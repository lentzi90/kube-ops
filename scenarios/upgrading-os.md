# Upgrading OS & packages

This is a walk through with commands needed to update packages on a node while
ignoring kubernetes related packages.

Start by draining the node that is to be updated. This should minimize downtime
and avoid issues since all pods are moved to other nodes before the maintenance.
```
kubectl drain <node-name> --ignore-daemonsets --delete-local-data
```

Make sure all kubernetes-related packages are on hold:
```
sudo apt-mark hold kubelet kubeadm kubectl docker
```
Now you can upgrade the packages:
```
sudo apt update
sudo apt upgrade
```
Remember to reboot if needed.

When you are finished, make the node schedulable again:
```
kubectl uncordone <node-name>
```
