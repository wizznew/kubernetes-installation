This will guide you to setup minimal kubernetes for learning

### REQUIREMENTS
---
#### Virtual Machine Image
You can download from VM Image from [HERE](https://bit.ly/Centos7VM-Kubernetes-Image)


#### Virtual Machine Requirement
| Type | Specs| Note
| --- | --- | ---
| Control Plane (Master) - 1 VM | 2 CPU|
| |2 GB Memory|
| | 40 GB Storage|
| | Network 1| Offline
| | Network 2| Bridged to Ethernet (internet connectivity required for packages installation)
|   |   |  
| Worker Nodes - 2 VMs| 2 CPU|
| |2 GB Memory|
| | 40 GB Storage|
| | Network 1| Offline
| | Network 2| Bridged to Ethernet (internet connectivity required for packages installation)


Above requirements are to give some space for application containers (microservices) to run

## INSTALLATION
Below steps will give a brief information on how to set-up kubernetes

### [STEP 1: On All Nodes](https://github.com/wizznew/kubernetes-installation/blob/main/01-On%20All%20Nodes.md)
### [STEP 2: On Master Node](https://github.com/wizznew/kubernetes-installation/blob/main/02-On%20Master%20Node.md)
### [STEP 3: On Worker Nodes](https://github.com/wizznew/kubernetes-installation/blob/main/03-On%20Worker%20Nodes.md)
### [STEP 4: Deploy Microservice](https://github.com/wizznew/kubernetes-installation/blob/main/04-nginx%20Deployment.md)
