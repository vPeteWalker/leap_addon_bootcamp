.. title:: Leap Add-on Bootcamp

.. toctree::
  :maxdepth: 2
  :caption: Unplanned Failover (UPFO)
  :name: onpremleap2_UPFO/onpremleap2_UPFO
  :hidden:

  onpremleap2_UPFO/onpremleap2_UPFO

.. :maxdepth: 2
.. :caption: [FUTURE] Planned Failover (PFO)
.. :name: onpremleap3_PFO/onpremleap3_PFO
.. :hidden:
..
.. onpremleap3_PFO/onpremleap3_PFO

.. _onpremleap1_setup:

----------------------
Setup and Requirements
----------------------

Nutanix AOS 5.17 offer significant enhancements to Leap for on-premises failover operations, including support for execution of guest scripts and synchronous replication with AHV.

**In this lab you deploy a multi-tier application, protect your VMs, build a Recovery Plan for runbook automation, and perform a failover operation to another Nutanix cluster.**
|
Leap Requirements
+++++++++++++++++

#. Two AHV clusters running AOS 5.17 or newer, each registered to a different Prism Central.

#. The on-prem clusters must be running AHV version 20190916.189 or newer.

#. The storage container name of the protected VMs must be the same on both the primary and recovery clusters. Therefore, a storage container must exist on the recovery cluster with the same name as the one on the primary cluster. For example, if the protected VMs are in the *SelfServiceContainer* storage container on the primary cluster, there must also be a *SelfServiceContainer* storage container on the recovery cluster.

#. Set the virtual IP address and the data services IP address in the primary and the recovery clusters.

#. The clusters on the primary availability zone (site) and the recovery site communicate over the ports 2030, 2036, 2073, and 2090. Ensure that these ports have open access between both the primary cluster and recovery clusters (Prism Elements). For more information about the required ports, see General Requirements of Leap.

   .. note:: **This step is necessary even if using two HPOC clusters in the same datacenter**

   To open the ports for communication to the *RecoverySite* cluster, run the following command from any CVM on the **PrimarySite** cluster.

   .. code-block:: bash

      allssh 'modify_firewall -f -r recovery_cvm_ip2/cidr,recovery_cvm_ip3/cidr,recovery_cvm_ip4/cidr,recovery_cvm_ip1/cidr,recovery_virtual_ip/cidr -p 2030,2036,2073,2090 -i eth0'

   - Replace *recovery_cvm_ip#* with the IP address of the *RecoverySite* cluster CVMs.

   - Replace *remote_virtual_ip* with the virtual IP address of the *RecoverySite* cluster.

|

   To open the ports for communication to the *PrimarySite* cluster, run the following command from any CVM on the **RecoverySite** cluster.

   .. code-block:: bash

      allssh 'modify_firewall -f -r primary_cvm_ip1/cidr,primary_cvm_ip2/cidr,primary_cvm_ip3/cidr,primary_cvm_ip4/cidr,primary_virtual_ip/cidr -p 2030,2036,2073,2090 -i eth0'

   - Replace *source_cvm_ip#* with the IP address of the *PrimarySite* cluster CVMs.

   - Replace *source_virtual_ip* with the virtual IP address of the *PrimarySite* cluster.

|

   Example to run on *PrimarySite* (destination addresses are for *RecoverySite*)

   .. code-block:: bash

      allssh 'modify_firewall -f -o open -r 10.38.212.29/25,10.38.212.30/25,10.38.212.31/25,10.38.212.32/25,10.38.212.37/25 -p 2030,2036,2073,2090 -i eth0'

|

   Example to run on *RecoverySite* (destination addresses are for *PrimarySite*)

   .. code-block:: bash

      allssh 'modify_firewall -f -o open -r 10.42.54.29/25,10.42.54.30/25,10.42.54.31/25,10.42.54.32/25,10.42.54.37/25 -p 2030,2036,2073,2090 -i eth0'
|
Lab Requirements
++++++++++++++++

#. Calm is enabled on the *PrimarySite* cluster.

#. SSH client installed (ex. Putty for Windows)

#. Leap is enabled on both clusters.

#. If you are using the HPOC environment, reserve two clusters in the same datacenter to ensure synchronous replication latency recommendations are met.

#. Designate one cluster as *PrimarySite*, and one as *RecoverySite*. We recommend you rename each cluster within Prism to aid with identification during this lab.

#. Configure a Primary (and optionally a Secondary) network within Prism, including IP Address Management (IPAM) on one or both networks, including DNS, and an IP Pool. This lab requires 2 IP addresses per attendee, per physical cluster (2 at the Primary site, 2 at the Recovery site).

.. note::

   When utilizing the HPOC, both Primary and Secondary network information will be provided with your reservation.

.. note::
   .. class:: center

   Recommended IPAM pools when using HPOC

   - Primary
      - Range = .50 - .125
      - IPAM DHCP = .126
      - (76 available IPs)

   - Secondary
      - Range = .132 - .253
      - IPAM DHCP = .254
      - (122 available IPs)
|
.. note::
.. class:: center

   Recommended IPAM pools when using HPOC

   - Primary
      - Range = .50 - .125
      - IPAM DHCP = .126
      - (76 available IPs)

   - Secondary
      - Range = .132 - .253
      - IPAM DHCP = .254
      - (122 available IPs)
|
Leap Limitations
++++++++++++++++

(As of June 2nd, 2020)

#. Planned for 5.17.1
   - Test Failover Planned Failover
|
#. Not supported
|
   - Volume Groups (Planned 5.21)
   - Files (Planned 5.19+)
   - Single PC (Planned 5.19)
   - Automatic resumption from outage when Automatic Failure Handling is set
   - 1-node and 2-node configurations
   - Multi-site Sync Rep Multi-site Sync + NearSync/Async
|
AHV Sync-Rep FAQ [**INTERNAL LINK**]
++++++++++++++++++++++++++++++++++++

|
https://docs.google.com/document/d/1jqJ8bENTS8LW0oAOezXlucgB9Zfylj6n5HupeJUOE5s/edit?ts=5e7b7d65

|
Xi Leap Admin Guide
+++++++++++++++++++

|
https://portal.nutanix.com/page/documents/details/?targetId=Xi-Leap-Admin-Guide%3AXi-Leap-Admin-Guide

|
Synchronous Replication Limitations
+++++++++++++++++++++++++++++++++++

|
.. note::

   These limitations are in addition to the general limitations of Leap.

(As of June 2nd, 2020)

   - Volume Groups (VGs) are not supported.

   - Witness VMs are not supported.

   - VMs that are configured as part of a network function chain are not supported.

   - VMs with affinity policies are not supported.

   - VMs with incompatible GPUs on the recovery cluster are not supported.

   - Only unplanned failover is supported.

Synchronous Replication Recommendation
++++++++++++++++++++++++++++++++++++++

|
   - For optimal performance, Nutanix recommends that the round trip latency (RTT) between clusters be less than 5 ms. Maintain adequate bandwidth to accommodate peak writes and have a redundant physical network between the clusters.

|
Lab Roadmap
+++++++++++

   - Implement staging to automate aspects of the setup process: Network creation, deployment of PC, enable Calm/Leap, deploy Calm blueprint for specified number of users, etc.

   - Add alternative instructions to deploying a multi-VM application via Calm (ex. customers/prospects interested in Leap, but do not own Calm)

   - Add Windows-based activity

|
Calm configuration
++++++++++++++++++

#. In **Prism Central**, select :fa:`bars` **> Services > Calm**.

#. Select **Projects** from the lefthand menu and click **+ Create Project**.

   .. figure:: images/Calm/23.png

|
#. Fill out the following fields:

   - **Project Name** - *Initials*\ -FiestaProject

   - Under **Infrastructure**, select **Select Provider > Nutanix**

   - Click **Select Clusters & Subnets**

   - Select *Your PrimarySite cluster*

   - Under **Subnets**, select **Primary**, (optionally) **Secondary**. Click **Confirm**

   - Mark *Your Preferred Network (Primary or Secondary)* as the default network by clicking the :fa:`star`

#. Click **Save & Configure Environment**.

This will redirect you to the Envrionments page, but there is nothing needed to configure here. You may now move on to the next step.

|
Staging Blueprints
..................

A Blueprint is the framework for every application that you model by using Nutanix Calm. Blueprints are templates that describe all the steps that are required to provision, configure, and execute tasks on the services and applications that are created. A Blueprint also defines the lifecycle of an application and its underlying infrastructure, starting from the creation of the application to the actions that are carried out on a application (updating software, scaling out, etc.) until the termination of the application.

You can use Blueprints to model applications of various complexities; from simply provisioning a single virtual machine to provisioning and managing a multi-node, multi-tier application.

#. `Download the Fiesta-Multi Blueprint by right-clicking here <https://github.com/vPeteWalker/leap_addon_bootcamp/raw/master/Fiesta-Multi-GITHUB.json>`_.

#. Log in to Prism Central for your **PrimarySite** cluster.

#. Open :fa:`bars` **Prism Central > Calm**, select **Blueprints** from the lefthand menu and click **Upload Blueprint**.

   .. figure:: images/Calm/1.png

|
#. Select **Fiesta-Multi-GITHUB.json**.

#. Update the **Blueprint Name** to include your initials. Even across different projects, Calm Blueprint names must be unique.

#. Select your *Initials*\ -FiestaProject project and click **Upload**.

   .. figure:: images/Calm/2.png

|
#. In order to launch the Blueprint you must first assign a network to the VM. Select the **NodeReact** Service, and in the **VM** Configuration menu on the right, select *Your Preferred Network* as the **NIC 1** network.

   .. figure:: images/Calm/3.png

|
#. Repeat the **NIC 1** assignment for the **MySQL** Service.

#. Expand the *db_password* section, and within the *Value* entry, type *nutanix/4u* as the password.

   .. figure:: images/Calm/4.png

|
#. Click **Credentials** to define a private key used to authenticate to the CentOS VM that will be provisioned by the Blueprint.

   .. figure:: images/Calm/5.png

|
#. Expand the **CENTOS** credential and paste in the following value as the **SSH Private Key**:

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

Deploy a multi-VM application via Calm
......................................

#. Open :fa:`bars` **> Services > Calm** and select **Blueprints** from the sidebar.

#. Select the **FiestaApp** Blueprint and click **Actions > Launch**.

   .. figure:: images/Calm/6.png

|
#. Fill out the following fields and then click **Create** to begin provisioning your application:

   - **Name of the Application** - *Initials*\ -FiestaApp
   - **user_initials** - *Initials*

.. figure:: images/Calm/7.png

|
#. Monitor the status of the application in the **Audit** tab and proceed once your application enters a **Running** state.

#. On the **Services** tab, select the **NodeReact** service and note the IP Address. This is the web server hosting the front end of your application.

#. Open http://*NodeReact-VM-IP-Address*:5001 in a new browser tab and validate you can access the Fiesta Inventory Management app.

   .. figure:: images/Calm/8.png

|
Installing Nutanix Guest Tools
++++++++++++++++++++++++++++++

#. Open :fa:`bars` **> Virtual Infrastructure > VMs**.

#. Select your *Initials*\ **-WebServer-...** VM and click **Actions > Update**.

#. Under **Disks**, click :fa:`eject` beside **CD-ROM** to unmount the Cloud-Init disk mounted during the Calm deployment.

#. Click **Save**.

#. Repeat **Steps 2-4** to eject the **CD-ROM** on your *Initials*\ **-MySQL-...** VM.

#. Select both VMs and click **Actions > Install NGT**.

   .. figure:: images/NGT/1.png

|
#. Select **Restart as soon as the install is completed** and click **Confirm & Enter Password**.

   .. figure:: images/NGT/2.png

|
#. Provide the following credentials and click **Done** to begin the NGT installation:

   - **User Name** - centos
   - **Password** - nutanix/4u

   .. figure:: images/NGT/3.png

|
#. Once both VMs have rebooted, validate both VMs now have empty CD-ROM drives and **NGT Status** displays **Latest** in Prism Central.

   .. figure:: images/NGT/4.png

|
Creating a new Availability Zone
++++++++++++++++++++++++++++++++

.. note::

   **Instructor will demonstrate the following steps**

|
#. Log in to Prism Central for your **PrimarySite** cluster.

#. Open :fa:`bars` **> Administration > Availability Zones** and observe that a Local AZ has already been created by default. Click **Connect to Availability Zone**

.. figure:: images/AZ/1.png

|
#. In the *Availability Zone Type* dropdown, select **Physical Location**. Enter the IP, username, and password for the **RecoverySite** PC, and click **Connect**.

.. figure:: images/AZ/2.png

|
.. figure:: images/AZ/3.png

|
Observe that the **RecoverySite** cluster is now listed as *Physical*, and its *Connectivity Status* is listed as *Reachable*

.. figure:: images/AZ/4.png

|
