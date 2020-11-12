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
Follow steps descibed in [**03-On Worker Nodes**](https://github.com/wizznew/kubernetes-installation/blob/main/03-On%20Worker%20Nodes.md) to join worker nodes to the cluster

#### 7. Check and Verify Nodes
Check your nodes availability by issuing below command
```
$ kubectl get nodes -o wide
NAME                    STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
kubemaster1.kubelocal   Ready    master   15h   v1.19.3   202.22.13.11   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://19.3.13
worker1.kubelocal       Ready    <none>   13h   v1.19.3   202.22.13.13   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://19.3.13
worker2.kubelocal       Ready    <none>   13h   v1.19.3   202.22.13.14   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://19.3.13
worker3.kubelocal       Ready    <none>   13h   v1.19.3   202.22.13.15   <none>        CentOS Linux 7 (Core)   3.10.0-1127.19.1.el7.x86_64   docker://19.3.13

```
The status **NotReady** may shown a bit longer when you issue some actions.
