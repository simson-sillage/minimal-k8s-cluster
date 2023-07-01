# Setup a minimal Kubernetes cluster with Ansible

## Requirments

You need to setup servers running the default [Ubuntu 22.04 LTS cloud image](https://cloud-images.ubuntu.com/jammy/current/) and be able to ssh into them. The servers need to be able to communicate with each other.

The playbooks assume root access. If you can't login via root you need to configure `become` on the Ansible cli.

Copy the IPs of your servers into the hosts/inventory file. You need at least one IP for each: `master`, `control_planes`, `worker`.

Your inventory might look like this:

```ini
[master]
192.168.2.1

[control_planes]
192.168.2.2
192.168.2.3

[worker]
192.168.2.4
192.168.2.5
192.168.2.6
```

`master` is just the control-plane used to boostrap the k8s cluster with kubeadm.

## Usage

Change into the `ansible/` directory.

To prepare the Ubuntu servers for Kubernetes, run:

```sh
ansible-playbook -i hosts -u root prepare-ubuntu-k8s-nodes.yaml
```

As long as the playbook doesn't fail in the `setup-containerd` task, it is idempotent.

Now you can create a Kubernetes cluster with:

```sh
ansible-playbook -i hosts -u root create-k8s-cluster.yaml
```

This playbook is **NOT** idempotent. Don't run it twice.

If everything worked out, you can ssh into you `master`-node and run:

```sh
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl get nodes
```

Which should print something like this:

```txt
NAME   STATUS   ROLES           AGE   VERSION
c1     Ready    control-plane   20m   v1.27.3
c2     Ready    control-plane   18m   v1.27.3
c3     Ready    control-plane   18m   v1.27.3
w1     Ready    <none>          18m   v1.27.3
w2     Ready    <none>          18m   v1.27.3
w3     Ready    <none>          18m   v1.27.3
```

You now have bootstrapped a minimal but usable k8s-cluster.
