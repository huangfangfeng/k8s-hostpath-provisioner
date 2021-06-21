# Kubernetes hostPath PV provisioner

This is temporary workaround for Kubernetes local volume management until
[this feature](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/local-storage-overview.md)
will be implemented.


## Usage

This provisioner requires custom kube-scheduler to be running in your cluster. You should edit your scheduler manifest and
change container image there:(already exist)

```yaml
# ...
containers:
- image: image: nailgun/kube-scheduler-amd64:v1.6.2_hostpath.0
  command:
  - kube-scheduler
  - # your args here
# ...
```

* [List of available image tags](https://hub.docker.com/r/nailgun/kube-scheduler-amd64/tags/)
* [Sources for all image tags](https://github.com/nailgun/kubernetes/branches/all) (based on upstream with one simple commit
added)

After you have running custom scheduler, install the provisioner:

```
$ cd example
$ kubectl create -f hostpath-provisioner.yaml
```


Now you can test it. You should annotate your nodes with information of available storage:

```
$ kubectl annotate node NODE_NAME hostpath.kubevirt.io/ssd=/tmp/ssd
$ kubectl annotate node NODE_NAME hostpath.kubevirt.io/hdd=/tmp/hdd
$ kubectl annotate node NODE_NAME hostpath.kubevirt.io/nvme=/tmp/nvme
```

Then create test StorageClass, PVC and Pod:

```
$ kubectl create -f sc.yaml -f pvc.yaml -f test-pod.yaml
```

The test-pod will be always scheduled on the same node where the PVC has been provisioned.

Note StorageClass has `hostPathName` parameter equal to `ssd` which means that provisioner will search for a node having
`hostpath.kubevirt.io/ssd` annotation and it will use hostPath specified in the annotation. If you have more than one node
with this annotation, node will be selected randomly.



## Limitations

This provisioner doesn't support any kind of storage isolation nor quotas. This is temporary solution to have working PVCs with local storage.

If a pod uses more then one PVC, they can be scheduled to different nodes and Pod will be failed to be scheduled.


## Thanks

Thanks to @TamalSaha for [the idea and first implementation](https://github.com/kubernetes-incubator/external-storage/issues/15).
