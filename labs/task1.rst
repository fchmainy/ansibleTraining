Task 1. Create an inventory
===========================

Let's create a simple ansible inventory! For our lab we will use the main inventory file (/etc/ansible/hosts) with the following content:

.. parsed-literal::

	[paris]
	192.168.1.101
	192.168.1.102
	192.168.2.11
	192.168.2.12

	[bigip]
	192.168.1.101
	192.168.1.102

	[webservers]
	192.168.2.11	apache_version=2.6
	192.168.2.12	apache_version=2.6

	[production]
	192.168.1.102
	192.168.2.12

	[lab]
	192.168.1.101
	192.168.2.11

	[waf]
	mywaf.f5demo.fch

This is as simple as that!!!

Besides being simple, this is really powerful in the defintion of targets:
	* **webservers:bigip** is a logical OR operation so it means **paris**
	* **paris:!webserverss** means paris devices *except* webservers, so: ***bigip**
	* **bigip:&production** is a logical intersection, so:  **192.168.1.102**
	* **paris:&bigip:!lab** is a combinaison corresponding to 192.168.1.102


Although it is not recommended to add variables to the main inventory file we will do it for lab purpose only. In real life, we higly recommend using dedicated INI files such as:
	* /etc/ansible/group_vars/bigip
	* /etc/ansible/group_vars/webservers
	* /etc/ansible/host_vars/mywaf.f5demo.fch


In order to check the inventory

.. code::

	$ ansible-inventory --list
	{
	    "_meta": {
		"hostvars": {
		    "192.168.1.101": {},
		    "192.168.1.102": {},
		    "192.168.2.11": {
			"apache_version": 2.6
		    },
		    "192.168.2.12": {
			"apache_version": 2.6
		    },
		    "mywaf.f5demo.fch": {}
		}
	    },
	    "all": {
		"children": [
		    "bigip",
		    "lab",
		    "paris",
		    "production",
		    "ungrouped",
		    "waf",
		    "webservers"
		]
	    },
	    "bigip": {
		"hosts": [
		    "192.168.1.101",
		    "192.168.1.102"
		]
	    },
	    "lab": {
		"hosts": [
		    "192.168.1.101",
		    "192.168.2.11"
		]
	    },
	    "paris": {
		"hosts": [
		    "192.168.1.101",
		    "192.168.1.102",
		    "192.168.2.11",
		    "192.168.2.12"
		]
	    },
	    "production": {
		"hosts": [
		    "192.168.1.102",
		    "192.168.2.12"
		]
	    },
	    "ungrouped": {},
	    "waf": {
		"hosts": [
		    "mywaf.f5demo.fch"
		]
	    },
	    "webservers": {
		"hosts": [
		    "192.168.2.11",
		    "192.168.2.12"
		]
	    }
	}
