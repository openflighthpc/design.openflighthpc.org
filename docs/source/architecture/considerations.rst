.. _architecture-considerations:

Architecture Considerations: General
====================================

The scope of cluster architecture is broad, designing a cluster involves:

- Identifying the cluster platform
- Considering network configuration
- Deciding on operating system configuration
- Choosing node structure for workflow
- Identifying levels of resistance and redundancy

Platforms
---------

With the rise of cloud computing and the constant evolution of metal and container solutions there are many platforms on which a cluster architecture can be built. To name a few:

- Cloud

  - AWS
  - Azure
  - Google Cloud

- Metal

  - Kickstart (or other automated OS build systems)

- Virtual

  - Libvirt
  - VirtualBox

Cluster architectures no longer have to be restricted to just one platform with the rise of *Hybrid Clusters*, creating overflow nodes in the cloud for on-site clusters to ensure that workloads can be completed within expected timescales.

Cloud
^^^^^

Using a cloud platform for a cluster has many advantages. It allows for flexible resource management, such as, scaling resources to match workload such that resources are less likely to be left sat around idle than in metal clusters. Cloud is also an incredibly quick platform to deploy architectures on as there is no need to order, install and test on-site hardware (like with traditional metal solutions). Furthermore, cloud platforms offer a higher level of redundancy and stability than traditional metal solutions for a fraction of the cost.

Cloud can be a worryingly external service as opposed to on-site solutions depending on the workload and data privacy. Additionally, permanent, non-scaled cloud solutions are likely to be more expensive than dedicated on-site hardware.

Metal
^^^^^

This is the most traditional platform for cluster architecture. Metal solutions provide complete control over the hardware choices and physical configuration of the servers. Additionally, a well-configured metal cluster will have security advantages over cloud with most, or all, of the cluster existing within the network of a site. 

Maintaining the hardware and physical configuration of a metal cluster can be time-consuming, create server downtime and can also be costly when replacing or repairing the infrastructure. 

Virtual
^^^^^^^

Depending on the workflow, virtual solutions can provide a low-cost, lightweight, transferrable platform for cluster computing. Virtual machines or containers do require a system to run on which would most likely be a metal machine. The use of virtualisation allows for services and nodes to be separated on shared resources, optimising the usage of the hardware it's running on. 

Hybrid
^^^^^^

Using a hybrid platform solution can provide solutions to some of the constraints of the various platforms. For example, a metal compute cluster can be configured to overflow to cloud resources when the load of the cluster is peaking. Additionally, the use of virtualisation for system services can reduce the quantity of multiple hardware or cloud instances (and therefore, the costs).

Networks
--------

The network in the cluster may be broken up (physically or virtually with VLANs) into separate networks to serve different usages and isolate traffic. Potential networks that may be in the cluster architecture are:

  - **Primary Network** - The main network that all systems are connected to.
  - **Out-of-Band Network** - A separate network for management traffic. This could contain on-board BMCs, switch management ports and disk array management ports. Typically this network would only be accessible by system administrators from within the HPC network.
  - **High Performance Network** - Usually built on an Infiniband fabric, the high performance network would be used by the compute nodes for running large parallel jobs over MPI. This network can also be used for storage servers to provide performance improvements to data access.
  - **External Networks** - The network outside of the HPC environment that nodes may need to access. For example, the *Master Node* could be connected to an *Active Directory* server on the external network and behave as a slave to relay user information to the rest of the HPC environment.
  - **Build Network** - This network can host a DHCP server for deploying operating systems via PXE boot kickstart installations. It allows for systems that require a new build or rebuild to be flipped over and provisioned without disturbing the rest of the network.
  - **DMZ** - A demilitarised zone would contain any externally-facing services, this could be setup in conjunction with the external networks access depending on the services and traffic passing through.

The above networks could be physically or virtually separated from one another. In a physical separation scenario there will be a separate network switch for each one, preventing any sort of cross-communication. In a virtually separated network there will be multiple bridged switches that separate traffic by dedicating ports (or tagging traffic) to different VLANs. The benefit of the VLAN solution is that the bridged switches (along with bonded network interfaces) provides additional network redundancy.

.. note:: If a cloud environment is being used then it is most likely that all systems will reside on the primary network and no others. This is due to the network configuration from the cloud providers.

Operating System
----------------

There are many different operating systems which can provide the base software stability and functionality for a given workflow. With OS selection it is worth considering:

- Package versions and support for the intended software usage
- Documentation and support availability
- Licensing costs

This community project uses CentOS 7 as it is stable and free with a large amount of documentation and community support available.

Node Structure
--------------

A cluster will most likely be comprised of systems that serve different purposes within the network. Ideas of node types along with the services and purpose of those nodes can be seen below.

  - **Login Node** - A login node will usually provide access to the cluster and will be the central system that users access to run applications. How users will access the system should be considered, usually this will be SSH and some graphical login service, such as, VNC.
  - **Master Node** - A master node will usually run services for the cluster. Such as, the master process for a job scheduler, monitoring software and user management services.
  - **Compute Node** - Compute nodes are usually used for running HPC applications that are queued through a job scheduler. Additionally, these can be used for VM deployments (via software like OpenStack) or other computational uses. Compute nodes usually have large amounts of cores and memory as well as high bandwidth interconnect (like Infiniband).
  - **Special-purpose Node** - Some compute nodes may feature a particular specification to be used for a particular job, or stage in your workflow. Examples may include nodes with more memory, larger amounts of local scratch storage, or GPU/FPGA devices installed.
  - **Storage Node** - The storage node will serve network storage solutions to systems on the network. It would have some sort of storage array connected to it which would provide large and resilient storage.

The above types are not strict. Services can be mixed, matched and moved around to create the desired balance and distribution of services and functions for the architecture and workflow.

Resilience
----------

How well a system can cope with failures is crucial when designing cluster architecture. Adequate resilience can allow for maximum system availability with a minimal chance of failures disrupting the user. System resilience can be improved with many hardware and software solutions, such as:

  - **RAID Arrays** - A RAID array is a collection of disks configured in such a way that they become a single storage device. There are different RAID levels which improve data redundancy or storage performance (and maybe even both). Depending on the RAID level used, a disk in the array can fail without disrupting the access to data and can be hot swapped to rebuild the array back to full functionality. [#f1]_
  - **Service Redundancy** - Many software services have the option to configure a slave/failover server that can take over the service management should the master process be unreachable. Having a secondary server that mirrors critical network services would provide suitable resilience to master node failure.
  - **Failover Hardware** - For many types of hardware there is the possibility of setting up failover devices. For example, in the event of a power failure (either on the circuit or in a power supply itself) a redundant power supply will continue to provide power to the server without any downtime occurring.

There are many more options than the examples above for improving the resilience of the cluster, it is worth exploring and considering available solutions during design.

.. note:: Cloud providers are most likely to implement all of the above resilience procedures and more to ensure that their service is available at least 99.99% of the time.

Hostname and Domain Names
-------------------------

Using proper domain naming conventions during design of the cluster architecture is best practice for ensuring a clear, logical and manageable network. Take the below fully qualified domain name::

  node01.pri.cluster1.compute.estate

Which can be broken down as follows:

  - ``node01`` - The hostname of the system
  - ``pri`` - The network that the interface of the system is sat on (in this case, pri = primary)
  - ``cluster1`` - The cluster that ``node01`` is a part of
  - ``compute`` - The subdomain of the greater network that ``cluster1`` is a part of
  - ``estate`` - The top level domain

Security
--------

Network security is key for both the internal and external connections of the cluster. Without proper security control the system configuration and data is at risk to attack or destruction from user error. Some tips for improving network security are below:

  - Restrict external access points where possible. This will reduce the quantity of points of entry, minimising the attack surface from external sources.
  - Limit areas that users have access to. In general, there are certain systems that users would never (and should never) have access to so preventing them from reaching these places will circumvent any potential user error risks.
  - Implement firewalls to limit the types of traffic allowed in/out of systems.

It is also worth considering the performance and usability impacts of security measures.

Much like with resilience, a Cloud provider will most likely implement the above security features - it is worth knowing what security features and limitations are in place when selecting a cloud environment.

.. note:: Non-Ethernet networks usually cannot usually be secured to the same level as Ethernet so be aware of what the security drawbacks are for the chosen network technology.


.. [#f1] For more information on RAID arrays see https://en.wikipedia.org/wiki/RAID
