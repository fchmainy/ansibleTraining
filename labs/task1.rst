Task 1. Create an inventory
===========================

Once inventory hosts are listed, variables can be assigned to them in simple text files (in a subdirectory called ‘group_vars/’ or ‘host_vars/’ or directly in the inventory file.

As described above, it is easy to assign variables to hosts that will be used later in playbooks:

.. code::
	[atlanta]
	host1 http_port=80 maxRequestsPerChild=808
	host2 http_port=303 maxRequestsPerChild=909


Variables can also be applied to an entire group at once:

.. code::
	[atlanta]
	host1
	host2

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com

Or, as already mentioned, use a dynamic inventory to pull your inventory from data sources like EC2, Rackspace, or OpenStack.


**The preferred practice in Ansible is to not store variables in the main inventory file.**

Assuming the inventory file path is:

/etc/ansible/hosts
If the host is named ‘foosball’, and in groups ‘raleigh’ and ‘webservers’, variables in YAML files at the following locations will be made available to the host:

.. code::
	/etc/ansible/group_vars/raleigh # can optionally end in '.yml', '.yaml', or '.json'
	/etc/ansible/group_vars/webservers
	/etc/ansible/host_vars/foosball

For instance, suppose you have hosts grouped by datacenter, and each datacenter uses some different servers. The data in the groupfile ‘/etc/ansible/group_vars/raleigh’ for the ‘raleigh’ group might look like:

.. code::
	---
	ntp_server: acme.example.org
	database_server: storage.example.org

It is okay if these files do not exist, as this is an optional feature.
As an advanced use case, you can create directories named after your groups or hosts, and Ansible will read all the files in these directories. An example with the ‘raleigh’ group:

.. code::
	/etc/ansible/group_vars/raleigh/db_settings
	/etc/ansible/group_vars/raleigh/cluster_settings


In order to check the inventory
.. code::
	# ansible-inventory --list
