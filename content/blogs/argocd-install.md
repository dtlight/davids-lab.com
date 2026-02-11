---
title: "Setting up ArgoCD"
date: 2026-05-28T09:09:00+00:00
layout: "posts"
draft: false
tags: ["argocd", "traefik", "helm"]
categories: ["argocd"]
hiddenInHomeList: true
---

# ArgoCD SetUp

## Prepare Helm
Add ArgoCD Helm Repository
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

Inside argocd-values.yaml

```bash
server:
  service:
    type: NodePort
    ports:
      http: 80
configs:
  params:
    server.insecure: "true" # ArgoCD needs to run with TLS disabled since Traefik will handle it.
controller:
  resources:
    requests:
      cpu: 250m
      memory: 256Mi
```

Because I will use argocd to deploy traefik, i'll have to access argocd via node ip or port forward. I'll configure a nicer way once I have my ingress up and running.

## Install ArgoCD

```
kubectl create namespace argocd
helm install argocd argo/argo-cd \
  --namespace argocd \
  --values argocd-values.yaml \
  --version 5.46.8 \
  --timeout 5m \
  --atomic \
  --wait
```

Why version 5.46.8 when much newer is available? I will upgrade argocd in a future blog to version 3.., things will certainly break and so I'll explain how I fix them. The wait parameter is to ensure that Kubernetes confirms resources are Ready.. meaning deployement replicas = desired, service endpoints are populated and configmaps and secrets are created.

## Access ArgoCD

port forward
```bash
kubectl -n argocd port-forward svc/argocd-server 8080:80
```

get password
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d ; echo
```

Check to see all pods are in ready state
```bash
david@cp1:~/homelab/argocd $ k get po -n argocd -o wide
NAME                                                READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
argocd-application-controller-0                     1/1     Running   0          33m   10.244.3.12   wn2    <none>           <none>
argocd-applicationset-controller-84d89c598f-92b22   1/1     Running   0          33m   10.244.3.10   wn2    <none>           <none>
argocd-dex-server-5cf45b5db7-29c2m                  1/1     Running   0          33m   10.244.3.11   wn2    <none>           <none>
argocd-notifications-controller-7974665955-6hzgj    1/1     Running   0          33m   10.244.3.13   wn2    <none>           <none>
argocd-redis-76bf9bb495-8c2bb                       1/1     Running   0          33m   10.244.3.8    wn2    <none>           <none>
argocd-repo-server-6f54586b55-p6scq                 1/1     Running   0          32m   10.244.3.9    wn2    <none>           <none>
argocd-server-68745bc9d9-vvb9f                      1/1     Running   0          33m   10.244.2.2    wn1    <none>           <none>
```

SSH to cp1 (use explicit ip if you haven't configured in ssh config) then port forward 
```bash
ssh david@cp1 'kubectl -n argocd port-forward svc/argocd-server 8080:80'
```

In local browser `http://localhost:8080` will give you the argo ui with username `admin` and password from the `get password` step above.

# Installing Traefik

Doing this the GitOps way means everything should be defined in Git and deployed via ArgocD. I will 

* Add my manifests to [argocd/apps/](https://github.com/dtlight/homelab/argocd/apps/)

* Deploy Traefik using an ArgoCD Application

* Create an IngressRoute for ArgoCD

## Install Traefik via ArgoCD

In apps/traefik.yaml

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: traefik
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://traefik.github.io/charts
    chart: traefik
    targetRevision: "*"
    helm:
      values: |
        service:
          type: LoadBalancer
  destination:
    server: https://kubernetes.default.svc
    namespace: traefik
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

This is then applied 

## Create IngressRoute for ArgoCD
Create argocd-ingressroute.yaml in the argocd directory of my homelab git repo

```yaml
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: argocd-server
  namespace: argocd
spec:
  entryPoints:
    - websecure
  routes:
    - kind: Rule
      match: Host(`argocd.davids-lab.com`)
      priority: 10
      services:
        - name: argocd-server
          port: 80
    - kind: Rule
      match: Host(`argocd.davids-lab.com`) && Headers(`Content-Type`, `application/grpc`)
      priority: 11
      services:
        - name: argocd-server
          port: 80
          scheme: h2c
  tls:
    certResolver: default  # TODO Configure cert-manager or Let's Encrypt
```
The second route with h2c scheme handles gRPC traffic for the ArgoCD CLI.

## Apply via ArgoCD
Commit the IngressRoute to Git and create an ArgoCD Application to sync it:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: argocd-ingress
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/yourusername/your-repo
    path: argocd
    targetRevision: main
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```
I'll run kubectl apply locally once to bootstrap the ArgoCD Application after which ArgoCD takes over. From this point forward, ArgoCD watches Git and automatically syncs changes.


## Alternative: App of Apps Pattern
For a more "GitOps native" approach, I can use an alternative App of Apps pattern where ArgoCD manages its own Applications:

* Create a "root" Application that points to a directory containing all your Application manifests

* `kubectl apply` only the root Application once
​
* Add new Applications by committing them to that directory - ArgoCD creates them automatically

This means I only ever run kubectl apply once for the root app, and everything else (including new Applications) is managed through Git.




