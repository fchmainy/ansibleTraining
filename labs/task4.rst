Task 4. Create an ansible role
==============================

1. Create the Role Structure
------------------------
Use the init command to initialize the base structure of a new role, saving time on creating the various directories and main.yml files a role requires

.. code::

	$ ansible-galaxy init **username**.lbsvc --offline
	- username.lbsvc was created successfully

You should then have the framework for your role:

.. parsed-literal::

	$ ls -R **username**.lbsvc/
	username.lbsvc/:
	README.md  defaults  files  handlers  meta  tasks  templates  tests  vars

	username.lbsvc/defaults:
	main.yml

	username.lbsvc/files:

	username.lbsvc/handlers:
	main.yml

	username.lbsvc/meta:
	main.yml

	username.lbsvc/tasks:
	main.yml

	username.lbsvc/templates:

	username.lbsvc/tests:
	inventory  test.yml

	username.lbsvc/vars:
	main.yml

2. Create the Role Variables
----------------------------

In the defaults/main.yml file, copy the following content:

.. parsed-literal::

	---
	username: "admin"
	password: "supernetops"

	app_name: "myAppTask4"
	pool_name: "{{ app_name }}_pool"
	redirect_port: "80"
	vip_ip: "10.1.20.101"
	vip_port: "443"

	pool_members:
	- port: "9081"
	  host: "10.1.10.20"
	- port: "9082"
	  host: "10.1.10.20"
	- port: "9083"
	  host: "10.1.10.20"
	  
This is the key/value pairs variables to use in your role.
here, variables are passed to the **defaults** folder. In the existing fch.run_docker role, variables are passed to the **vars** folder. Both solutions are valid depending on what you want to achieve and the precedence

Note:
-----
If multiple variables of the same name are defined in different places, they win in a certain order, which is:
	* extra vars (-e in the command line) always win
	* then comes connection variables defined in inventory (ansible_ssh_user, etc)
	* then comes "most everything else" (command line switches, vars in play, included vars, role vars, etc)
	* then comes the rest of the variables defined in inventory
	* then comes facts discovered about a system
	* then "role defaults", which are the most "defaulty" and lose in priority to everything.

*https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable*


3. Create the Role Tasks
------------------------
Copy/paste the following tasks in your tasks/main.yml file:

.. parsed-literal::
	
	---
	  - name: Create nodes
	    bigip_node:
	      server: "{{ inventory_hostname }}"
	      user: "{{ username }}"
	      password: "{{ password }}"
	      host: "{{item.host}}"
	      name: "{{item.host}}"
	      validate_certs: False
	    with_items: "{{pool_members}}"
	    delegate_to: localhost

	  - name: Create pool
	    bigip_pool:
	      server: "{{ inventory_hostname }}"
	      user: "{{ username }}"
	      password: "{{ password }}"
	      name: "{{pool_name}}"
	      lb_method: "round-robin"
	      monitors: "/Common/http"
	      validate_certs: False
	    delegate_to: localhost

	  - name: Add Pool members
	    bigip_pool_member:
	      server: "{{ inventory_hostname }}"
	      user: "{{ username }}"
	      password: "{{ password }}"
	      name: "{{item.host}}"
	      host: "{{item.host}}"
	      port: "{{item.port}}"
	      pool: "{{pool_name}}"
	      validate_certs: False
	    with_items: "{{pool_members}}"
	    delegate_to: localhost

	  - name: Add Virtual Server
	    bigip_virtual_server:
	      server: "{{ inventory_hostname }}"
	      user: "{{ username }}"
	      password: "{{ password }}"
	      name: "{{ app_name }}_vs_https"
	      destination: "{{ vip_ip }}"
	      port: "{{ vip_port }}"
	      all_profiles:
	       - http
	       - name: clientssl
		 context: client-side
	      pool: "{{pool_name}}"
	      snat: "automap"
	      validate_certs: False
	    delegate_to: localhost

	  - name: Add Redirect Virtual Server
	    bigip_virtual_server:
	      server: "{{ inventory_hostname }}"
	      user: "{{ username }}"
	      password: "{{ password }}"
	      name: "{{ app_name }}_vs_http_redirect"
	      destination: "{{ vip_ip }}"
	      port: "80"
	      all_profiles:
	       - http
	      irules:
	      - "_sys_https_redirect"
	      validate_certs: False
	    delegate_to: localhost


4. (Optional)Create your role meta file
---------------------------------------

This is mainly for documentation, and to help you find the best role for reuse. The file to edit is ./meta/main.yml :

.. parsed-literal::
	galaxy_info:
	  author: <Your name>
	  company: <Your Company
	  license: license (GPLv2, CC-BY, etc)
	  min_ansible_version: 2.5
	  platforms:
	    - name: Ubuntu
	      versions:
	      - all
	  categories:
	      - ….
	  galaxy_tags:
	    - bigip
	    - networking
	    - selfip
	    - bigip
	    - F5



5. Securing sensitive information
---------------------------------------

Keeping passwords in clear text in probably the worst thing we have done yet :( Let’s secure it using ansible vault (https://docs.ansible.com/ansible/2.4/vault.html).
"Vault" is a feature of ansible that allows keeping sensitive data such as passwords or keys in encrypted files, rather than as plaintext in your playbooks or roles. These vault files can then be distributed or placed in source control.

The default and easiest way is to encrypt the whole variable file and ask for the vault password when running the playbook.
As of version 2.3, Ansible also supports encrypting single values inside a YAML file, using the !vault tag to let YAML and Ansible know it uses special processing. This feature is covered in more details below.

The ansible-vault encrypt_string command will encrypt and format a provided string into a format that can be included in ansible-playbook YAML files.

To encrypt your admin password as a cli arg, you need to provide a Vault password (you can put anything as soon as you remember it as You will need to unlock your vault several times in the upcoming tasks):

.. parsed-literal::

	$ ansible-vault encrypt_string 'supernetops' --name 'password'
	New Vault password:
	Confirm New Vault password:
	password: !vault |
		  $ANSIBLE_VAULT;1.1;AES256
		  38623963623565303065613262316437313830643032336364663938383238343864623661633264
		  3730396430336332623563303861616632633630376139300a326664356235373565363138323533
		  61626537336361633464636135393864616332376231363137643732666563303739323438633266
		  3864663163643433390a333132643562663034393862383861616635666335313032663638663937
		  6665
	Encryption successful


Then replace the password line in your defaults/main.yml file

.. parsed-literal::

	username: "admin"
	password: "supernetops"
	…

by the encrypted string previously generated:

.. parsed-literal::

	username: "admin"
	password: !vault |
		  $ANSIBLE_VAULT;1.1;AES256
		  38623963623565303065613262316437313830643032336364663938383238343864623661633264
		  3730396430336332623563303861616632633630376139300a326664356235373565363138323533
		  61626537336361633464636135393864616332376231363137643732666563303739323438633266
		  3864663163643433390a333132643562663034393862383861616635666335313032663638663937
		  6665

Finally copy your role to /etc/ansible/roles:

.. code::

	$ sudo cp -R **username**.lbsvc /etc/ansible/roles/

Running your playbook:
-------------------------------

create a playbook called /tmp/task4.yml and paste the following content (be sure your rename username.lbsvc with your **username**):

.. parsed-literal::

	---
	- name: Configure http service
	  hosts: production:&bigip
	  gather_facts: false
	  roles:
	    - { role: username.lbsvc }

then run your playbook:

.. parsed-literal::

	$ ansible-playbook /tmp/task4.yml --ask-vault-pass -vvv

you can check on your BigIP the service have been created.

You can easily run the same role to add pool members to the configuration (remember: F5 ansible playbooks are idempotent):

.. parsed-literal::

	$ ansible-playbook /tmp/task4.yml --ask-vault-pass --extra-vars 'pool_members=[{"port":"9084","host:"10.1.10.20"},{"port":"9085","host:"10.1.10.20"}]”'

or run the same playbook for a new service without touching the playbook YAML file:

.. parsed-literal::

	$ ansible-playbook /tmp/task4.yml --ask-vault-pass --extra-vars 'pool_members=[{"port":"9082","host:"10.1.10.20"},{"port":"9081","host:"10.1.10.20"}] app_name="my2ndApp_task4" vip_ip="10.1.20.102"'

You can run it as many time as you want as it is... did I already told you about idempotency?


