# Running ArgoCd on a vSPhere7 with K8s supervisor cluster

* Create a namespace `argocd` in supervisor cluster, using the vCenter interface
* Attach storage, configure access and resources limits to the `argocd` namespace, using the vCenter interface (if needed)
* Install ArgoCD on the supervisor cluster.
    * Login to the SUpervisor control plane. 
    * `kubectl apply -f https://raw.githubusercontent.com/papivot/argocd-on-supervisor/main/install-argocd.yaml -n argocd`
* Get the argocd-server service IP address and podname

```shell
kubectl get pod -n argocd |grep argocd-server|awk '{print $1}'
kubectl get svc -n argocd argocd-server -o json|jq -r ".status.loadBalancer.ingress[].ip"
```
* Login to argocd SVC_IPADDRESS
    * login is admin
    * password is argocd-server podname
* `argocd login SVC_IPADDRESS --username admin --password POD_NAME`
* `argocd account update-password`
* `argocd cluster list`
* `kubectl get secrets -n demo1 workload-vsphere-tkg1-kubeconfig -o json|jq -r '.data.value'|base64 -d > /tmp/workload-cluster.cfg`
* `argocd cluster add --kubeconfig /tmp/workload-cluster.cfg workload-vsphere-tkg1-admin@workload-vsphere-tkg1`
* `argocd repo add https://github.com/papivot/kube-shell --name kube-shell --insecure-skip-server-verification --type git`
* Create a yaml to create a new project

```yaml
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
* `argocd app create --name kube-shell --project navlab --sync-policy auto-prune --repo https://github.com/papivot/kube-shell --path http --dest-server https://192.168.10.162:6443`
