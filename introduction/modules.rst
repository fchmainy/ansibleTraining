What is an ansible module?
-------------------------
Ansible works by connecting to your nodes and pushing out small programs, called **"Ansible Modules"** to them. These programs are written to be resource models of the desired state of the system. Ansible then executes these modules (over SSH by default), and removes them when finished.

Your library of modules can reside on any machine, and there are no servers, daemons, or databases required. Typically you will work with your favorite terminal program, a text editor, and probably a version control system to keep track of changes to your content.

F5 Provides and support today almost 100 ansible modules to cover:
* BigIP system provisioning 
* L4/L7 LTM configuration
* ASM Policy deployment
* BigIQ dynamic licensing
* DNS/GTM configuration
. . .

list of F5 modules are available on https://github.com/F5Networks/f5-ansible/tree/stable-2.5/library and are already available when installing ansible (https://clouddocs.f5.com/products/orchestration/ansible/devel/usage/getting_started.html)


F5 commits to invest strong efforts on developing additional modules to cover other use cases but We are also very welcome to contribute by developing and share your own modules.
Before you can contribute to any project sponsored by F5 Networks, Inc. (F5) on GitHub, you must sign a Contributor License Agreement (CLA).
https://clouddocs.f5.com/products/orchestration/ansible/devel/development/cla-landing.html

When you develop a module, it goes through review before F5 accepts it. This review process may be difficult at times, but it ensures the published modules are good quality.



Module Documentation and syntax:
--------------------------------
if you want to have documentation on a specific module, you can either check on Clouddocs, on the F5 Networks Github or use the **ansible-doc** command. ansible-doc displays a short description of the module, what os ot aimed for and a short snippet with required and optional attributes so you can just copy / paste any of the example provided:

.. code::

Modules documentation
displays information on modules installed in Ansible libraries. It displays a terse listing of plugins and their short descriptions, provides a printout of their DOCUMENTATION strings, and it can create a short “snippet” which can be pasted into a playbook.
# ansible-doc bigip_virtual



