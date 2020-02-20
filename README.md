Deploy Kubernetes 1.17.2 & bento/ubuntu-16.04  & VirtualBox & Vagrant & Ansible
========================================

Description
--------------------------------
This document describe the process to install Kubernetes 1.17.2  in VirtualBox. Vagrant generate the VMs and resources for us. Ansible will install Kubernetes 1.17.2.
This is a modification of https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant (Author: Naresh L J (Infosys))


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
```
kubectl get nodes

NAME         STATUS   ROLES    AGE     VERSION

k8s-master   Ready    master   5m43s   v1.17.2

node-1       Ready    <none>   3m2s    v1.17.2

node-2       Ready    <none>   33s     v1.17.2
```

```
kubectl get namespaces

NAME              STATUS   AGE

default           Active   5m52s

kube-node-lease   Active   5m53s

kube-public       Active   5m53s

kube-system       Active   5m53s
```


```
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
```

Deploy NFS Storage
-----------------------------------------

Create a directory for nfs in the master vm:
```
cd /home/vagrant/storageDeploy
sudo mkdir -p /storage/dynamic
sudo chmod -R 777 /storage/dynamic
```

Label the node and delete NoSchedule label from master:
```
kubectl label nodes node-1  role=master
kubectl taint node k8s-master  node-role.kubernetes.io/master:NoSchedule-
```

Create the service:
```
k create -f nfs-deployment-icpservice.yaml
...
...
...
service/nfs-provisioner created
```

Create the DeploymentConfig:

```
k create -f nfs-deployment-icp.yaml
deployment.apps/nfs-provisioner created
```

Create the Service Account and associate a clusterrolebinding:
```
kubectl create serviceaccount nfs-provisioner-admin-sa
kubectl create clusterrolebinding nfs-provisioner-admin-sa  --clusterrole=cluster-admin --serviceaccount=default:nfs-provisioner-admin-sa
```

Add the serviceAccount to the DC
```
k edit deployment nfs-provisioner

spec:
  template:
    # Below is the podSpec.
    metadata:
      name: ...
    spec:
      serviceAccountName: nfs-provisioner-admin-sa
```

Create the STORAGECLASS:
```
k create -f nfs-class.yaml
...
...
...
storageclass.storage.k8s.io/nfs-dynamic created
```

Patch the storageclass as the default storageclass:

*kubectl patch storageclass <your-class-name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'*

```
kubectl patch storageclass nfs-dynamic -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
...
...
...
storageclass.storage.k8s.io/nfs-dynamic patched
```

Create pvc to test StorageClass:
```
k create -f nfs-test-claim.yaml
...
...
...
persistentvolumeclaim/nfs created
```

Check that the pvc is bounded:
```
k get pvc
NAME   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nfs    Bound    pvc-57289cf4-bc7d-4699-9ca0-70c3140c4d5e   1Mi        RWX            nfs-dynamic    3s
```

Sources:
-----------------------------------------
https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant/

https://github.com/kubernetes/kubernetes/issues/44665

https://docs.projectcalico.org/v3.9/getting-started/kubernetes/installation/calico#installing-with-the-etcd-datastore
