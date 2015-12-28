# Bambu API

## Border Gateway (BGW)
### 1. Introduction
L2 Border Gateway is an enhancement to the current Networking/L2GW project.
The current project aims to connect overlay network with external, usually bare metal, servers via hardware switches using OVSDB protocol. 
The enhancement will provide ability to connect layer2 of two or more overlay networks that are located on different base networks (cloud network) to create inter-cloud connection.
The L2GW has few commands to support the above functionality. In order to add the needed functionality, few more commands need to be added to the current API.
Following is a description of the new API that includes the commands for inter-cloud connection.

### 2. API Commands

##### 2.1 Gateway Commands
The current API that runs CRUD commands for gateway. The command l2-gateway-create creates a new gateway. The command has –device parameter that get the name of the device and a list of interfaces that should be connected to the overlay network. The list is mandatory. In case of Border Gateway, the interfaces are optional, as we can use the gateway for inter-cloud connection only without connecting bare metal servers.

##### 2.2. Remote Gateway Commands
Optimal link utilization, path protection & aggregation, flow-based load balancing, flow-based QoS/COS, HA
Extend local LAN to remote sites with secured tunneling (GRE or VxLAN over IPSec)

###### 2.2.1.	l2-remote-gateway-create
This command creates a remote gateway with its IP address with which the local gateway can communicate to build inter-cloud tunnels.
```
Usage:
neutron l2-remote-gateway-create <gateway-name> <ip-addr>
```
###### 2.2.2. l2-remote-gateway-update
This command updates the remote gateway name or/and its IP address.
```
Usage:
neutron l2-remote-gateway-update <remote-gw-name/uuid> 
[--name=remote-gw-name] 
[--ip-addr]
```
###### 2.2.3. l2-remote-gateway-delete
This command deletes remote gateway.
```
Usage:
neutron l2-remote-gateway-delete <remote-gw-name/uuid>
````
###### 2.2.3. l2-remote-gateway-list
This command lists all the remote gateways.
```
Usage:
neutron l2-remote-gateway-list 
```
##### 2.3 Gateway Commands
To be able to connect two overlay networks, the following command will be added to the API:

###### 2.3.1.	l2-remote-gateway-connection-create
This command creates a new connection to a remote gateway
```
Usage:
neutron l2-remote-gateway-connection-create <gateway-name/uuid>  
–remote=<remote-gw-name/uuid>
--segmentation-id=<sed-id> 
````
###### 2.3.2. l2-remote-gateway-connection-update
This command can be used to update the remote gateway or/and the segmentation ID for the connection.
```
Usage:
neutron l2-remote-gateway-connection-update <uuid>  
[–remote=<remote-gw-name/uuid>]
[--segmentation-id=<sed-id>]
````
###### 2.3.3.	l2-remote-gateway-connection-delete
This command can be used to delete a connection to a remote gateway.
```
Usage:
neutron l2-remote-gateway-connection-delete <uuid>  
````
###### 2.3.4.	l2-remote-gateway-connection-list
This command lists all the remote gateway connections.
```
Usage:
l2-remote-gateway-connection-list 
````

##### 2.4 MAC Commands
The following commands add, update, remote and list the MAC addresses on a remote gateway. 

###### 2.4.1.	l2-remote-mac-create
This command adds MAC address of a host that is located in a remote overlay network that can be reached via a remote gateway connection.
```
Usage:
l2-remote-mac-create <MAC> <gw-connection-uuid>  [--ipaddr=<ip-address>]
Note: the IP address is the remote hosts’
````

###### 2.4.2.	l2-remote-mac-update
This command updates the IP address of a host with MAC address <MAC>.
```
Usage:
l2-remote-mac-update <MAC> <gw-connection-uuid>  --ipaddr=<ip-address>]
````

###### 2.4.3.	l2-remote-mac-delete
This command deletes MAC on a remote gateway connection.
```
Usage:
l2-remote-mac-delete <MAC-uuid> 
````

###### 2.4.4.	l2-remote-mac-list
This command lists all the MAC addresses on all remote gateway connections.
```
Usage:
l2-remote-mac-list
````
