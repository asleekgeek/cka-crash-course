# Solution

Open an interactive shell to the control plane node.

```
$ vagrant ssh kube-control-plane
```

Have a look at the status of the nodes. The worker node has an issue indicated by "NotReady".

```
$ kubectl get nodes
NAME                 STATUS      ROLES           AGE     VERSION
kube-control-plane   Ready       control-plane   4m27s   v1.31.1
kube-worker-1        NotReady    <none>          48s     v1.31.1
```

The conditions section of the worker node indicates an issue with the kubelet.

```
$ kubectl describe node kube-worker-1
...
Conditions:
  Type             Status    LastHeartbeatTime                 LastTransitionTime                Reason              Message
  ----             ------    -----------------                 ------------------                ------              -------
  MemoryPressure   Unknown   Thu, 29 May 2025 11:58:18 +0000   Thu, 29 May 2025 11:58:59 +0000   NodeStatusUnknown   Kubelet stopped posting node status.
  DiskPressure     Unknown   Thu, 29 May 2025 11:58:18 +0000   Thu, 29 May 2025 11:58:59 +0000   NodeStatusUnknown   Kubelet stopped posting node status.
  PIDPressure      Unknown   Thu, 29 May 2025 11:58:18 +0000   Thu, 29 May 2025 11:58:59 +0000   NodeStatusUnknown   Kubelet stopped posting node status.
  Ready            Unknown   Thu, 29 May 2025 11:58:18 +0000   Thu, 29 May 2025 11:58:59 +0000   NodeStatusUnknown   Kubelet stopped posting node status.
```

Exit the control plane node.

```
$ exit
```

Open an interactive shell to the worker node.

```
$ vagrant ssh kube-worker-1
```

The `journalctl` can provide useful information about the service. It looks like the client CA file is misconfigured.

```
$ sudo journalctl -u kubelet.service
Jan 21 23:46:49 kube-worker-1 kubelet[6560]: F0121 23:46:49.183791    6560 server.go:257] unable to load client CA file /etc/kubernetes/pki/non-existent-ca.crt: open /etc/kubernetes/pki/non-existent-ca.crt: no such file or directory
```

The configuration file can be discovered by rendering the status of the service. The drop-in value points to `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`.

```
$ systemctl status kubelet.service -l
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: activating (auto-restart) (Result: exit-code) since Fri 2021-01-22 00:01:31 UTC; 9s ago
     Docs: https://kubernetes.io/docs/home/
  Process: 9644 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=255)
 Main PID: 9644 (code=exited, status=255)
 
$ sudo cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```

The value of the environment variable `KUBELET_CONFIG_ARGS` is `--config=/var/lib/kubelet/config.yaml`. Upon inspection of the file, the value of `authentication.x509.clientCAFile` is `/etc/kubernetes/pki/non-existent-ca.crt`. This file does not exist. Let's fix the file by changing to `clientCAFile: /etc/kubernetes/pki/ca.crt`.

```
$ sudo ls /etc/kubernetes/pki
ca.crt
$ sudo vim /var/lib/kubelet/config.yaml
```

Now, let's restart the service.

```
$ sudo systemctl daemon-reload
$ sudo systemctl restart kubelet
```

After waiting a couple of seconds, the worker node should transition into the "Ready" status.

```
$ kubectl get nodes
NAME                 STATUS   ROLES           AGE     VERSION
kube-control-plane   Ready    control-plane   4m27s   v1.31.1
kube-worker-1        Ready    <none>          48s     v1.31.1
```
