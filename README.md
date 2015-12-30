# Bambu - Hyper Network
## Overview
The Hyper Network provides a stretched L2 connectivity across the hyper virtual private cloud (Hyper VPC).   
In other words, it enables the connectivity between tenant VMs that run on top of different bottom cloud providers.   
For example, it enables a cross-cloud application deployment:   

![Cross-Cloud Application Deployment](https://raw.githubusercontent.com/Hybrid-Cloud/hybrid_cloud/master/doc/source/hypernetwork/images/cross_cloud_application_deployment.png)   

The application components run as different VMs (or containers), connected via the same virtual network and the same subnet.  
Each of the application components (i.e. VMs) uses a Hyper IP address, which reside in the same Hyper Subnet.   
For example, the hyper subnet we will use is ``10.0.1.0/24``.  
The application VMs are deployed on different clouds, which assign the VMs that run in them with IP addresses on local subnets (for example, Public Cloud 1 will assign from ``172.16.15.0/24`` and Public Cloud 2 will assign from ``192.168.10.0/24``).  
Normally, in order to connect two VMs that reside on different clouds, we would need to use Public IP addresses (or a VPN, which also uses Public IP addresses).  If we do that, we would be using Layer 3 network to connect the VMs, which will force us to define a different Subnet on each side of the VPN (e.g. ``10.0.1.0/24`` and ``10.0.2.0/24``).  
However, we want to provide the tenant with the ability to set fixed IP addresses to their VMs, which will persist regardless of the cloud to which they are deployed.  For example, use just the ``10.0.1.0/24`` subnet.

## Main Challenges

* Tenant VMs are maintained by the tenant and cannot run our code
* Bottom Cloud infrastructure features differ between vendors
* Making the tenant VM use the Hyper IP address in spite of the bottom Cloud provider
* Ensure tenant isolation
* Support the ability to migrate workload across clouds (retain IP address, MAC, security groups, etc.)
* Incur up-to 20% network performance degradation, compared with the native cloud

## Solution Overview

We currently see several different solutions, which tackle different use cases.  
Each solution has some advantages and some disadvantages.

### Agent-based Solutions 

Agent-based solutions run on the tenant's VM, employing mechanisms to alter the network traffic that comes from the VM at the source, in order to apply the network features of the Hyper Cloud (security groups, L2, L3 and metadata service).   
The main advantages of agent-based solutions is that they split the workload of traffic manipulation on the compute resources of the tenant (incurring some overhead), and that they don't add additional network hops, which incur latency.   
The main disadvantages of these approaches is that the tenant becomes aware (at varying degrees) to the fact that they run in a hyper cloud environment (e.g. need to install software into their VM, or package their software in a special manner).

#### Intrusive

In this Agent-based solution, the tenant needs to install a software agent into its VMs.    
This agent creates a TAP device(s) with the HyperIP address(es), and implements the additional services directly in the user VM.  
The agent uses OVS to implement the packet manipulation (encapsulation, routing, etc.).  

#### HyperVM (container-based)

In this solution, the tenant workload is packaged as a *Container Image* and executed inside a *Container*.  
The Agent runs outside the *Container* (i.e. directly in the hosting VM) and provides the network services directly on the datapath of the *Container*.

#### Compare *Intrusive* and *HyperVM* Agent-based

The following diagram shows the basic components and concept of the two Agent-based solution options:

![Agent-based Solutions Comparison](https://raw.githubusercontent.com/Hybrid-Cloud/hybrid_cloud/master/doc/source/hypernetwork/images/agent-based_solutions_comparison.png)

As you can see, the main difference is that in Option 2 (HyperVM), the *User Workload* is isolated using the *Container Visor* (e.g. LXC, Docker) from the Agent.  
By isolating the User Workload and the Agent we mitigate some very large disadvantages:
* Security - The User cannot interfere with the working of the Agent
* User Experience - The User works in a clean namespace (90% equivalent of a VM)
* Maintainability - The User is not forced to run alien software components that need their own updates and life-cycle management
* Portability - The User manages a single Container Image, regardless of the Agent, which can run on any Bottom Cloud Provider that we support, without any conversion 

The main disadvantage of the *HyperVM* solution is that it can only work for user workloads that can be packaged in a Container Image and support a Container-based runtime environment (e.g. Microsoft Windows-based workloads may not work, until Microsoft releases its Container technology, Complex software that requires certification for support, like Oracle Database, will not be able to run this way).


### Agent-less Solutions

Agent-less solutions run on a separate VM from the *Tenant*, requiring that the datapath is routed through them, in order for them to provide the hyper network features (e.g. security groups, L2, L3, metadata service).  
In order to do that, there are different mechanisms in the various Bottom Cloud providers.  
For example, Amazon AWS lets us manipulate the Routing Table, which enables us to introduce our HyperNode into the datapath of the tenant's VMs.  
The main advantages of the Agent-less solutions are that they require no software installation to the user's VM, the user can work with an actual VM and deploy any kind of runtime environment they want, that the Cloud Provider supports (usually, all types).  Also, there is no compute resource overhead that is incurred at the Tenant's VM (like in the Agent-based solution), as everything is done at the HyperNode (which is a different VM).  
The main disadvantages of the Agent-less solutions are that they incur additional latency, due to the routing of the datapath traffic through additional hop(s), and that the tenant user can see the Cloud-provided IP address in the VM, instead of (or in addition to) the Hyper IP Address that they actually want.

#### HyperNode with Custom Route

This solution works only in Amazon AWS, as it is based on the ability to manipulate the Custom Route Table of the subnet.  
The Tenant's VMs are deployed regularly in an AWS VPC subnet, and receive IP addresses from the IPAM of the VPC.  
The HyperVM introduces itself into the Routing Table, as the default gateway for the HyperSubnet (a range of IP addresses that is not recognized by the AWS VPC).  
In addition, the HyperVM disables the AWS security check that prohibits IP spoofing, so that it can fulfill its routing functionality.  
By applying AWS security groups on the subnet that block all VMs from communicating with anything else but the HyperVMs, we ensure that the Tenant user is not bypassing the HyperVM for communicating between collocated VMs.  

#### HyperNode with VPN Gateways

*This solution is still under initial design.  *
The main principle is that the Tenant user opens a VPN gateway from his VM to the HyperNode, and receives the Hyper IP address on the TAP device that is created from this VPN connection.  
This enables us to use the Agent-less HyperNode solution on all Bottom Cloud providers, as it is completely agnostic to the underlying cloud's capabilities.

#### Compare HyperNode *Custom Route* and *VPN Gateway* Agentless

![Agentless Solutions Comparison](https://raw.githubusercontent.com/Hybrid-Cloud/hybrid_cloud/master/doc/source/hypernetwork/images/agent-less_solutions_comparison.png)



