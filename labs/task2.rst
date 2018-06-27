Task 2. Create a simple playbook
===========================
Introduction
------------
Playbooks are Ansible’s configuration, deployment, and orchestration language. They can describe a policy you want your remote systems to enforce, or a set of steps in a general IT process.

If Ansible modules are the tools in your workshop, playbooks are your instruction manuals, and your inventory of hosts are your raw material.

At a basic level, playbooks can be used to manage configurations of and deployments to remote machines. At a more advanced level, they can sequence multi-tier rollouts involving rolling updates, and can delegate actions to other hosts, interacting with monitoring servers and load balancers along the way.

While there’s a lot of information here, there’s no need to learn everything at once. You can start small and pick up more features over time as you need them.

Playbooks are designed to be human-readable and are developed in a basic text language. There are multiple ways to organize playbooks and the files they include, and we’ll offer up some suggestions on that and making the most out of Ansible.

https://docs.ansible.com/ansible/2.5/user_guide/playbooks.html


Variables
------------



Conditions
-------------



Loops
--------


copy and paste the following content in a YAML playbookfile. let’s call it: task2.yml

.. literalinclude:: task2.yml
   :language: yaml

run the playbook using the following command:

.. code::

  $ ansible-playbook task1.yaml -vvv

*Note: You can run the playbook multiple time as F5 ansible modules are idempotent (https://en.wikipedia.org/wiki/Idempotence) *



