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
* `argocd account update-password`
* `argocd cluster list`
* `argocd cluster add workload-cluster`
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
