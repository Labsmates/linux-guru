# ☸️ OpenShift & Kubernetes - Guide Complet

Maîtriser l'orchestration de containers en production avec Kubernetes et OpenShift.

---

## 📚 Introduction

**Kubernetes (K8s) :**
- Orchestration de containers open-source (Google)
- Standard de l'industrie
- Vanilla Kubernetes

**OpenShift :**
- Distribution Kubernetes entreprise (Red Hat)
- Kubernetes + outils supplémentaires
- Routes, Build configs, Security enhanced
- Interface web complète

**Ce guide couvre les deux** (commandes compatibles sauf mention contraire).

---

## 🔧 kubectl - CLI Kubernetes

### Installation & Configuration

```bash
# Installer kubectl (Linux)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Vérifier version
kubectl version --client

# Configuration cluster
kubectl config view

# Lister contexts
kubectl config get-contexts

# Changer context
kubectl config use-context my-cluster

# Définir namespace par défaut
kubectl config set-context --current --namespace=myapp
```

### oc - OpenShift CLI

```bash
# Installer oc (OpenShift CLI)
# Télécharger depuis: https://mirror.openshift.com/pub/openshift-v4/clients/ocp/

# Login OpenShift
oc login https://api.cluster.example.com:6443
# Demande username + password

# Login avec token
oc login --token=sha256~xxxxx --server=https://api.cluster.example.com:6443

# Vérifier user courant
oc whoami

# Projet courant
oc project

# Changer projet
oc project myapp
```

---

## 🏗️ Pods - Unité de Base

### kubectl run - Créer Pod

```bash
# Créer pod simple
kubectl run nginx --image=nginx

# Avec commande custom
kubectl run busybox --image=busybox --command -- sleep 3600

# Dry-run (générer YAML sans créer)
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

# Avec variables d'environnement
kubectl run nginx --image=nginx --env="ENV=prod"

# Avec port
kubectl run nginx --image=nginx --port=80

# Avec labels
kubectl run nginx --image=nginx --labels="app=web,tier=frontend"
```

### kubectl get - Lister Ressources

```bash
# Lister pods
kubectl get pods
kubectl get pods -o wide  # Plus de détails (IP, Node)

# Tous namespaces
kubectl get pods --all-namespaces
kubectl get pods -A

# Format custom
kubectl get pods -o json
kubectl get pods -o yaml
kubectl get pods -o name  # Seulement noms

# Avec labels
kubectl get pods --show-labels
kubectl get pods -l app=nginx

# Trier
kubectl get pods --sort-by=.metadata.creationTimestamp

# Watch (temps réel)
kubectl get pods -w
```

### kubectl describe - Détails Ressource

```bash
# Describe pod
kubectl describe pod nginx

# Voir events
kubectl describe pod nginx | grep -A10 Events

# Describe service
kubectl describe svc nginx

# Describe node
kubectl describe node worker-1
```

### kubectl logs - Logs Pod

```bash
# Logs pod
kubectl logs nginx

# Follow (temps réel)
kubectl logs -f nginx

# Container spécifique (si multi-container pod)
kubectl logs nginx -c sidecar

# Dernières 100 lignes
kubectl logs --tail=100 nginx

# Depuis timestamp
kubectl logs --since=1h nginx
kubectl logs --since=2024-02-21T10:00:00Z nginx

# Logs de tous les pods avec label
kubectl logs -l app=nginx

# Previous container (si crashé)
kubectl logs nginx --previous
```

### kubectl exec - Exécuter Commande

```bash
# Shell interactif
kubectl exec -it nginx -- /bin/bash

# Commande simple
kubectl exec nginx -- ls /usr/share/nginx/html

# Container spécifique (multi-container pod)
kubectl exec -it nginx -c sidecar -- /bin/sh

# Copier fichiers
kubectl cp nginx:/etc/nginx/nginx.conf ./nginx.conf
kubectl cp ./index.html nginx:/usr/share/nginx/html/
```

### kubectl delete - Supprimer Ressource

```bash
# Supprimer pod
kubectl delete pod nginx

# Force delete
kubectl delete pod nginx --force --grace-period=0

# Supprimer par label
kubectl delete pods -l app=nginx

# Supprimer tous pods dans namespace
kubectl delete pods --all

# Depuis fichier YAML
kubectl delete -f pod.yaml
```

---

## 🚀 Deployments - Production Workloads

### kubectl create deployment

```bash
# Créer deployment
kubectl create deployment nginx --image=nginx

# Avec replicas
kubectl create deployment nginx --image=nginx --replicas=3

# Générer YAML
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
```

### Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            CPU: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        env:
        - name: ENV
          value: "production"
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html
        persistentVolumeClaim:
          claimName: html-pvc
```

### kubectl apply - Appliquer Configuration

```bash
# Appliquer YAML
kubectl apply -f deployment.yaml

# Appliquer dossier complet
kubectl apply -f ./manifests/

# Appliquer depuis URL
kubectl apply -f https://example.com/deployment.yaml

# Dry-run
kubectl apply -f deployment.yaml --dry-run=client

# Server-side apply
kubectl apply -f deployment.yaml --server-side
```

### kubectl scale - Scaling

```bash
# Scale deployment
kubectl scale deployment nginx --replicas=5

# Autoscaling (HPA)
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=80

# Voir HPA
kubectl get hpa
```

### kubectl rollout - Gestion Déploiements

```bash
# Status rollout
kubectl rollout status deployment/nginx

# Historique
kubectl rollout history deployment/nginx

# Rollback
kubectl rollout undo deployment/nginx

# Rollback vers revision spécifique
kubectl rollout undo deployment/nginx --to-revision=2

# Pause rollout
kubectl rollout pause deployment/nginx

# Resume rollout
kubectl rollout resume deployment/nginx

# Restart deployment (recreate pods)
kubectl rollout restart deployment/nginx
```

### kubectl set - Modifier Ressource

```bash
# Changer image
kubectl set image deployment/nginx nginx=nginx:1.26

# Changer resources
kubectl set resources deployment nginx --limits=cpu=500m,memory=512Mi

# Variables d'environnement
kubectl set env deployment/nginx ENV=production
```

---

## 🌐 Services - Networking

### Types de Services

```yaml
# ClusterIP (interne seulement)
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80

---
# NodePort (accès externe via port node)
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080  # 30000-32767

---
# LoadBalancer (cloud provider)
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

### kubectl expose - Créer Service

```bash
# Expose deployment
kubectl expose deployment nginx --port=80 --type=ClusterIP

# NodePort
kubectl expose deployment nginx --port=80 --type=NodePort

# LoadBalancer
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Avec nom custom
kubectl expose deployment nginx --port=80 --name=web-service
```

### kubectl port-forward - Port Forwarding

```bash
# Forward local port vers pod
kubectl port-forward pod/nginx 8080:80
# Accès: http://localhost:8080

# Forward vers service
kubectl port-forward svc/nginx 8080:80

# Forward vers deployment
kubectl port-forward deployment/nginx 8080:80

# Écouter sur toutes interfaces
kubectl port-forward --address 0.0.0.0 pod/nginx 8080:80
```

---

## 📊 ConfigMaps & Secrets

### ConfigMap

```bash
# Créer depuis literal
kubectl create configmap app-config \
  --from-literal=ENV=production \
  --from-literal=LOG_LEVEL=debug

# Créer depuis fichier
kubectl create configmap app-config --from-file=config.properties

# Créer depuis dossier
kubectl create configmap app-config --from-file=./configs/

# YAML
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  ENV: "production"
  LOG_LEVEL: "debug"
  config.properties: |
    server.port=8080
    server.host=0.0.0.0
```

**Utiliser ConfigMap dans Pod :**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: myapp:1.0
    # Option 1: Variables d'environnement
    env:
    - name: ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: ENV
    # Option 2: Toutes les clés comme env vars
    envFrom:
    - configMapRef:
        name: app-config
    # Option 3: Volume mount
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: app-config
```

### Secrets

```bash
# Créer secret
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123

# Depuis fichiers
kubectl create secret generic ssl-cert \
  --from-file=tls.crt \
  --from-file=tls.key

# TLS secret
kubectl create secret tls tls-secret \
  --cert=tls.crt \
  --key=tls.key

# Docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass \
  --docker-email=user@example.com

# Voir secret (encodé base64)
kubectl get secret db-secret -o yaml

# Décoder
kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 -d
```

**Utiliser Secret :**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    volumeMounts:
    - name: secrets
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secrets
    secret:
      secretName: db-secret
  # Pull depuis registry privé
  imagePullSecrets:
  - name: regcred
```

---

## 💾 Persistent Volumes (PV/PVC)

### PersistentVolume (PV)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce     # RWO: un seul node
  # - ReadOnlyMany    # ROX: lecture multiple nodes
  # - ReadWriteMany   # RWX: lecture/écriture multiple nodes
  persistentVolumeReclaimPolicy: Retain  # Retain|Delete|Recycle
  storageClassName: standard
  hostPath:
    path: /mnt/data
  # Ou NFS:
  # nfs:
  #   server: nfs-server.example.com
  #   path: /exported/path
```

### PersistentVolumeClaim (PVC)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-data
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```

**Utiliser PVC dans Pod :**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: pvc-data
```

**Commandes PV/PVC :**
```bash
# Lister PV
kubectl get pv

# Lister PVC
kubectl get pvc

# Describe PVC
kubectl describe pvc pvc-data

# Supprimer PVC
kubectl delete pvc pvc-data
```

---

## 🌍 Ingress & Routes

### Ingress (Kubernetes)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
  # TLS
  tls:
  - hosts:
    - app.example.com
    secretName: tls-secret
```

**Créer Ingress :**
```bash
# Depuis YAML
kubectl apply -f ingress.yaml

# Voir ingress
kubectl get ingress

# Describe
kubectl describe ingress app-ingress
```

### Routes (OpenShift)

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: app-route
spec:
  host: app.apps.cluster.example.com
  to:
    kind: Service
    name: app-service
  port:
    targetPort: 80
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
```

**Commandes Routes (OpenShift) :**
```bash
# Créer route
oc expose service app-service

# Avec hostname custom
oc expose service app-service --hostname=app.example.com

# HTTPS (edge termination)
oc create route edge app-route --service=app-service

# Lister routes
oc get routes

# Voir URL
oc get route app-route -o jsonpath='{.spec.host}'
```

---

## 🔐 RBAC - Sécurité

### ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
```

### Role & RoleBinding

```yaml
# Role (namespace-scoped)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: ServiceAccount
  name: app-sa
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRole & ClusterRoleBinding

```yaml
# ClusterRole (cluster-wide)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin-custom
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]

---
# ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-binding
subjects:
- kind: User
  name: john@example.com
roleRef:
  kind: ClusterRole
  name: cluster-admin-custom
  apiGroup: rbac.authorization.k8s.io
```

**Commandes RBAC :**
```bash
# Créer ServiceAccount
kubectl create serviceaccount app-sa

# Vérifier permissions
kubectl auth can-i create pods --as=system:serviceaccount:default:app-sa

# Lister roles
kubectl get roles
kubectl get clusterroles

# Describe role
kubectl describe role pod-reader
```

---

## 🎯 Namespaces - Organisation

```bash
# Lister namespaces
kubectl get namespaces
kubectl get ns

# Créer namespace
kubectl create namespace dev

# YAML
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  labels:
    env: development

# Supprimer namespace
kubectl delete namespace dev

# Tout opérer dans namespace spécifique
kubectl get pods -n dev
kubectl apply -f deployment.yaml -n dev

# Définir namespace par défaut
kubectl config set-context --current --namespace=dev
```

---

## 📈 Monitoring & Debugging

### kubectl top - Resource Usage

```bash
# Usage nodes
kubectl top nodes

# Usage pods
kubectl top pods

# Tous namespaces
kubectl top pods --all-namespaces

# Trier par CPU
kubectl top pods --sort-by=cpu

# Trier par RAM
kubectl top pods --sort-by=memory
```

### kubectl get events

```bash
# Events cluster
kubectl get events

# Events namespace
kubectl get events -n dev

# Trier par timestamp
kubectl get events --sort-by=.metadata.creationTimestamp

# Watch events
kubectl get events -w
```

### kubectl debug - Debugging

```bash
# Debug pod avec ephemeral container
kubectl debug nginx -it --image=busybox

# Debug node
kubectl debug node/worker-1 -it --image=busybox

# Copy pod avec debugger
kubectl debug nginx --copy-to=nginx-debug --container=nginx --image=nginx:debug
```

### Troubleshooting Commands

```bash
# Voir pourquoi pod ne démarre pas
kubectl describe pod failing-pod | grep -A10 Events

# Logs de tous containers d'un pod
kubectl logs failing-pod --all-containers=true

# Previous container logs (si crashé)
kubectl logs failing-pod --previous

# Exec dans container qui crash
kubectl run -it debug --image=busybox --restart=Never -- sh

# Port-forward pour debug
kubectl port-forward svc/app 8080:80

# Proxy API server
kubectl proxy
# Accès: http://localhost:8001/api/v1/namespaces/default/pods

# Exécuter commande dans tous pods
kubectl get pods -o name | xargs -I {} kubectl exec {} -- date
```

---

## 🔨 OpenShift Spécifiques

### oc new-app - Créer Application

```bash
# Depuis image Docker
oc new-app nginx

# Depuis Git repo
oc new-app https://github.com/user/myapp.git

# Depuis Dockerfile
oc new-app https://github.com/user/myapp.git --strategy=docker

# Source-to-Image (S2I)
oc new-app php:7.4~https://github.com/user/php-app.git

# Avec variables d'environnement
oc new-app nginx -e ENV=production

# Avec nom custom
oc new-app nginx --name=webserver
```

### BuildConfig & ImageStream

```bash
# Lister builds
oc get builds

# Démarrer build
oc start-build myapp

# Annuler build
oc cancel-build myapp-1

# Logs build
oc logs -f bc/myapp

# Lister ImageStreams
oc get imagestreams
oc get is

# Tag image
oc tag myapp:latest myapp:v1.0
```

### DeploymentConfig (OpenShift)

```bash
# Lister DC
oc get dc

# Rollout latest
oc rollout latest dc/myapp

# Rollback
oc rollback myapp

# Scale
oc scale dc/myapp --replicas=3
```

### Templates

```bash
# Lister templates
oc get templates -n openshift

# Process template
oc process -f template.yaml | oc apply -f -

# Avec parameters
oc process -f template.yaml \
  -p NAME=myapp \
  -p REPLICAS=3 | oc apply -f -
```

---

## 🛠️ Helm - Package Manager

### Installation

```bash
# Installer Helm (Linux)
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Vérifier version
helm version
```

### Helm Commands

```bash
# Ajouter repo
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami

# Mettre à jour repos
helm repo update

# Rechercher chart
helm search repo nginx

# Voir valeurs chart
helm show values bitnami/nginx

# Installer chart
helm install my-nginx bitnami/nginx

# Avec values custom
helm install my-nginx bitnami/nginx -f values.yaml

# Avec override
helm install my-nginx bitnami/nginx --set replicaCount=3

# Lister releases
helm list
helm list --all-namespaces

# Status release
helm status my-nginx

# Upgrade release
helm upgrade my-nginx bitnami/nginx

# Upgrade avec nouveau values
helm upgrade my-nginx bitnami/nginx -f new-values.yaml

# Rollback
helm rollback my-nginx

# Rollback vers revision
helm rollback my-nginx 2

# Historique
helm history my-nginx

# Uninstall
helm uninstall my-nginx

# Dry-run
helm install my-nginx bitnami/nginx --dry-run --debug
```

### Créer Chart

```bash
# Créer structure chart
helm create mychart

# Structure:
# mychart/
#   Chart.yaml       # Metadata
#   values.yaml      # Default values
#   charts/          # Dependencies
#   templates/       # K8s manifests
#     deployment.yaml
#     service.yaml

# Valider chart
helm lint mychart/

# Package chart
helm package mychart/

# Install depuis fichier local
helm install myapp ./mychart
```

---

## 📝 Exemples Complets

### Application Full Stack

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: myapp

---
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: myapp
data:
  DATABASE_HOST: mysql-service
  DATABASE_NAME: myapp

---
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: myapp
type: Opaque
stringData:
  DATABASE_PASSWORD: secretpass

---
# mysql-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DATABASE_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DATABASE_NAME
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-data
        persistentVolumeClaim:
          claimName: mysql-pvc

---
# mysql-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
  namespace: myapp
spec:
  selector:
    app: mysql
  ports:
  - port: 3306

---
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  namespace: myapp
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi

---
# app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
  namespace: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: app
        image: myapp:1.0
        ports:
        - containerPort: 8080
        envFrom:
        - configMapRef:
            name: app-config
        env:
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DATABASE_PASSWORD
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5

---
# app-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service
  namespace: myapp
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080

---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: myapp
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```

**Déployer :**
```bash
kubectl apply -f namespace.yaml
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
kubectl apply -f pvc.yaml
kubectl apply -f mysql-deployment.yaml
kubectl apply -f mysql-service.yaml
kubectl apply -f app-deployment.yaml
kubectl apply -f app-service.yaml
kubectl apply -f ingress.yaml

# Ou tout en une fois:
kubectl apply -f .
```

---

## 🎓 Ressources & Bonnes Pratiques

### Bonnes Pratiques

**1. Labels et Annotations**
```yaml
metadata:
  labels:
    app: myapp
    tier: frontend
    environment: production
    version: v1.0
  annotations:
    description: "Main web application"
    maintainer: "team@example.com"
```

**2. Resource Limits**
```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

**3. Health Checks**
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

**4. Security Context**
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000
  readOnlyRootFilesystem: true
```

**5. Pod Disruption Budget**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web
```

### Commandes Utiles

```bash
# Kubectl cheat sheet
kubectl cheat

# Explain ressource
kubectl explain pod
kubectl explain pod.spec.containers

# Dry-run + output YAML (génération rapide)
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml

# Diff avant apply
kubectl diff -f deployment.yaml

# Edit ressource live
kubectl edit deployment nginx

# Patch ressource
kubectl patch deployment nginx -p '{"spec":{"replicas":5}}'

# Replace (force update)
kubectl replace -f deployment.yaml --force

# Taint nodes
kubectl taint nodes worker-1 key=value:NoSchedule

# Cordon/Uncordon node (maintenance)
kubectl cordon worker-1
kubectl uncordon worker-1

# Drain node (évacuer pods)
kubectl drain worker-1 --ignore-daemonsets --delete-emptydir-data

# Shell quick debug
kubectl run -it --rm debug --image=alpine --restart=Never -- sh
```

---

**🎉 Félicitations ! Vous maîtrisez maintenant Linux, Docker, et Kubernetes/OpenShift !**

**Ressources officielles :**
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [OpenShift Documentation](https://docs.openshift.com/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kubernetes Patterns](https://www.redhat.com/en/engage/kubernetes-containers-architecture-s-201910240918)
