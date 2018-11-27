# Downgrading k8s

This should be possible with `kubeadm` according to the documentation, but in
reality it can be tricky. And it is not documented.

## Downgrade before kubelets are upgraded

Make a **backup before** upgrading (see backup and restore scenario).

In this scenario we practice downgrading immediately after the
`kubeadm upgrade apply` command. You should follow the upgrade guide up until
control plane is upgraded but don't upgrade the kubelet packages.

```
# Downgrade kubeadm
sudo apt-mark unhold kubeadm
sudo apt install kubeadm=1.11.4-00
```

You can now try downgrading by executing `kubeadm upgrade apply v1.11.4`.
If it doesn't work, reset the master: `sudo kubeadm reset` and continue by
restoring and initializing the master just as in the restore scenario.

## Downgrade after complete upgrade

Make a **backup before** upgrading (see backup and restore scenario).

This scenario describes how to downgrade after having finished an upgrade all
the way. You should follow the upgrade guide to completion before continuing.

```
# Downgrade node config on all non-master nodes
sudo kubeadm upgrade node config --kubelet-version v1.11.4
# Downgrade kubelet and kubeadm on all nodes
sudo apt-mark unhold kubelet kubeadm
sudo apt install kubelet=1.11.4-00 kubeadm=1.11.4-00
sudo apt-mark hold kubelet kubeadm
```

It seems to be possible to downgrade now with `kubeadm upgrade apply v1.11.4`.
In case this fails, just reset the master and restore the backup.
