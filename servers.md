# Servers

> See also: [network.md](./network.md) | [kubernetes.md](./kubernetes.md) | [storage.md](./storage.md)

---

## Physical servers

### Server 1 (active)

| Property   | Value                                          |
|------------|------------------------------------------------|
| IP         | `192.168.1.100`                                |
| Hypervisor | Proxmox VE                                     |
| Role       | All VMs, LXC containers, k8s cluster, storage  |
| Status     | Production                                     |

### Server 2 (planned)

| Property   | Value                                                       |
|------------|-------------------------------------------------------------|
| Hypervisor | Proxmox VE (to be configured)                               |
| Role       | Backups + additional k8s worker nodes                       |
| Status     | Not yet configured                                          |
| Notes      | Will join Proxmox cluster for live migration and VM HA      |

---

## Proxmox inventory

```mermaid
graph TD
    PVE[Proxmox VE<br>192.168.1.100]

    PVE --> NFS[VM — NFS Server<br>192.168.1.110]
    PVE --> Static[VM — Static / Misc<br>192.168.1.130]
    PVE --> Build[LXC — Build Server<br>192.168.1.131]
    PVE --> CP[VM — k8s Control Plane<br>192.168.1.120]
    PVE --> W1[VM — k8s Worker Node 1<br>192.168.1.121]
    PVE --> W2[VM — k8s Worker Node 2<br>192.168.1.122]
```

---

## VM / LXC details

### NFS Server VM

| Property  | Value                                                          |
|-----------|----------------------------------------------------------------|
| IP        | `192.168.1.110`                                                |
| Type      | VM                                                             |
| Purpose   | Centralised persistent storage for all VMs and k8s workloads  |
| Protocol  | NFS                                                            |
| Consumers | k8s PVCs (all stateful workloads), direct app mounts           |
| Notes     | Single point of failure for all stateful data. Highest backup priority. See [storage.md](./storage.md). |

---

### Static / Misc VM

| Property    | Value                                                         |
|-------------|---------------------------------------------------------------|
| IP          | `192.168.1.130`                                               |
| Type        | VM                                                            |
| Purpose     | Hosts Docker containers for non-k8s workloads + Minecraft     |
| Runtime     | Docker                                                        |
| Integration | Apps wired into k8s via manual Endpoint objects               |
| Open port   | `:25565` TCP — forwarded directly from router (Minecraft only)|
| Notes       | This is supposed to be an easy way to deploy simple things that don't require a full k8s architecture |

#### Deployed workloads

| App         | Stack                            | Domain                      | k8s Endpoint |
|-------------|----------------------------------|-----------------------------|--------------|
| Minecraft   | Minecraft Java Edition           | —                           | No (raw TCP port forward) |

---

### Build Server LXC

| Property  | Value                                                            |
|-----------|------------------------------------------------------------------|
| IP        | `192.168.1.131`                                                  |
| Type      | LXC container                                                    |
| Purpose   | CI/CD — build Docker images and deploy to k8s                    |
| Runtime   | GitHub Actions self-hosted runners                               |
| Deploys to| Kubernetes cluster via `kubectl` / Helm                          |
| Pushes to | `registry.r16a.cloud` (Docker Registry in k8s)                   |

---

### k8s Control Plane VM

| Property   | Value                                                             |
|------------|-------------------------------------------------------------------|
| IP         | `192.168.1.120`                                                   |
| Type       | VM                                                                |
| Purpose    | Kubernetes control plane                                          |
| Components | kube-apiserver, etcd, kube-scheduler, kube-controller-manager     |
| Notes      | Single control plane — no HA yet... maybe one day if AI stops eating RAM lol |

---

### k8s Worker Node 1 VM

| Property   | Value                                     |
|------------|-------------------------------------------|
| IP         | `192.168.1.121`                           |
| Type       | VM                                        |
| Purpose    | Kubernetes worker                         |
| Components | kubelet, kube-proxy, container runtime    |

---

### k8s Worker Node 2 VM

| Property   | Value                                     |
|------------|-------------------------------------------|
| IP         | `192.168.1.122`                           |
| Type       | VM                                        |
| Purpose    | Kubernetes worker                         |
| Components | kubelet, kube-proxy, container runtime    |
