.. _onpremleap1_setup:

----------------------
Setup and Requirements
----------------------

Nutanix AOS 5.17 offer significant enhancements to Leap for on-premises failover operations, including support for execution of guest scripts and synchronous replication with AHV.

In this lab you will:

- Deploy a multi-tier application, Fiesta, using Calm
- Protect your VMs with a Protection Policy
- Build a Recovery Plan for runbook automation
- Create changes to the Fiesta application database
- Perform a failover to a physical Nutanix cluster
- Access the Fiesta application to ensure changes were retained
- Make additional changes to the Fiesta application database
- Perform a failback from a physical Nutanix cluster
- Access the Fiesta application to ensure changes were retained

Environment Requirements
++++++++++++++++++++++++

- Two AHV clusters running AOS 5.17.1 or newer, each registered to a different Prism Central. Reserve two clusters in the same HPOC datacenter to ensure synchronous replication latency requirements are met (<5ms RTT).

- The clusters must be running AHV version 20190916.189 or newer.

- If you are using the HPOC environment,

.. #. The storage container name of the protected VMs must be the same on both the primary and recovery clusters. This is addressed by the staging script.

.. Therefore, a storage container must exist on the recovery cluster with the same name as the one on the primary cluster. For example, if the protected VMs are in the *SelfServiceContainer* storage container on the primary cluster, there must also be a *SelfServiceContainer* storage container on the recovery cluster.

.. #. Set the virtual IP address and the data services IP address in the primary and the recovery clusters.

- The clusters on the **PrimarySite** and **RecoverySite** availability zones communicate over the following ports: 2030, 2036, 2073, and 2090. Follow the steps below to open the required ports. **This step is necessary even if using two HPOC clusters in the same datacenter.**

   #. Run the following command on any CVM of the **PrimarySite** cluster by remoting in via SSH (e.g. ``ssh nutanix@<CLUSTER-VIRTUAL-IP>``):

   .. code-block:: bash

      allssh 'modify_firewall -f -p 2030,2036,2073,2090 -i eth0'

   #. Run the following command on any CVM of the **RecoverySite** cluster by remoting in via SSH (e.g. ``ssh nutanix@<CLUSTER-VIRTUAL-IP>``):

      .. code-block:: bash

         allssh 'modify_firewall -f -p 2030,2036,2073,2090 -i eth0'


   #. Exit both SSH sessions.

   .. note::

      For more information about the required ports, see `General Requirements of Leap <https://portal.nutanix.com/page/documents/details/?targetId=Xi-Leap-Admin-Guide%3AXi-Leap-Admin-Guide>`_.

.. - (RECOMMENDED) By default, the staging used for the **PrimarySite** will provide a dynamic cluster name (e.g. **PHX**)

Reference
+++++++++

The following resources are provided for reference purposes and aren't required to complete the lab exercises.

- `Xi Leap Administration Guide <https://portal.nutanix.com/page/documents/details/?targetId=Xi-Leap-Admin-Guide%3AXi-Leap-Admin-Guide>`_

..   Lab Requirements
   ++++++++++++++++

.. #. Calm is enabled on the *PrimarySite* cluster.

.. #. SSH client installed (ex. Putty for Windows).

.. #. If you are using the HPOC environment, reserve two clusters in the same datacenter to ensure synchronous replication latency recommendations are met.

.. #. Designate one cluster as *PrimarySite*, and one as *RecoverySite*. We recommend you rename each cluster within Prism to aid with identification during this lab.

.. #. Configure a Primary and a VM network within Prism, including IP Address Management (IPAM) on one or both networks, including DNS, and an IP Pool. This lab requires 2 IP addresses per attendee, per physical cluster (2 at the Primary site, 2 at the Recovery site). Do not rename these networks once the lab has begun.

   .. note::

      When utilizing the HPOC, both *Primary* and *VM Network* information will be provided with your reservation.

      If you only have a single VLAN available to you, continue to create the *VM Network* using the same VLAN information as your primary. This is to allow the instructions to be easily followed in either situation.

      Recommended IPAM pools when using HPOC

      - Primary
         - Range = .50 - .125
         - IPAM DHCP = .126
         - (76 available IPs)

      - VM Network
         - Range = .132 - .253
         - IPAM DHCP = .254
         - (122 available IPs)


.. Synchronous Replication Recommendation
.. ++++++++++++++++++++++++++++++++++++++

..   - For optimal performance, Nutanix recommends that the round trip latency (RTT) between clusters be less than 5 ms., maintain adequate bandwidth to accommodate peak writes, and have a redundant physical network between the clusters.

.. Future Additions
.. ++++++++++++++++
..
..    - Implement staging to automate aspects of the setup process: Network creation, deployment of PC, enable Calm/Leap, deploy Calm blueprint for specified number of users, etc.
..    - Add alternative instructions to deploying a multi-VM application via Calm (ex. customers/prospects interested in Leap, but do not own Calm)
..    - Add Windows-based activity
      - Protect via Categories, vs. selecting VMs

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

      - Under **Subnets**, select **VM Network**. Click **Confirm**

      - Mark *VM Network* as the default network by clicking the :fa:`star`.

   #. Click **Save & Configure Environment**.

   This will redirect you to the Envrionments page, but there is nothing needed to configure here. You may now move on to the next step.

   Staging Blueprints
   ..................

   A Blueprint is the framework for every application that you model by using Nutanix Calm. Blueprints are templates that describe all the steps that are required to provision, configure, and execute tasks on the services and applications that are created. A Blueprint also defines the lifecycle of an application and its underlying infrastructure, starting from the creation of the application to the actions that are carried out on a application (updating software, scaling out, etc.) until the termination of the application.

   You can use Blueprints to model applications of various complexities; from simply provisioning a single virtual machine to provisioning and managing a multi-node, multi-tier application.

   #. `Download the Fiesta-Multi Blueprint by right-clicking here <https://github.com/vPeteWalker/leap_addon_bootcamp/raw/master/Fiesta-Multi-GITHUB.json>`_.

   #. Log in to Prism Central for your **PrimarySite** cluster.

   #. Open :fa:`bars` **Prism Central > Calm**, select **Blueprints** from the lefthand menu and click **Upload Blueprint**.

      .. figure:: images/Calm/25.png

   #. Select **Fiesta-Multi-GITHUB.json**.

   #. Update the **Blueprint Name** to include your initials. Even across different projects, Calm Blueprint names must be unique.

   #. Select your *Initials*\ -FiestaProject project and click **Upload**.

      .. figure:: images/Calm/26.png

   #. On the right hand side, expand the *db_password* section, and within the *Value* entry, type **nutanix/4u** as the password.

      .. figure:: images/Calm/26b.png

      If you accidentally clicked away from this screen, and need to revisit it, click **AHV** under *Application Profile* in the lower left hand corner

            .. figure:: images/Calm/26c.png

   #. In order to launch the Blueprint you must first assign a network to the VM. Select the **NodeReact** Service, and in the **VM** Configuration menu on the right, select *VM Network* as the **NIC 1** network.

      .. figure:: images/Calm/27.png

   #. Repeat the **NIC 1** assignment for the **MySQL** Service.

   #. Click **Credentials** to define a private key used to authenticate to the CentOS VM that will be provisioned by the Blueprint.

      .. figure:: images/Calm/27b.png

   #. Expand the **CENTOS** credential and paste in the following value as the **SSH Private Key**. Click the icon in the upper right hand corner of the below window to copy the entire private key to your clipboard.

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

      .. figure:: images/Calm/28.png

   #. Click **Save** and click **Back** once the Blueprint has completed saving.

   Deploy a multi-VM application via Calm
   ......................................

   We'll be utilizing Calm to quickly and easily deploy a multi-tier application (web and database) via two VMs. This enables us to demonstrate a real-world scenario: enable scripting to automate the configuration of the web server to accomodate the IP addresses of both VMs changing during a failover or failback.

   #. Open :fa:`bars` **> Services > Calm** and select **Blueprints** from the sidebar.

   #. Select the **FiestaApp** Blueprint and click **Actions > Launch**.

      .. figure:: images/2.png

   #. Fill out the following fields and then click **Create** to begin provisioning your application:

      - **Name of the Application** - *Initials*\ -FiestaApp
      - **user_initials**           - *Initials*



   #. Monitor the status of the application in the **Audit** tab and proceed once your application enters a **Running** state. This will take approximately 15 minutes to complete.

   #. On the **Services** tab, select the **NodeReact** service and note the IP Address. This is the web server hosting the front end of your application.

   #. Open `<http://NodeReact-VM-IP-Address:5001>`_ in a new browser tab and validate you can access the Fiesta Inventory Management app. (ex. `<http://10.42.212.50:5001>`_)

      .. figure:: images/5.png

Installing Nutanix Guest Tools
++++++++++++++++++++++++++++++

#. Open :fa:`bars` **> Virtual Infrastructure > VMs**.

#. Select both your *Initials*\ **-WebServer** and *Initials*\ **-MySQL** VMs. Click **Actions > Install NGT**.

   .. figure:: images/4.png

#. Select **Restart as soon as the install is completed**, then click **Confirm & Enter Password**.

   .. figure:: images/4b.png

#. Provide the following credentials and click **Done** to begin the NGT installation:

   - **User Name** - centos
   - **Password**  - nutanix/4u

   .. figure:: images/4c.png

#. Once both VMs have rebooted, validate both VMs now have empty CD-ROM drives and **NGT Status** displays **Latest** in Prism Central.

   .. figure:: images/6.png

..   Remove Categories
   +++++++++++++++++

   Calm creates a number of categories when deploying a blueprint. Since these categories won't be replicated to the Recovery Site, we will remove them to avoid validation warnings or errors.

   #. Open :fa:`bars` **> Virtual Infrastructure > VMs**.

   #. Select your *Initials*\ **-WebServer** VM. Click **Actions > Manage Categories**.

   #. Remove all categories by clicking the **-** next to each one. Click **Save** once complete.

   #. Select your *Initials*\ **-MySQL** VM. Click **Actions > Manage Categories**.

   #. Remove all categories by clicking the **-** next to each one. Click **Save** once complete.
