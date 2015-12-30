# Border Gateway

## 1. Introduction

![Overview](https://github.com/Hybrid-Cloud/hybrid_cloud/blob/master/doc/BGW/images/L2GW_overview.png)

### L2 Border-Gateway – What is it?
Connect overlay networks with other external overlay networks in Layer 2.

* L2 connection between multiple OpenStack clouds
* Connect multiple overlay tenant networks
* Use L2GW as baseline
    - Same API, DB, OVSDB Hardware-VTEP Schema with minor modifications
* Can use any tunneling protocols (VxLAN, NVGRE, Geneve, ETC.)  in the LAN and WAN
* Layer 3 will be handled by DVR / DragonFlow / OVN, and alike.

### Motivation:
* Tricircle is an OpenStack project that aims to deal with OpenStack deployment in multiple sites. See https://docs.google.com/document/d/19BXf0RhkH8wEEymE2eHHqoDZ67gnzgvpr3atk4qwdGs/edit#heading=h.5r6zgqbiehsh
Main characteristics are
* Deferent neutron network
* Single Keystone
* TOP “Special Purpose” OpenStack to manage and orchestrate all the sites
* Bottom, “Ordinary” OpenStack on each site
* L2 Border Gateway is needed to add Layer 2 connectivity between overlay networks in deferent sites
* Network orchestration will be done by the “TOP” OpenStack

### Current implementation
![currentImplementation](https://github.com/Hybrid-Cloud/hybrid_cloud/blob/master/doc/BGW/images/L2GW_currentImplementation.png)
* Connect overlay networks with physical network
* Use VxLAN towards overlay network and VLAN towards physical network
* Bare-Metal server info (MAC, IP, port location) is in Hardware-VTEP db

### Needed Functionality
* Use any network segmentation (VLAN, VxLAN, GRE, ETC.)
* VxLAN only in first phase, but with support for deferent VNI on every tunnel.


# 2. Deployment

![Deployment](https://github.com/Hybrid-Cloud/hybrid_cloud/blob/master/doc/WANV/images/WANV_deployment.png)

## Single-ended
* Transparent
    - NAT is done by gateway device (e.g. DSL modem/router)
* Stickiness
    - Flows remain on selected link and are not             re-routed
* User Experience
    - Best fitting link selected for each flow, based on round robin on available links
    - Queuing discipline guarantees fairness and prevents bandwidth hogging
    - Link based internet service delay measurement method 
* Path Balancing
    - Optimize flow distribution over all links
    - Passive learning algorithm for long-term link utilization optimization

## Multi-ended
* Secured Cross-Branch Tunneling
    - Paths between branches use tunneling protocols (e.g. GRE, GRE over IPSEC)
* Efficient
    - WANV IP addresses are public, NAT is not done between branches
* High Fidelity (site-to-site)
    - Original packets not modified
    - Internal IP addresses are retained
* User Experience
    - Flows are transparently re-routed to better paths in real-time
    - Path performance is monitored in real-time (state, delay, delay variation, packet loss)
    - Path protection with no noticeable interruption to ongoing user sessions
    - Queuing discipline guarantees fairness and prevents bandwidth hogging
    - Online hybrid model to predict network performance in real time
* Path Balancing
    - Optimize flow distribution over all paths

