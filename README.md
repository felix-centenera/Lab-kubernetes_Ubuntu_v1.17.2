Deploy Kubernetes 1.17.2 & bento/ubuntu-16.04  & VirtualBox & Vagrant & Ansible
========================================

Description
--------------------------------
This document describe the process to install Kubernetes 1.17.2  in VirtualBox. Vagrant generate the VMs and resources for us. Ansible will install Kubernetes 1.17.2.
This is a modification of https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant(Author: Naresh L J (Infosys))


Requirements
--------------------------------
VirtualBox 6.1

Vagrant version: Installed Version: 2.2.6
    Vagrant box list:

      ubuntu-16.04 (virtualbox, 201912.15.0)


Infrastructure
--------------------------------
1 master nodes.

    k8s-master

2 app node.

    node-1
    node-2

Details
--------
User Virtual Machine:

user: root

password: vagrant

user: vagrant

password: vagrant


Download the project
-----------------------------------------
```
git clone https://github.com/felix-centenera/kubernetes_Ubuntu_v1.17.2.git
```
Generate VirtualBox Machines with Vagrant
-----------------------------------------
```
vagrant up
```
Login in the bastion
-----------------------------------------
```
vagrant ssh k8s-master

```


Start using Kubernetes
-----------------------------------------

kubectl get nodes

NAME         STATUS   ROLES    AGE     VERSION

k8s-master   Ready    master   5m43s   v1.17.2

node-1       Ready    <none>   3m2s    v1.17.2

node-2       Ready    <none>   33s     v1.17.2



kubectl get namespaces

NAME              STATUS   AGE

default           Active   5m52s

kube-node-lease   Active   5m53s

kube-public       Active   5m53s

kube-system       Active   5m53s




kubectl get pods -n kube-system

NAME                                       READY   STATUS    RESTARTS   AGE

calico-kube-controllers-6b9d4c8765-49gg5   1/1     Running   0          5m40s

calico-node-9cbps                          1/1     Running   0          5m40s

calico-node-fjn4p                          1/1     Running   0          50s

calico-node-nlzph                          1/1     Running   0          3m19s


coredns-6955765f44-nctjs                   1/1     Running   0          5m40s

coredns-6955765f44-s6n6z                   1/1     Running   0          5m40s

etcd-k8s-master                            1/1     Running   0          5m53s

kube-apiserver-k8s-master                  1/1     Running   0          5m53s

kube-controller-manager-k8s-master         1/1     Running   0          5m53s

kube-proxy-bwgtn                           1/1     Running   0          50s

kube-proxy-rvf7n                           1/1     Running   0          5m40s

kube-proxy-xs4ms                           1/1     Running   0          3m19s

kube-scheduler-k8s-master                  1/1     Running   0          5m53s



Sources:
-----------------------------------------
https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant/

https://github.com/kubernetes/kubernetes/issues/44665

https://docs.projectcalico.org/v3.9/getting-started/kubernetes/installation/calico#installing-with-the-etcd-datastore
