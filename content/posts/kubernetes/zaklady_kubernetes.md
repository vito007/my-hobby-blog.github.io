---
title: "Kubernetes - Kompletný sprievodca orchestráciou kontajnerov"
description: "Komplexný návod na Kubernetes pre začiatočníkov s praktickými príkladmi. Naučte sa základy orchestrácie, pods, services, deployments a pokročilé koncepty."
date: 2025-01-15
draft: false
tags: ["kubernetes", "k8s", "orchestrácia", "devops", "kontajnery"]
categories: ["Technológie", "DevOps", "Cloud"]
author: "Váš Author"
slug: "kubernetes-orchestracia-sprievodca"
toc: true
---

> **Prečo Kubernetes?** Kubernetes (k8s) je de facto štandard pre orchestráciu kontajnerov. Umožňuje automatické nasadzovanie, škálovanie a správu kontajnerizovaných aplikácií v produkčnom prostredí.

## 1. Úvod do Kubernetes

### Čo je Kubernetes?
Kubernetes je open-source platforma pre orchestráciu kontajnerov. Automatizuje nasadzovanie, škálovanie a správu kontajnerizovaných aplikácií naprieč klastrom serverov.

### Základné pojmy:
- **Cluster** - skupina uzlov (nodes) spravovaných Kubernetes
- **Node** - fyzický alebo virtuálny server v klastri
- **Pod** - najmenšia deployovateľná jednotka (jeden alebo viac kontajnerov)
- **Service** - abstrakcia pre prístup k aplikáciám
- **Deployment** - deklaratívny spôsob správy aplikácií

### Kubernetes vs Docker Compose:

| Aspekt | Docker Compose | Kubernetes |
|--------|----------------|------------|
| **Škála** | Jeden host | Multi-host cluster |
| **Škálovanie** | Manuálne | Automatické |
| **Load balancing** | Základný | Pokročilý |
| **Self-healing** | Nie | Áno |
| **Použitie** | Development/Testing | Production |

---

## 2. Kubernetes Architecture

### Teoretický úvod
Kubernetes má master-slave architektúru s control plane a worker nodes.

#### Control Plane komponenty:
- **API Server** - centrálny komunikačný bod
- **etcd** - distribuovaná databáza stavu klastra
- **Controller Manager** - spravuje kontroléry
- **Scheduler** - rozhoduje o umiestnení pods

#### Worker Node komponenty:
- **kubelet** - agent komunikujúci s control plane
- **kube-proxy** - sieťový proxy
- **Container Runtime** - Docker/containerd/CRI-O

### Inštalácia a setup:

```bash
# Inštalácia kubectl (MacOS)
brew install kubectl

# Overenie inštalácie
kubectl version --client

# Inštalácia minikube pre lokálny vývoj
brew install minikube

# Spustenie lokálneho klastra
minikube start

# Overenie stavu klastra
kubectl cluster-info
kubectl get nodes
```

### Základné kubectl príkazy:
```bash
# Zobrazenie všetkých zdrojov
kubectl get all

# Detailné informácie o uzloch
kubectl describe nodes

# Kontexty (prepínanie medzi klastrami)
kubectl config get-contexts
kubectl config use-context minikube

# Pomoc
kubectl --help
kubectl get --help
```

---

## 3. Pods - Základné jednotky

### Teoretický úvod
Pod je najmenšia deployovateľná jednotka v Kubernetes. Obsahuje jeden alebo viac úzko súvisiacich kontajnerov, ktoré zdieľajú sieť a storage.

### Základné operácie s Pods:

```bash
# Vytvorenie pod z image
kubectl run nginx-pod --image=nginx

# Zobrazenie pods
kubectl get pods

# Detailné informácie o pod
kubectl describe pod nginx-pod

# Prístup do pod
kubectl exec -it nginx-pod -- /bin/bash

# Zobrazenie logov
kubectl logs nginx-pod

# Odstránenie pod
kubectl delete pod nginx-pod
```

### Pod Manifest (YAML):
```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
  labels:
    app: web
    version: v1
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: sidecar
    image: busybox
    command: ['sh', '-c', 'while true; do echo "$(date): Sidecar running"; sleep 30; done']
```

```bash
# Aplikovanie manifesta
kubectl apply -f pod.yaml

# Monitorovanie pod
kubectl get pod web-pod -w
```

### Praktický príklad - Multi-container Pod:
```yaml
# multi-container-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-logging
spec:
  containers:
  # Hlavná aplikácia
  - name: app
    image: nginx:alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  
  # Logging sidecar
  - name: log-processor
    image: busybox
    command: ['sh', '-c']
    args: ['tail -f /var/log/nginx/access.log']
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  
  volumes:
  - name: shared-logs
    emptyDir: {}
```

---

## 4. Services - Sieťová abstrakcia

### Teoretický úvod
Services poskytujú stabilný endpoint pre prístup k pods. Riešia problém dynamických IP adries a load balancing.

#### Typy Services:
- **ClusterIP** - interná komunikácia v klastri
- **NodePort** - prístup z vonka cez port na uzle
- **LoadBalancer** - externý load balancer (cloud provider)
- **ExternalName** - DNS alias pre externé služby

### Základné príkazy:
```bash
# Vytvorenie service pre existujúce pods
kubectl expose pod web-pod --type=ClusterIP --port=80

# Zobrazenie services
kubectl get services
kubectl get svc

# Detailné informácie
kubectl describe service web-pod

# Testovanie konektivity
kubectl run test-pod --image=busybox -it --rm -- wget -qO- web-pod
```

### Service Manifests:

#### ClusterIP Service:
```yaml
# service-clusterip.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

#### NodePort Service:
```yaml
# service-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service-external
spec:
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
```

#### LoadBalancer Service (pre cloud):
```yaml
# service-loadbalancer.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service-lb
spec:
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

### Praktický príklad - Service Discovery:
```bash
# Vytvorenie deployment s viacerými pods
kubectl create deployment web --image=nginx --replicas=3

# Vytvorenie service
kubectl expose deployment web --port=80 --type=ClusterIP

# Test load balancingu
kubectl run test --image=busybox -it --rm -- sh
# V kontajneri:
# while true; do wget -qO- web; sleep 1; done
```

---

## 5. Deployments - Deklaratívna správa aplikácií

> **Tip:** Deployments sú preferovaný způsob nasazovania aplikácií v Kubernetes. Poskytujú rolling updates, rollbacks a self-healing.

### Teoretický úvod
Deployment je kontrolér, ktorý spravuje ReplicaSets a tým aj Pods. Umožňuje deklaratívne aktualizácie a automatické škálovanie.

### Základné operácie:
```bash
# Vytvorenie deployment
kubectl create deployment nginx-app --image=nginx:1.21

# Škálovanie
kubectl scale deployment nginx-app --replicas=5

# Rolling update
kubectl set image deployment nginx-app nginx=nginx:1.22

# História zmien
kubectl rollout history deployment nginx-app

# Rollback na predchádzajúcu verziu
kubectl rollout undo deployment nginx-app

# Status rolling update
kubectl rollout status deployment nginx-app
```

### Deployment Manifest:
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
  labels:
    app: web
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
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
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
```

### Deployment stratégie:

#### RollingUpdate (default):
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
```

#### Recreate:
```yaml
spec:
  strategy:
    type: Recreate
```

### Praktický príklad - Blue-Green deployment:
```yaml
# blue-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: app
        image: myapp:v1.0
        ports:
        - containerPort: 8080

---
# service pre blue-green
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: myapp
    version: blue  # Prepnutie na green pre switch
  ports:
  - port: 80
    targetPort: 8080
```

---

## 6. ConfigMaps a Secrets - Konfigurácia

### Teoretický úvod
ConfigMaps a Secrets umožňujú oddelenie konfigurácie od kódu aplikácie.

#### Rozdiely:
- **ConfigMap** - nesitlivé konfiguračné dáta (plain text)
- **Secret** - citlivé údaje (base64 encoded, môže byť šifrované)

### ConfigMaps:

```bash
# Vytvorenie ConfigMap z command line
kubectl create configmap app-config \
  --from-literal=DATABASE_URL=postgresql://db:5432/myapp \
  --from-literal=DEBUG=true

# Vytvorenie z súboru
kubectl create configmap nginx-config --from-file=nginx.conf

# Zobrazenie ConfigMap
kubectl get configmaps
kubectl describe configmap app-config
```

#### ConfigMap Manifest:
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://db:5432/myapp"
  debug_mode: "true"
  log_level: "info"
  # Môže obsahovať aj celé súbory
  nginx.conf: |
    server {
        listen 80;
        location / {
            proxy_pass http://backend;
        }
    }
```

### Secrets:

```bash
# Vytvorenie Secret
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=supersecret

# Vytvorenie TLS secret
kubectl create secret tls tls-secret \
  --cert=tls.crt \
  --key=tls.key

# Zobrazenie secrets
kubectl get secrets
```

#### Secret Manifest:
```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=    # base64 encoded 'admin'
  password: c3VwZXJzZWNyZXQ=  # base64 encoded 'supersecret'
```

### Použitie v Pod:

#### Environment Variables:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
```

#### Volume Mounts:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: config-volume
    configMap:
      name: app-config
  - name: secret-volume
    secret:
      secretName: db-credentials
```

---

## 7. Persistent Volumes - Trvalé úložisko

### Teoretický úvod
Kubernetes poskytuje abstrakciu pre storage cez Persistent Volumes (PV) a Persistent Volume Claims (PVC).

#### Komponenty:
- **Persistent Volume (PV)** - fyzické úložisko v klastri
- **Persistent Volume Claim (PVC)** - požiadavka na úložisko
- **Storage Class** - dynamické provisionovanie

### Storage Classes:
```yaml
# storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
allowVolumeExpansion: true
reclaimPolicy: Delete
```

### Persistent Volume:
```yaml
# persistent-volume.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast-ssd
  hostPath:
    path: /data/mysql
```

### Persistent Volume Claim:
```yaml
# persistent-volume-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: fast-ssd
```

### Použitie v Pod/Deployment:
```yaml
# mysql-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
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
              name: mysql-secret
              key: password
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pvc
```

### Praktický príklad - StatefulSet pre databázu:
```yaml
# mysql-statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
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
          value: "rootpassword"
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

---

## 8. Ingress - HTTP Load Balancing

### Teoretický úvod
Ingress poskytuje HTTP/HTTPS routing do služieb v klastri. Umožňuje definovať pravidlá pre domain-based a path-based routing.

### Ingress Controller:
Najprv musíte mať nainštalovaný Ingress Controller:

```bash
# Nginx Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Overenie inštalácie
kubectl get pods -n ingress-nginx

# Pre minikube
minikube addons enable ingress
```

### Základný Ingress:
```yaml
# basic-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

### Pokročilý Ingress s TLS:
```yaml
# advanced-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
spec:
  tls:
  - hosts:
    - api.myapp.com
    - www.myapp.com
    secretName: tls-secret
  rules:
  - host: www.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
  - host: api.myapp.com
    http:
      paths:
      - path: /api/v1
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
      - path: /api/v2
        pathType: Prefix
        backend:
          service:
            name: backend-v2-service
            port:
              number: 8080
```

### Testovanie Ingress:
```bash
# Pridanie do /etc/hosts
echo "$(minikube ip) myapp.local" | sudo tee -a /etc/hosts

# Test cez curl
curl http://myapp.local

# Získanie Ingress informácií
kubectl get ingress
kubectl describe ingress web-ingress
```

---

## 9. Namespaces - Izolacia zdrojov

### Teoretický úvod
Namespaces poskytujú virtuálnu izoláciu zdrojov v rámci jedného klastra. Umožňujú organizáciu aplikácií podľa prostredí alebo tímov.

### Základné operácie:
```bash
# Zobrazenie namespaces
kubectl get namespaces

# Vytvorenie namespace
kubectl create namespace development
kubectl create namespace staging
kubectl create namespace production

# Prepnutie do namespace
kubectl config set-context --current --namespace=development

# Zobrazenie všetkých zdrojov v namespace
kubectl get all -n development

# Odstránenie namespace
kubectl delete namespace development
```

### Namespace Manifest:
```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    environment: dev
    team: frontend
```

### Resource Quotas:
```yaml
# resource-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: development
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    persistentvolumeclaims: "10"
    pods: "20"
```

### Network Policies:
```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: development
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: development
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

---

## 10. Horizontal Pod Autoscaler (HPA)

### Teoretický úvod
HPA automaticky škáluje počet pods na základe využitia CPU, pamäte alebo custom metrík.

### Požiadavky:
```bash
# Inštalácia metrics-server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Overenie
kubectl get pods -n kube-system | grep metrics-server
```

### Vytvorenie HPA:
```bash
# HPA na základe CPU
kubectl autoscale deployment web-deployment \
  --cpu-percent=50 \
  --min=2 \
  --max=10

# Zobrazenie HPA
kubectl get hpa
```

### HPA Manifest:
```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-deployment
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
```

### Testovanie autoscalingu:
```bash
# Generovanie load
kubectl run load-generator --image=busybox -it --rm -- /bin/sh
# V kontajneri:
# while true; do wget -q -O- http://web-service; done

# Sledovanie HPA
kubectl get hpa -w

# Sledovanie pods
kubectl get pods -w
```

---

## 11. Jobs a CronJobs - Dávkové úlohy

### Jobs
Jobs zabezpečujú jednorazové spustenie úloh s garantovaným dokončením.

```yaml
# job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: database-migration
spec:
  completions: 1
  parallelism: 1
  backoffLimit: 3
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: migration
        image: migrate/migrate:latest
        command: ["migrate"]
        args: ["-path", "/migrations", "-database", "postgres://user:pass@db:5432/dbname?sslmode=disable", "up"]
        env:
        - name: DB_HOST
          value: "postgresql"
```

### CronJobs
CronJobs spúšťajú úlohy podľa cronového rozvrhu.

```yaml
# cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-job
spec:
  schedule: "0 2 * * *"  # Každý deň o 2:00
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: backup
            image: postgres:13
            command:
            - /bin/bash
            - -c
            args:
            - pg_dump -h postgresql -U postgres mydb > /backup/backup-$(date +%Y%m%d).sql
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
            volumeMounts:
            - name: backup-storage
              mountPath: /backup
          volumes:
          - name: backup-storage
            persistentVolumeClaim:
              claimName: backup-pvc
```

---

## 12. Monitoring a Logging

### Kubernetes Dashboard:
```bash
# Inštalácia Dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# Vytvorenie admin usera
kubectl create serviceaccount admin-user -n kubernetes-dashboard
kubectl create clusterrolebinding admin-user --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:admin-user

# Získanie tokenu
kubectl -n kubernetes-dashboard create token admin-user

# Proxy pre prístup
kubectl proxy
# http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

### Prometheus a Grafana stack:
```yaml
# prometheus-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring

---
# prometheus-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: prometheus-config
          mountPath: /etc/prometheus/
        - name: prometheus-storage
          mountPath: /prometheus/
      volumes:
      - name: prometheus-config
        configMap:
          name: prometheus-config
      - name: prometheus-storage
        emptyDir: {}
```

### Centralizované logovanie s EFK:
```yaml
# elasticsearch.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch
  namespace: logging
spec:
  serviceName: elasticsearch
  replicas: 1
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: elasticsearch:7.17.9
        env:
        - name: discovery.type
          value: single-node
        - name: ES_JAVA_OPTS
          value: "-Xms512m -Xmx512m"
        ports:
        - containerPort: 9200
        volumeMounts:
        - name: elasticsearch-data
          mountPath: /usr/share/elasticsearch/data
  volumeClaimTemplates:
  - metadata:
      name: elasticsearch-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

---

## 13. Bezpečnosť v Kubernetes

> **⚠️ DÔLEŽITÉ:** Bezpečnosť v Kubernetes je komplexná téma. Implementujte viacvrstvový security model.

### RBAC (Role-Based Access Control):
```yaml
# rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-reader
  namespace: development

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: development
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: development
subjects:
- kind: ServiceAccount
  name: app-reader
  namespace: development
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### Pod Security Standards:
```yaml
# pod-security.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: app
    image: nginx:alpine
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      runAsNonRoot: true
      runAsUser: 1000
      capabilities:
        drop:
        - ALL
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m### Network Security Policies:
```yaml
# network-security.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx-ingress
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 80
```

### Secrets Management - Sealed Secrets:
```bash
# Inštalácia Sealed Secrets
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml

# Inštalácia kubeseal CLI
brew install kubeseal

# Vytvorenie sealed secret
echo -n mypassword | kubectl create secret generic mysecret --dry-run=client --from-file=password=/dev/stdin -o yaml | kubeseal -o yaml > mysealedsecret.yaml

# Aplikovanie sealed secret
kubectl apply -f mysealedsecret.yaml
```

---

## 14. Helm - Package Manager pre Kubernetes

### Teoretický úvod
Helm je package manager pre Kubernetes, ktorý umožňuje definovať, inštalovať a upgradovať komplexné Kubernetes aplikácie pomocou Charts.

### Inštalácia Helm:
```bash
# MacOS
brew install helm

# Ubuntu/Debian
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm

# Overenie inštalácie
helm version
```

### Základné Helm operácie:
```bash
# Pridanie Helm repozitára
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable https://charts.helm.sh/stable
helm repo update

# Vyhľadávanie charts
helm search repo nginx
helm search hub wordpress

# Inštalácia aplikácie
helm install my-nginx bitnami/nginx

# Zobrazenie nainštalovaných releases
helm list

# Upgrade aplikácie
helm upgrade my-nginx bitnami/nginx --set replicaCount=3

# Rollback
helm rollback my-nginx 1

# Odinstalovanie
helm uninstall my-nginx
```

### Vytvorenie vlastného Chart:
```bash
# Vytvorenie nového chart
helm create myapp

# Štruktúra chart
myapp/
  Chart.yaml          # Metadata o chart
  values.yaml         # Defaultné hodnoty
  templates/          # Kubernetes manifesty
    deployment.yaml
    service.yaml
    ingress.yaml
    _helpers.tpl      # Template helpers
  charts/             # Závislosti
  .helmignore         # Ignorované súbory
```

#### Chart.yaml:
```yaml
# Chart.yaml
apiVersion: v2
name: myapp
description: A Helm chart for my application
type: application
version: 0.1.0
appVersion: "1.0"
dependencies:
- name: postgresql
  version: "11.9.13"
  repository: "https://charts.bitnami.com/bitnami"
  condition: postgresql.enabled
```

#### values.yaml:
```yaml
# values.yaml
replicaCount: 2

image:
  repository: myapp
  pullPolicy: IfNotPresent
  tag: "latest"

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  className: "nginx"
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
  hosts:
    - host: myapp.local
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: myapp-tls
      hosts:
        - myapp.local

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80

postgresql:
  enabled: true
  auth:
    postgresPassword: "supersecret"
    database: "myapp"
```

#### Template deployment.yaml:
```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
          env:
            - name: DATABASE_URL
              value: "postgresql://postgres:{{ .Values.postgresql.auth.postgresPassword }}@{{ include "myapp.fullname" . }}-postgresql:5432/{{ .Values.postgresql.auth.database }}"
          livenessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: http
            initialDelaySeconds: 5
            periodSeconds: 5
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

### Testovanie a deploying Chart:
```bash
# Validácia chart
helm lint myapp/

# Dry-run inštalácia
helm install myapp-test ./myapp --dry-run --debug

# Inštalácia s custom hodnotami
helm install myapp ./myapp \
  --set replicaCount=3 \
  --set image.tag=v1.2.0 \
  --set postgresql.auth.postgresPassword=newpassword

# Inštalácia s values súborom
helm install myapp ./myapp -f production-values.yaml

# Package chart
helm package myapp/

# Push do repozitára (ak máte vlastný)
helm repo index --url https://myrepo.com/charts .
```

---

## 15. Kubernetes Operators

### Teoretický úvod
Operators sú Kubernetes aplikácie, ktoré používajú Custom Resources a Controllers na automatizáciu správy komplexných aplikácií.

### Custom Resource Definition (CRD):
```yaml
# mysql-crd.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: mysqls.database.example.com
spec:
  group: database.example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              size:
                type: integer
                minimum: 1
                maximum: 100
              version:
                type: string
                enum: ["5.7", "8.0"]
              storageSize:
                type: string
                pattern: '^[0-9]+Gi$'
          status:
            type: object
            properties:
              conditions:
                type: array
                items:
                  type: object
                  properties:
                    type:
                      type: string
                    status:
                      type: string
                    reason:
                      type: string
  scope: Namespaced
  names:
    plural: mysqls
    singular: mysql
    kind: MySQL
    shortNames:
    - mysql
```

### Custom Resource:
```yaml
# mysql-instance.yaml
apiVersion: database.example.com/v1
kind: MySQL
metadata:
  name: my-database
spec:
  size: 3
  version: "8.0"
  storageSize: "10Gi"
```

### Populárne Operators:

#### Prometheus Operator:
```bash
# Inštalácia Prometheus Operator
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack

# Vytvorenie ServiceMonitor
kubectl apply -f - <<EOF
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: myapp-metrics
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
  - port: metrics
    path: /metrics
    interval: 30s
EOF
```

#### ArgoCD Operator:
```bash
# Inštalácia ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Prístup k ArgoCD
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Získanie admin hesla
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

---

## 16. GitOps s ArgoCD

### Teoretický úvod
GitOps je metodológia continuous deployment, kde Git repozitár slúži ako single source of truth pre infrastruktúru a aplikácie.

### ArgoCD Application:
```yaml
# application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myapp-k8s
    targetRevision: main
    path: manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
  revisionHistoryLimit: 10
```

### App of Apps pattern:
```yaml
# apps.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: apps
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/gitops-apps
    targetRevision: main
    path: apps
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Štruktúra GitOps repozitára:
```
gitops-repo/
├── apps/
│   ├── frontend.yaml
│   ├── backend.yaml
│   └── database.yaml
├── infrastructure/
│   ├── namespaces/
│   ├── ingress-controllers/
│   └── monitoring/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── production/
└── README.md
```

---

## 17. Service Mesh s Istio

### Teoretický úvod
Service Mesh poskytuje komunikačnú infraštruktúru medzi mikroservices s funkciami ako load balancing, service discovery, encryption, observability.

### Inštalácia Istio:
```bash
# Stiahnutie Istio
curl -L https://istio.io/downloadIstio | sh -
export PATH=$PWD/istio-1.19.0/bin:$PATH

# Inštalácia do klastra
istioctl install --set values.defaultRevision=default

# Povolenie sidecar injection
kubectl label namespace default istio-injection=enabled
```

### Istio Gateway a VirtualService:
```yaml
# istio-gateway.yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: myapp-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - myapp.com
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: myapp-tls
    hosts:
    - myapp.com

---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: myapp-vs
spec:
  hosts:
  - myapp.com
  gateways:
  - myapp-gateway
  http:
  - match:
    - uri:
        prefix: /api/
    route:
    - destination:
        host: backend-service
        port:
          number: 8080
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: frontend-service
        port:
          number: 80
```

### Traffic Splitting (Canary Deployment):
```yaml
# canary-virtualservice.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: backend-canary
spec:
  hosts:
  - backend-service
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: backend-service
        subset: v2
  - route:
    - destination:
        host: backend-service
        subset: v1
      weight: 90
    - destination:
        host: backend-service
        subset: v2
      weight: 10

---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: backend-destination
spec:
  host: backend-service
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

---

## 18. Troubleshooting a Debugging

### Základné debugging príkazy:
```bash
# Zobrazenie problematických pods
kubectl get pods --field-selector=status.phase!=Running

# Detailné informácie o pod
kubectl describe pod <pod-name>

# Logs z pod
kubectl logs <pod-name> -f
kubectl logs <pod-name> -c <container-name>

# Logs z predchádzajúceho crash
kubectl logs <pod-name> --previous

# Exec do pod
kubectl exec -it <pod-name> -- /bin/bash

# Port forwarding pre debugging
kubectl port-forward pod/<pod-name> 8080:80

# Debug networking
kubectl run debug --image=nicolaka/netshoot -it --rm
```

### Problematické scenáre:

#### Pod stuck v Pending:
```bash
# Kontrola resource constraints
kubectl describe pod <pod-name>
kubectl top nodes
kubectl describe nodes

# Kontrola taints a tolerations
kubectl get nodes -o json | jq '.items[].spec.taints'
```

#### ImagePullBackOff:
```bash
# Kontrola image name a tag
kubectl describe pod <pod-name>

# Kontrola image pull secrets
kubectl get secrets
kubectl describe secret <image-pull-secret>

# Test pull z node
docker pull <image-name>
```

#### CrashLoopBackOff:
```bash
# Kontrola aplikačných logov
kubectl logs <pod-name> --previous

# Kontrola liveness/readiness probes
kubectl describe pod <pod-name>

# Dočasné vypnutie probes pre debugging
kubectl patch deployment <deployment-name> -p '{"spec":{"template":{"spec":{"containers":[{"name":"<container-name>","livenessProbe":null,"readinessProbe":null}]}}}}'
```

### Performance monitoring:
```bash
# Resource utilization
kubectl top nodes
kubectl top pods
kubectl top pods -A

# Podrobné metriky
kubectl get --raw /metrics

# Events v klastri
kubectl get events --sort-by='.lastTimestamp'
kubectl get events -A --sort-by='.lastTimestamp'
```

---

## 19. Production Best Practices

### Resource Management:
```yaml
# production-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: production-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    spec:
      containers:
      - name: app
        image: myapp:v1.2.3  # Vždy špecifický tag
        resources:
          requests:
            cpu: "500m"
            memory: "1Gi"
          limits:
            cpu: "1000m"
            memory: "2Gi"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        env:
        - name: ENVIRONMENT
          value: "production"
        - name: LOG_LEVEL
          value: "info"
```

### Pod Disruption Budgets:
```yaml
# pdb.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```

### Quality Gates:
```yaml
# Admission Controllers - OPA Gatekeeper
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        properties:
          labels:
            type: array
            items:
              type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        violation[{"msg": msg}] {
          required := input.parameters.labels
          provided := input.review.object.metadata.labels
          missing := required[_]
          not provided[missing]
          msg := sprintf("Missing required label: %v", [missing])
        }

---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: must-have-owner
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment"]
  parameters:
    labels: ["owner", "environment"]
```

### Multi-cluster Strategy:
```yaml
# cluster-api-cluster.yaml
apiVersion: cluster.x-k8s.io/v1beta1
kind: Cluster
metadata:
  name: production-cluster
spec:
  clusterNetwork:
    services:
      cidrBlocks: ["10.128.0.0/12"]
    pods:
      cidrBlocks: ["192.168.0.0/16"]
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
    kind: AWSCluster
    name: production-cluster
  controlPlaneRef:
    kind: KubeadmControlPlane
    apiVersion: controlplane.cluster.x-k8s.io/v1beta1
    name: production-cluster-control-plane
```

---

## 20. Záver a ďalšie kroky

> **🎉 Gratulujeme!** Prešli sste kompletným sprievodcom Kubernetes orchestráciou. Teraz máte solídny základ pre prácu s produkčnými Kubernetes klastrami.

### Kľúčové koncepty na zapamätanie:

1. **Pods** sú základné jednotky, **Deployments** ich spravujú
2. **Services** poskytujú stabilný prístup k aplikáciám  
3. **ConfigMaps/Secrets** oddeľujú konfiguráciu od kódu
4. **Ingress** rieši HTTP routing a load balancing
5. **Namespaces** poskytujú izoláciu a organizáciu
6. **HPA** zabezpečuje automatické škálovanie
7. **Helm** zjednodušuje deployment komplexných aplikácií

### Odporúčaný learning path:

#### Začiatočníci:
- Praktické cvičenia s minikube
- Vytvorenie jednoduchých aplikácií
- Experimentovanie s kubectl príkazmi
- Pochopenie YAML manifestov

#### Pokročilí:
- Implementácia CI/CD pipeline s GitOps
- Service Mesh (Istio/Linkerd)  
- Custom Controllers a Operators
- Multi-cluster management
- Advanced security (Falco, OPA)

#### Produkcia:
- Cluster autoscaling
- Disaster recovery stratégie
- Cost optimization
- Compliance a governance
- Performance tuning

### Užitočné zdroje:
- [Kubernetes oficiálna dokumentácia](https://kubernetes.io/docs/)
- [Kubernetes by Example](https://kubernetesbyexample.com/)
- [CNCF Landscape](https://landscape.cncf.io/)
- [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/)

> **💡 Tip:** Kubernetes má strmú learning curve, ale systematické učenie a praktické skúsenosti vám pomôžu zvládnuť túto mocnú technológiu. Začnite s jednoduchými projektmi a postupne pridávajte komplexnejšie funkcie.
*Návod vytvoreny pomocou [AI](https://claude.ai/)*
