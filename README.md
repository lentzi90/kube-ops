# README

This folder contains a Vagrantfile to set up a local kubernetes cluster.
Ansible is used for provisioning (see `playbooks`).

## Quick start

The cluster is set up and provisioned by running `vagrant up --provider libvirt`.

If you wish to run playbooks manually from the host, do it like this:
```
ansible-playbook playbooks/provision-all.yml
```
There is no need to specify the inventory since it is already configured in
`ansible.cfg`.

## Requirements

- [Vagrant](https://www.vagrantup.com/)
- [Libvirt](https://libvirt.org/) OR [Virtualbox](https://www.virtualbox.org/)
- [Ansible](https://www.ansible.com/) >= 2.6

If you want to use vagrant with the libvirt provider, make sure you set up
libvirt with KVM first and install the `vagrant-libvirt` plugin.
Instructions for setting up libvirt with KVM on Ubuntu can be found [here](https://help.ubuntu.com/community/KVM/Installation).
The `vagrant-libvirt` plugin can be found [here](https://github.com/vagrant-libvirt/vagrant-libvirt).

## Options

There are multiple ways to configure the vagrant cluster. This section describes
some of the possibilities, but by no means all of them.

Check the `examples` folder for some example configurations. It contains
inventory files ending with `.ini` and vagrant files starting with
`Vagrantfile`. If you wish to try one of the examples, replace the `Vagrantfile`
with the example vagrant file (e.g. `mv examples/Vagrantfile.ha Vagrantfile`).
It is already configured to use the corresponding example inventory.

### Virtualization provider

If you want to use virtualbox instead of libvirt, just install virtualbox and
change the `inventory.ini` file to use the virtualbox block instead of the
libvirt one. Like this:

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

### Operating system

You can choose between Ubuntu and CentOS as operating system for the nodes.
Simply uncomment the vagrant box you want to use in the `Vagrantfile`:
```ruby
config.vm.box = "centos/7"
# config.vm.box = "generic/ubuntu1604"
# config.vm.box = "generic/ubuntu1804"
```
OR add the box in the `hosts` hash (see `worker2` and `worker3`):
```ruby
hosts = {
    "master" => { "memory" => 1536, "ip" => "192.168.10.10"},
    "worker1" => { "memory" => 1536, "ip" => "192.168.10.30"},
    "worker2" => { "memory" => 1536, "ip" => "192.168.10.31", "box" => "generic/ubuntu1604"},
    "worker3" => { "memory" => 1024, "ip" => "192.168.10.32", "box" => "generic/ubuntu1804"},
    "nfs" => { "memory" => 512, "ip" => "192.168.10.20"}
}
```

### High availability control plane

The provided playbooks support setting up a HA control plane for kubernetes
versions 1.12 and 1.13. You will need at least three master nodes configured
both in the `Vagrantfile` and in the `inventory.ini` file. Additionally, you
probably want to set up an external client instead of relying on one of the
masters for talking to the API. You can for example use the nfs/loadbalancer
machine. See `examples/ha.ini` and `examples/Vagrantfile.ha` for an example
on how to configure this.

### Running playbooks manually

You may want to run some playbooks manually, for example if something fails or
if you don't want to do the full provisioning. Rerunning playbooks should not be
a problem, they are written to be idempotent. Check the `playbooks` folder for
more information about the playbooks.

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
Add `192.168.10.40 example.com wordpress.example.com` to `/etc/hosts` in order
to access wordpress at `wordpress.example.com`. Or maybe [set up dnsmasq](https://www.linux.com/learn/intro-to-linux/2018/2/advanced-dnsmasq-tips-and-tricks)
to resolve the whole kube-ops domain to the loadbalancer.

See `playbooks/templates/wordpress-production.yaml.j2` for configuration values.
The login credentials are `kube-ops:p455w0rd` at http://wordpress.example.com/wp-admin.

There are a few other components deployed in the cluster as base infrastructure.

- [nfs-client-provisioner](https://hub.kubeapps.com/charts/stable/nfs-client-provisioner)
for persistent volume provisioning
- [nginx-ingress](https://hub.kubeapps.com/charts/stable/nginx-ingress) as
ingress controller
- [metallb](https://hub.kubeapps.com/charts/stable/metallb) as loadbalancer

## Issues to be aware of

- There is a [bug](https://github.com/kubernetes/kubernetes/issues/55713)
that causes pods to remain in unknown state on nodes that are not ready, if they
have volumes attached.
    - [shutdown taint issue](https://github.com/kubernetes/kubernetes/issues/58635)
- Helm does not automatically update repository information. Make sure you get
the version you want of all charts, and if needed run `helm repo update`.
- Be careful what you wish for: pod disruption budgets won't stop nodes from
going down, but they will prevent you from evicting pods!
- Backing up only etcd data and root certificate is not enough. Old secrets
(tokens) will not be valid and cause applications to fail. (Kube-proxy is one
example.)
- Coredns running on the master at first. This will make all pods loose
DNS if/when the master goes down. If the coredns pods are moved to some other
node (by killing them after other nodes have joined) all applications will
continue to operate normally when the master goes down.
- Vagrant may occasionally try to run the ansible provisioner for all VMs. If
this happens, just `Ctrl + C` and start it again with `vagrant provision`.
(You'll notice this by looking for duplicate tasks and output.)
