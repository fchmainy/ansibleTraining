Task5: Create a Jenkins Freestyle Job
=============================

1. Create a new Job
---------------------------
On the left hand menu, click on ** New Item**

.. image:: ../images/image001.jpeg
   :scale: 50 %
   :align: center 


Let’s call this job **task5**
Select **Freestyle Project**

.. image:: ../images/image002.jpeg
   :scale: 50 %
   :align: center 

.. image:: ../images/image003.jpeg
   :scale: 50 %
   :align: center 

click **Ok**


2. Configure the job
--------------------------
**Add the rundocker playbook invocation to the jenkins job**

(optional) add into the description field: *"This is simple freestyle job to run sequentially 2 ansible playbooks.”*

scroll down to the end of the job until you reach the **Build** category.

.. image:: ../images/image004.jpeg
   :scale: 50 %
   :align: center 

Then add a build step by clicking on **“Add a build step"**

It will open a drop down list with all the baked in plugins and the additional plugins you could have installed on your Jenkins server.

.. image:: ../images/image005.jpeg
   :scale: 50 %
   :align: center 

Click on **Invoke Ansible Playbook**

Firstly, let’s configure the invocation of the run docker playbook.

in the playbook path, add the following line:
**/tmp/task3.yml**

Do not specify Inventory, as we will use the main inventory (/etc/ansible/hosts).

We need to specify this ansible-playbook needs sudo privileges to be executed:

.. image:: ../images/image006.jpeg
   :scale: 50 %
   :align: center 



**Add the lbsvc playbook invocation to the jenkins job**

Add a new **Invoke Ansible Playbook** as an additional build step of your job:

.. image:: ../images/image007.jpeg
   :scale: 50 %
   :align: center 

We will now run the task4.yml Ansible playbook, which is in charge of deploying the configuration on your bigip:

.. image:: ../images/image008.jpeg
   :scale: 50 %
   :align: center 

Again, we will use the main host file, so you don’t need to specify an extra inventory.

there are yet no vault credentials available in our configuration. Let’s create a new credential entry in Jenkins to store our ansible vault password.
Click on “Add” on the “Vault Credentials” line.

.. image:: ../images/image009.jpeg
   :scale: 50 %
   :align: center 

Select **Secret text** as a credential *kind*:

.. image:: ../images/image010.jpeg
   :scale: 50 %
   :align: center 

Then, fill the required fields with the following values:
	* secret: default
	* ID: vaultTask4
	* Description: Ansible vault password for the lbsvc role execution

.. image:: ../images/image011.jpeg
   :scale: 50 %
   :align: center 


Note:
	You have seen Jenkins can store different kind of confidential data in its credential store.
	We could have created our BigIP passwords in the Jenkins store which is still a valid method and probably an easier way of managing all the credentials from a central standpoint but it will come to the discussion about what is the safest solution between ansible vault (AES256) and Jenkins credential store (AES128).


You can reorder the two playbooks invocations easily by clicking and dragging the 2 **Invoke Ansible Playbook** on their top left corner as the two playbooks are independent of each other.

.. image:: ../images/image012.jpeg
   :scale: 50 %
   :align: center 

Click ** Save **



3. Run the freestyle job
-------------------------
On the left hand menu, you can run the playbook by clicking on the *Build now** button

.. image:: ../images/image013.jpeg
   :scale: 50 %
   :align: center 

On the Build History window on the bottom left corner, you can have details on the execution of your job:

.. image:: ../images/image014.jpeg
   :scale: 50 %
   :align: center 

click on the #number of your execution.
it will open a new page for this job execution (called build).

.. image:: ../images/image015.jpeg
   :scale: 50 %
   :align: center 

the **Console Output** button will give you insights on the build steps:

.. parsed-literal::

    Started by user Jenkins admin
    Building in workspace /var/lib/jenkins/workspace/task5
    [task5] $ ansible-playbook /tmp/task3.yml -f 5
    [DEPRECATION WARNING]: DEFAULT_SUDO_USER option, In favor of Ansible Become, 
    which is a generic framework. See become_user. , use become instead. This 
    feature will be removed in version 2.8. Deprecation warnings can be disabled by
    setting deprecation_warnings=False in ansible.cfg.
    [WARNING]: Found variable using reserved name: remote_user

    PLAY [me] **********************************************************************

    TASK [Gathering Facts] *********************************************************
    ok: [127.0.0.1]

    TASK [fch.rundocker : create and run a docker container] ***********************
    changed: [127.0.0.1 -> localhost] => (item=9081)
    changed: [127.0.0.1 -> localhost] => (item=9082)
    changed: [127.0.0.1 -> localhost] => (item=9083)

    PLAY RECAP *********************************************************************
    127.0.0.1                  : ok=2    changed=1    unreachable=0    failed=0   

    [task5] $ ansible-playbook /tmp/task4.yml -f 5 --vault-password-file /var/lib/jenkins/workspace/task5/vault8754706889869935773.password
    [DEPRECATION WARNING]: DEFAULT_SUDO_USER option, In favor of Ansible Become, 
    which is a generic framework. See become_user. , use become instead. This 
    feature will be removed in version 2.8. Deprecation warnings can be disabled by
    setting deprecation_warnings=False in ansible.cfg.

    PLAY [Configure http service] **************************************************

    TASK [fch.lbsvc : Create nodes] ************************************************
    changed: [192.168.1.143 -> localhost] => (item={u'host': u'10.100.26.144', u'port': u'80'})
    changed: [192.168.1.143 -> localhost] => (item={u'host': u'10.100.26.145', u'port': u'80'})
    [DEPRECATION WARNING]: Param 'user' is deprecated. See the module docs for more
    information. This feature will be removed in version 2.9. Deprecation warnings
    can be disabled by setting deprecation_warnings=False in ansible.cfg.
    [DEPRECATION WARNING]: Param 'password' is deprecated. See the module docs for 
    more information. This feature will be removed in version 2.9. Deprecation 
    warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
    [DEPRECATION WARNING]: Param 'server' is deprecated. See the module docs for 
    more information. This feature will be removed in version 2.9. Deprecation 
    warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
    [DEPRECATION WARNING]: Param 'validate_certs' is deprecated. See the module 
    docs for more information. This feature will be removed in version 2.9. 
    Deprecation warnings can be disabled by setting deprecation_warnings=False in 
    ansible.cfg.

    TASK [fch.lbsvc : Create pool] *************************************************
    changed: [192.168.1.143 -> localhost]

    TASK [fch.lbsvc : Add Pool members] ********************************************
    changed: [192.168.1.143 -> localhost] => (item={u'host': u'10.100.26.144', u'port': u'80'})
    changed: [192.168.1.143 -> localhost] => (item={u'host': u'10.100.26.145', u'port': u'80'})

    TASK [fch.lbsvc : Add Virtual Server] ******************************************
    changed: [192.168.1.143 -> localhost]

    TASK [fch.lbsvc : Add Redirect Virtual Server] *********************************
    changed: [192.168.1.143 -> localhost]

    PLAY RECAP *********************************************************************
    192.168.1.143              : ok=5    changed=5    unreachable=0    failed=0   

    Finished: SUCCESS
