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
![NeededFunctionality](https://github.com/Hybrid-Cloud/hybrid_cloud/blob/master/doc/BGW/images/L2GW_neededFunctionality.png)
* Use any network segmentation (VLAN, VxLAN, GRE, ETC.)
* VxLAN only in first phase, but with support for deferent VNI on every tunnel.

### L2GW building blocks
![buildingBlocks](https://github.com/Hybrid-Cloud/hybrid_cloud/blob/master/doc/BGW/images/L2GW_buildingBlocks.png)

### HW VTEP Schema
![hwvtepschema](https://github.com/Hybrid-Cloud/hybrid_cloud/blob/master/doc/BGW/images/L2GW_hwVtepSchema.png)

1. Changes needed 

1.1. OVSDB Schema
- Physical_Locator TABLE 
- tunnel_key FIELD
- Need to add this field to support per logical_port + physical_locator  (see Local_Switch table in the spec: http://openvswitch.org/docs/vtep.5.pdf )

1.2 Neutron DB
- L2GW DB Model
    - Add McastMacRemote
        - Use this record to instruct the switch to flood packets to unknown MAC to list of tunnels (Physical_Locator list)
- L2gatewayconnections DB Table
    - Add tunnel_id filed that will hold the Physical_Locator ID
- physical_locators DB Table
    - Add tunnel_key field

1.3. API
- Command: l2-gateway-create
    - Allow device name without interfaces

- Add Command: l2-border-gateway-connection-CRUD (similar to l2-gateway-connection-create)
    - Create Physical_Locator to the remote GW with tunnel_key field that was supplied in the command in segmentation_id field
    - Add Physical_Locator to Mcast_Macs_Remote table with keyword ‘unknown_dst’ in MAC field (used for flooding)

- Add command: mac-CRUD
    - Fields:  gw_name, tunnel_id, mac, ip (optional)
    - Insert and remove MAC addresses to the Ucast_Macs_Remote table

