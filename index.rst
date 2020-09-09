.. title:: Leap Add-On Bootcamp

.. toctree::
   :maxdepth: 2
   :name: _onpremleap
   :caption: Leap Labs
   :hidden:

   onpremleap3_PFO/onpremleap3_PFO
   onpremleap2_UPFO/onpremleap2_UPFO

-------------------------
Getting Started with Leap
-------------------------

Overview
++++++++

New in 5.17, Nutanix AOS offers significant enhancements to Leap for on-premises failover operations, including support for execution of guest scripts, and synchronous replication with AHV.

In this lab you will:

- Use a multi-tier application - Fiesta - deployed by Calm
- Protect your VMs with a Protection Policy
- Build a Recovery Plan for runbook automation
- Create changes to the Fiesta application database
- Perform a failover to a physical Nutanix cluster
- Access the Fiesta application to ensure changes were retained
- Make additional changes to the Fiesta application database
- Perform a failback from a physical Nutanix cluster
- Access the Fiesta application to ensure changes were retained

This add-on Bootcamp provides exercises for BOTH **Unplanned** and **Planned** failover scenarios. If you plan to complete one exercise, you may skip the setup steps for the second exercise, as you will use the same VMs, Protection Policy, and Recovery Plan for both. This is detailed in the introduction to each lab.

Environment Requirements
++++++++++++++++++++++++

- Two AHV clusters running AOS 5.17.1/AHV 20190916.189 or newer. Reserve two clusters in the same HPOC datacenter to ensure synchronous replication latency requirements are met (<5ms RTT).

   .. note::

      Single node clusters are supported for the **Leap Add-On Bootcamp**.

- You can run any standard Bootcamp staging on one cluster (e.g. **Enterprise Private Cloud Bootcamp**, **Databases: Era with MSSQL Bootcamp**, etc.) This will be your **RecoverySite** cluster.

- You **must** run the **Leap Add-On Bootcamp** staging on your additional cluster. This will be your **PrimarySite** cluster.

- Follow the steps below to open the required ports: 2030, 2036, 2073, and 2090.

   .. note::

      This step is necessary even if using two HPOC clusters in the same datacenter.

   #. To open the ports for communication to the recovery cluster, run the following command on all CVMs of the **PrimarySite** cluster by remoting in via SSH (e.g. ssh nutanix@<CLUSTER-VIRTUAL-IP>):

      .. code-block:: bash

          allssh 'modify_firewall -f -r remote_cvm_ip,remote_virtual_ip -p 2030,2036,2073,2090 -i eth0'

      Replace remote_cvm_ip with the IP address of the recovery cluster CVM. If there are multiple CVMs, replace remote_cvm_ip with the IP addresses of the CVMs separated by comma.

      Replace remote_virtual_ip with the virtual IP address of the recovery cluster.

   #. To open the ports for communication to the primary cluster, run the following command on all CVMs of the **RecoverySite** cluster by remoting in via SSH (e.g. ssh nutanix@<CLUSTER-VIRTUAL-IP>):

      .. code-block:: bash

         allssh 'modify_firewall -f -r source_cvm_ip,source_virtual_ip -p 2030,2036,2073,2090 -i eth0'

      Replace source_cvm_ip with the IP address of the primary cluster CVM. If there are multiple CVMs, replace source_cvm_ip with the IP addresses of the CVMs separated by comma.

      Replace source_virtual_ip with the virtual IP address of the primary cluster.

   #. Exit both SSH sessions.

   .. note::

      For more information about the required ports, see the **General Requirements of Leap** section within the `Xi Leap Administration Guide <https://portal.nutanix.com/page/documents/details/?targetId=Xi-Leap-Admin-Guide%3AXi-Leap-Admin-Guide>`_.

- This staging script will automatically deploy **10** instances of the Fiesta application:

   - **USER01-MYSQL-...**; **USER01-WebServer-...**
   - **USER02-MYSQL-...**; **USER02-WebServer-...**
   - And so on...

- The instructor should assign a **USER** number to each participant. The lab guide will reference entity names with **USERXX** which should be substituted for their specific number (e.g. **USER01**).

- It is recommended to rename the clusters within Prism to **Primary** and **Recovery** respectively. This will aid in identification during the labs.

Reference
+++++++++

The following resources are provided for reference purposes and aren't required to complete the lab exercises.

- `Xi Leap Administration Guide <https://portal.nutanix.com/page/documents/details/?targetId=Xi-Leap-Admin-Guide%3AXi-Leap-Admin-Guide>`_
