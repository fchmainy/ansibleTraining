Task 3. Introduction to Ansible Roles
============================
1. Introduction
(https://docs.ansible.com/ansible/2.5/user_guide/playbooks_reuse_roles.html)
Roles are ways of automatically loading certain vars_files, tasks, and handlers based on a known file structure. Grouping content by roles also allows easy sharing of roles with other users.


Example project structure:

site.yml
webservers.yml
fooservers.yml
roles/
   common/
     tasks/
     handlers/
     files/
     templates/
     vars/
     defaults/
     meta/
   webservers/
     tasks/
     defaults/
     meta/
Roles expect files to be in certain directory names. Roles must include at least one of these directories, however it is perfectly fine to exclude any which are not being used. When in use, each directory must contain a main.yml file, which contains the relevant content:

tasks - contains the main list of tasks to be executed by the role.
handlers - contains handlers, which may be used by this role or even anywhere outside this role.
defaults - default variables for the role (see Variables for more information).
vars - other variables for the role (see Variables for more information).
files - contains files which can be deployed via this role.
templates - contains templates which can be deployed via this role.
meta - defines some meta data for this role. See below for more details.
Other YAML files may be included in certain directories. For example, it is common practice to have platform-specific tasks included from the tasks/main.yml file:

# roles/example/tasks/main.yml
- name: added in 2.4, previously you used 'include'
  import_tasks: redhat.yml
  when: ansible_os_platform|lower == 'redhat'
- import_tasks: debian.yml
  when: ansible_os_platform|lower == 'debian'

# roles/example/tasks/redhat.yml
- yum:
    name: "httpd"
    state: present

# roles/example/tasks/debian.yml
- apt:
    name: "apache2"
    state: present
Roles may also include modules and other plugin types. For more information, please refer to the Embedding Modules and Plugins In Roles section below.

Using Roles

The classic (original) way to use roles is via the roles: option for a given play:

---
- hosts: webservers
  roles:
     - common
     - webservers
This designates the following behaviors, for each role ‘x’:

If roles/x/tasks/main.yml exists, tasks listed therein will be added to the play.
If roles/x/handlers/main.yml exists, handlers listed therein will be added to the play.
If roles/x/vars/main.yml exists, variables listed therein will be added to the play.
If roles/x/defaults/main.yml exists, variables listed therein will be added to the play.
If roles/x/meta/main.yml exists, any role dependencies listed therein will be added to the list of roles (1.3 and later).
Any copy, script, template or include tasks (in the role) can reference files in roles/x/{files,templates,tasks}/ (dir depends on task) without having to path them relatively or absolutely.
When used in this manner, the order of execution for your playbook is as follows:

Any pre_tasks defined in the play.
Any handlers triggered so far will be run.
Each role listed in roles will execute in turn. Any role dependencies defined in the roles meta/main.ymlwill be run first, subject to tag filtering and conditionals.
Any tasks defined in the play.
Any handlers triggered so far will be run.
Any post_tasks defined in the play.
Any handlers triggered so far will be run.
Note

See below for more information regarding role dependencies.
Note

If using tags with tasks (described later as a means of only running part of a playbook), be sure to also tag your pre_tasks, post_tasks, and role dependencies and pass those along as well, especially if the pre/post tasks and role dependencies are used for monitoring outage window control or load balancing.
As of Ansible 2.4, you can now use roles inline with any other tasks using import_role or include_role:

---

- hosts: webservers
  tasks:
  - debug:
      msg: "before we run our role"
  - import_role:
      name: example
  - include_role:
      name: example
  - debug:
      msg: "after we ran our role"
When roles are defined in the classic manner, they are treated as static imports and processed during playbook parsing.

Note

The include_role option was introduced in Ansible 2.3. The usage has changed slightly as of Ansible 2.4 to match the include (dynamic) vs. import (static) usage. See Dynamic vs. Static for more details.
The name used for the role can be a simple name (see Role Search Path below), or it can be a fully qualified path:

---

- hosts: webservers
  roles:
    - { role: '/path/to/my/roles/common' }
Roles can accept parameters:

---

- hosts: webservers
  roles:
    - common
    - { role: foo_app_instance, dir: '/opt/a', app_port: 5000 }
    - { role: foo_app_instance, dir: '/opt/b', app_port: 5001 }
Or, using the newer syntax:

---

- hosts: webservers
  tasks:
  - include_role:
       name: foo_app_instance
    vars:
      dir: '/opt/a'
      app_port: 5000
  ...
You can conditionally execute a role. This is not generally recommended with the classic syntax, but is common when using import_role or include_role:

---

- hosts: webservers
  tasks:
  - include_role:
      name: some_role
    when: "ansible_os_family == 'RedHat'"
Finally, you may wish to assign tags to the roles you specify. You can do so inline:

---

- hosts: webservers
  roles:
    - { role: foo, tags: ["bar", "baz"] }
Or, again, using the newer syntax:

---

- hosts: webservers
  tasks:
  - import_role:
      name: foo
    tags:
    - bar
    - baz
Note

This tags all of the tasks in that role with the tags specified, appending to any tags that are specified inside the role. The tags in this example will not be added to tasks inside an include_role. Tag the include_roletask directly in order to apply tags to tasks in included roles. If you find yourself building a role with lots of tags and you want to call subsets of the role at different times, you should consider just splitting that role into multiple roles.
Role Duplication and Execution

Ansible will only allow a role to execute once, even if defined multiple times, if the parameters defined on the role are not different for each definition. For example:

---
- hosts: webservers
  roles:
  - foo
  - foo
Given the above, the role foo will only be run once.

To make roles run more than once, there are two options:

Pass different parameters in each role definition.
Add allow_duplicates: true to the meta/main.yml file for the role.
Example 1 - passing different parameters:

---
- hosts: webservers
  roles:
  - { role: foo, message: "first" }
  - { role: foo, message: "second" }
In this example, because each role definition has different parameters, foo will run twice.

Example 2 - using allow_duplicates: true:

# playbook.yml
---
- hosts: webservers
  roles:
  - foo
  - foo

# roles/foo/meta/main.yml
---
allow_duplicates: true
In this example, foo will run twice because we have explicitly enabled it to do so.

Role Default Variables

New in version 1.3.

Role default variables allow you to set default variables for included or dependent roles (see below). To create defaults, simply add a defaults/main.yml file in your role directory. These variables will have the lowest priority of any variables available, and can be easily overridden by any other variable, including inventory variables.

Role Dependencies

New in version 1.3.

Role dependencies allow you to automatically pull in other roles when using a role. Role dependencies are stored in the meta/main.yml file contained within the role directory, as noted above. This file should contain a list of roles and parameters to insert before the specified role, such as the following in an example roles/myapp/meta/main.yml:

---
dependencies:
  - { role: common, some_parameter: 3 }
  - { role: apache, apache_port: 80 }
  - { role: postgres, dbname: blarg, other_parameter: 12 }
Note

Role dependencies must use the classic role definition style.
Role dependencies are always executed before the role that includes them, and may be recursive. Dependencies also follow the duplication rules specified above. If another role also lists it as a dependency, it will not be run again based on the same rules given above.

Note

Always remember that when using allow_duplicates: true, it needs to be in the dependent role’s meta/main.yml, not the parent.
For example, a role named car depends on a role named wheel as follows:

---
dependencies:
- { role: wheel, n: 1 }
- { role: wheel, n: 2 }
- { role: wheel, n: 3 }
- { role: wheel, n: 4 }
And the wheel role depends on two roles: tire and brake. The meta/main.yml for wheel would then contain the following:

---
dependencies:
- { role: tire }
- { role: brake }
And the meta/main.yml for tire and brake would contain the following:

---
allow_duplicates: true
The resulting order of execution would be as follows:

tire(n=1)
brake(n=1)
wheel(n=1)
tire(n=2)
brake(n=2)
wheel(n=2)
...
car
Note that we did not have to use allow_duplicates: true for wheel, because each instance defined by caruses different parameter values.

Task 4. Use an existing local ansible role
===============================

Ansible Galaxy refers to the Galaxy website (https://galaxy.ansible.com/)  where users can share roles, and to a command line tool for installing, creating and managing roles.
Galaxy, is a free site for finding, downloading, and sharing community developed roles. Downloading roles from Galaxy is a great way to jumpstart your automation projects.

You can also use the site to share roles that you create. By authenticating with the site using your GitHub account, you’re able to import roles, making them available to the Ansible community. Imported roles become available in the Galaxy search index and visible on the site, allowing users to discover and download them.

The ansible-galaxy command comes bundled with Ansible, and you can use it to install roles from Galaxy or directly from a git based SCM. You can also use it to create a new role, remove roles, or perform tasks on the Galaxy website.

The command line tool by default communicates with the Galaxy website API using the server address https://galaxy.ansible.com. Since the Galaxy project is an open source project, you may be running your own internal Galaxy server and wish to override the default server address. You can do this using the –serveroption or by setting the Galaxy server value in your ansible.cfg file. For information on setting the value in ansible.cfg visit Galaxy Settings.


Installing Roles
--------------------
Use the ansible-galaxy command to download roles from the Galaxy website

$ ansible-galaxy install username.role_name,v1.0.0


Search for Roles
----------------------
Search the Galaxy database by tags, platforms, author and multiple keywords. For example:

.. code ::
$ ansible-galaxy search bigip

Found 11 roles matching your search:

 Name                                     Description
 ----                                     -----------
 f5devcentral.bigip-onboarding            Modules to on board the BIG-IP
 f5devcentral.bigip-toggle-nodeStatus     Ansible role to enable/disable pool member on BIG-IP
 f5devcentral.bigip-ansible-deploy-iapp   Ansible role to deploy an F5 iApp
 f5devcentral.bigip-hardening             Ansible role to automate base BIG-IP hardening, and STIG/SRG configuration
 f5devcentral.bigip-ansible-virtualserver Ansible role to configure nodes/pools and virtual server on the BIG-IP
 mikefaille.ansible-bigdata               Playbook for boostratping Big data env.
 erjac77.module-f5bigip                   Ansible module for F5 BIG-IP
[…]

List installed roles
-----------------------
Use list to show the name and version of each role installed in the roles_path.

.. code::
$ ansible-galaxy list
- fch.rundocker, (unknown version)




Get more information about a role

Use the info command to view more detail about a specific role:
.. code::
$ ansible-galaxy info fch.rundocker

Role: fch.rundocker
        description:
        dependencies: []
        galaxy_info:
                author: Fouad Chmainy
                company: F5 Demo
                galaxy_tags: []
                license: license (GPLv2, CC-BY, etc)
                min_ansible_version: 2.3
        path: [u'/etc/ansible/roles']


Now, let’s run this role with a simple playbook. There is already a test playbook in the tests directory of the role:

.. code::
---
- hosts: me
  remote_user: fchmainy
  strategy: debug
  gather_facts: yes

  vars:
    container_ports:
      - "9081"
      - "9082"
      - "9083"

  roles:
    - { role: fch.rundocker, become: yes, myports: "{{ container_ports }}” }

copy this content in a new file: /tmp/task4.yml 

Then run the playbook:
ansible-playbook /tmp/task4.yml --ask-sudo

There are already 3 instances of the same container in the tests file:
.. code::
  vars:
    container_ports:
      - "9081"
      - "9082"
      - "9083"

let’s check if our containers have been created:
.. code::
$ sudo docker ps
CONTAINER ID        IMAGE                      COMMAND             CREATED             STATUS              PORTS                  NAMES
f026c78b0f74        f5devcentral/f5-demo-app   "npm start"         14 minutes ago      Up 14 minutes       0.0.0.0:9083->80/tcp   myapp_9083
134e85ab982e        f5devcentral/f5-demo-app   "npm start"         14 minutes ago      Up 14 minutes       0.0.0.0:9082->80/tcp   myapp_9082
d95802d44ced        f5devcentral/f5-demo-app   "npm start"         14 minutes ago      Up 14 minutes       0.0.0.0:9081->80/tcp   myapp_9081

These variables can be overridden easily by passing the variables as **extra-vars** while running the playbook
.. code::
ansible-playbook fch.rundocker/tests/test.yml --ask-sudo --extra-vars 'container_ports=["9084","9085"]'

$ sudo docker ps
CONTAINER ID        IMAGE                      COMMAND             CREATED             STATUS              PORTS                  NAMES
d95802d44ced        f5devcentral/f5-demo-app   "npm start"         14 minutes ago      Up 14 minutes       0.0.0.0:9085->80/tcp   myapp_9085
037a4b004339        f5devcentral/f5-demo-app   "npm start"         14 minutes ago      Up 14 minutes       0.0.0.0:9084->80/tcp   myapp_9084
9c10a5e70584        f5devcentral/f5-demo-app   "npm start"         5 days ago          Up 17 minutes       0.0.0.0:9083->80/tcp   myapp_9083
f510d393ed53        f5devcentral/f5-demo-app   "npm start"         5 days ago          Up 17 minutes       0.0.0.0:9082->80/tcp   myapp_9082
796c06cb7437        f5devcentral/f5-demo-app   "npm start"         5 days ago          Up 17 minutes       0.0.0.0:9081->80/tcp   myapp_9081


