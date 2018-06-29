Task 1. Create an inventory
===========================

Let's create a simple ansible inventory! For our lab we will use the main inventory file (/etc/ansible/hosts) with the following content:

.. parsed-literal::

	[datacenter]
	10.1.1.2
	10.1.1.3
	10.1.1.5
	10.1.1.10
	10.1.10.20
	10.1.10.100
	
	[bigip]
	10.1.1.10
	10.1.1.5
	10.1.1.11
	
	[webservers]
	10.1.10.20	apache_version=2.6
	10.1.10.100	apache_version=2.6
	10.1.10.101
	10.1.10.102
	10.1.10.103
	
	[production]
	10.1.1.10
	1.1.10.20
	192.168.1.12

	[lab]
	10.1.1.3
	10.1.1.5
	192.168.2.11

	[waf]
	mywaf.f5demo.fch

This is as simple as that!!!

Besides being simple, this is really powerful in the defintion of targets:
	* **webservers:bigip** is a logical OR operation so it corresponds to all IPs in these groups
	* **datacenter:!webservers** means paris devices *except* webservers, so: ***10.1.1.2,10.1.1.3**
	* **bigip:&production** is a logical intersection, so:  **10.1.1.10**
	* **datacenter:&bigip:!lab** is a combinaison corresponding to **10.1.1.10**


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
		    "1.1.10.20": {},
		    "10.1.1.10": {},
		    "10.1.1.11": {},
		    "10.1.1.2": {},
		    "10.1.1.3": {},
		    "10.1.1.5": {},
		    "10.1.10.100": {
			"apache_version": 2.6
		    },
		    "10.1.10.101": {},
		    "10.1.10.102": {},
		    "10.1.10.103": {},
		    "10.1.10.20": {
			"apache_version": 2.6
		    },
		    "192.168.1.12": {},
		    "192.168.2.11": {},
		    "mywaf.f5demo.fch": {}
		}
	    },
	    "all": {
		"children": [
		    "bigip",
		    "datacenter",
		    "lab",
		    "production",
		    "ungrouped",
		    "waf",
		    "webservers"
		]
	    },
	    "bigip": {
		"hosts": [
		    "10.1.1.10",
		    "10.1.1.11",
		    "10.1.1.5"
		]
	    },
	    "datacenter": {
		"hosts": [
		    "10.1.1.10",
		    "10.1.1.2",
		    "10.1.1.3",
		    "10.1.1.5",
		    "10.1.10.100",
		    "10.1.10.20"
		]
	    },
	    "lab": {
		"hosts": [
		    "10.1.1.3",
		    "10.1.1.5",
		    "192.168.2.11"
		]
	    },
	    "production": {
		"hosts": [
		    "1.1.10.20",
		    "10.1.1.10",
		    "192.168.1.12"
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
		    "10.1.10.100",
		    "10.1.10.101",
		    "10.1.10.102",
		    "10.1.10.103",
		    "10.1.10.20"
		]
	    }
	}
