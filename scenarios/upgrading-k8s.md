# Upgrading k8s (single master)

This is a walk through with commands needed to upgrade kubernetes from 1.11.4
to 1.12.2.

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

If the kubernetes related packages are on hold, you must undo this before it is
possible to upgrade them:
```
sudo apt-mark unhold kubeadm
```

Upgrade kubeadm:
```
sudo apt install kubeadm=1.12.2-00
sudo apt-mark hold kubeadm
```

Upgrade control plane:
```
sudo kubeadm upgrade apply v1.12.2
```

Upgrade kubelet on the master
```
kubectl drain master --ignore-daemonsets --delete-local-data
sudo apt-mark unhold kubelet
sudo apt install kubelet=1.12.2-00
sudo apt-mark hold kubelet
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
sudo apt-mark unhold kubelet kubeadm
sudo apt install kubelet=1.12.2-00 kubeadm=1.12.2-00
sudo apt-mark hold kubelet kubeadm
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
