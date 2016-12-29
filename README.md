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

Check volumes and claims:
```
$ kubectl get pv,pvc
NAME              CAPACITY   ACCESSMODES   STATUS     CLAIM                 REASON    AGE
pv/pv5gdb         5Gi        RWX           Bound      default/myclaim-db              6m
pv/pv5gweb        5Gi        RWX           Bound      default/myclaim-web             6m
NAME              STATUS     VOLUME        CAPACITY   ACCESSMODES           AGE
pvc/myclaim-db    Bound      pv5gdb        5Gi        RWX                   14s
pvc/myclaim-web   Bound      pv5gweb       5Gi        RWX                   10s
```

Now we are ready to deploy application cluster.

### 3. Deploy MySQL

MySQL is the backend for the Wordpress. We need to create a definition file called mysql-pod.yaml. Before that, we need create password file and import password from that file into k8s secure store. Make sure password.txt does not have a trailing newline

```
kubectl create secret generic mysql-pass --from-file=password.txt
```

... and then  deploy:

```
$ kubectl create -f https://raw.githubusercontent.com/kratochj/kubernetes-cluster-vagrant/master/pods/mysql-pod.yaml
```

Next, we need dedicatexd IP address for MySQL service (clusterIP) so all Wordpress pods are connected to this one single MySQL database.

```
apiVersion: v1
kind: Service
metadata:
  labels:
    name: mysql
  name: mysql
spec:
  clusterIP: 10.254.10.20
  ports:
    - port: 3306
  selector:
    name: mysql
```	

deploy it: 

```
kubectl create -f https://raw.githubusercontent.com/kratochj/kubernetes-cluster-vagrant/master/pods/mysql-service.yaml
```

Here is how to check if everything is deployed:
```
$ kubectl get pods,services
```
















