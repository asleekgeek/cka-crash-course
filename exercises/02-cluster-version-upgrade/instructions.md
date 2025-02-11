# Exercise 2

<details>
<summary><b>Quick Reference</b></summary>
<p>

* Namespace: N/A<br>
* Documentation: [Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

</p>
</details>

In this exercise, you will learn how to upgrade the cluster version from 1.31.1 to 1.31.5 using kubeadm. The cluster will contain of a single control plane node named `kube-control-plane`, and one worker node named `kube-worker-1`. The existing setup uses virtual machines (VMs) to emulate the cluster environment.

> [!NOTE]
> The file [vagrant-setup.md](../common/vagrant-setup.md) describes the setup instructions and commands for Vagrant and VMware. If you do not want to use the Vagrant environment, you can use the O'Reilly interactive lab ["Upgrading a cluster version"](https://learning.oreilly.com/interactive-lab/cka-prep-upgrading/9781492095514).

## Upgrading the Control Plane Node

1. Shell into the control plane node with `vagrant ssh kube-control-plane`.
2. Upgrade kubeadm to `1.31.5-1.1` via `apt-get`. Verify that the correct version has been set.
3. Drain the control plane node.
4. Use the appropriate kubeadm command to upgrade the control plane to `1.31.5`.
5. Uncordon the control plane node.
6. Upgrade kubelet and kubectl to `1.31.5-1.1`.
7. Restart the kubelet.
8. Exit out of the node.

## Upgrading the Worker Node

1. Shell into the worker node with `vagrant ssh kube-worker-1`.
2. Upgrade kubeadm to `1.31.5-1.1` via `apt-get`. Verify that the correct version has been set.
3. Drain the node.
4. Upgrade the kubelet configuration.
5. Upgrade kubelet and kubectl to `1.31.5-1.1`.
6. Restart the kubelet.
7. Uncordon the node.
8. Exit out of the node.
