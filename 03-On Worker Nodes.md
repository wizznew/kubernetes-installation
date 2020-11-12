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

#### 3. Check and Verify
Continue to step #7 on [**02-On Master Node**](https://github.com/wizznew/kubernetes-installation/blob/main/02-On%20Master%20Node.md)
