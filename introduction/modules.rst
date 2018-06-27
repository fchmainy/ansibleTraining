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
if you want to have documentation on a specific module, you can either check on **F5 Clouddocs**, on the **F5 Networks Github** or use the **ansible-doc** command. ansible-doc displays a short description of the module, what is it aimed for and a short snippet with required and optional attributes so you can just copy / paste any of the example provided:

.. parsed-literal::

  $ ansible-doc bigip_pool
  > BIGIP_POOL    (/usr/local/lib/python2.7/dist-packages/ansible/modules/network/f5/bigip_pool.py)

          Manages F5 BIG-IP LTM pools via iControl REST API.

  OPTIONS (= is mandatory):

  - description
          Specifies descriptive text that identifies the pool.
          [Default: (null)]
          version_added: 2.3

  - lb_method
          Load balancing method. When creating a new pool, if this value is not specified, the default of `round-
          robin' will be used.
          (Choices: dynamic-ratio-member, dynamic-ratio-node, fastest-app-response, fastest-node, least-connections-
          member, least-connections-node, least-sessions, observed-member, observed-node, predictive-member,
          predictive-node, ratio-least-connections-member, ratio-least-connections-node, ratio-member, ratio-node,
          ratio-session, round-robin, weighted-least-connections-member, weighted-least-connections-nod)[Default:
          (null)]
          version_added: 1.3
  . . .
  = user
          The username to connect to the BIG-IP with. This user must have administrative privileges on the device. You
          can omit this option if the environment variable `F5_USER' is set.


  - validate_certs
          If `no', SSL certificates will not be validated. Use this only on personally controlled sites using self-
          signed certificates. You can omit this option if the environment variable `F5_VALIDATE_CERTS' is set.
          [Default: True]
          type: bool
          version_added: 2.0



  NOTES:
        * Requires BIG-IP software version >= 12.
        * To add members do a pool, use the `bigip_pool_member' module. Previously, the `bigip_pool' module
          allowed the management of users, but this has been removed in version 2.5 of Ansible.
        * For more information on using Ansible to manage F5 Networks devices see
          https://www.ansible.com/integrations/networks/f5.
        * Requires the f5-sdk Python package on the host. This is as easy as `pip install f5-sdk'.

  REQUIREMENTS:  f5-sdk >= 3.0.9

  AUTHOR: Tim Rupp (@caphrim007), Wojciech Wypior (@wojtek0806)
          METADATA:
            status:
            - preview
            supported_by: community


  EXAMPLES:
  - name: Create pool
    bigip_pool:
      server: lb.mydomain.com
      user: admin
      password: secret
      state: present
      name: my-pool
      partition: Common
      lb_method: least-connection-member
      slow_ramp_time: 120
    delegate_to: localhost

  - name: Modify load balancer method
    bigip_pool:
      server: lb.mydomain.com
      user: admin
      password: secret
      state: present
      name: my-pool
      partition: Common
      lb_method: round-robin
    delegate_to: localhost
  
  



