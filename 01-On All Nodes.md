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
  $ echo "202.22.13.11          masternode          master               node1           kubemaster1.kubelocal"| sudo tee -a /etc/hosts
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

Then you may continue to next step [**02-On Master Node**](https://github.com/wizznew/kubernetes-installation/blob/main/02-On%20Master%20Node.md)

