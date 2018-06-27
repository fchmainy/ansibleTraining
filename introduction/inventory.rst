By default, Ansible represents what machines it manages using a very simple INI file that puts all of your managed machines in groups of your own choosing.

Here is what a plain text inventory file looks like:

.. parsed-literal::

[bigip]
10.1.1.101
10.1.1.102
10.1.1.103
10.1.1.104

[production]
10.1.1.101
10.1.1.102

[qa]
10.1.1.103
10.1.1.104

