# Playbooks for deploying Kubernetes

The main playbook is `provision-all.yml`. It applies all roles and imports the
other playbooks in order to install Kubernetes, nfs server, loadbalancer and
example applications.

You can use tags or individual playbooks to in order to apply only specific
parts. The available top level tags are:

- **prereq:** for installing basic packages needed to use ansible
- **loadbalancer:** for installing the loadbalancer
- **nfs:** for provisioning the nfs server
- **charts:** for installing helm and all charts
- **kubernetes:** for deploying kubernetes
- **client:** for installing `kubectl` and kubeconfig

Individual playbooks:

- **deploy-kubernetes.yml** sets up the Kubernetes cluster by applying the
kubernetes-node role and then running the following playbooks
    - **initialize-master.yml** starts (the first) control plane node and
    installs a network provider.
    - **ha-control-plane.yml** adds additional control plane nodes
        - **ha-1.12.yml** specific procedure for adding control plane nodes in
        version 1.12
    - **join-cluster.yml** adds worker nodes to the cluster
- **helm-charts.yml** installs helm and all charts
- **set-up-client.yml** installs `kubectl` and adds the kubeconfig file

## Example usage

Say you want to automatically install all needed packages, but manually
initialize the cluster. You could accomplish this by first starting the VMs
without provisioning:

```
vagrant up --no-provision
```

Then install kubernetes packages and prerequisites for ansible managed hosts:

```
ansible-playbook playbooks/provision-all.yml --tags prereq,node
```

When you are finished setting up the cluster, you may want to add helm and some
charts:

```
ansible-playbook playbooks/helm-charts.yml --skip-tags wordpress
```
