# Upgrading k8s (single master)

This is a walk through with commands needed to upgrade kubernetes from 1.11.4
to 1.13.0.

In general, start by checking this page for information related to the specific
upgrade you are doing: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-upgrade/

## On the master node

Plan the upgrade:
```
sudo kubeadm upgrade plan
```

Note the output from the above command, specifically that you need to upgrade
`kubeadm` before you attempt the upgrade. After the upgrade, you also have to
upgrade `kubelet` on all nodes.

Upgrade kubeadm (ubuntu):
```
sudo apt-mark unhold kubeadm
sudo apt install kubeadm=1.13.0-00
sudo apt-mark hold kubeadm
```
Upgrade kubeadm (centos):
```
sudo yum install kubeadm-1.13.0-0 --disableexcludes=kubernetes
```


Upgrade control plane:
```
sudo kubeadm upgrade apply v1.13.0
```

Upgrade kubelet on the master
```
kubectl drain master --ignore-daemonsets --delete-local-data
# ubuntu
sudo apt-mark unhold kubelet
sudo apt install kubelet=1.13.0-00
sudo apt-mark hold kubelet
# centos
sudo yum install kubelet-1.13.0-0 --disableexcludes=kubernetes
```

When it is ready for action again, mark it as schedulable:
```
kubectl uncordon master
```

## On non-master nodes - one at a time

Drain the node:
```
kubectl drain <node-name> --ignore-daemonsets --delete-local-data
```

Upgrade kubelet and kubeadm:
```
# ubuntu
sudo apt-mark unhold kubelet kubeadm
sudo apt install kubelet=1.13.0-00 kubeadm=1.13.0-00
sudo apt-mark hold kubelet kubeadm
# centos
sudo yum install kubelet-1.13.0-0 kubeadm-1.13.0-0 --disableexcludes=kubernetes
```

Upgrade kubelet config:
```
sudo kubeadm upgrade node config --kubelet-version $(kubelet --version | cut -d ' ' -f 2)
sudo systemctl restart kubelet
```

When the node is ready for action again, mark it as schedulable:
```
kubectl uncordon <node-name>
```

# Upgrading HA k8s

Upgrading a cluster with HA control plane is quite similar to the above.
See [this page](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade-ha-1-13/)
for precise instructions.

The main difference is that you should run
```
kubeadm upgrade node experimental-control-plane
```
on all additional control plane nodes. For the upgrade from 1.12 to 1.13 you
also have to edit the kubeadm configmap manually.
