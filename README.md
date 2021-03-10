# Running ArgoCd on a vSPhere7 with K8s supervisor cluster

* Create a namespace `argocd` in supervisor cluster, using the vCenter interface
* Attach storage, configure access and resources limits to the `argocd` namespace, using the vCenter interface (if needed)
* Install ArgoCD on the supervisor cluster.
    * Login to the SUpervisor control plane. 
    * `curl https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml -o argo-install.yaml`
    * modify the file to install service type `LoadBalancer` for argocd-server
    * `kubectl apply -n argocd argocd-install.yaml`
* Login to argocd SVC_IPADDRESS
    * login is admin
    * password is argocd-server podname
* `argocd login admin`
* `argocd account update-password`
* `argocd cluster list`
* `kubectl get secrets -n demo1 workload-vsphere-tkg1-kubeconfig -o json|jq -r '.data.value'|base64 -d > /tmp/workload-cluster.cfg`
* `argocd cluster add --kubeconfig /tmp/workload-cluster.cfg workload-vsphere-tkg1-admin@workload-vsphere-tkg1`
* `argocd repo add https://github.com/papivot/kube-shell --name kube-shell --insecure-skip-server-verification --type git`
* Create a yaml to create a new project

```
metadata:
  name: navlab
  namespace: argocd
spec:
  clusterResourceWhitelist:
  - group: '*'
    kind: '*'
  destinations:
  - namespace: '*'
    server: https://192.168.10.162:6443
  orphanedResources: {}
  sourceRepos:
  - https://github.com/papivot/kube-shell
```
* `argocd proj create navlab -f /tmp/proj`
* Create and deploy applications

```yaml
spec:
  destination:
    namespace: kube-shell
    server: https://192.168.10.163:6443
  project: default
  source:
    path: http
    repoURL: https://github.com/papivot/kube-shell.git
    targetRevision: HEAD
  syncPolicy:
    automated: {}
    syncOptions:
    - CreateNamespace=true
```

```yaml
spec:
  destination:
    namespace: kube-shell
    server: https://kubernetes.default.svc
  project: default
  source:
    path: http
    repoURL: https://github.com/papivot/kube-shell.git
    targetRevision: HEAD
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      - CreateNamespace=true
```
