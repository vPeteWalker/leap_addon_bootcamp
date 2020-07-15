.. _onpremleap2_UPFO:

----------------------------
Unplanned Failover with Leap
----------------------------

In this exercise you will perform an **Unplanned** failover of your application, demonstrating what one would experience in the event of a real cluster/site failure. The key differences from a **Planned** failover are...

.. note::

   If you have already completed the :ref:`onpremleap3_PFO` exercise, you can skip the repeat setup instructions and skip to `Performing An Unplanned Failover`_.

Enabling Leap
+++++++++++++

.. raw:: html

   <strong><font color="red">The following steps should be performed by either the instructor or a designated user, as enabling Leap and configuring the Availability Zone are one-time operations.</font></strong>

Enable Leap
...........

#. Within your *PrimarySite Prism Central*, select :fa:`bars` **> Prism Central Settings**.

#. Within the *Setup* section, click **Enable Leap > Enable**.

#. Within *RecoverySite Prism Central*, select :fa:`bars` **> Prism Central Settings**.

#. Within the *Setup* section, click **Enable Leap > Enable**.

Creating a New Availability Zone
................................

#. Within *PrimarySite Prism Central*, select :fa:`bars` **> Administration > Availability Zones** and observe that a Local AZ has already been created by default.

#. Click **Connect to Availability Zone**

   .. figure:: images/AZ/1.png

#. In the *Availability Zone Type* dropdown, select **Physical Location**. Enter the IP, username, and password for the **RecoverySite** PC, and click **Connect**.

   .. figure:: images/AZ/2.png
       :align: left

   .. figure:: images/AZ/3.png
       :align: right

#. Observe that the **RecoverySite** cluster is now listed as *Physical*, and its *Connectivity Status* is listed as *Reachable*.

Staging Guest Script
++++++++++++++++++++

New in 5.17, Leap allows you to execute scripts within a guest to update configuration files, or perform other critical functions as part of the runbook. In this exercise, you'll stage a script on your WebServer VM that will automatically update its configured IP information for the MySQL VM connection. This modification allows the WebServer to connect to the MySQL database after failover to your **RecoverySite** network.

#. SSH into your *Initials*\ **-WebServer** VM using the following credentials:

   - **User Name** - centos
   - **Password**  - nutanix/4u



#. Within the SSH session, execute the following. Click the icon in the upper right hand corner of the below window to copy the commands to your clipboard. You may then paste that within your SSH session.

   .. code-block:: bash

      cd /usr/local/sbin
      sudo wget https://github.com/vPeteWalker/leap_addon_bootcamp/raw/master/production_vm_recovery
      sudo chmod +x /usr/local/sbin/production_vm_recovery

   .. note::

      If you'd like to view the contents of the failover script, execute:

      ``sudo cat /usr/local/sbin/production_vm_recovery``

#. You may now exit the SSH session.

Installing Nutanix Guest Tools
++++++++++++++++++++++++++++++

In order to take advantage of the guest script functionality, Nutanix Guest Tools must first be installed within the guest VMs being protected.

#. Open :fa:`bars` **> Virtual Infrastructure > VMs**.

#. Select both your *USERxx*\ **-WebServer** and *Userxx*\ **-MySQL** VMs. Click **Actions > Install NGT**.

   .. figure:: images/22.png

#. Select **Restart as soon as the install is completed**, then click **Confirm & Enter Password**.

   .. figure:: images/23.png

#. Provide the following credentials and click **Done** to begin the NGT installation:

   - **User Name** - centos
   - **Password**  - nutanix/4u

   .. figure:: images/24.png

#. Once both VMs have rebooted, validate both VMs now have empty CD-ROM drives and **NGT Status** displays **Latest** in Prism Central.

   .. figure:: images/25.png

Creating A Protection Policy
++++++++++++++++++++++++++++

A protection policy is where you specify your Recovery Point Objectives (RPO) and retention policies.

#. In Prism Central, open :fa:`bars` **> Policies > Protection Policies**.

#. Click **Create Protection Policy**.

#. Fill out the following fields and click **Save**.

   - **Name**                 - *Initials*\ -FiestaProtection
   - **Primary Cluster(s)**   - PrimarySite
   - **Recovery Location**    - PC_*RecoverySite PrismIP*
   - **Target Cluster**       - RecoverySite
   - **Policy Type**          - Synchronous
   - **Failure Handling**     - Automatic
   - **Timeout After**        - 10 Seconds

      .. figure:: images/Protection/1.png

Assigning A Protection Policy
+++++++++++++++++++++++++++++

.. note::

   Protection policies can be automatically applied based on category assignment, allowing VMs to be automatically protected from their initial provisioning. You can also add VMs individually to any protection policy.

   **Choose one of the methods below.**

Method 1 - Add VM to a protection policy
........................................

#. In Prism Central, open :fa:`bars` **> Virtual Infrastructure > VMs**.

#. Select both of your VMs and click **Actions > Protect**.

#. Select your *Initials*\ **-FiestaProtection** policy and click **Protect**.

   .. figure:: images/Protection/2.png

#. In the **VM List**, click **Focus** and select **Data Protection** from the drop down menu.

   .. figure:: images/Protection/3.png

#. Observe the **Protection Status** of each of your VMs move to **Synced**.

   .. figure:: images/Protection/4.png

Method 2 - Add category to a protection policy
..............................................

#. In Prism Central, open :fa:`bars` **> Policies > Protection Policies**.

#. Select your *Initials*\ -FiestaProtection Protection Policy, and from the *Actions* dropdown, choose **Update**.

#. Under *Associated Categories* add both **CalmService: MySQL** and **CalmService: NodeReact** categories.

   .. figure:: images/Protection/5.png

#. Click **Save**.

Creating A Recovery Plan
++++++++++++++++++++++++

.. note::

   Continue using the same method you chose when configuring your protection policy in the previous section.

Method 1 - Add VM to a Recovery Plan
....................................

#. In Prism Central, open :fa:`bars` **> Policies > Recovery Plans**.

#. Click **Create Recovery Plan**.

#. Select *RecoverySite PC* as your **Recovery Location** and click **Proceed**.

#. Specify *Initials*\ **-FiestaRecovery** as your **Recovery Plan Name** and click **Next**.

#. Under **Power On Sequence** we will add our VMs in stages to the plan. Click **+ Add Entities**.

#. Select your *Initials*\ **-MySQL** VM and click **Add**.

   .. figure:: images/Recovery/1.png

#. Click **+ Add New Stage**. Under **Stage 2**, click **+ Add Entities**.

   .. figure:: images/Recovery/2.png

#. Select your *Initials*\ **-WebServer** VM and click **Add**.

#. Select your *Initials*\ **-WebServer** VM and click **Manage Scripts > Enable**. This will run the **production_vm_recovery** script within the guest VM you staged in a previous exercise.

   .. figure:: images/Recovery/3.png

#. Click **+ Add Delay** between your two stages.

   .. figure:: images/Recovery/4.png

#. Specify **60** seconds and click **Add**.

#. Click **Next**.

   In this step you will configure network settings which enable you to map networks in the local availability zone (*PrimarySite*) to networks at the recovery location (*RecoverySite*).

#. Select **VM Network** for all *Virtual Network or Port Group* entries.

   .. figure:: images/Recovery/15.png

.. note::

   You are able to override the IP address failover scheme by clicking the *Advance Settings > + Custom IP Mapping*. The VMs must have a static IP address assigned already, before those VMs are available in this section. You can modify the *Test Failback* (Primary Site), *Production* (Recovery Site), and *Test Failover* (Recovery Site). Click *Save* once your modifications are complete.

   .. figure:: images/Recovery/customIP1.png

#. Click **Done**.

Method 2 - Add category to a recovery plan
..........................................

#. In Prism Central, open :fa:`bars` **> Policies > Recovery Plans**.

#. Click **Create Recovery Plan**.

#. Select *RecoverySite PC* as your **Recovery Location** and click **Proceed**.

#. Specify *Initials*\ **-FiestaRecovery** as your **Recovery Plan Name** and click **Next**.

#. Under **Power On Sequence** we will add our VMs in stages to the plan. Click **+ Add Entities**.

#. From the dropdown, choose **Category**. Type **CalmService** in the text box to the right, and select **CalmService: MySQL** in the lower window.

   .. figure:: images/Recovery/category1.png

#. Click **+ Add New Stage**. Under **Stage 2**, click **+ Add Entities**.

   .. figure:: images/Recovery/category2.png

#. From the dropdown, choose **Category**. Type **CalmService** in the text box to the right, and select **CalmService: NodeReact** in the lower window.

#. Select your **CalmService: NodeReact** category and click **Manage Scripts > Enable**. This will run the **production_vm_recovery** script within the guest VM you staged in a previous exercise.

   .. figure:: images/Recovery/category3.png

#. Click **+ Add Delay** between your two stages.

   .. figure:: images/Recovery/4.png

#. Specify **60** seconds and click **Add**.

#. Click **Next**.

   In this step you will configure network settings which enable you to map networks in the local availability zone (*PrimarySite*) to networks at the recovery location (*RecoverySite*).

#. Select **VM Network** for all *Virtual Network or Port Group* entries.

   .. figure:: images/15.png

#. Click **Done**.

.. note::

   Leap guest script locations
      - **Windows** (Relative to Nutanix directory in Program Files)

         Production: scripts/production/vm_recovery.bat

         Test: scripts/test/vm_recovery.bat

      - **Linux**

         Production: /usr/local/sbin/production_vm_recovery

         Test: /usr/local/sbin/test_vm_recovery for Windows and Linux guests.

Performing An Unplanned Failover
++++++++++++++++++++++++++++++++

There are 3 types of failovers: Test, Planned and Unplanned.

- Test failovers are for testing a recovery plan. VMs are started in the test network as specified in the recovery plan. VMs at the primary location are not affected.

- Planned failovers (PFO) are when disruption of services is predicted at the primary site. The recovery plan will first create a snapshot of each VM, replicates, then starts them at the recovery location. They no longer run at the primary site after a planned failover has occurred. Replication then begins in the reverse direction (from *RecoverySite* to *PrimarySite*)

- Unplanned failovers (UPFO) occur when a disaster has already occurred at the primary location. VMs are recovered from the most recent snapshot, and are started at the recovery site.

**In this exercise, you will perform an Unplanned Failover (UPFO).**

Failovers are initiated from the remote site, which can either be another on-prem Prism Central located at your DR site, or Xi Cloud Servies.

In this exercise, we will be connecting to an on-prem Prism Central at the *RecoverySite*, which we've already paired with the *PrimarySite* on-prem cluster.

Before performing our failover, let's make a quick update to our application.

#. Open `<http://Initials-WebServer-IP-address:5001>`_ in another browser tab. (ex. `<http://10.42.212.50:5001>`_)

#. Under **Stores**, click **Add New Store** and fill out the required fields. Validate your new store appears in the UI.

   .. figure:: images/Failover/1.png

#. Log in to Prism Central for your **RecoverySite**.

#. Open :fa:`bars` **> Policies > Recovery Plans**.

#. Select your *Initials*\ **-FiestaRecovery** plan and click **Actions > Failover**.

   .. figure:: images/Failover/2.png

#. Under **Failover Type**, select **Unplanned Failover** and click **Failover**.

   .. figure:: images/Failover/3.png

#. Ignore any warnings in the Recovery AZ (*RecoverySite*) and click **Execute Anyway**.

#. Click on *Initials*\ **-FiestaRecovery** to monitor status of plan execution. Select **Tasks > Failover** for full details.

   .. figure:: images/Failover/4.png

   .. note::

      If you had validation warnings before initiating failover, it is normal for the *Validating Recovery Plan* step to show a Status of *Failed*.

#. Once the Recovery Plan reaches 100%, open :fa:`bars` **> Virtual Infrastructure > VMs** and note the *RECOVERYSITE* IP Address of your *Initials*\ **-WebServer**.

#. Open `<http://Initials-WebServer-VM-RECOVERYSITE-IP-Address:5001>`_ in another browser tab and verify the change you'd made to your application is present.

Congratulations! You've completed your first DR failover with Nutaix AHV, leveraging native Leap runbook capabilities and synchronous replication.

Performing An Unplanned Failback
++++++++++++++++++++++++++++++++

Before performing our failback, let's make another update to our application.

#. Open `<http://Initials-WebServer-VM-RECOVERYSITE-IP-Address:5001>`_ in another browser tab.

#. Under **Stores**, click **Add New Store** and fill out the required fields. Validate your new store appears in the UI.

   .. figure:: images/Failover/1.png

#. Log in to Prism Central for your **PrimarySite**.

#. Open :fa:`bars` **> Virtual Infrastructure > VMs**.

#. Select both of your VMs and click **Actions > Delete**. Confirm by clicking **Delete**.

#. Open :fa:`bars` **> Policies > Recovery Plans**.

#. Select your *Initials*\ **-FiestaRecovery** plan and click **Actions > Failover**.

   .. figure:: images/Failover/2.png

#. Under **Failover Type**, select **Unplanned Failover** and click **Failover**.

   .. figure:: images/Failover/3.png

#. Ignore any warnings in the Recovery AZ (*PrimarySite*) and click **Execute Anyway**.

#. Click the **Name** of your Recovery Plan to monitor status of plan execution. Select **Tasks > Failover** for full details.

   .. figure:: images/Failover/4.png

.. note::

   If you had validation warnings before initiating failover, it is normal for the *Validating Recovery Plan* step to show a Status of *Failed*.

#. Once the Recovery Plan reaches 100%, open :fa:`bars` **> Virtual Infrastructure > VMs** and note the *PRIMARYSITE* IP Address of your *Initials*\ **-WebServer**.

#. Open `<http://Initials-WebServer-VM-PRIMARYSITE-IP-Address:5001>`_ in another browser tab and verify the change you'd made to your application is present.

Congratulations! You've completed your first DR failback with Nutanix AHV, leveraging native Leap runbook capabilities and synchronous replication.
