.. _onpremleap3_PFO:

-----------------------------------
[FUTURE] Planned Failover with Leap
-----------------------------------

.. note::

   Planned failover functionality is scheduled to be released in AOS v5.17.1


Staging Guest Script
++++++++++++++++++++

New in 5.17, Leap allows you to execute scripts within a guest to update configuration files or perform other critical functions as part of the runbook. In this exercise you'll stage a script on your WebServer VM that will update its configuration file responsible for the MySQL VM connection, allowing the WebServer to connect to the MySQL database after failover to our **RecoverySite** network.

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

#. In Prism Central, open :fa:`bars` **> Policies > Protection Policies**.

#. Click **Create Protection Policy**.

#. Fill out the following fields:

   - **Name** - *Initials*\ -FiestaProtection
   - **Primary Cluster(s)** - PrimarySite
   - **Recovery Location** - PC_*RecoverySite IP*
   - **Target Cluster** - RecoverySite
   - Under **Policy Type**, select **Synchronous**
   - Under **Failure Handling**, select **Automatic**
   - **Timeout After** - 10 Seconds

   .. figure:: images/7.png

   .. note::

      Protection policies can be automatically applied based on Category assignment, allowing VMs to be automatically protected from their initial provisioning. We will not assign categories in this lab.

#. Click **Save**.

Assigning A Protection Policy
+++++++++++++++++++++++++++++

#. In Prism Central, open :fa:`bars` **> Virtual Infrastructure > VMs**.

#. Select both of your VMs and click **Actions > Protect**.

#. Select your *Initials*\ **-FiestaProtection** policy and click **Protect**.

   .. figure:: images/9.png

#. In the **VM List**, click **Focus** and select **Data Protection** from the drop down menu.

   .. figure:: images/10.png

#. Observe the **Protection Status** of each of your VMs move to **Synced**.

   .. figure:: images/11.png

Creating A Recovery Plan
++++++++++++++++++++++++

#. In Prism Central, open :fa:`bars` **> Policies > Recovery Plans**.

#. Click **Create Recovery Plan**.

#. Select **PC_10.38.173.40** as your **Recovery Location** and click **Proceed**.

#. Specify *Initials*\ **-FiestaRecovery** as your **Recovery Plan Name** and click **Next**.

#. Under **Power On Sequence** we will add our VMs in stages to the plan. Click **+ Add Entities**.

#. Select your *Initials*\ **-MySQL-...** VM and click **Add**.

   .. figure:: images/12.png

#. Click **+ Add New Stage**. Under **Stage 2**, click **+ Add Entities**.

   .. figure:: images/13.png

#. Select your *Initials*\ **-WebServer-...** VM and click **Add**.

#. Select your *Initials*\ **-WebServer-...** VM and click **Manage Scripts > Enable**. This will run the **production_vm_recovery** script within the guest VM you staged in a previous exercise.

   .. figure:: images/21.png

   .. note::

      You can mouse over **Script Path** to see where Leap expects guest scripts for Windows and Linux guests.

#. Click **+ Add Delay** between your two stages.

   .. figure:: images/14.png

#. Specify **60** seconds and click **Add**.

#. Click **Next**.

   In this step you will map VM networks from your primary site to your recovery site.

#. Select **VLAN1943** for **Local AZ Production** and **Local AZ Test Failback** Virtual Networks. Select **VLAN1733** for **PC_10.38.173.40 Production** and **PC_10.38.173.40 Test Failback** Virtual Networks.

   .. figure:: images/15.png

#. Click **Done**.

Performing An Unplanned Failover
++++++++++++++++++++++++++++++++

Before performing our failover, we'll make a quick update to our application.

#. Open http://*Initials-WebServer-VM-IP-Address*:5001 in another browser tab.

#. Under **Stores**, click **Add New Store** and fill out the required fields. Validate your new store appears in the UI.

   .. figure:: images/16.png

#. Log in to Prism Central for your **RecoverySite**

#. Open :fa:`bars` **> Policies > Recovery Plans**.

#. Select your *Initials*\ **-FiestaRecovery** plan and click **Actions > Failover**.

   .. figure:: images/17.png

#. To simulate a true DR event, under **Failover Type**, select **Unplanned Failover** and click **Failover**.

   .. figure:: images/18.png

#. Ignore any warnings related to Calm categories not found in the Recovery AZ and click **Execute Anyway**.

#. Click the **Name** of your Recovery Plan to monitor status of plan execution. Select **Tasks > Failover** for full details.

   .. figure:: images/20.png

#. Once the Recovery Plan reaches 100%, open :fa:`bars` **> Virtual Infrastructure > VMs** and note the *new* IP Address of your *Initials*\ **-WebServer-...**.

#. Open http://<*Initials-WebServer-VM-NEW-IP-Address*:5001> in another browser tab and verify the change you'd made to your application is present.

Congratulations! You've completed your first DR failover with Nutaix AHV, leveraging native Leap runbook capabilities and synchronous replication.
