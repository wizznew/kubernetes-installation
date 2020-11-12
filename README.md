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

### ON ALL NODES
#### 1. Update your system

On CENTOS7
```
$ sudo yum update -y
$ sudo yum upgrade -y
$ sudo yum install -y yum-utils
```

On CENTOS8
```
$ sudo dnf update -y
$ sudo dnf upgrade -y
```
#### 2. Add repositories


On CENTOS7
```
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

On CENTOS8
```
$ sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
```

On Both CENTOS7 & CENTOS8
```
$ sudo su
# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
  [kubernetes]
  name=Kubernetes
  baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
  enabled=1
  gpgcheck=1
  repo_gpgcheck=1
  gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
  EOF
# exit
```

#### 3. Remove old version docker installation (optional, if you have installed docker previously)

On CENTOS7
```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
#### 4. Install required software packages

On CENTOS7
```
$ sudo yum install -y docker-ce docker-ce-cli containerd.io
$ sudo yum install -y kubelet kubeadm kubectl
```

On CENTOS8
```
$ sudo dnf install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
$ sudo dnf install -y docker-ce
$ sudo dnf install -y kubeadm
```
#### 5. Setup Bridging

Make sure that the `br_netfilter` module is loaded


This can be done by running `lsmod | grep br_netfilter`


To load it explicitly call `sudo modprobe br_netfilter`


As a requirement for your Linux Node's iptables to correctly see bridged traffic, you should ensure `net.bridge.bridge-nf-call-iptables` is set to `1` in your sysctl


```
$ sudo su
# cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
  net.bridge.bridge-nf-call-ip6tables = 1
  net.bridge.bridge-nf-call-iptables = 1
  EOF
$ sudo sysctl --system
```

#### 6. Disable SE Linux
```
$ sudo setenforce 0
$ sudo sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

```
#### 7. Disable `/swap`


Open `/etc/fstab` on editor and put a `#` character on the beginning of swap partition definition
```
# /dev/mapper/centos-swap /swap
```
Turn off swap
```
$ sudo swapoff -a
```
#### 8. Setup `/etc/hosts`
  
  
  Add entries to your /etc/hosts covering IP and hostname of your nodes
  ```
  $ echo "202.22.13.11          masternode          master               node1           master.kubelocal"| sudo tee -a /etc/hosts
  $ echo "202.22.13.11          slavenode           slave                node2           slave.kubelocal"| sudo tee -a /etc/hosts
  $ echo "202.22.13.11          workera             worker1              node3           worker1.kubelocal"| sudo tee -a /etc/hosts
  $ echo "202.22.13.11          workerb             worker2              node4           worker2.kubelocal"| sudo tee -a /etc/hosts
  $ echo "202.22.13.11          workerc             worker3              node5           worker3.kubelocal"| sudo tee -a /etc/hosts
  ```
  Changing hostname will be covered in setting up master node and worker nodes
  
#### 9. Finalizing and Reboot
```
$ sudo systemctl enable docker
$ sudo systemctl enable kubelet
$ sudo systemctl daemon-reload
$ sudo systemctl start docker
$ sudo systemctl start kubelet
$ sudo shutdown -r now
```
#### 10. Notes

You may consider to clone the VM in this step.

### ON MASTER NODE ONLY
#### 1. Open Network Ports
```
$ sudo firewall-cmd --permanent --add-port=6443/tcp
$ sudo firewall-cmd --permanent --add-port=2379-2380/tcp
$ sudo firewall-cmd --permanent --add-port=10250/tcp
$ sudo firewall-cmd --permanent --add-port=10251/tcp
$ sudo firewall-cmd --permanent --add-port=10252/tcp
$ sudo firewall-cmd --permanent --add-port=10255/tcp
$ sudo firewall-cmd --reload
```
#### 2. Download Control Plane Required Software Packages
```
$ sudo kubeadm config images pull
```
#### 3. Initialize Control Plane
If you need to bind a specific IP address in master node as communication point with worker nodes, use `--apiserver-advertise-address` option to determine the IP address

If you need to bind a specific IP address range for pods when you create after kubernetes cluster formed, use `--pod-network-cidr` option to determine the IP address subnet

The `--pod-network-cidr` is a mandatory when you are choosing networking pod other than _weave_ within next step of **setup networking pod**.

> Other networking options are *Calico*, *Flannel*, *Cilium*, etc.  You can only use 1 networing pod within a cluster
```
$ sudo kubeadm init --apiserver-advertise-address [IP-ADDRESS-OF-MASTER-NODE] --pod-network-cidr [IP-ADDRESS-OF-POD-NETWORK]
```

If you were not sure and choose for automatic creation of kubernetes cluster just run `sudo kubeadm init`, will use a weave network within next step.

> Note: 
>
> In this step, you may get notification of command to run on worker nodes in order to join the cluster, please keep it for next step of joining worker nodes.
>
> Below is sample notification for joining worker nodes 
>
> kubeadm join 202.22.13.11:6443 --token nu06lu.xrsux0ss0ixtnms5  \ --discovery-token-ca-cert-hash ha256:f996ea35r4353d342fdea2997a1cf8caeddafd6d4360d606dbc82314683478hjmf7
#### 4. Setup Environment
```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 5. Setup Networking Pod
Use only 1 that suits your previous step


**weave** networking pod
```
$ sudo su
# export kubever=$(kubectl version | base64 | tr -d '\n')
# kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever
# exit
```
**calico** networking pod
```
$ sudo su
# kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
# exit
```
#### 6. Setup Worker Nodes
Follow steps descibed in **ON WORKER NODES ONLY** to join worker nodes to the cluster

#### 7. Check and Verify Nodes
Check your nodes availability by issuing below command
```
$ kubectl get nodes
```
The status **NotReady** may shown a bit longer when you issue some actions.

### ON WORKER NODES ONLY
#### 1. Open Network Ports
```
$ sudo firewall-cmd --permanent --add-port=10251/tcp
$ sudo firewall-cmd --permanent --add-port=10255/tcp
$ sudo firewall-cmd --reload
```

If you are going to use NodePort, you will need to open port 30000 and above
```
$ sudo firewall-cmd --permanent --add-port=30000-32767/tcp
$ sudo firewall-cmd --reload
```

#### 2. Join to Control Plane
Use command that shown when you initialize your kubernetes master (control plane) node.

below only sample of join command

```
$ sudo kubeadm join 202.22.13.11:6443 --token nu06lu.xrsux0ss0ixtnms5  \ --discovery-token-ca-cert-hash ha256:f996ea35r4353d342fdea2997a1cf8caeddafd6d4360d606dbc82314683478hjmf7
```

