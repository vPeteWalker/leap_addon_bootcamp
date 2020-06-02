.. _onpremleap1_setup:

----------------------
Setup and Requirements
----------------------

Nutanix AOS 5.17 offer significant enhancements to Leap for on-premises failover operations, including support for execution of guest scripts and synchronous replication with AHV.

**In this lab you deploy a multi-tier application, protect your VMs, build a Recovery Plan for runbook automation, and perform a failover operation to another Nutanix cluster.**

Lab Requirements
++++++++++++++++

#. Two clusters - each running AOS and PC 5.17 or later.

#. Each cluster has a Data Services IP configured within Prism.

#. If you are using the HPOC environment, reserve two clusters in the same datacenter to ensure synchronous replication latency requirements are met **LINK TO SYNC REQUIREMENTS**.

#. Designate one cluster as *PrimarySite*, and one as *SecondarySite*. Recommend you rename each cluster within Prism to aid with identification during this lab.

#. Configure a Primary (and optionally a Secondary) network within Prism, including IP Address Management (IPAM) on one or both networks. When utilizing the HPOC, both Primary and Secondary network information will be provided.  **HOW MANY PEOPLE CAN DO THE LAB BASED ON # OF IPS?**

#. Calm is enabled.

Limitations
+++++++++++

(As of June 2nd, 2020)

#. Planned for 5.17.1
   Test Failover
   Planned Failover

#. Not supported
   Volume Groups (Planned 5.21)
   Files (Planned 5.19+)
   Single PC (Planned 5.19)
   Automatic resumption from outage when Automatic Failure Handling is set.
   1-node and 2-node configurations
   Multi-site Sync Rep
   Multi-site Sync + NearSync/Async

AHV Sync-Rep FAQ
#. https://docs.google.com/document/d/1jqJ8bENTS8LW0oAOezXlucgB9Zfylj6n5HupeJUOE5s/edit?ts=5e7b7d65

Future
++++++

#. Add alternative instructions to deploying a multi-VM application via Calm (ex. customers/prospects interested in Leap, but do not own Calm)

#. Add Windows-based activity

Calm configuration
++++++++++++++++++

#. In **Prism Central**, select :fa:`bars` **> Services > Calm**.

#. Select **Projects** from the lefthand menu and click **+ Create Project**.

   .. figure:: images/Calm/23.png

#. Fill out the following fields:

   - **Project Name** - *Initials*\ -FiestaProject

   - Under **Infrastructure**, select **Select Provider > Nutanix**

   - Click **Select Clusters & Subnets**

   - Select *Your PrimarySite cluster*

   - Under **Subnets**, select **Primary**, (optionally) **Secondary**. Click **Confirm**

   - Mark *Your Preferred Network (Primary or Secondary)* as the default network by clicking the :fa:`star`

#. Click **Save & Configure Environment**.

``This will redirect you to the Envrionments page, but there is nothing needed to configure here. You may now move on to the next step.``

Staging Blueprints
..................

A Blueprint is the framework for every application that you model by using Nutanix Calm. Blueprints are templates that describe all the steps that are required to provision, configure, and execute tasks on the services and applications that are created. A Blueprint also defines the lifecycle of an application and its underlying infrastructure, starting from the creation of the application to the actions that are carried out on a application (updating software, scaling out, etc.) until the termination of the application.

You can use Blueprints to model applications of various complexities; from simply provisioning a single virtual machine to provisioning and managing a multi-node, multi-tier application.

#. `Download the Fiesta-Multi Blueprint by right-clicking here <https://raw.githubusercontent.com/nutanixworkshops/ts2020/master/pc/dayinlife/Fiesta-Multi.json>`_.

#. Log in to Prism Central for your **PrimarySite** cluster.

#. Open :fa:`bars` **Prism Central > Calm**, select **Blueprints** from the lefthand menu and click **Upload Blueprint**.

   .. figure:: images/Calm/25.png

#. Select **Fiesta-Multi.json**.

#. Update the **Blueprint Name** to include your initials. Even across different projects, Calm Blueprint names must be unique.

#. Select your Calm project and click **Upload**.

   .. figure:: images/Calm/26.png

#. In order to launch the Blueprint you must first assign a network to the VM. Select the **NodeReact** Service, and in the **VM** Configuration menu on the right, select *Your Preferred Network* as the **NIC 1** network.

#. Specify the *Initials*\ **-Team: Fiesta** and **Environment: Dev** categories for the **NodeReact** Service.

   .. figure:: images/Calm/27.png

#. Repeat the **NIC 1** and **Category** assignment for the **MySQL** Service.

#. Click **Credentials** to define a private key used to authenticate to the CentOS VM that will be provisioned by the Blueprint.

   .. figure:: images/Calm/27b.png

#. Expand the **CENTOS** credential and use your preferred SSH key, or paste in the following value as the **SSH Private Key**:

   ::

      -----BEGIN RSA PRIVATE KEY-----
      MIIEowIBAAKCAQEAii7qFDhVadLx5lULAG/ooCUTA/ATSmXbArs+GdHxbUWd/bNG
      ZCXnaQ2L1mSVVGDxfTbSaTJ3En3tVlMtD2RjZPdhqWESCaoj2kXLYSiNDS9qz3SK
      6h822je/f9O9CzCTrw2XGhnDVwmNraUvO5wmQObCDthTXc72PcBOd6oa4ENsnuY9
      HtiETg29TZXgCYPFXipLBHSZYkBmGgccAeY9dq5ywiywBJLuoSovXkkRJk3cd7Gy
      hCRIwYzqfdgSmiAMYgJLrz/UuLxatPqXts2D8v1xqR9EPNZNzgd4QHK4of1lqsNR
      uz2SxkwqLcXSw0mGcAL8mIwVpzhPzwmENC5OrwIBJQKCAQB++q2WCkCmbtByyrAp
      6ktiukjTL6MGGGhjX/PgYA5IvINX1SvtU0NZnb7FAntiSz7GFrODQyFPQ0jL3bq0
      MrwzRDA6x+cPzMb/7RvBEIGdadfFjbAVaMqfAsul5SpBokKFLxU6lDb2CMdhS67c
      1K2Hv0qKLpHL0vAdEZQ2nFAMWETvVMzl0o1dQmyGzA0GTY8VYdCRsUbwNgvFMvBj
      8T/svzjpASDifa7IXlGaLrXfCH584zt7y+qjJ05O1G0NFslQ9n2wi7F93N8rHxgl
      JDE4OhfyaDyLL1UdBlBpjYPSUbX7D5NExLggWEVFEwx4JRaK6+aDdFDKbSBIidHf
      h45NAoGBANjANRKLBtcxmW4foK5ILTuFkOaowqj+2AIgT1ezCVpErHDFg0bkuvDk
      QVdsAJRX5//luSO30dI0OWWGjgmIUXD7iej0sjAPJjRAv8ai+MYyaLfkdqv1Oj5c
      oDC3KjmSdXTuWSYNvarsW+Uf2v7zlZlWesTnpV6gkZH3tX86iuiZAoGBAKM0mKX0
      EjFkJH65Ym7gIED2CUyuFqq4WsCUD2RakpYZyIBKZGr8MRni3I4z6Hqm+rxVW6Dj
      uFGQe5GhgPvO23UG1Y6nm0VkYgZq81TraZc/oMzignSC95w7OsLaLn6qp32Fje1M
      Ez2Yn0T3dDcu1twY8OoDuvWx5LFMJ3NoRJaHAoGBAJ4rZP+xj17DVElxBo0EPK7k
      7TKygDYhwDjnJSRSN0HfFg0agmQqXucjGuzEbyAkeN1Um9vLU+xrTHqEyIN/Jqxk
      hztKxzfTtBhK7M84p7M5iq+0jfMau8ykdOVHZAB/odHeXLrnbrr/gVQsAKw1NdDC
      kPCNXP/c9JrzB+c4juEVAoGBAJGPxmp/vTL4c5OebIxnCAKWP6VBUnyWliFhdYME
      rECvNkjoZ2ZWjKhijVw8Il+OAjlFNgwJXzP9Z0qJIAMuHa2QeUfhmFKlo4ku9LOF
      2rdUbNJpKD5m+IRsLX1az4W6zLwPVRHp56WjzFJEfGiRjzMBfOxkMSBSjbLjDm3Z
      iUf7AoGBALjvtjapDwlEa5/CFvzOVGFq4L/OJTBEBGx/SA4HUc3TFTtlY2hvTDPZ
      dQr/JBzLBUjCOBVuUuH3uW7hGhW+DnlzrfbfJATaRR8Ht6VU651T+Gbrr8EqNpCP
      gmznERCNf9Kaxl/hlyV5dZBe/2LIK+/jLGNu9EJLoraaCBFshJKF
      -----END RSA PRIVATE KEY-----

#. Click **Save** and click **Back** once the Blueprint has completed saving.

#. Click on **AHV** in the lower left corner.

   .. figure:: images/Calm/1.png

#. Expand the **db_password** entry within the *Application Profile Name* area, along the right hand side.

   .. figure:: images/Calm/2.png

     .. Within the *Value* text box, enter *nutanix/4u* as the password.

#. Click **Save**.

#. Verify no remaining warnings or errors are present for the blueprint before proceeding.

Deploy a multi-VM application via Calm
......................................

#. Open :fa:`bars` **> Services > Calm** and select **Blueprints** from the sidebar.

#. Select the **FiestaApp** Blueprint and click **Actions > Launch**.

   .. figure:: images/2.png

#. Fill out the following fields and then click **Create** to begin provisioning your application:

   - **Name of the Application** - *Initials*\ -FiestaApp
   - **UserInitials** - *Initials*

#. Monitor the status of the application in the **Audit** tab and proceed once your application enters a **Running** state.

#. On the **Services** tab, select the **NodeReact** service and note the IP Address. This is the web server hosting the front end of your application.

#. Open http://<*NodeReact-VM-IP-Address:5001*> in a new browser tab and validate you can access the Fiesta Inventory Management app.

   .. figure:: images/5.png

Installing Nutanix Guest Tools
++++++++++++++++++++++++++++++

#. Open :fa:`bars` **> Virtual Infrastructure > VMs**.

#. Select your *Initials*\ **-WebServer-...** VM and click **Actions > Update**.

#. Under **Disks**, click :fa:`eject` beside **CD-ROM** to unmount the Cloud-Init disk mounted during the Calm deployment.

#. Click **Save**.

#. Repeat **Steps 2-4** to eject the **CD-ROM** on your *Initials*\ **-MySQL-...** VM.

#. Select both VMs and click **Actions > Install NGT**.

   .. figure:: images/4.png

#. Select **Restart as soon as the install is completed** and click **Confirm & Enter Password**.

   .. figure:: images/4b.png

#. Provide the following credentials and click **Done** to begin the NGT installation:

   - **User Name** - centos
   - **Password** - nutanix/4u

   .. figure:: images/4c.png

   ..   #. Click **Confirm and Enter Password > Skip and Mount** to mount the NGT .iso to your VMs.

      .. note::

         Nutanix Calm currently supports automatic installation of NGT for single VM blueprints, and plans to support multi-VM blueprints in the future.

      #. SSH into your *Initials*\ **-WebServer-...** VM using the following credentials:

         - **User Name** - centos
         - **Password** - nutanix/4u

      #. Within the VM SSH session, execute the following to install NGT:

         .. code-block:: bash

            sudo mount /dev/sr0 /mnt
            sudo /mnt/installer/linux/install_ngt.py
            sudo reboot

      #. Repeat **Steps 8-9** with your *Initials*\ **-MySQL-...** VM.

#. Once both VMs have rebooted, validate both VMs now have empty CD-ROM drives and **NGT Status** displays **Latest** in Prism Central.

   .. figure:: images/6.png

Staging Guest Script
++++++++++++++++++++

New in 5.17, Leap allows you to execute scripts within a guest to update configuration files or perform other critical functions as part of the runbook. In this exercise you'll stage a script on your WebServer VM that will update its configuration file responsible for the MySQL VM connection, allowing the WebServer to connect to the MySQL database after failover to our **SecondarySite** network.

#. SSH into your *Initials*\ **-WebServer-...** VM using the following credentials:

   - **User Name** - centos
   - **Password** - nutanix/4u

#. Within the VM SSH session, execute the following:

   .. code-block:: bash

      cd /usr/local/sbin
      sudo wget https://raw.githubusercontent.com/vPeteWalker/leap_addon_bootcamp/master/production_vm_recovery
      sudo chmod +x /usr/local/sbin/production_vm_recovery

   .. note::

      Run ``sudo cat /usr/local/sbin/production_vm_recovery`` to view the contents of the failover script``.

Creating a new Availability Zone
++++++++++++++++++++++++++++++++

   .. note::

      Creating an Availability Zone (AZ) can only be done once per set of physical clusters, and should be done by an instructor, so that all participants can experience the process documented below.

#. Log in to Prism Central for your **PrimarySite** cluster.

#. Open :fa:`bars` **> Administration > Availability Zones** and observe that a Local AZ has already been created by default. Click **Connect to Availability Zone**

.. figure:: images/AZ/1.png

#. In the *Availability Zone Type* dropdown, select **Physical Location**. Enter the IP, username, and password for the **SecondarySite** PC, and click **Connect**.

.. figure:: images/AZ/2.png

.. figure:: images/AZ/3.png

#. Observe that the **SecondarySite** cluster is now listed as *Physical*, and its *Connectivity Status* is listed as *Reachable*

MOVE ONTO NEXT PART - ADD LINK HERE
