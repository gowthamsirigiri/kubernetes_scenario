
```markdown
# Incident Report: Kubelet Service Down on Worker Node

## 1. Overview
* **Incident:** Kubernetes node agent (`kubelet`) became inactive (`dead`) on `node01`, causing the node to go unmonitored and pods to stop syncing.
* **Environment:** On-premises / Cloud Kubernetes Cluster (v1.37.1)
* **Affected Component:** `node01` (Worker Node)

---

## 2. Detection & Symptoms
Initial checks from the control plane showed standard node status, but internal checks revealed connectivity issues and service degradation:

```bash
root@controlplane:~$ kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   31d   v1.37.1
node01         Ready    <none>          31d   v1.37.1

```

Checking the service status directly on `node01` via SSH confirmed the failure:

```bash
root@node01:~$ systemctl status kubelet
○ kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
     Active: inactive (dead) since Thu 2026-11-05 15:49:34 UTC; 1min 20s ago
   Main PID: 721 (code=exited, status=0/SUCCESS)

```

---

## 3. Root Cause Analysis

The systemd logs indicated that the `kubelet` service had cleanly exited/stopped without triggering an automatic recovery loop.

---

## 4. Remediation Steps

1. Logged into the affected worker node (`node01`).
2. Started and enabled the `kubelet` service using `systemd`:
```bash
sudo systemctl start kubelet
sudo systemctl enable kubelet

```


3. Verified the service was actively running and healthy:
```bash
systemctl status kubelet

```


*Output:*
```text
● kubelet.service - kubelet: The Kubernetes Node Agent
     Active: active (running) since Thu 2026-11-05 15:51:49 UTC

```



---

## 5. Verification

Switched back to the control plane to ensure the node successfully re-established communication and reported a `Ready` state:

```bash
root@controlplane:~$ kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   31d   v1.37.1
node01         Ready    <none>          31d   v1.37.1

```

```

```
