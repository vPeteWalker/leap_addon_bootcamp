.. _onpremleap2_UPFO:

----------------------------
Unplanned Failover with Leap
----------------------------

Staging Guest Script
++++++++++++++++++++

New in 5.17, Leap allows you to execute scripts within a guest to update configuration files or perform other critical functions as part of the runbook. In this exercise, you'll stage a script on your WebServer VM that will update its configuration file responsible for the MySQL VM connection, allowing the WebServer to connect to the MySQL database after failover to our **RecoverySite** network.

#. SSH into your *Initials*\ **-WebServer-...** VM using the following credentials:

   - **User Name** - centos
   - **Password** - nutanix/4u

#. Within the VM SSH session, execute the following:

   .. code-block:: bash

      cd /usr/local/sbin
      sudo wget https://github.com/vPeteWalker/leap_addon_bootcamp/raw/master/production_vm_recovery
      sudo chmod +x /usr/local/sbin/production_vm_recovery

   .. note::

      Run ``sudo cat /usr/local/sbin/production_vm_recovery`` to view the contents of the failover script``.

Creating A Protection Policy
++++++++++++++++++++++++++++

A protection policy is where you specify your Recovery Point Objectives (RPO) and retention policies.

#. In Prism Central, open :fa:`bars` **> Policies > Protection Policies**.

#. Click **Create Protection Policy**.

#. Fill out the following fields and click **Save**.

   - **Name** - *Initials*\ -FiestaProtection
   - **Primary Cluster(s)** - PrimarySite
   - **Recovery Location** - `PC_` *RecoverySite IP*
   - **Target Cluster** - RecoverySite
   - Under **Policy Type**, select **Synchronous**
   - Under **Failure Handling**, select **Automatic**
   - **Timeout After** - 10 Seconds

   .. figure:: images/Protection/1.png

   .. note::

      Protection policies can be automatically applied based on Category assignment, allowing VMs to be automatically protected from their initial provisioning. We will not assign categories in this lab.

Assigning A Protection Policy
+++++++++++++++++++++++++++++

#. In Prism Central, open :fa:`bars` **> Virtual Infrastructure > VMs**.

#. Select both of your VMs and click **Actions > Protect**.

#. Select your *Initials*\ **-FiestaProtection** policy and click **Protect**.

   .. figure:: images/Protection/2.png

#. In the **VM List**, click **Focus** and select **Data Protection** from the drop down menu.

   .. figure:: images/Protection/3.png

#. Observe the **Protection Status** of each of your VMs move to **Synced**.

   .. figure:: images/Protection/4.png

Creating A Recovery Plan
++++++++++++++++++++++++

#. In Prism Central, open :fa:`bars` **> Policies > Recovery Plans**.

#. Click **Create Recovery Plan**.

#. Select *RecoverySite PC* as your **Recovery Location** and click **Proceed**.

#. Specify *Initials*\ **-FiestaRecovery** as your **Recovery Plan Name** and click **Next**.

#. Under **Power On Sequence** we will add our VMs in stages to the plan. Click **+ Add Entities**.

#. Select your *Initials*\ **-MySQL-...** VM and click **Add**.

   .. figure:: images/Recovery/1.png

#. Click **+ Add New Stage**. Under **Stage 2**, click **+ Add Entities**.

   .. figure:: images/Recovery/2.png

#. Select your *Initials*\ **-WebServer-...** VM and click **Add**.

#. Select your *Initials*\ **-WebServer-...** VM and click **Manage Scripts > Enable**. This will run the **production_vm_recovery** script within the guest VM you staged in a previous exercise.

   .. figure:: images/Recovery/3.png

   .. note::

      Leap guest script locations
         - **Windows** (Relative to Nutanix directory in Program Files)

            Production: scripts/production/vm_recovery.bat

            Test: scripts/test/vm_recovery.bat

         - **Linux**

            Production: /usr/local/sbin/production_vm_recovery

            Test: /usr/local/sbin/test_vm_recovery for Windows and Linux guests.

#. Click **+ Add Delay** between your two stages.

   .. figure:: images/Recovery/4.png

#. Specify **60** seconds and click **Add**.

#. Click **Next**.

   In this step you will configure network settings which enable you to map networks in the local availability zone (*PrimarySite*) to networks at the recovery location (*RecoverySite*).

#. Select the networks where your VMs reside for **Local AZ (Primary) - Production** and **Local AZ (Primary) - Test Failback**. Repeat for **PC_ *RecoverySite PC IP* (Recovery) - Production** and ** PC_ *RecoverySite PC IP* (Recovery) - Test Failback.**

   .. figure:: images/Recovery/5.png

#. Click **Done**.

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

#. Open http:// *Initials-WebServer-VM-IP-Address* :5001 in another browser tab.

#. Under **Stores**, click **Add New Store** and fill out the required fields. Validate your new store appears in the UI.

   .. figure:: images/Failover/1.png

#. Log in to Prism Central for your **RecoverySite**.

#. Open :fa:`bars` **> Policies > Recovery Plans**.

#. Select your *Initials*\ **-FiestaRecovery** plan and click **Actions > Failover**.

   .. figure:: images/Failover/2.png

#. Under **Failover Type**, select **Unplanned Failover** and click **Failover**.

   .. figure:: images/Failover/3.png

#. Ignore any warnings in the Recovery AZ (*RecoverySite*) and click **Execute Anyway**.

#. Click the **Name** of your Recovery Plan to monitor status of plan execution. Select **Tasks > Failover** for full details.

   .. figure:: images/Failover/4.png

.. note::

   If you had validation warnings before initiating failover, it is normal for the *Validating Recovery Plan* step to show a Status of *Failed*.

#. Once the Recovery Plan reaches 100%, open :fa:`bars` **> Virtual Infrastructure > VMs** and note the *new* IP Address of your *Initials*\ **-WebServer-...**.

#. Open `http://` `Initials-WebServer-VM-NEW-IP-Address` :5001 in another browser tab and verify the change you'd made to your application is present.

Congratulations! You've completed your first DR failover with Nutaix AHV, leveraging native Leap runbook capabilities and synchronous replication.

Performing An Unplanned Failback
++++++++++++++++++++++++++++++++

Before performing our failback, let's make another update to our application.

#. Open http:// *Initials-WebServer-VM-IP-Address* :5001 in another browser tab.

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

#. Once the Recovery Plan reaches 100%, open :fa:`bars` **> Virtual Infrastructure > VMs** and note the *new* IP Address of your *Initials*\ **-WebServer-...**.

#. Open `http://` `Initials-WebServer-VM-NEW-IP-Address` :5001 in another browser tab and verify the change you'd made to your application is present.

Congratulations! You've completed your first DR failback with Nutaix AHV, leveraging native Leap runbook capabilities and synchronous replication.
