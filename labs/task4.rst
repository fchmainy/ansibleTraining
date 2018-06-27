Task 4. Create an ansible role
=======================
Use the init command to initialize the base structure of a new role, saving time on creating the various directories and main.yml files a role requires

.. code::

	$ ansible-galaxy init username.task4 --offline
	- username.task4 was created successfully

You should then have the framework for your role:
.. parsed-literal::

	$ ls -R fch.task4/
	fch.task4/:
	README.md  defaults  files  handlers  meta  tasks  templates  tests  vars

	fch.task4/defaults:
	main.yml

	fch.task4/files:

	fch.task4/handlers:
	main.yml

	fch.task4/meta:
	main.yml

	fch.task4/tasks:
	main.yml

	fch.task4/templates:

	fch.task4/tests:
	inventory  test.yml

	fch.task4/vars:
	main.yml

**Role Variables**
In the defaults/main.yml file, copy the following content:
.. parsed-literal::

	username: "admin"
	password: "admin"

	app_name: "myApp"
	pool_name: "{{ app_name }}_pool"
	redirect_port: "80"
	vip_ip: "10.100.26.143"
	vip_port: "443"

	pool_members:
	- port: "80"
	  host: "10.100.26.144"
	- port: "80"
	  host: “10.100.26.145"

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

**Role Inventory**

paste the following inventory to your hosts file.
.. parsed-literal::

	[bigip]
	192.168.1.143

**Role Tasks**
Copy/paste the following tasks in your tasks/main.yml file:

.. parsed-literal::

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
	      irules:
	      - "_sys_https_redirect"
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

**(Optional)Create your meta  file** 
This is mainly for documentation, and to help you find the best role for reuse…

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

Securing sensitive information
---------------------------------------

Keeping passwords in clear text in probably the worst thing we have done yet :( Let’s secure it using ansible vault (https://docs.ansible.com/ansible/2.4/vault.html).
"Vault" is a feature of ansible that allows keeping sensitive data such as passwords or keys in encrypted files, rather than as plaintext in your playbooks or roles. These vault files can then be distributed or placed in source control.

The default and easiest way is to encrypt the whole variable file and ask for the vault password when running the playbook.
As of version 2.3, Ansible also supports encrypting single values inside a YAML file, using the !vault tag to let YAML and Ansible know it uses special processing. This feature is covered in more details below.

The ansible-vault encrypt_string command will encrypt and format a provided string into a format that can be included in ansible-playbook YAML files.

To encrypt your admin password as a cli arg:

.. parsed-literal::

	$ ansible-vault encrypt_string 'admin' --name 'password'
	New Vault password:
	Confirm New Vault password:
	password: !vault |
		  $ANSIBLE_VAULT;1.1;AES256
		  38616233643963386663646565666535316639353634666636656338643562363961333362323134
		  6663633034333936303936393666303165356232373230330a356635326663393262383331656438
		  30323265646362383339646438376366643430393930333139356433626634616635386465666239
		  3333646665643662630a376237643064343466313066626333356439633330336538616461323364
		  3865
	Encryption successful


Then replace the password line in your defaults/main.yml file
.. parsed-literal::
	username: "admin"
	password: "admin"
	…

by the encrypted string previously generated:

.. parsed-literal::

	username: "admin"
	password: !vault |
		  $ANSIBLE_VAULT;1.1;AES256
		  38616233643963386663646565666535316639353634666636656338643562363961333362323134
		  6663633034333936303936393666303165356232373230330a356635326663393262383331656438
		  30323265646362383339646438376366643430393930333139356433626634616635386465666239
		  3333646665643662630a376237643064343466313066626333356439633330336538616461323364
		  3865

Running your playbook:
-------------------------------

create a playbook called task5.yml and paste the following content:

.. parsed-literal::

	---
	- name: Configure http service
	  hosts: prod
	  gather_facts: false
	  roles:
	    - { role: fch.lbsvc }

then run your playbook:

.. parsed-literal::

$ ansible-playbook task5.yml -vvv

you can check on your BigIP the service have been created.

You can easily run the same role to add pool members to the configuration (remember: F5 ansible playbooks are idempotent):
.. parsed-literal::

	$ ansible-playbook task5.yml --ask-sudo --extra-vault-pass 'pool_members=[{"port":"80","host:"10.100.26.146"},{"port":"80","host:"10.100.26.146"}]”'

or run the same playbook for a new service without touching the playbook YAML file:

.. parsed-literal::

	$ ansible-playbook task5.yml --ask-sudo --extra-vault-pass 'pool_members=[{"port":"80","host:"10.100.26.146"},{"port":"80","host:"10.100.26.146"}] app_name="my2ndApp" vip_ip="10.100.26.43"'




