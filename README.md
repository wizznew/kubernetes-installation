### REQUIREMENTS
---
| Type | Specs| Note
| --- | --- | ---
| Control Plane (Master) - 1 VM | 2 CPU|
| |2 GB Memory|
| | 40 GB Storage|
| | Network 1| Host-only network
| | Network 2| NAT or Bridge
|   |   |  
| Worker Nodes - 2 VMs| 2 CPU|
| |2 GB Memory|
| | 40 GB Storage|
| | Network 1| Host-only network
| | Network 2| NAT or Bridge

Above requirements are to give some space for application containers to run

## INSTALLATION
Below steps will give a brief information on how to set-up kubernetes

### [STEP 1: On All Nodes](https://github.com/wizznew/kubernetes-installation/blob/main/01-On%20All%20Nodes.md)
### [STEP 2: On Master Node](https://github.com/wizznew/kubernetes-installation/blob/main/02-On%20Master%20Node.md)
### [STEP 3: On Worker Nodes](https://github.com/wizznew/kubernetes-installation/blob/main/03-On%20Worker%20Nodes.md)
### [STEP 4: Let's create a microservice!](https://github.com/wizznew/kubernetes-installation/blob/main/04-nginx%20Deployment.md)
