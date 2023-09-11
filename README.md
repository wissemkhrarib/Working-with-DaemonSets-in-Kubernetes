# DAEMONSETS OBJECTS IN KUBERNETES

## Why DaemonSets??

In some case, we need to run an application on each node in the k8s cluster
for example for 
* monitoring the node(collectd)
* collecting logs (fluentd) or
* storage daemon (ceph)

This can't be achieved with a ReplicaSet or a Deployment object

And for that, we have another object in kubernetes that can perfrom this task
which is the DaemonSet object

## What are DaemonSets?
A DaemonSet is a k8s object/controllers that ensures that all or some nodes in the k8s cluster runs a instance of a specific pod

For running a pod on a subset of nodes, first of all, we tag the nodes with labels and then, we use these labels in DaemonSet manifest file.

If a pod controlled by a DaemonSet goes down, the DaemonSet will notice that
and will recreate the pod on that node

If a new node is added to the cluster, the DaemonSet will create a pod on the
new added node

To delete the pods created by the DaemonSet, all you have to do is to delete the DaemonSet itself and then all pods managed by this DaemonSet will be deleted.

If you delete only the pods and not the DaemonSet, the pods will be recreated
as long as the DaemonSet stills exist in the cluster

## Creating DaemonSets 
To create a DaemonSet object, let's create the yaml manifest file need to create a DaemonSet
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-ds
spec:
  template:
    metadata:
      labels: 
        app: fluentd
    spec:
      containers:
        - image: fluent/fluentd:v1.16-1
          name: fluentd-container
  selector:
    matchLabels:
      app: fluentd
```
We can verify that on each node, DaemonSet creates a pod we can use the command `kubectl get pods -o wide`

To display the DaemonSets, we can use the command `kubectl get ds`

To get more infos about a DaemonSet object, we can use `kubectl describe daemonset <daemonset-name>`

To delete the DaemonSet, we can use the command `kubectl delete daemonset <daemonset-name>` or `kubectl delete -f <manifest-file-name>`

We can use DaemonSets to create pods on a set of nodes

First of all, we need to tag the nodes with labels using the command `kubectl label nodes <node-name> key=value`

For example `kubectl label nodes minikube-m02 disktype=ssd`

And then in template.spec section we add the property nodeSelector and here we mention the label given to the node

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-ds
spec:
  template:
    metadata:
      labels: 
        app: fluentd
    spec:
      containers:
        - image: fluent/fluentd:v1.16-1
          name: fluentd-container
      nodeSelector:
        disktype: ssd
  selector:
    matchLabels:
      app: fluentd
```