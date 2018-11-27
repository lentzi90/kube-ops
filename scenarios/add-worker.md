# Add worker node

The same playbook that was used to provision the kubernetes cluster can also be
used to add new nodes. You just need to add a new entry in the `hosts` hash and
the `workers` list in the `Vagrantfile`:
```
hosts = {
    "master" => { "memory" => 1536, "ip" => "192.168.10.10"},
    "worker1" => { "memory" => 2048, "ip" => "192.168.10.30"},
    "worker2" => { "memory" => 2048, "ip" => "192.168.10.31"},
    "worker3" => { "memory" => 1024, "ip" => "192.168.10.32"},
    "nfs" => { "memory" => 512, "ip" => "192.168.10.20"}
}
workers = ["worker1", "worker2", "worker3"]
```

Make sure that the `vars` hash also is up to date with the correct `k8s_version`
for example. Then run the provisioner:
```
vagrant provision --provision-with scale-up
```

## How is this done and how can I do it manually?

The procedure described above uses the playbook
`playbooks/deploy-kubernetes.yml` to first configure the new node correctly,
then it gets a join command from the master which is finally used to add the
node to the cluster.

In short, the manual steps are as follows:

- Install all necessary packages (such as `docker`,`kubelet` and `kubeadm`)
- Configure the node (disable swap, open ports and so on)
- Get the join command from the master `kubeadm token create --print-join-command`
- Use the join command to add the node to the cluster.

This is basically the same procedure as when deploying the cluster the first
time, except that the master is not initialized. See this web page for more:
https://kubernetes.io/docs/setup/independent/install-kubeadm/
