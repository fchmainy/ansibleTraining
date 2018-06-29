Task 3. Use an existing local ansible role
===========================================

Ansible Galaxy refers to the Galaxy website (https://galaxy.ansible.com/)  where users can share roles, and to a command line tool for installing, creating and managing roles.
Galaxy, is a free site for finding, downloading, and sharing community developed roles. Downloading roles from Galaxy is a great way to jumpstart your automation projects.

You can also use the site to share roles that you create. By authenticating with the site using your GitHub account, you’re able to import roles, making them available to the Ansible community. Imported roles become available in the Galaxy search index and visible on the site, allowing users to discover and download them.

The ansible-galaxy command comes bundled with Ansible, and you can use it to install roles from Galaxy or directly from a git based SCM. You can also use it to create a new role, remove roles, or perform tasks on the Galaxy website.

The command line tool by default communicates with the Galaxy website API using the server address https://galaxy.ansible.com. Since the Galaxy project is an open source project, you may be running your own internal Galaxy server and wish to override the default server address. You can do this using the –serveroption or by setting the Galaxy server value in your ansible.cfg file. For information on setting the value in ansible.cfg visit Galaxy Settings.


Installing Roles
--------------------
Use the ansible-galaxy command to download roles from the Galaxy website

.. code::

 $ ansible-galaxy install username.role_name,v1.0.0


Search for Roles
----------------------
Search the Galaxy database by tags, platforms, author and multiple keywords. For example:

.. code::

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
 . . .


List installed roles
-----------------------
Use list to show the name and version of each role installed in the roles_path.

.. code::

 $ ansible-galaxy list
 - fch.rundocker, (unknown version)




Get more information about a role
---------------------------------
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

.. parsed-literal::
 ---
 - hosts: me
   remote_user: fchmainy
   gather_facts: no

   vars:
     container_ports:
       - "9081"
       - "9082"
       - "9083"

   roles:
     - { role: fch.rundocker, become: yes, myports: "{{ container_ports }}” }

copy this content in a new file: /tmp/task3.yml 

Then run the playbook:

.. parsed-literal::

 $ ansible-playbook /tmp/task3.yml --ask-sudo

There are already 3 instances of the same container in the tests file:

.. parsed-literal::

 vars:
    container_ports:
      - "9081"
      - "9082"
      - "9083"

let’s check if our containers have been created:

.. parsed-literal::

 $ sudo docker ps
 CONTAINER ID        IMAGE                      COMMAND             CREATED             STATUS              PORTS                  NAMES
 f026c78b0f74        f5devcentral/f5-demo-app   "npm start"         14 minutes ago      Up 14 minutes       0.0.0.0:9083->80/tcp   myapp_9083
 134e85ab982e        f5devcentral/f5-demo-app   "npm start"         14 minutes ago      Up 14 minutes       0.0.0.0:9082->80/tcp   myapp_9082
 d95802d44ced        f5devcentral/f5-demo-app   "npm start"         14 minutes ago      Up 14 minutes       0.0.0.0:9081->80/tcp   myapp_9081

These variables can be overridden easily by passing the variables as **extra-vars** while running the playbook

.. parsed-literal::

 $ ansible-playbook /tmp/task3.yml --ask-sudo --extra-vars 'container_ports=["9084","9085"]'
 $ sudo docker ps
 CONTAINER ID        IMAGE                      COMMAND             CREATED             STATUS              PORTS                  NAMES
 d95802d44ced        f5devcentral/f5-demo-app   "npm start"         14 minutes ago      Up 14 minutes       0.0.0.0:9085->80/tcp   myapp_9085
 037a4b004339        f5devcentral/f5-demo-app   "npm start"         14 minutes ago      Up 14 minutes       0.0.0.0:9084->80/tcp   myapp_9084
 9c10a5e70584        f5devcentral/f5-demo-app   "npm start"         5 days ago          Up 17 minutes       0.0.0.0:9083->80/tcp   myapp_9083
 f510d393ed53        f5devcentral/f5-demo-app   "npm start"         5 days ago          Up 17 minutes       0.0.0.0:9082->80/tcp   myapp_9082
 796c06cb7437        f5devcentral/f5-demo-app   "npm start"         5 days ago          Up 17 minutes       0.0.0.0:9081->80/tcp   myapp_9081
