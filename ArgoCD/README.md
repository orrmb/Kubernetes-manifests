## Install ArgoCD

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

```

```bash
Linux@Kubernetes# kubectl get namespaces
NAME              STATUS   AGE
argocd            Active   12s
default           Active   47h
kube-node-lease   Active   47h
kube-public       Active   47h
kube-system       Active   47h
```

The pods run:

```bash
Linux@Kubernetes# kubectl get pods -n argocd
NAME                                               READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                    1/1     Running   0          3m26s
argocd-applicationset-controller-7d7c89f5f-4f2qf   1/1     Running   0          3m26s
argocd-dex-server-77b75c4cff-87jl5                 1/1     Running   0          3m26s
argocd-notifications-controller-64775dbfc4-c6chx   1/1     Running   0          3m26s
argocd-redis-7d85c5d7b8-nqbk6                      1/1     Running   0          3m26s
argocd-repo-server-75bf446df7-hn2vm                1/1     Running   0          3m26s
argocd-server-7bb58d96d7-9qf6b                     1/1     Running   0          3m26s

```

Access to Argo dashboard:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443

```

The user name is admin and the secret we need to get from the secrets:

```bash
Linux@Kubernetes# kubectl get secrets -n argocd argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 --decode

```

![alt text](ArgoCD/Pic/image.png)

There are two way we can create app in the ArgoCD:

1. With yaml file
   For example:
   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
   name: example-app
   namespace: argocd
   spec:
   project: default
   source:
   repoURL: https://github.com/marcel-dempers/docker-development-youtube-series.git
   targetRevision: HEAD
   path: argo/example-app
   directory:
   recurse: true
   destination:
   server: https://kubernetes.default.svc
   namespace: example-app
   syncPolicy:
   automated:
   prune: false
   selfHeal: false
   ```
   Create app for ArgoCD:

```bash
Linux@Kubernetes# kubectl -n argocd apply -f app.yaml

application.argoproj.io/example-app created
```

After why apply the Yaml, the app create for us in ArgoCD:
![alt text]()
