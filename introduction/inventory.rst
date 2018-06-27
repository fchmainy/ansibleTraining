By default, Ansible represents what machines it manages using a very simple INI file that puts all of your managed machines in groups of your own choosing.

To add new machines, there is no additional SSL signing server involved, so there is never any hassle deciding why a particular machine didn't get linked up due to obscure NTP or DNS issues.

If there is another source of truth in your infrastructure, Ansible can also plugin to that, such as drawing inventory, group, and variable information from sources like EC2, Rackspace, OpenStack, and more.

Here is what a plain text inventory file looks like:

.. code::
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

