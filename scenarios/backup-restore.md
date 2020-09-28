# Backup and restore

This scenario describes how to backup and restore a master node.
Most of the instructions here are based on [this guide](https://labs.consol.de/kubernetes/2018/05/25/kubeadm-backup.html).

## Backup

The most important backup is the etcd snapshot. It contains all the information
about deployments, pods, volumes and so on. In addition to etcd, the
certificates should also be backed up. Without them, it is not possible to
restore the master "in place". A new cluster would have to be created that the
old nodes could then join. With the method below, the master should simply
re-appear in the old cluster.

Optionally, you may want to backup the kubeadm configuration file.

```
mkdir backup
sudo mount nfs:/nfs/export/www-data backup
# Backup certificates
sudo cp -r /etc/kubernetes/pki backup/

# Make etcd snapshot
sudo docker run --rm -v $(pwd)/backup:/backup \
    --network host \
    -v /etc/kubernetes/pki/etcd:/etc/kubernetes/pki/etcd \
    --env ETCDCTL_API=3 \
    k8s.gcr.io/etcd:3.4.3-0 \
    etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt --key=/etc/kubernetes/pki/etcd/healthcheck-client.key snapshot save /backup/etcd-snapshot-latest.db

# Backup kubeadm-config
sudo cp /etc/kubeadm/kubeadm-config.yaml backup/
```

## Oops, the master failed

Make the master go down by reverting everything that `kubeadm init` did.

```
sudo kubeadm reset --skip-phases remove-etcd-member
```

A more drastic alternative is to destroy the node completely:

```
vagrant destroy master
vagrant up master --no-provision
ansible-playbook playbooks/deploy-kubernetes.yml --tags prereq,node

# On master
mkdir backup
sudo mount nfs:/nfs/export/www-data backup
```

## Restore

```
# Restore certificates
sudo cp -r backup/pki /etc/kubernetes/

# Restore etcd backup
sudo mkdir -p /var/lib/etcd
sudo docker run --rm \
    -v $(pwd)/backup:/backup \
    -v /var/lib/etcd:/var/lib/etcd \
    --env ETCDCTL_API=3 \
    k8s.gcr.io/etcd:3.4.3-0 \
    /bin/sh -c "etcdctl snapshot restore '/backup/etcd-snapshot-latest.db' ; mv /default.etcd/member/ /var/lib/etcd/"

# Restore kubeadm-config
sudo mkdir /etc/kubeadm
sudo cp backup/kubeadm-config.yaml /etc/kubeadm/

# Initialize the master with backup
sudo kubeadm init --ignore-preflight-errors=DirAvailable--var-lib-etcd \
    --config /etc/kubeadm/kubeadm-config.yaml
```
