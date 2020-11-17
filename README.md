# argocd-on-supervisor
* Create namespace argocd in supervisor cluster
* attach storage
* `curl https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml -o argo-install.yaml`
* modify the file to install servicetype LoadBalancer for argocd-server
* `kubectl apply -n argocd argocd-install.yaml`
* Login to argocd SVC_IPADDRESS
    login is admin/password is argocd-server podname
* `argocd accoutn update-password`
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
