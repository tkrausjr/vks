# vCluster on VKS — `goad-test-02a`

Running [vCluster](https://www.vcluster.com/) (virtual Kubernetes clusters) on the
VKS/TKG cluster **`goad-test-02a`**. A vCluster is a normal workload — a StatefulSet
plus a syncer pod — that runs inside one host namespace and presents its own API
server. Your workloads' **pods get synced down** onto the real nodes; everything else
(Deployments, Services, ConfigMaps, RBAC, CRDs) lives only in the virtual cluster. You
get what looks like full cluster-admin on a "real" cluster without touching the host.

## Feasibility on `goad-test-02a`

The cluster is Kubernetes **v1.35.5+vmware.1** (1 control-plane + 3 amd64 workers,
containerd). The four things that decide feasibility are all green:

| Requirement | `goad-test-02a` status | Verdict |
|---|---|---|
| **Storage** for the virtual control plane's datastore | `vsan-esa-default-policy-raid5` (vSAN CSI) is default, supports expansion | ✅ |
| **Expose** the vCluster API server | Avi/AKO LoadBalancer works — hands out `10.47.7.x` externals | ✅ |
| **Pod Security Admission** | System namespaces are `privileged`; user namespaces have no enforce label → no restriction on the syncer / synced pods | ✅ |
| **CNI / network policy traps** | Antrea is the CNI. The `baseline-default-deny` ACNP is scoped **only to `music-store`**, not cluster-wide | ✅ |

## Steps

### 1. Install the CLI (on your Mac)

```bash
brew install loft-sh/tap/vcluster
# or:
# curl -L -o vcluster "https://github.com/loft-sh/vcluster/releases/latest/download/vcluster-darwin-arm64" \
#   && install -m755 vcluster /usr/local/bin/
vcluster version
```

The CLI drives Helm internally, so you don't need Helm separately for the basic path.

### 2. Namespace for the vCluster

Keep it out of `music-store` so the default-deny ACNP doesn't apply:

```bash
kubectl create namespace vcluster-demo
```

### 3. Review `vcluster.yaml`

The included [`vcluster.yaml`](./vcluster.yaml) pins the vSAN storage class and exposes
the API server through Avi:

```yaml
controlPlane:
  statefulSet:
    persistence:
      volumeClaim:
        enabled: true
        storageClass: vsan-esa-default-policy-raid5
        size: 5Gi
  service:
    spec:
      type: LoadBalancer        # Avi/AKO assigns a 10.47.7.x VIP
```

### 4. Create it

```bash
vcluster create demo -n vcluster-demo -f vcluster.yaml
```

The CLI waits for the pod, grabs the Avi LoadBalancer VIP, writes a kubeconfig context,
and **auto-connects** you to it.

### 5. Use it

You're now talking to the virtual cluster:

```bash
kubectl get ns              # virtual cluster's namespaces, not the host's
kubectl create ns team-a
kubectl -n team-a run web --image=nginx
# that pod appears on a real node, inside host ns vcluster-demo, renamed
```

### 6. Switch back / tear down

```bash
vcluster disconnect                     # back to goad-test-02a context
vcluster delete demo -n vcluster-demo   # removes everything + the PVC
```

## Cluster-specific gotchas

- **Expose = Avi VIP.** `type: LoadBalancer` is the clean path since AKO already works.
  To avoid burning a VIP per vCluster, use a Gateway API HTTPRoute on `avi-lb`, or
  `vcluster connect` port-forward for quick tests.
- **Ties into the WAF/microseg demo.** A vCluster gives tenants "their own cluster"
  while the host's Antrea ACNPs and the Avi WAF at ingress still govern real traffic — a
  clean multi-tenant isolation story on top of the segmentation story.
- **If `baseline-default-deny` later goes cluster-wide**, add an Antrea allow rule for
  the `vcluster-demo` namespace (ingress to the API-server service and syncer), or the
  control plane won't be reachable. Today it's music-store-only, so no action needed.
- **Distro:** the default works fine. For vanilla upstream etcd-backed K8s inside
  (matching VKS conformance), uncomment `controlPlane.distro.k8s.enabled: true` in
  `vcluster.yaml`.
- **Node arch:** workers are amd64. vCluster's own images are multi-arch, so no issue.
