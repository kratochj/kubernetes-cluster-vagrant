# Kubernetes Cluster with Vagrant

This is example how to easily install Kubernetes cluster with Vagrant. By default vagrant spin up one master and three nodes (workers). 

## Requirements

* Virtual Box
* Vagrant

## Quick start

Run `vagrant` command to spin machines up:

	vagrant up 


...wait couple of minutes...

You have running _k8s_ cluster 

## Install HA Wordpress on running cluster

Steps in this section should be performed on Kubernetes master (or Kubernete client via kubectl).

### 1. Create Persistent Volumes

```
kubectl create -f https://raw.githubusercontent.com/kratochj/kubernetes-cluster-vagrant/master/pods/nfs-db.yaml
kubectl create -f https://raw.githubusercontent.com/kratochj/kubernetes-cluster-vagrant/master/pods/nfs-web.yaml
```

Check whether volumes were created:
```
$ kubectl get pv
NAME      CAPACITY   ACCESSMODES   STATUS      CLAIM     REASON    AGE
pv5gdb    5Gi        RWX           Available                       1m
pv5gweb   5Gi        RWX           Available                       1m
```

### 2. Create Persistent Volume Claims

```
kubectl create -f https://raw.githubusercontent.com/kratochj/kubernetes-cluster-vagrant/master/pods/claim-db.yaml
kubectl create -f https://raw.githubusercontent.com/kratochj/kubernetes-cluster-vagrant/master/pods/claim-web.yaml
```

Check whether volumes were created:
```
$ kubectl get pv
NAME      CAPACITY   ACCESSMODES   STATUS      CLAIM     REASON    AGE
pv5gdb    5Gi        RWX           Available                       1m
pv5gweb   5Gi        RWX           Available                       1m
```
