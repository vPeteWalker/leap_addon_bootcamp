.. title:: Leap Add-On Bootcamp

.. toctree::
   :maxdepth: 2
   :name: _onpremleap
   :caption: Leap Labs
   :hidden:

   onpremleap3_PFO/onpremleap3_PFO
   onpremleap2_UPFO/onpremleap2_UPFO

Overview
++++++++

Nutanix AOS 5.17 offer significant enhancements to Leap for on-premises failover operations, including support for execution of guest scripts and synchronous replication with AHV.

In this lab you will:

- Use a multi-tier application, Fiesta, deployed by Calm
- Protect your VMs with a Protection Policy
- Build a Recovery Plan for runbook automation
- Create changes to the Fiesta application database
- Perform a failover to a physical Nutanix cluster
- Access the Fiesta application to ensure changes were retained
- Make additional changes to the Fiesta application database
- Perform a failback from a physical Nutanix cluster
- Access the Fiesta application to ensure changes were retained

This add-on Bootcamp provides exercises for BOTH **Unplanned** and **Planned** failover scenarios. If you plan to complete each exercise, you may skip the setup steps for the second exercise, as you will use the same VMs and Protection Plan for each. This is detailed in the introduction to each lab.

Environment Requirements
++++++++++++++++++++++++

- Two AHV clusters running AOS 5.17.1/AHV 20190916.189 or newer. Reserve two clusters in the same HPOC datacenter to ensure synchronous replication latency requirements are met (<5ms RTT).

   .. note::

      Single node clusters are supported for the **Leap Add-On Bootcamp**.

- You can run any standard Bootcamp staging on one cluster (e.g. **Enterprise Private Cloud Bootcamp**, **Databases: Era with MSSQL Bootcamp**, etc.)

- You **must** run the **Leap Add-On Bootcamp** staging on your additional cluster.

- Both clusters communicate over the following ports: 2030, 2036, 2073, and 2090. Follow the steps below to open the required ports.

   .. note::

      This step is necessary even if using two HPOC clusters in the same datacenter.

   #. Run the following command on any CVM of the **PrimarySite** cluster by remoting in via SSH (e.g. ``ssh nutanix@<CLUSTER-VIRTUAL-IP>``):

   .. code-block:: bash

      allssh 'modify_firewall -f -p 2030,2036,2073,2090 -i eth0'

   #. Run the following command on any CVM of the **RecoverySite** cluster by remoting in via SSH (e.g. ``ssh nutanix@<CLUSTER-VIRTUAL-IP>``):

      .. code-block:: bash

         allssh 'modify_firewall -f -p 2030,2036,2073,2090 -i eth0'


   #. Exit both SSH sessions.

   .. note::

      For more information about the required ports, see `General Requirements of Leap <https://portal.nutanix.com/page/documents/details/?targetId=Xi-Leap-Admin-Guide%3AXi-Leap-Admin-Guide>`_.

- This staging script will automatically deploy **10** instances of the Fiesta application:

   - **USER01-MYSQL-...**; **USER01-WebServer-...**
   - **USER02-MYSQL-...**; **USER02-WebServer-...**
   - And so on...

- The instructor should assign a **USER** number to each participant. The lab guide will reference entity names with **USERxx** which should be substituted for their specific number (e.g. **USER01**).

Reference
+++++++++

The following resources are provided for reference purposes and aren't required to complete the lab exercises.

- `Xi Leap Administration Guide <https://portal.nutanix.com/page/documents/details/?targetId=Xi-Leap-Admin-Guide%3AXi-Leap-Admin-Guide>`_
