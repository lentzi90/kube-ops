# README

This folder contains a Vagrantfile to set up a local kubernetes cluster.
Ansible is used for provisioning (see `playbooks`).

## Quick start

The cluster is set up and provisioned by running `vagrant up`.

If you wish to run playbooks manually from the host, do it like this:
```
ansible-playbook playbooks/provision-all.yml
```
There is no need to specify the inventory since it is already configured in
`ansible.cfg`.

## Requirements

- [Vagrant](https://www.vagrantup.com/)
- [Libvirt](https://libvirt.org/) OR [Virtualbox](https://www.virtualbox.org/)
- [Ansible](https://www.ansible.com/) >= 2.7

If you want to use vagrant with the libvirt provider, make sure you set up
libvirt with KVM first and install the `vagrant-libvirt` plugin.
Instructions for setting up libvirt with KVM on Ubuntu can be found [here](https://help.ubuntu.com/community/KVM/Installation).
The `vagrant-libvirt` plugin can be found [here](https://github.com/vagrant-libvirt/vagrant-libvirt).

If you want to use virtualbox, just install virtualbox and vagrant and change
the `inventory.ini` file to use the virtualbox block instead of the libvirt one.
Like this:

```
; Libvirt
; master ansible_host=192.168.10.10 ansible_port=22 ansible_user='vagrant' ansible_ssh_private_key_file='.vagrant/machines/master/libvirt/private_key'
; worker1 ansible_host=192.168.10.30 ansible_port=22 ansible_user='vagrant' ansible_ssh_private_key_file='.vagrant/machines/worker1/libvirt/private_key'
; worker2 ansible_host=192.168.10.31 ansible_port=22 ansible_user='vagrant' ansible_ssh_private_key_file='.vagrant/machines/worker2/libvirt/private_key'
; worker3 ansible_host=192.168.10.32 ansible_port=22 ansible_user='vagrant' ansible_ssh_private_key_file='.vagrant/machines/worker3/libvirt/private_key'
; nfs ansible_host=192.168.10.20 ansible_port=22 ansible_user='vagrant' ansible_ssh_private_key_file='.vagrant/machines/nfs/libvirt/private_key'
; Virtualbox
master ansible_host=192.168.10.10 ansible_port=22 ansible_user='vagrant' ansible_ssh_private_key_file='.vagrant/machines/master/virtualbox/private_key'
worker1 ansible_host=192.168.10.30 ansible_port=22 ansible_user='vagrant' ansible_ssh_private_key_file='.vagrant/machines/worker1/virtualbox/private_key'
worker2 ansible_host=192.168.10.31 ansible_port=22 ansible_user='vagrant' ansible_ssh_private_key_file='.vagrant/machines/worker2/virtualbox/private_key'
worker3 ansible_host=192.168.10.32 ansible_port=22 ansible_user='vagrant' ansible_ssh_private_key_file='.vagrant/machines/worker3/virtualbox/private_key'
nfs ansible_host=192.168.10.20 ansible_port=22 ansible_user='vagrant' ansible_ssh_private_key_file='.vagrant/machines/nfs/virtualbox/private_key'
```

## Ops Scenarios

A number of scenarios with instructions and commands are available in the
`scenarios` folder.

- [upgrading k8s](scenarios/upgrading-k8s.md)
- [downgrading k8s](scenarios/downgrading-k8s.md)
- [upgrading OS and packages](scenarios/upgrading-os.md)
- [add worker node](scenarios/add-worker.md)
- [backup & restore](scenarios/backup-restore.md)

## Test application: wordpress

The vagrant environment contains a wordpress deployment to simulate a workload.
Add `192.168.10.20 example.com wordpress.example.com` to `/etc/hosts` in order
to access wordpress at `wordpress.example.com`. Or maybe [set up dnsmasq](https://www.linux.com/learn/intro-to-linux/2018/2/advanced-dnsmasq-tips-and-tricks)
to resolve the whole kube-ops domain to the loadbalancer.

See `playbooks/templates/wordpress-production.yaml.j2` for configuration values.
The login credentials are `kube-ops:p455w0rd` at http://wordpress.example.com/wp-admin.

There are a few other components deployed in the cluster as base infrastructure.

- [nfs-client-provisioner](https://hub.kubeapps.com/charts/stable/nfs-client-provisioner)
for persistent volume provisioning
- [nginx-ingress](https://hub.kubeapps.com/charts/stable/nginx-ingress) as
ingress controller

## Issues to be aware of

- There is a [bug](https://github.com/kubernetes/kubernetes/issues/55713)
that causes pods to remain in unknown state on nodes that are not ready, if they
have volumes attached.
    - [shutdown taint issue](https://github.com/kubernetes/kubernetes/issues/58635)
- Helm does not automatically update repository information. Make sure you get
version you want of all charts, and if needed run `helm repo update`.
- Be careful what you wish for: pod disruption budgets won't stop nodes from
going down, but they will prevent you from evicting pods!
- `kubeadm` is supposed to support downgrading using the same method as when
upgrading. Unfortunately, it does not seem to work well. It does not support
planning the downgrade and applying it directly makes weave crash loop.
- Backing up only etcd data and root certificate is not enough. Old secrets
(tokens) will not be valid and cause applications to fail. (Kube-proxy is one
example.)
- Wordpress is sometimes unable to connect to the database while the master is
down. This is due to coredns running on the master at first. If the pods are
moved to some other node all applications continue to operate normally when the
master goes down.
