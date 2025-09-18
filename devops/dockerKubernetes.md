# Docker & Kubernetes Advanced Guide for Senior Developers

## Table of Contents
1. [Docker Deep Dive](#docker-deep-dive)
2. [Kubernetes Architecture & Components](#kubernetes-architecture--components)
3. [Advanced Deployment Strategies](#advanced-deployment-strategies)
4. [Production Patterns & Best Practices](#production-patterns--best-practices)
5. [Scenario-Based Questions & Solutions](#scenario-based-questions--solutions)
6. [Troubleshooting & Debugging](#troubleshooting--debugging)
7. [Security & Compliance](#security--compliance)
8. [Performance Optimization](#performance-optimization)

---

## Docker Deep Dive

### Container Architecture & Internals

#### Linux Namespaces and Control Groups
```bash
# Understanding container isolation
# PID Namespace - Process isolation
docker run -it --pid=host ubuntu ps aux  # Share host PID namespace
docker run -it ubuntu ps aux             # Isolated PID namespace

# Network Namespace - Network isolation
docker run -it --network=host nginx      # Share host network
docker run -it nginx                     # Isolated network

# Mount Namespace - Filesystem isolation
docker run -it -v /host/path:/container/path ubuntu ls /container/path

# User Namespace - User ID mapping
docker run -it --user 1000:1000 ubuntu id

# UTS Namespace - Hostname isolation
docker run -it --hostname=custom-host ubuntu hostname
```

#### Container Runtime Architecture
```yaml
# Container Runtime Interface (CRI) Stack
Host OS
├── Container Runtime (Docker Engine/containerd/CRI-O)
│   ├── containerd (High-level runtime)
│   │   ├── containerd-shim (Process lifecycle management)
│   │   └── runc (Low-level runtime - OCI compliant)
│   └── Docker Daemon (API layer)
├── Linux Kernel
│   ├── Namespaces (Isolation)
│   ├── Cgroups (Resource limiting)
│   └── Capabilities (Security)
└── Hardware
```

### Advanced Dockerfile Patterns

#### Multi-Stage Builds for Production
```dockerfile
# Build stage
FROM maven:3.8.4-openjdk-17-slim AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -B

COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM openjdk:17-jre-slim AS runtime
RUN addgroup --system --gid 1001 appgroup && \
    adduser --system --uid 1001 --gid 1001 appuser

WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
COPY --chown=appuser:appgroup entrypoint.sh .

USER appuser
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["./entrypoint.sh"]
```

#### Distroless Images for Security
```dockerfile
# Using Google's distroless images
FROM gcr.io/distroless/java17-debian11:nonroot
COPY --from=builder /app/target/*.jar /app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app.jar"]

# Custom distroless with specific dependencies
FROM scratch
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /app/target/*.jar /app.jar
USER nonroot:nonroot
ENTRYPOINT ["/app.jar"]
```

### Docker Networking Deep Dive

#### Network Drivers and Use Cases
```bash
# Bridge Network (Default)
docker network create --driver bridge \
  --subnet=172.20.0.0/16 \
  --ip-range=172.20.240.0/20 \
  custom-bridge

# Host Network (Performance critical)
docker run --network host nginx

# Overlay Network (Multi-host)
docker network create --driver overlay \
  --subnet=10.0.9.0/24 \
  --attachable \
  multi-host-network

# Macvlan Network (Direct hardware access)
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  macvlan-net
```

#### Custom Network Configuration
```yaml
# docker-compose.yml with custom networks
version: '3.8'
services:
  web:
    image: nginx
    networks:
      frontend:
        ipv4_address: 172.20.0.10
      backend:
        ipv4_address: 172.21.0.10
  
  api:
    image: api:latest
    networks:
      backend:
        ipv4_address: 172.21.0.20
      database:
        ipv4_address: 172.22.0.20

networks:
  frontend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.21.0.0/16
  database:
    driver: bridge
    internal: true  # No external access
    ipam:
      config:
        - subnet: 172.22.0.0/16
```

### Docker Storage and Volumes

#### Volume Types and Performance
```bash
# Named volumes (Docker managed)
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.100,rw \
  --opt device=:/path/to/dir \
  nfs-volume

# Bind mounts (Host filesystem)
docker run -v /host/data:/container/data:ro nginx

# tmpfs mounts (Memory-based)
docker run --tmpfs /tmp:rw,noexec,nosuid,size=100m nginx

# Volume plugins for distributed storage
docker volume create --driver rexray/ebs \
  --opt size=10 \
  --opt volumetype=gp2 \
  ebs-volume
```

#### Storage Drivers Comparison
```yaml
Storage Drivers:
  overlay2:
    description: "Default, best performance"
    use_case: "Production workloads"
    pros: ["Fast", "Efficient", "Copy-on-write"]
    cons: ["Linux only"]
  
  aufs:
    description: "Legacy, being deprecated"
    use_case: "Older systems"
    pros: ["Stable"]
    cons: ["Slower", "Not in mainline kernel"]
  
  devicemapper:
    description: "Block-level storage"
    use_case: "Enterprise storage systems"
    pros: ["Thin provisioning", "Snapshots"]
    cons: ["Complex configuration"]
```

---

## Kubernetes Architecture & Components

### Control Plane Components

#### API Server Deep Dive
```yaml
# API Server Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-apiserver-config
data:
  config.yaml: |
    apiVersion: apiserver.k8s.io/v1alpha1
    kind: APIServerConfiguration
    etcdServersOverrides:
      - /events#https://etcd-events:2379
    storageConfig:
      urls:
        - https://etcd-main:2379
      keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
      certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
      caFile: /etc/kubernetes/pki/etcd/ca.crt
    admission:
      plugins:
        - name: NamespaceLifecycle
        - name: LimitRanger
        - name: ServiceAccount
        - name: DefaultStorageClass
        - name: ResourceQuota
        - name: PodSecurityPolicy
          configuration:
            apiVersion: podsecuritypolicy.admission.k8s.io/v1beta1
            kind: PodSecurityPolicyConfiguration
```

#### etcd Cluster Configuration
```yaml
# etcd StatefulSet for HA
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: etcd
spec:
  serviceName: etcd
  replicas: 3
  selector:
    matchLabels:
      app: etcd
  template:
    metadata:
      labels:
        app: etcd
    spec:
      containers:
      - name: etcd
        image: quay.io/coreos/etcd:v3.5.0
        command:
        - etcd
        - --name=$(HOSTNAME)
        - --data-dir=/var/lib/etcd
        - --initial-cluster=etcd-0=https://etcd-0.etcd:2380,etcd-1=https://etcd-1.etcd:2380,etcd-2=https://etcd-2.etcd:2380
        - --initial-cluster-state=new
        - --initial-cluster-token=etcd-cluster
        - --listen-client-urls=https://0.0.0.0:2379
        - --advertise-client-urls=https://$(HOSTNAME).etcd:2379
        - --listen-peer-urls=https://0.0.0.0:2380
        - --initial-advertise-peer-urls=https://$(HOSTNAME).etcd:2380
        - --cert-file=/etc/etcd/pki/server.crt
        - --key-file=/etc/etcd/pki/server.key
        - --trusted-ca-file=/etc/etcd/pki/ca.crt
        - --peer-cert-file=/etc/etcd/pki/peer.crt
        - --peer-key-file=/etc/etcd/pki/peer.key
        - --peer-trusted-ca-file=/etc/etcd/pki/ca.crt
        - --peer-client-cert-auth
        - --client-cert-auth
        env:
        - name: HOSTNAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        volumeMounts:
        - name: etcd-data
          mountPath: /var/lib/etcd
        - name: etcd-certs
          mountPath: /etc/etcd/pki
  volumeClaimTemplates:
  - metadata:
      name: etcd-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
      storageClassName: fast-ssd
```

#### Scheduler Configuration
```yaml
# Custom Scheduler Configuration
apiVersion: kubescheduler.config.k8s.io/v1beta2
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: custom-scheduler
  plugins:
    score:
      enabled:
      - name: NodeResourcesFit
        weight: 1
      - name: NodeAffinity
        weight: 2
      - name: PodTopologySpread
        weight: 2
      disabled:
      - name: NodeResourcesLeastAllocated
    filter:
      enabled:
      - name: NodeUnschedulable
      - name: NodeResourcesFit
      - name: NodeAffinity
      - name: PodTopologySpread
  pluginConfig:
  - name: NodeResourcesFit
    args:
      scoringStrategy:
        type: LeastAllocated
        resources:
        - name: cpu
          weight: 1
        - name: memory
          weight: 1
```

### Node Components Architecture

#### kubelet Configuration
```yaml
# kubelet configuration
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: 0.0.0.0
port: 10250
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
clusterDomain: cluster.local
clusterDNS:
- 10.96.0.10
containerRuntimeEndpoint: unix:///var/run/containerd/containerd.sock
cpuManagerPolicy: static
cpuManagerReconcilePeriod: 10s
evictionHard:
  memory.available: "100Mi"
  nodefs.available: "10%"
  nodefs.inodesFree: "5%"
evictionSoft:
  memory.available: "200Mi"
  nodefs.available: "15%"
evictionSoftGracePeriod:
  memory.available: "2m"
  nodefs.available: "2m"
featureGates:
  CPUManager: true
  TopologyManager: true
imageGCHighThresholdPercent: 85
imageGCLowThresholdPercent: 80
maxPods: 110
memoryManagerPolicy: Static
systemReserved:
  cpu: 100m
  memory: 100Mi
  ephemeral-storage: 1Gi
kubeReserved:
  cpu: 100m
  memory: 100Mi
  ephemeral-storage: 1Gi
topologyManagerPolicy: single-numa-node
```

#### Container Runtime Interface (CRI)
```yaml
# containerd configuration
version = 2

[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    sandbox_image = "k8s.gcr.io/pause:3.6"
    
    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "runc"
      
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          runtime_type = "io.containerd.runc.v2"
          
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true
            
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.kata]
          runtime_type = "io.containerd.kata.v2"
          
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.gvisor]
          runtime_type = "io.containerd.runsc.v1"
    
    [plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://registry-1.docker.io"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."private-registry.company.com"]
          endpoint = ["https://private-registry.company.com"]
```

### Networking Architecture

#### CNI Plugin Configuration
```yaml
# Calico CNI Configuration
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 10.244.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()
    - blockSize: 122
      cidr: 2001:db8::/32
      encapsulation: None
      natOutgoing: Enabled
      nodeSelector: all()
    nodeAddressAutodetectionV4:
      interface: "eth.*"
    nodeAddressAutodetectionV6:
      interface: "eth.*"
  registry: quay.io/
  imagePullSecrets:
  - name: tigera-pull-secret
```

#### Service Mesh Integration (Istio)
```yaml
# Istio Service Mesh Configuration
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  name: control-plane
spec:
  values:
    global:
      meshID: mesh1
      multiCluster:
        clusterName: cluster1
      network: network1
  components:
    pilot:
      k8s:
        env:
          - name: PILOT_TRACE_SAMPLING
            value: "100"
          - name: PILOT_ENABLE_WORKLOAD_ENTRY_AUTOREGISTRATION
            value: true
        resources:
          requests:
            cpu: 500m
            memory: 2048Mi
    ingressGateways:
    - name: istio-ingressgateway
      enabled: true
      k8s:
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
        service:
          type: LoadBalancer
          ports:
          - port: 15021
            targetPort: 15021
            name: status-port
          - port: 80
            targetPort: 8080
            name: http2
          - port: 443
            targetPort: 8443
            name: https
```

---

## Advanced Deployment Strategies

### Blue-Green Deployments

#### Implementation with Kubernetes
```yaml
# Blue-Green Deployment Strategy
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: blue-green-app
spec:
  replicas: 5
  strategy:
    blueGreen:
      activeService: active-service
      previewService: preview-service
      autoPromotionEnabled: false
      scaleDownDelaySeconds: 30
      prePromotionAnalysis:
        templates:
        - templateName: success-rate
        args:
        - name: service-name
          value: preview-service
      postPromotionAnalysis:
        templates:
        - templateName: success-rate
        args:
        - name: service-name
          value: active-service
  selector:
    matchLabels:
      app: blue-green-app
  template:
    metadata:
      labels:
        app: blue-green-app
    spec:
      containers:
      - name: app
        image: myapp:v2.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10

---
apiVersion: v1
kind: Service
metadata:
  name: active-service
spec:
  selector:
    app: blue-green-app
  ports:
  - port: 80
    targetPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: preview-service
spec:
  selector:
    app: blue-green-app
  ports:
  - port: 80
    targetPort: 8080
```

### Canary Deployments with Traffic Splitting

#### Flagger Configuration
```yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: canary-app
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: canary-app
  progressDeadlineSeconds: 60
  service:
    port: 80
    targetPort: 8080
    gateways:
    - public-gateway.istio-system.svc.cluster.local
    hosts:
    - app.example.com
    trafficPolicy:
      tls:
        mode: DISABLE
  analysis:
    interval: 1m
    threshold: 5
    maxWeight: 50
    stepWeight: 10
    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99
      interval: 1m
    - name: request-duration
      thresholdRange:
        max: 500
      interval: 30s
    webhooks:
    - name: acceptance-test
      type: pre-rollout
      url: http://flagger-loadtester.test/
      timeout: 30s
      metadata:
        type: bash
        cmd: "curl -sd 'test' http://canary-app-canary/token | grep token"
    - name: load-test
      url: http://flagger-loadtester.test/
      timeout: 5s
      metadata:
        cmd: "hey -z 1m -q 10 -c 2 http://app.example.com/"
```

### GitOps with ArgoCD

#### Application Configuration
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: microservice-app
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/company/k8s-manifests
    targetRevision: HEAD
    path: apps/microservice
    helm:
      valueFiles:
      - values-prod.yaml
      parameters:
      - name: image.tag
        value: v1.2.3
      - name: replicaCount
        value: "3"
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
  revisionHistoryLimit: 10

---
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production-project
  namespace: argocd
spec:
  description: Production applications
  sourceRepos:
  - 'https://github.com/company/*'
  destinations:
  - namespace: 'production'
    server: https://kubernetes.default.svc
  - namespace: 'staging'
    server: https://kubernetes.default.svc
  clusterResourceWhitelist:
  - group: ''
    kind: Namespace
  - group: 'rbac.authorization.k8s.io'
    kind: ClusterRole
  - group: 'rbac.authorization.k8s.io'
    kind: ClusterRoleBinding
  namespaceResourceWhitelist:
  - group: ''
    kind: ConfigMap
  - group: ''
    kind: Secret
  - group: ''
    kind: Service
  - group: 'apps'
    kind: Deployment
  - group: 'apps'
    kind: StatefulSet
  roles:
  - name: admin
    description: Admin access to production
    policies:
    - p, proj:production-project:admin, applications, *, production-project/*, allow
    - p, proj:production-project:admin, repositories, *, *, allow
    groups:
    - company:production-admins
```

---

## Production Patterns & Best Practices

### Resource Management and Limits

#### Comprehensive Resource Configuration
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    requests.cpu: "100"
    requests.memory: 200Gi
    limits.cpu: "200"
    limits.memory: 400Gi
    requests.storage: 1Ti
    persistentvolumeclaims: "50"
    pods: "100"
    services: "20"
    secrets: "50"
    configmaps: "50"

---
apiVersion: v1
kind: LimitRange
metadata:
  name: production-limits
  namespace: production
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    type: Container
  - max:
      cpu: "2"
      memory: "4Gi"
    min:
      cpu: "50m"
      memory: "64Mi"
    type: Container
  - max:
      storage: "100Gi"
    min:
      storage: "1Gi"
    type: PersistentVolumeClaim

---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
  namespace: production
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: critical-app
```

### Horizontal Pod Autoscaling (HPA)

#### Advanced HPA Configuration
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: advanced-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 3
  maxReplicas: 100
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
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1k"
  - type: Object
    object:
      metric:
        name: requests-per-second
      describedObject:
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        name: main-route
      target:
        type: Value
        value: "10k"
  - type: External
    external:
      metric:
        name: queue_messages_ready
        selector:
          matchLabels:
            queue: worker_tasks
      target:
        type: AverageValue
        averageValue: "30"
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
      - type: Pods
        value: 2
        periodSeconds: 60
      selectPolicy: Min
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 4
        periodSeconds: 15
      selectPolicy: Max
```

### Vertical Pod Autoscaling (VPA)

#### VPA Configuration
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: vpa-recommender
  namespace: production
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: resource-intensive-app
  updatePolicy:
    updateMode: "Auto"
    minReplicas: 2
  resourcePolicy:
    containerPolicies:
    - containerName: app
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 4Gi
      controlledResources: ["cpu", "memory"]
      controlledValues: RequestsAndLimits
    - containerName: sidecar
      mode: "Off"
```

### StatefulSet Patterns

#### Database StatefulSet with Persistent Storage
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgresql-cluster
  namespace: database
spec:
  serviceName: postgresql-headless
  replicas: 3
  selector:
    matchLabels:
      app: postgresql
  template:
    metadata:
      labels:
        app: postgresql
    spec:
      securityContext:
        fsGroup: 999
      initContainers:
      - name: init-permissions
        image: busybox:1.35
        command:
        - sh
        - -c
        - |
          chown -R 999:999 /var/lib/postgresql/data
          chmod 700 /var/lib/postgresql/data
        volumeMounts:
        - name: postgresql-data
          mountPath: /var/lib/postgresql/data
      containers:
      - name: postgresql
        image: postgres:14-alpine
        env:
        - name: POSTGRES_DB
          value: "myapp"
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: postgresql-secret
              key: username
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgresql-secret
              key: password
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        ports:
        - containerPort: 5432
          name: postgresql
        volumeMounts:
        - name: postgresql-data
          mountPath: /var/lib/postgresql/data
        - name: postgresql-config
          mountPath: /etc/postgresql/postgresql.conf
          subPath: postgresql.conf
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
          limits:
            cpu: 2
            memory: 4Gi
        readinessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - pg_isready -U $POSTGRES_USER -d $POSTGRES_DB
          initialDelaySeconds: 15
          periodSeconds: 5
        livenessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - pg_isready -U $POSTGRES_USER -d $POSTGRES_DB
          initialDelaySeconds: 30
          periodSeconds: 10
      volumes:
      - name: postgresql-config
        configMap:
          name: postgresql-config
  volumeClaimTemplates:
  - metadata:
      name: postgresql-data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 100Gi

---
apiVersion: v1
kind: Service
metadata:
  name: postgresql-headless
  namespace: database
spec:
  clusterIP: None
  selector:
    app: postgresql
  ports:
  - port: 5432
    targetPort: 5432

---
apiVersion: v1
kind: Service
metadata:
  name: postgresql-read
  namespace: database
spec:
  selector:
    app: postgresql
  ports:
  - port: 5432
    targetPort: 5432
```

---

## Scenario-Based Questions & Solutions

### Scenario 1: High-Traffic E-commerce Platform

**Problem**: You're architecting a Kubernetes cluster for a high-traffic e-commerce platform that experiences 10x traffic spikes during sales events. The platform consists of:
- Frontend (React SPA)
- API Gateway
- Microservices (User, Product, Order, Payment, Inventory)
- Databases (PostgreSQL, Redis, Elasticsearch)
- Message Queue (RabbitMQ)

**Requirements**:
- Handle 100K concurrent users during peak
- 99.99% uptime
- Sub-200ms API response times
- Cost optimization during low traffic
- Multi-region deployment

**Solution**:

```yaml
# Cluster Autoscaler Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-autoscaler-status
  namespace: kube-system
data:
  nodes.max: "1000"
  nodes.min: "10"
  scale-down-delay-after-add: "10m"
  scale-down-unneeded-time: "10m"
  scale-down-utilization-threshold: "0.5"

---
# Multi-tier Application Architecture
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  namespace: production
spec:
  replicas: 5
  selector:
    matchLabels:
      app: api-gateway
      tier: gateway
  template:
    metadata:
      labels:
        app: api-gateway
        tier: gateway
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - api-gateway
              topologyKey: kubernetes.io/hostname
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-type
                operator: In
                values:
                - compute-optimized
      containers:
      - name: api-gateway
        image: nginx:1.21-alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 1
            memory: 512Mi
        readinessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20

---
# HPA for API Gateway
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-gateway-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-gateway
  minReplicas: 5
  maxReplicas: 100
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
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 100
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60

---
# Microservice with Circuit Breaker Pattern
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  namespace: production
spec:
  replicas: 10
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: order-service
        image: order-service:v1.2.3
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: database-secret
              key: url
        - name: REDIS_URL
          valueFrom:
            configMapKeyRef:
              name: redis-config
              key: url
        resources:
          requests:
            cpu: 300m
            memory: 512Mi
          limits:
            cpu: 1
            memory: 1Gi
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 30
          failureThreshold: 3

---
# Service Mesh Configuration (Istio)
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: order-service-vs
  namespace: production
spec:
  hosts:
  - order-service
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: order-service
        subset: canary
      weight: 100
  - route:
    - destination:
        host: order-service
        subset: stable
      weight: 100
    timeout: 10s
    retries:
      attempts: 3
      perTryTimeout: 3s
      retryOn: 5xx,reset,connect-failure,refused-stream

---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: order-service-dr
  namespace: production
spec:
  host: order-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        maxRequestsPerConnection: 10
    circuitBreaker:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
  subsets:
  - name: stable
    labels:
      version: stable
  - name: canary
    labels:
      version: canary
```

**Architecture Decisions**:

1. **Multi-tier Node Architecture**:
   ```yaml
   # Node groups for different workload types
   compute-optimized:  # For CPU-intensive services
     instance_type: c5.2xlarge
     min_size: 5
     max_size: 50
   
   memory-optimized:   # For caching and databases
     instance_type: r5.xlarge
     min_size: 3
     max_size: 20
   
   storage-optimized:  # For databases and logging
     instance_type: i3.large
     min_size: 3
     max_size: 10
   ```

2. **Database Strategy**:
   ```yaml
   # Read replicas for scaling reads
   postgresql-primary:
     replicas: 1
     resources:
       cpu: 4
       memory: 16Gi
       storage: 1Ti
   
   postgresql-read-replicas:
     replicas: 5
     resources:
       cpu: 2
       memory: 8Gi
       storage: 1Ti
   
   # Redis cluster for caching
   redis-cluster:
     replicas: 6  # 3 masters, 3 slaves
     resources:
       cpu: 1
       memory: 4Gi
   ```

### Scenario 2: Multi-tenant SaaS Platform

**Problem**: Design a Kubernetes architecture for a multi-tenant SaaS platform where:
- Each tenant needs isolated resources
- Some tenants require dedicated nodes for compliance
- Shared services (monitoring, logging, ingress)
- Different SLA tiers (Basic, Premium, Enterprise)

**Solution**:

```yaml
# Namespace per tenant with resource quotas
apiVersion: v1
kind: Namespace
metadata:
  name: tenant-enterprise-001
  labels:
    tenant-id: "enterprise-001"
    tier: "enterprise"
    compliance: "required"

---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: enterprise-quota
  namespace: tenant-enterprise-001
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    persistentvolumeclaims: "10"
    services: "10"
    secrets: "20"

---
# Network Policy for tenant isolation
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: tenant-isolation
  namespace: tenant-enterprise-001
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: istio-system
    - namespaceSelector:
        matchLabels:
          name: tenant-enterprise-001
    - namespaceSelector:
        matchLabels:
          name: shared-services
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: shared-services
  - to: []
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
  - to: []
    ports:
    - protocol: TCP
      port: 443

---
# Dedicated node pool for enterprise tenants
apiVersion: v1
kind: Node
metadata:
  name: enterprise-node-001
  labels:
    node-type: enterprise
    tenant-tier: enterprise
    compliance: required
spec:
  taints:
  - key: tenant-tier
    value: enterprise
    effect: NoSchedule

---
# Tenant application with node affinity
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tenant-app
  namespace: tenant-enterprise-001
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tenant-app
      tenant-id: enterprise-001
  template:
    metadata:
      labels:
        app: tenant-app
        tenant-id: enterprise-001
    spec:
      nodeSelector:
        tenant-tier: enterprise
      tolerations:
      - key: tenant-tier
        operator: Equal
        value: enterprise
        effect: NoSchedule
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: compliance
                operator: In
                values:
                - required
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - tenant-app
              topologyKey: kubernetes.io/hostname
      containers:
      - name: app
        image: tenant-app:v1.0.0
        env:
        - name: TENANT_ID
          value: "enterprise-001"
        - name: TIER
          value: "enterprise"
        resources:
          requests:
            cpu: 1
            memory: 2Gi
          limits:
            cpu: 2
            memory: 4Gi
        securityContext:
          runAsNonRoot: true
          runAsUser: 1000
          readOnlyRootFilesystem: true
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL

---
# Tenant-specific ingress with rate limiting
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tenant-ingress
  namespace: tenant-enterprise-001
  annotations:
    nginx.ingress.kubernetes.io/rate-limit: "1000"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/auth-tls-verify-client: "on"
    nginx.ingress.kubernetes.io/auth-tls-secret: "tenant-enterprise-001/client-ca"
spec:
  tls:
  - hosts:
    - enterprise-001.saas-platform.com
    secretName: enterprise-001-tls
  rules:
  - host: enterprise-001.saas-platform.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: tenant-app
            port:
              number: 80
```

### Scenario 3: CI/CD Pipeline with Security Scanning

**Problem**: Implement a secure CI/CD pipeline that:
- Builds and scans container images
- Runs security and compliance checks
- Implements progressive deployment
- Provides rollback capabilities
- Integrates with external security tools

**Solution**:

```yaml
# Tekton Pipeline for CI/CD
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: secure-build-deploy
  namespace: cicd
spec:
  params:
  - name: git-url
    type: string
  - name: git-revision
    type: string
    default: main
  - name: image-name
    type: string
  - name: deployment-namespace
    type: string
  workspaces:
  - name: shared-data
  - name: docker-credentials
  tasks:
  
  # Source code checkout
  - name: fetch-source
    taskRef:
      name: git-clone
    workspaces:
    - name: output
      workspace: shared-data
    params:
    - name: url
      value: $(params.git-url)
    - name: revision
      value: $(params.git-revision)
  
  # Security: Source code scanning
  - name: source-security-scan
    runAfter: ["fetch-source"]
    taskRef:
      name: sonarqube-scanner
    workspaces:
    - name: source
      workspace: shared-data
    params:
    - name: sonar-host-url
      value: "https://sonarqube.company.com"
    - name: sonar-project-key
      value: "microservice-app"
  
  # Build application
  - name: build-app
    runAfter: ["source-security-scan"]
    taskRef:
      name: maven
    workspaces:
    - name: source
      workspace: shared-data
    params:
    - name: GOALS
      value: ["clean", "compile", "test", "package"]
  
  # Build container image
  - name: build-image
    runAfter: ["build-app"]
    taskRef:
      name: buildah
    workspaces:
    - name: source
      workspace: shared-data
    - name: dockerconfig
      workspace: docker-credentials
    params:
    - name: IMAGE
      value: $(params.image-name):$(tasks.fetch-source.results.commit)
    - name: DOCKERFILE
      value: ./Dockerfile
    - name: CONTEXT
      value: .
  
  # Security: Container image scanning
  - name: image-security-scan
    runAfter: ["build-image"]
    taskSpec:
      params:
      - name: image
        type: string
      steps:
      - name: trivy-scan
        image: aquasec/trivy:latest
        script: |
          #!/bin/sh
          trivy image --exit-code 1 --severity HIGH,CRITICAL $(params.image)
          trivy image --format json --output /tmp/trivy-report.json $(params.image)
        volumeMounts:
        - name: trivy-cache
          mountPath: /root/.cache/trivy
      - name: upload-results
        image: curlimages/curl:latest
        script: |
          #!/bin/sh
          curl -X POST -H "Content-Type: application/json" \
            -d @/tmp/trivy-report.json \
            https://security-dashboard.company.com/api/scan-results
      volumes:
      - name: trivy-cache
        emptyDir: {}
    params:
    - name: image
      value: $(params.image-name):$(tasks.fetch-source.results.commit)
  
  # Security: Policy validation
  - name: policy-validation
    runAfter: ["image-security-scan"]
    taskSpec:
      params:
      - name: image
        type: string
      steps:
      - name: opa-conftest
        image: openpolicyagent/conftest:latest
        script: |
          #!/bin/sh
          # Validate Kubernetes manifests against security policies
          conftest verify --policy /policies k8s-manifests/
        volumeMounts:
        - name: security-policies
          mountPath: /policies
      volumes:
      - name: security-policies
        configMap:
          name: security-policies
    params:
    - name: image
      value: $(params.image-name):$(tasks.fetch-source.results.commit)
  
  # Deploy to staging
  - name: deploy-staging
    runAfter: ["policy-validation"]
    taskRef:
      name: argocd-task-sync-and-wait
    params:
    - name: application-name
      value: "app-staging"
    - name: argocd-version
      value: v2.4.7
    - name: flags
      value: --insecure
  
  # Integration tests
  - name: integration-tests
    runAfter: ["deploy-staging"]
    taskSpec:
      steps:
      - name: run-tests
        image: postman/newman:latest
        script: |
          #!/bin/sh
          newman run integration-tests.json \
            --environment staging.json \
            --reporters cli,json \
            --reporter-json-export /tmp/test-results.json
          
          # Upload test results
          curl -X POST -H "Content-Type: application/json" \
            -d @/tmp/test-results.json \
            https://test-dashboard.company.com/api/results
  
  # Performance tests
  - name: performance-tests
    runAfter: ["integration-tests"]
    taskSpec:
      steps:
      - name: k6-load-test
        image: loadimpact/k6:latest
        script: |
          #!/bin/sh
          k6 run --out json=/tmp/k6-results.json performance-test.js
          
          # Check performance thresholds
          if [ $? -ne 0 ]; then
            echo "Performance tests failed"
            exit 1
          fi
  
  # Production deployment (Blue-Green)
  - name: deploy-production
    runAfter: ["performance-tests"]
    taskRef:
      name: argocd-task-sync-and-wait
    params:
    - name: application-name
      value: "app-production"
    - name: argocd-version
      value: v2.4.7
    - name: flags
      value: --insecure

---
# Security Policies (OPA/Gatekeeper)
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredsecuritycontext
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredSecurityContext
      validation:
        properties:
          runAsNonRoot:
            type: boolean
          runAsUser:
            type: integer
          fsGroup:
            type: integer
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredsecuritycontext
        
        violation[{"msg": msg}] {
          container := input.review.object.spec.template.spec.containers[_]
          not container.securityContext.runAsNonRoot
          msg := "Container must run as non-root user"
        }
        
        violation[{"msg": msg}] {
          container := input.review.object.spec.template.spec.containers[_]
          container.securityContext.runAsUser == 0
          msg := "Container must not run as root (UID 0)"
        }

---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredSecurityContext
metadata:
  name: must-run-as-nonroot
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment", "StatefulSet", "DaemonSet"]
    namespaces: ["production", "staging"]
  parameters:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 1000
```

---

## Troubleshooting & Debugging

### Common Issues and Solutions

#### 1. Pod Stuck in Pending State

**Diagnostic Commands**:
```bash
# Check pod status and events
kubectl describe pod <pod-name> -n <namespace>

# Check node resources
kubectl top nodes
kubectl describe nodes

# Check resource quotas
kubectl describe resourcequota -n <namespace>

# Check PVC status
kubectl get pvc -n <namespace>
kubectl describe pvc <pvc-name> -n <namespace>
```

**Common Causes and Solutions**:

```yaml
# Insufficient resources - Add resource requests
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 500m
        memory: 512Mi

# Node selector issues - Check node labels
kubectl get nodes --show-labels
kubectl label nodes <node-name> disktype=ssd

# Taints and tolerations
kubectl describe node <node-name> | grep -i taint
```

#### 2. Service Discovery Issues

**Diagnostic Commands**:
```bash
# Check service endpoints
kubectl get endpoints <service-name> -n <namespace>

# Test DNS resolution
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup <service-name>.<namespace>.svc.cluster.local

# Check kube-dns/CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

**Solutions**:
```yaml
# Correct service selector
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app  # Must match pod labels exactly
  ports:
  - port: 80
    targetPort: 8080

# Headless service for StatefulSet
apiVersion: v1
kind: Service
metadata:
  name: my-statefulset-headless
spec:
  clusterIP: None
  selector:
    app: my-statefulset
  ports:
  - port: 80
```

#### 3. Persistent Volume Issues

**Diagnostic Commands**:
```bash
# Check PV and PVC status
kubectl get pv,pvc -A
kubectl describe pv <pv-name>
kubectl describe pvc <pvc-name> -n <namespace>

# Check storage class
kubectl get storageclass
kubectl describe storageclass <storage-class-name>

# Check volume mounts in pods
kubectl describe pod <pod-name> -n <namespace>
```

**Solutions**:
```yaml
# Proper PVC configuration
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 10Gi

# StatefulSet with volume claim template
apiVersion: apps/v1
kind: StatefulSet
spec:
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 10Gi
```

### Advanced Debugging Techniques

#### 1. Network Debugging
```bash
# Create debug pod with network tools
kubectl run netshoot --rm -it --image=nicolaka/netshoot -- /bin/bash

# Inside the debug pod:
# Test connectivity
nc -zv <service-name> <port>
curl -v http://<service-name>:<port>/health

# Check DNS
nslookup <service-name>.<namespace>.svc.cluster.local
dig <service-name>.<namespace>.svc.cluster.local

# Network policies debugging
kubectl get networkpolicies -A
kubectl describe networkpolicy <policy-name> -n <namespace>
```

#### 2. Performance Debugging
```bash
# Resource usage
kubectl top pods -n <namespace> --sort-by=cpu
kubectl top pods -n <namespace> --sort-by=memory

# Detailed resource metrics
kubectl get --raw /apis/metrics.k8s.io/v1beta1/namespaces/<namespace>/pods

# Application profiling
kubectl port-forward pod/<pod-name> 6060:6060 -n <namespace>
# Access http://localhost:6060/debug/pprof/
```

#### 3. Security Debugging
```bash
# Check RBAC permissions
kubectl auth can-i <verb> <resource> --as=<user> -n <namespace>
kubectl auth can-i create pods --as=system:serviceaccount:default:my-sa

# Check security contexts
kubectl get pod <pod-name> -o jsonpath='{.spec.securityContext}'
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].securityContext}'

# Check admission controllers
kubectl get validatingadmissionwebhooks
kubectl get mutatingadmissionwebhooks
```

---

## Security & Compliance

### Pod Security Standards

#### Pod Security Policy (Deprecated) to Pod Security Standards Migration
```yaml
# Pod Security Standards at namespace level
apiVersion: v1
kind: Namespace
metadata:
  name: secure-namespace
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted

---
# Compliant pod specification
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
  namespace: secure-namespace
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: my-app:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      runAsNonRoot: true
      runAsUser: 1000
      runAsGroup: 1000
      capabilities:
        drop:
        - ALL
        add:
        - NET_BIND_SERVICE
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: var-run
      mountPath: /var/run
  volumes:
  - name: tmp
    emptyDir: {}
  - name: var-run
    emptyDir: {}
```

### RBAC Configuration

#### Comprehensive RBAC Setup
```yaml
# Service Account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
  namespace: production
automountServiceAccountToken: false

---
# Role for namespace-specific permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: app-role
rules:
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: production
subjects:
- kind: ServiceAccount
  name: app-service-account
  namespace: production
roleRef:
  kind: Role
  name: app-role
  apiGroup: rbac.authorization.k8s.io

---
# ClusterRole for cluster-wide permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring-clusterrole
rules:
- apiGroups: [""]
  resources: ["nodes", "nodes/metrics", "services", "endpoints", "pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["extensions", "apps"]
  resources: ["deployments", "daemonsets", "replicasets"]
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]

---
# ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: monitoring-clusterrolebinding
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: monitoring
roleRef:
  kind: ClusterRole
  name: monitoring-clusterrole
  apiGroup: rbac.authorization.k8s.io
```

### Network Security

#### Network Policies for Micro-segmentation
```yaml
# Default deny all ingress and egress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

---
# Allow ingress from specific namespaces
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-ingress
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: web-app
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 8080

---
# Allow egress to specific services
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-to-database
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: web-app
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: database
    - podSelector:
        matchLabels:
          app: postgresql
    ports:
    - protocol: TCP
      port: 5432
  - to: []  # Allow DNS
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

### Secrets Management

#### External Secrets Operator Configuration
```yaml
# SecretStore for AWS Secrets Manager
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets-manager
  namespace: production
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-west-2
      auth:
        secretRef:
          accessKeyID:
            name: aws-credentials
            key: access-key-id
          secretAccessKey:
            name: aws-credentials
            key: secret-access-key

---
# External Secret
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-credentials
  namespace: production
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: database-secret
    creationPolicy: Owner
  data:
  - secretKey: username
    remoteRef:
      key: prod/database
      property: username
  - secretKey: password
    remoteRef:
      key: prod/database
      property: password

---
# Vault SecretStore
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-backend
  namespace: production
spec:
  provider:
    vault:
      server: "https://vault.company.com"
      path: "secret"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "external-secrets"
          serviceAccountRef:
            name: external-secrets-sa
```

---

## Performance Optimization

### Resource Optimization

#### CPU and Memory Tuning
```yaml
# QoS Classes demonstration
apiVersion: v1
kind: Pod
metadata:
  name: guaranteed-pod
spec:
  containers:
  - name: app
    resources:
      requests:
        cpu: 500m
        memory: 1Gi
      limits:
        cpu: 500m      # Same as requests = Guaranteed QoS
        memory: 1Gi    # Same as requests = Guaranteed QoS

---
apiVersion: v1
kind: Pod
metadata:
  name: burstable-pod
spec:
  containers:
  - name: app
    resources:
      requests:
        cpu: 100m
        memory: 256Mi
      limits:
        cpu: 1         # Higher than requests = Burstable QoS
        memory: 1Gi    # Higher than requests = Burstable QoS

---
# CPU Manager and Topology Manager
apiVersion: v1
kind: Pod
metadata:
  name: cpu-intensive-pod
spec:
  containers:
  - name: app
    resources:
      requests:
        cpu: 2         # Whole CPU cores for CPU Manager
        memory: 4Gi
      limits:
        cpu: 2
        memory: 4Gi
```

### Storage Performance

#### Storage Class Optimization
```yaml
# High-performance SSD storage class
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  fsType: ext4
  encrypted: "true"
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true

---
# Local SSD for ultra-high performance
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-ssd
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer

---
# Database with optimized storage
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: high-perf-database
spec:
  serviceName: database
  replicas: 3
  template:
    spec:
      containers:
      - name: database
        image: postgres:14
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
        - name: wal
          mountPath: /var/lib/postgresql/wal
        resources:
          requests:
            cpu: 2
            memory: 8Gi
          limits:
            cpu: 4
            memory: 16Gi
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 100Gi
  - metadata:
      name: wal
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: local-ssd
      resources:
        requests:
          storage: 20Gi
```

### Network Performance

#### Optimized Service Configuration
```yaml
# Service with session affinity
apiVersion: v1
kind: Service
metadata:
  name: sticky-service
spec:
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 8080
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 3600

---
# Headless service for direct pod communication
apiVersion: v1
kind: Service
metadata:
  name: direct-communication
spec:
  clusterIP: None
  selector:
    app: cache-cluster
  ports:
  - port: 6379
    targetPort: 6379

---
# Service with topology-aware routing
apiVersion: v1
kind: Service
metadata:
  name: topology-aware-service
  annotations:
    service.kubernetes.io/topology-aware-hints: auto
spec:
  selector:
    app: distributed-app
  ports:
  - port: 80
    targetPort: 8080
```

---

## Ingress, Egress & Load Balancing

### Ingress Controllers Deep Dive

#### NGINX Ingress Controller Setup
```bash
# Install NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Verify installation
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

#### Advanced NGINX Ingress Configuration
```yaml
# Production-ready Ingress with SSL termination and rate limiting
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: production-ingress
  namespace: production
  annotations:
    # Basic configurations
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    
    # Rate limiting
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
    nginx.ingress.kubernetes.io/rate-limit-connections: "10"
    
    # SSL and TLS configurations
    nginx.ingress.kubernetes.io/ssl-protocols: "TLSv1.2 TLSv1.3"
    nginx.ingress.kubernetes.io/ssl-ciphers: "ECDHE-RSA-AES128-GCM-SHA256,ECDHE-RSA-AES256-GCM-SHA384"
    nginx.ingress.kubernetes.io/ssl-prefer-server-ciphers: "true"
    
    # Security headers
    nginx.ingress.kubernetes.io/configuration-snippet: |
      add_header X-Frame-Options "SAMEORIGIN" always;
      add_header X-Content-Type-Options "nosniff" always;
      add_header X-XSS-Protection "1; mode=block" always;
      add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
      add_header Content-Security-Policy "default-src 'self'" always;
    
    # Load balancing and upstream configuration
    nginx.ingress.kubernetes.io/upstream-hash-by: "$binary_remote_addr"
    nginx.ingress.kubernetes.io/load-balance: "round_robin"
    
    # Health checks and timeouts
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "10"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-next-upstream: "error timeout http_502 http_503 http_504"
    
    # Client body and buffer sizes
    nginx.ingress.kubernetes.io/client-body-buffer-size: "1m"
    nginx.ingress.kubernetes.io/client-max-body-size: "50m"
    nginx.ingress.kubernetes.io/proxy-buffer-size: "8k"
    nginx.ingress.kubernetes.io/proxy-buffers-number: "8"
    
    # Custom error pages
    nginx.ingress.kubernetes.io/custom-http-errors: "404,503"
    nginx.ingress.kubernetes.io/default-backend: "nginx-errors"
    
    # CORS configuration
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "https://myapp.com,https://admin.myapp.com"
    nginx.ingress.kubernetes.io/cors-allow-methods: "GET, POST, PUT, DELETE, OPTIONS"
    nginx.ingress.kubernetes.io/cors-allow-headers: "Authorization,Content-Type,X-Requested-With"
    
    # Authentication
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth-secret
    nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"
    
    # Canary deployments
    nginx.ingress.kubernetes.io/canary: "false"
    nginx.ingress.kubernetes.io/canary-weight: "10"
    nginx.ingress.kubernetes.io/canary-by-header: "X-Canary"
    
spec:
  tls:
  - hosts:
    - api.myapp.com
    - admin.myapp.com
    secretName: myapp-tls-secret
  rules:
  - host: api.myapp.com
    http:
      paths:
      - path: /api/v1
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /api/v2
        pathType: Prefix
        backend:
          service:
            name: api-v2-service
            port:
              number: 80
  - host: admin.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 80

---
# Custom error backend
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-errors
  namespace: production
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-errors
  template:
    metadata:
      labels:
        app: nginx-errors
    spec:
      containers:
      - name: nginx-errors
        image: nginx:alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: error-pages
          mountPath: /usr/share/nginx/html
      volumes:
      - name: error-pages
        configMap:
          name: error-pages-config

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-errors
  namespace: production
spec:
  selector:
    app: nginx-errors
  ports:
  - port: 80
    targetPort: 80

---
# SSL Certificate management with cert-manager
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@myapp.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
    - dns01:
        route53:
          region: us-west-2
          accessKeyID: AKIAIOSFODNN7EXAMPLE
          secretAccessKeySecretRef:
            name: route53-credentials
            key: secret-access-key

---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: myapp-tls
  namespace: production
spec:
  secretName: myapp-tls-secret
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - api.myapp.com
  - admin.myapp.com
  - "*.myapp.com"
```

#### Traefik Ingress Controller
```yaml
# Traefik v3 Configuration
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: api-ingressroute
  namespace: production
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`api.myapp.com`) && PathPrefix(`/api/v1`)
    kind: Rule
    services:
    - name: api-service
      port: 80
    middlewares:
    - name: api-auth
    - name: api-ratelimit
    - name: api-retry
  - match: Host(`api.myapp.com`) && PathPrefix(`/api/v2`)
    kind: Rule
    services:
    - name: api-v2-service
      port: 80
      weight: 90
    - name: api-v2-canary-service
      port: 80
      weight: 10
  tls:
    certResolver: letsencrypt

---
# Traefik Middlewares
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: api-auth
  namespace: production
spec:
  basicAuth:
    secret: api-auth-secret

---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: api-ratelimit
  namespace: production
spec:
  rateLimit:
    burst: 100
    average: 50
    period: 1m

---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: api-retry
  namespace: production
spec:
  retry:
    attempts: 3
    initialInterval: 100ms

---
# Traefik TCP/UDP routing
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRouteTCP
metadata:
  name: database-tcp
  namespace: production
spec:
  entryPoints:
    - postgres
  routes:
  - match: HostSNI(`db.myapp.com`)
    services:
    - name: postgresql-service
      port: 5432
  tls:
    passthrough: true

---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRouteUDP
metadata:
  name: dns-udp
  namespace: production
spec:
  entryPoints:
    - dns
  routes:
  - services:
    - name: dns-service
      port: 53
```

### Load Balancer Services

#### Cloud Provider Load Balancers
```yaml
# AWS Application Load Balancer (ALB)
apiVersion: v1
kind: Service
metadata:
  name: alb-service
  namespace: production
  annotations:
    # AWS Load Balancer Controller annotations
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "tcp"
    service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
    service.beta.kubernetes.io/aws-load-balancer-target-type: "ip"
    
    # Health check configuration
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-protocol: "http"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-path: "/health"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval-seconds: "10"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-timeout-seconds: "5"
    service.beta.kubernetes.io/aws-load-balancer-healthy-threshold-count: "2"
    service.beta.kubernetes.io/aws-load-balancer-unhealthy-threshold-count: "3"
    
    # SSL configuration
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "arn:aws:acm:us-west-2:123456789012:certificate/12345678-1234-1234-1234-123456789012"
    service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443"
    service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy: "ELBSecurityPolicy-TLS-1-2-2017-01"
    
    # Access control
    service.beta.kubernetes.io/aws-load-balancer-source-ranges: "10.0.0.0/8,172.16.0.0/12,192.168.0.0/16"
    
    # Additional configuration
    service.beta.kubernetes.io/aws-load-balancer-proxy-protocol: "*"
    service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled: "true"
    service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout: "60"
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
  - name: http
    port: 80
    targetPort: 8080
    protocol: TCP
  - name: https
    port: 443
    targetPort: 8080
    protocol: TCP
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 3600

---
# Google Cloud Load Balancer (GKE)
apiVersion: v1
kind: Service
metadata:
  name: gcp-lb-service
  namespace: production
  annotations:
    # GCP-specific annotations
    cloud.google.com/load-balancer-type: "External"
    cloud.google.com/backend-config: '{"ports": {"80":"backendconfig"}}'
    cloud.google.com/neg: '{"ingress": true}'
    
    # Firewall rules
    cloud.google.com/load-balancer-source-ranges: "0.0.0.0/0"
spec:
  type: LoadBalancer
  loadBalancerSourceRanges:
  - 10.0.0.0/8
  - 172.16.0.0/12
  - 192.168.0.0/16
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 8080

---
# Azure Load Balancer
apiVersion: v1
kind: Service
metadata:
  name: azure-lb-service
  namespace: production
  annotations:
    # Azure-specific annotations
    service.beta.kubernetes.io/azure-load-balancer-internal: "false"
    service.beta.kubernetes.io/azure-load-balancer-mode: "auto"
    service.beta.kubernetes.io/azure-dns-label-name: "myapp-prod"
    service.beta.kubernetes.io/azure-pip-name: "myapp-public-ip"
    service.beta.kubernetes.io/azure-load-balancer-health-probe-request-path: "/health"
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 8080
```

#### MetalLB for On-Premises Load Balancing
```yaml
# MetalLB Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: production-pool
      protocol: layer2
      addresses:
      - 192.168.1.100-192.168.1.150
    - name: bgp-pool
      protocol: bgp
      addresses:
      - 10.0.0.0/24
    bgp-communities:
      vpn-only: 1234:1
      web-facing: 1234:2

---
# Service using MetalLB
apiVersion: v1
kind: Service
metadata:
  name: metallb-service
  namespace: production
  annotations:
    metallb.universe.tf/address-pool: production-pool
    metallb.universe.tf/allow-shared-ip: "web-service"
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.1.100
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 8080
```

### Service Mesh Implementation

#### Istio Service Mesh Setup

##### Installation and Configuration
```bash
# Install Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.19.0
export PATH=$PWD/bin:$PATH

# Install Istio with custom configuration
istioctl install --set values.pilot.traceSampling=1.0 --set values.global.meshConfig.accessLogFile=/dev/stdout

# Enable automatic sidecar injection
kubectl label namespace production istio-injection=enabled
```

```yaml
# Istio Configuration for Production
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  name: production-istio
spec:
  values:
    global:
      meshID: production-mesh
      multiCluster:
        clusterName: production-cluster
      network: production-network
      meshConfig:
        accessLogFile: /dev/stdout
        defaultConfig:
          proxyStatsMatcher:
            inclusionRegexps:
            - ".*circuit_breakers.*"
            - ".*upstream_rq_retry.*"
            - ".*upstream_rq_pending.*"
            - ".*_cx_.*"
        extensionProviders:
        - name: jaeger
          envoyOtelAls:
            service: jaeger.istio-system.svc.cluster.local
            port: 14250
  components:
    pilot:
      k8s:
        env:
          - name: PILOT_TRACE_SAMPLING
            value: "1.0"
          - name: PILOT_ENABLE_CROSS_CLUSTER_WORKLOAD_ENTRY
            value: true
        resources:
          requests:
            cpu: 500m
            memory: 2048Mi
          limits:
            cpu: 2
            memory: 4096Mi
    ingressGateways:
    - name: istio-ingressgateway
      enabled: true
      k8s:
        service:
          type: LoadBalancer
          annotations:
            service.beta.kubernetes.io/aws-load-balancer-type: nlb
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 2
            memory: 1024Mi
        hpaSpec:
          minReplicas: 3
          maxReplicas: 10
          metrics:
          - type: Resource
            resource:
              name: cpu
              target:
                type: Utilization
                averageUtilization: 80
    egressGateways:
    - name: istio-egressgateway
      enabled: true
      k8s:
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi

---
# Gateway Configuration
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: production-gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*.myapp.com"
    tls:
      httpsRedirect: true
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: myapp-tls-secret
    hosts:
    - "*.myapp.com"
  - port:
      number: 15443
      name: tls
      protocol: TLS
    tls:
      mode: PASSTHROUGH
    hosts:
    - secure-api.myapp.com

---
# Virtual Service for Traffic Management
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: production-virtualservice
  namespace: production
spec:
  hosts:
  - api.myapp.com
  - admin.myapp.com
  gateways:
  - istio-system/production-gateway
  - mesh  # Internal traffic
  http:
  # Canary deployment configuration
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: api-service
        subset: canary
      weight: 100
    fault:
      delay:
        percentage:
          value: 0.1
        fixedDelay: 5s
    timeout: 10s
    retries:
      attempts: 3
      perTryTimeout: 3s
      retryOn: 5xx,reset,connect-failure,refused-stream
  
  # A/B testing
  - match:
    - headers:
        x-user-type:
          exact: "premium"
    route:
    - destination:
        host: api-service
        subset: premium
      weight: 100
  
  # Default routing with traffic splitting
  - match:
    - uri:
        prefix: "/api/v1"
    route:
    - destination:
        host: api-service
        subset: stable
      weight: 90
    - destination:
        host: api-service
        subset: canary
      weight: 10
    headers:
      request:
        add:
          x-request-id: "%REQ(x-request-id)%"
        remove:
        - x-internal-header
      response:
        add:
          x-response-time: "%RESPONSE_DURATION%"
    corsPolicy:
      allowOrigins:
      - exact: https://myapp.com
      - regex: ".*\\.myapp\\.com"
      allowMethods:
      - GET
      - POST
      - PUT
      - DELETE
      allowHeaders:
      - authorization
      - content-type
      - x-requested-with
      maxAge: 24h
  
  # Admin routes with authentication
  - match:
    - uri:
        prefix: "/admin"
    headers:
      authorization:
        regex: "Bearer .*"
    route:
    - destination:
        host: admin-service
        subset: stable
  
  # Websocket support
  - match:
    - uri:
        prefix: "/ws"
    headers:
      upgrade:
        exact: websocket
    route:
    - destination:
        host: websocket-service
        subset: stable
    timeout: 300s

---
# Destination Rules for Load Balancing and Circuit Breaking
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: api-service-dr
  namespace: production
spec:
  host: api-service
  trafficPolicy:
    loadBalancer:
      localityLbSetting:
        enabled: true
        distribute:
        - from: region1/zone1/*
          to:
            "region1/zone1/*": 80
            "region1/zone2/*": 20
        - from: region1/zone2/*
          to:
            "region1/zone2/*": 80
            "region1/zone1/*": 20
    connectionPool:
      tcp:
        maxConnections: 100
        connectTimeout: 30s
        tcpKeepalive:
          time: 7200s
          interval: 75s
      http:
        http1MaxPendingRequests: 50
        http2MaxRequests: 100
        maxRequestsPerConnection: 10
        maxRetries: 3
        idleTimeout: 60s
        h2UpgradePolicy: UPGRADE
    circuitBreaker:
      consecutiveGatewayErrors: 5
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
      minHealthPercent: 30
    outlierDetection:
      consecutiveGatewayErrors: 5
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
      minHealthPercent: 30
  subsets:
  - name: stable
    labels:
      version: stable
    trafficPolicy:
      loadBalancer:
        simple: LEAST_CONN
  - name: canary
    labels:
      version: canary
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
  - name: premium
    labels:
      tier: premium
    trafficPolicy:
      connectionPool:
        tcp:
          maxConnections: 200
        http:
          http1MaxPendingRequests: 100

---
# Service Entry for External Services
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: external-database
  namespace: production
spec:
  hosts:
  - external-database.company.com
  ports:
  - number: 5432
    name: postgres
    protocol: TCP
  - number: 443
    name: https
    protocol: HTTPS
  location: MESH_EXTERNAL
  resolution: DNS

---
# Egress Gateway for External Traffic
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: egress-gateway
  namespace: istio-system
spec:
  selector:
    istio: egressgateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    hosts:
    - external-api.company.com
    tls:
      mode: PASSTHROUGH

---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: egress-gateway-dr
  namespace: istio-system
spec:
  host: istio-egressgateway.istio-system.svc.cluster.local
  subsets:
  - name: external-api

---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: direct-external-api
  namespace: production
spec:
  hosts:
  - external-api.company.com
  gateways:
  - mesh
  - istio-system/egress-gateway
  http:
  - match:
    - gateways:
      - mesh
      port: 80
    route:
    - destination:
        host: istio-egressgateway.istio-system.svc.cluster.local
        subset: external-api
        port:
          number: 443
      weight: 100
  - match:
    - gateways:
      - istio-system/egress-gateway
      port: 443
    route:
    - destination:
        host: external-api.company.com
        port:
          number: 443
      weight: 100
```

#### Security Policies in Istio
```yaml
# Peer Authentication
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT

---
# Authorization Policy
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: api-access-control
  namespace: production
spec:
  selector:
    matchLabels:
      app: api-service
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/production/sa/frontend-sa"]
    - source:
        namespaces: ["istio-system"]
    to:
    - operation:
        methods: ["GET", "POST"]
        paths: ["/api/v1/*"]
    when:
    - key: request.headers[authorization]
      values: ["Bearer *"]
  - from:
    - source:
        principals: ["cluster.local/ns/production/sa/admin-sa"]
    to:
    - operation:
        methods: ["GET", "POST", "PUT", "DELETE"]
        paths: ["/api/v1/admin/*"]
    when:
    - key: source.ip
      values: ["10.0.0.0/8", "172.16.0.0/12"]

---
# Request Authentication (JWT)
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-auth
  namespace: production
spec:
  selector:
    matchLabels:
      app: api-service
  jwtRules:
  - issuer: "https://auth.myapp.com"
    jwksUri: "https://auth.myapp.com/.well-known/jwks.json"
    audiences:
    - "api.myapp.com"
    forwardOriginalToken: true
  - issuer: "https://accounts.google.com"
    jwksUri: "https://www.googleapis.com/oauth2/v3/certs"
    audiences:
    - "api.myapp.com"
```

### Pod Connectivity and Exposure Methods

#### Direct Pod Access Methods
```bash
# Method 1: Port forwarding for development/debugging
kubectl port-forward pod/my-pod-12345 8080:8080 -n production

# Method 2: Port forwarding for services
kubectl port-forward service/api-service 8080:80 -n production

# Method 3: Proxy access to pods
kubectl proxy --port=8001
# Access: http://localhost:8001/api/v1/namespaces/production/pods/my-pod-12345:8080/proxy/

# Method 4: Direct exec into pod
kubectl exec -it my-pod-12345 -n production -- /bin/bash

# Method 5: Debug with ephemeral containers (Kubernetes 1.23+)
kubectl debug my-pod-12345 -it --image=busybox:1.35 -n production
```

#### NodePort Service for External Access
```yaml
# NodePort Service Configuration
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
  namespace: production
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080  # Optional: K8s will assign if not specified (30000-32767)
    protocol: TCP
  externalTrafficPolicy: Local  # Preserves client IP
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 3600

---
# ExternalName Service for external services
apiVersion: v1
kind: Service
metadata:
  name: external-database-service
  namespace: production
spec:
  type: ExternalName
  externalName: database.company.com
  ports:
  - port: 5432
    targetPort: 5432
    protocol: TCP
```

#### Advanced Service Configurations
```yaml
# Headless Service for direct pod discovery
apiVersion: v1
kind: Service
metadata:
  name: statefulset-headless
  namespace: production
spec:
  clusterIP: None
  selector:
    app: database-cluster
  ports:
  - port: 5432
    targetPort: 5432
  publishNotReadyAddresses: true  # Include non-ready pods in DNS

---
# Service with multiple ports
apiVersion: v1
kind: Service
metadata:
  name: multi-port-service
  namespace: production
spec:
  selector:
    app: multi-app
  ports:
  - name: http
    port: 80
    targetPort: 8080
    protocol: TCP
  - name: https
    port: 443
    targetPort: 8443
    protocol: TCP
  - name: grpc
    port: 9090
    targetPort: 9090
    protocol: TCP
  - name: metrics
    port: 9092
    targetPort: 9092
    protocol: TCP

---
# Service with topology-aware routing
apiVersion: v1
kind: Service
metadata:
  name: topology-aware-service
  namespace: production
  annotations:
    service.kubernetes.io/topology-aware-hints: auto
spec:
  selector:
    app: distributed-app
  ports:
  - port: 80
    targetPort: 8080
  internalTrafficPolicy: Local  # Route to node-local endpoints when possible
```

### Advanced Networking Patterns

#### Multi-Cluster Service Mesh
```yaml
# Cross-cluster service discovery
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: cross-cluster-service
  namespace: production
spec:
  hosts:
  - api-service.production.global
  location: MESH_EXTERNAL
  ports:
  - number: 80
    name: http
    protocol: HTTP
  resolution: DNS
  addresses:
  - 240.0.0.1  # Virtual IP for cross-cluster communication
  endpoints:
  - address: api-service.production.svc.cluster.local
    locality: region1/zone1
    ports:
      http: 80
  - address: 10.0.1.100  # Remote cluster endpoint
    locality: region2/zone1
    ports:
      http: 80

---
# Workload Entry for external workloads
apiVersion: networking.istio.io/v1beta1
kind: WorkloadEntry
metadata:
  name: external-vm-workload
  namespace: production
spec:
  address: 10.0.2.100
  ports:
    http: 8080
    grpc: 9090
  labels:
    app: external-service
    version: v1
    env: production
  serviceAccount: external-workload-sa

---
apiVersion: networking.istio.io/v1beta1
kind: WorkloadGroup
metadata:
  name: external-workloads
  namespace: production
spec:
  metadata:
    labels:
      app: external-service
      version: v1
  template:
    ports:
      http: 8080
      grpc: 9090
    serviceAccount: external-workload-sa
```

#### Network Policies for Micro-segmentation
```yaml
# Zero-trust network policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: zero-trust-policy
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress: []  # Deny all ingress by default
  egress: []   # Deny all egress by default

---
# Application-specific network policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-service-network-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api-service
  policyTypes:
  - Ingress
  - Egress
  ingress:
  # Allow from frontend
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
  # Allow from ingress controller
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 8080
  # Allow from monitoring
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
    - podSelector:
        matchLabels:
          app: prometheus
    ports:
    - protocol: TCP
      port: 9090
  egress:
  # Allow to database
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  # Allow to cache
  - to:
    - podSelector:
        matchLabels:
          app: redis
    ports:
    - protocol: TCP
      port: 6379
  # Allow DNS
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
  # Allow HTTPS to external APIs
  - to: []
    ports:
    - protocol: TCP
      port: 443

---
# Namespace isolation policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: namespace-isolation
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  # Allow from same namespace
  - from:
    - namespaceSelector:
        matchLabels:
          name: production
  # Allow from system namespaces
  - from:
    - namespaceSelector:
        matchLabels:
          name: kube-system
  - from:
    - namespaceSelector:
        matchLabels:
          name: istio-system
  # Allow from monitoring
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
  egress:
  # Allow to same namespace
  - to:
    - namespaceSelector:
        matchLabels:
          name: production
  # Allow to system namespaces
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
  # Allow DNS
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
  # Allow HTTPS outbound
  - to: []
    ports:
    - protocol: TCP
      port: 443
```

---

## Health Checks and Probes

### Understanding Kubernetes Probes

Kubernetes provides three types of health checks to ensure application reliability:

#### Probe Types Overview
```yaml
# Probe Types and Their Purposes
probeTypes:
  livenessProbe:
    purpose: "Determines if container should be restarted"
    failure_action: "Restart container"
    use_case: "Detect deadlocks, infinite loops, unrecoverable errors"
    
  readinessProbe:
    purpose: "Determines if container is ready to serve traffic"
    failure_action: "Remove from service endpoints"
    use_case: "Application startup, dependency checks, graceful degradation"
    
  startupProbe:
    purpose: "Protects slow-starting containers during initialization"
    failure_action: "Restart container if startup fails"
    use_case: "Legacy applications, complex initialization processes"
```

### Liveness Probes - Detailed Configuration

#### HTTP Liveness Probe
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-with-probes
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-app
        image: myapp:v1.2.3
        ports:
        - containerPort: 8080
          name: http
        - containerPort: 8443
          name: https
        - containerPort: 9090
          name: metrics
        
        # HTTP Liveness Probe
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
            scheme: HTTP
            httpHeaders:
            - name: User-Agent
              value: "Kubernetes-Liveness-Probe"
            - name: Accept
              value: "application/json"
          initialDelaySeconds: 60    # Wait 60s before first check
          periodSeconds: 30          # Check every 30 seconds
          timeoutSeconds: 10         # Timeout after 10 seconds
          successThreshold: 1        # 1 success = healthy
          failureThreshold: 3        # 3 failures = restart container
        
        # Resources for proper probe timing
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
          limits:
            cpu: 2
            memory: 4Gi
        
        # Environment variables
        env:
        - name: JAVA_OPTS
          value: "-Xmx2g -Xms1g -XX:+UseG1GC"
        - name: MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE
          value: "health,metrics,info,prometheus"

---
# Liveness Probe Endpoint Implementation Example (Spring Boot)
apiVersion: v1
kind: ConfigMap
metadata:
  name: liveness-probe-config
  namespace: production
data:
  application.yml: |
    management:
      endpoints:
        web:
          exposure:
            include: health,metrics,info,prometheus
      health:
        livenessstate:
          enabled: true
        readinessstate:
          enabled: true
        probes:
          enabled: true
      endpoint:
        health:
          show-details: always
          probes:
            enabled: true
    
    # Custom health indicators
    custom:
      health:
        database:
          timeout: 5s
        external-service:
          timeout: 3s
        disk-space:
          threshold: 1GB
```

#### TCP Socket Liveness Probe
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: database-with-tcp-probe
  namespace: production
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgresql
  template:
    metadata:
      labels:
        app: postgresql
    spec:
      containers:
      - name: postgresql
        image: postgres:14-alpine
        ports:
        - containerPort: 5432
        
        # TCP Socket Liveness Probe
        livenessProbe:
          tcpSocket:
            port: 5432
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        env:
        - name: POSTGRES_DB
          value: "myapp"
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: username
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
      
      volumes:
      - name: postgres-data
        persistentVolumeClaim:
          claimName: postgres-pvc
```

#### Command/Exec Liveness Probe
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-with-exec-probe
  namespace: production
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        ports:
        - containerPort: 6379
        
        # Command/Exec Liveness Probe
        livenessProbe:
          exec:
            command:
            - redis-cli
            - --no-auth-warning
            - ping
          initialDelaySeconds: 20
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        # Redis configuration
        command:
        - redis-server
        - /etc/redis/redis.conf
        
        volumeMounts:
        - name: redis-config
          mountPath: /etc/redis
        - name: redis-data
          mountPath: /data
      
      volumes:
      - name: redis-config
        configMap:
          name: redis-config
      - name: redis-data
        emptyDir: {}
```

### Readiness Probes - Detailed Configuration

#### Advanced HTTP Readiness Probe
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: microservice-with-readiness
  namespace: production
spec:
  replicas: 5
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
      - name: order-service
        image: order-service:v2.1.0
        ports:
        - containerPort: 8080
          name: http
        
        # Readiness Probe with dependency checks
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
            scheme: HTTP
            httpHeaders:
            - name: User-Agent
              value: "Kubernetes-Readiness-Probe"
          initialDelaySeconds: 20    # Start checking after 20s
          periodSeconds: 5           # Check every 5 seconds
          timeoutSeconds: 3          # Timeout after 3 seconds
          successThreshold: 1        # 1 success = ready
          failureThreshold: 3        # 3 failures = not ready
        
        # Liveness Probe (different endpoint)
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 30
          timeoutSeconds: 10
          failureThreshold: 3
        
        # Environment configuration
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: database-secret
              key: url
        - name: REDIS_URL
          valueFrom:
            configMapKeyRef:
              name: redis-config
              key: url
        - name: EXTERNAL_API_URL
          value: "https://api.external-service.com"
        
        resources:
          requests:
            cpu: 200m
            memory: 512Mi
          limits:
            cpu: 1
            memory: 2Gi

---
# Readiness probe configuration for dependency checking
apiVersion: v1
kind: ConfigMap
metadata:
  name: readiness-probe-config
  namespace: production
data:
  application.yml: |
    management:
      health:
        readinessstate:
          enabled: true
        # Custom readiness indicators
        db:
          enabled: true
        redis:
          enabled: true
        external-api:
          enabled: true
    
    # Readiness probe implementation
    custom:
      readiness:
        checks:
          - name: database
            endpoint: "jdbc:postgresql://db:5432/myapp"
            timeout: 5s
            critical: true
          - name: redis
            endpoint: "redis://redis:6379"
            timeout: 3s
            critical: true
          - name: external-api
            endpoint: "${EXTERNAL_API_URL}/health"
            timeout: 5s
            critical: false  # Non-critical dependency
```

### Startup Probes - For Slow-Starting Applications

#### Startup Probe Configuration
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: legacy-app-with-startup-probe
  namespace: production
spec:
  replicas: 2
  selector:
    matchLabels:
      app: legacy-app
  template:
    metadata:
      labels:
        app: legacy-app
    spec:
      containers:
      - name: legacy-app
        image: legacy-app:v1.0.0
        ports:
        - containerPort: 8080
        
        # Startup Probe - Protects during slow initialization
        startupProbe:
          httpGet:
            path: /health/startup
            port: 8080
          initialDelaySeconds: 10    # Start checking after 10s
          periodSeconds: 10          # Check every 10 seconds
          timeoutSeconds: 5          # Timeout after 5 seconds
          failureThreshold: 30       # Allow up to 300s (30 * 10s) for startup
          successThreshold: 1        # 1 success = startup complete
        
        # Readiness Probe - Only starts after startup probe succeeds
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        
        # Liveness Probe - Only starts after startup probe succeeds
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          periodSeconds: 30
          timeoutSeconds: 10
          failureThreshold: 3
        
        # Application might need more time to start
        env:
        - name: STARTUP_TIMEOUT
          value: "300"
        - name: INIT_COMPLEXITY
          value: "high"
        
        resources:
          requests:
            cpu: 500m
            memory: 2Gi
          limits:
            cpu: 2
            memory: 8Gi
```

### Advanced Probe Patterns

#### Multi-Container Pod with Different Probes
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: multi-container-app
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: multi-container-app
  template:
    metadata:
      labels:
        app: multi-container-app
    spec:
      containers:
      # Main application container
      - name: app
        image: myapp:v1.0.0
        ports:
        - containerPort: 8080
        
        startupProbe:
          httpGet:
            path: /startup
            port: 8080
          periodSeconds: 10
          failureThreshold: 30
        
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          periodSeconds: 5
          failureThreshold: 3
        
        livenessProbe:
          httpGet:
            path: /live
            port: 8080
          periodSeconds: 30
          failureThreshold: 3
      
      # Sidecar proxy container
      - name: proxy
        image: envoy:v1.24.0
        ports:
        - containerPort: 9901
          name: admin
        - containerPort: 8000
          name: proxy
        
        # Envoy admin interface readiness
        readinessProbe:
          httpGet:
            path: /ready
            port: 9901
          periodSeconds: 5
          failureThreshold: 3
        
        # Envoy liveness check
        livenessProbe:
          httpGet:
            path: /server_info
            port: 9901
          periodSeconds: 30
          failureThreshold: 3
      
      # Metrics exporter sidecar
      - name: metrics-exporter
        image: prometheus/node-exporter:latest
        ports:
        - containerPort: 9100
        
        # Simple TCP check for metrics endpoint
        readinessProbe:
          tcpSocket:
            port: 9100
          periodSeconds: 10
        
        livenessProbe:
          tcpSocket:
            port: 9100
          periodSeconds: 30
```

#### Probe with Custom Health Check Script
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: custom-health-check-app
  namespace: production
spec:
  replicas: 2
  selector:
    matchLabels:
      app: custom-app
  template:
    metadata:
      labels:
        app: custom-app
    spec:
      containers:
      - name: custom-app
        image: custom-app:v1.0.0
        ports:
        - containerPort: 8080
        
        # Custom readiness check
        readinessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - |
              # Custom readiness logic
              curl -f http://localhost:8080/health/dependencies || exit 1
              if [ -f /tmp/maintenance-mode ]; then
                echo "Maintenance mode enabled"
                exit 1
              fi
              # Check database connectivity
              pg_isready -h database -p 5432 -U myapp || exit 1
              # Check Redis connectivity
              redis-cli -h redis ping | grep -q PONG || exit 1
              echo "All checks passed"
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 10
          failureThreshold: 3
        
        # Custom liveness check
        livenessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - |
              # Check if application is responsive
              response=$(curl -s -w "%{http_code}" http://localhost:8080/health/liveness -o /dev/null)
              if [ "$response" -eq 200 ]; then
                exit 0
              else
                echo "Health check failed with status: $response"
                exit 1
              fi
          initialDelaySeconds: 60
          periodSeconds: 30
          timeoutSeconds: 15
          failureThreshold: 3
        
        volumeMounts:
        - name: health-scripts
          mountPath: /health-scripts
          readOnly: true
      
      volumes:
      - name: health-scripts
        configMap:
          name: health-check-scripts
          defaultMode: 0755

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: health-check-scripts
  namespace: production
data:
  advanced-health-check.sh: |
    #!/bin/bash
    
    # Advanced health check script
    set -e
    
    # Function to check HTTP endpoint
    check_http() {
        local url=$1
        local expected_status=${2:-200}
        local timeout=${3:-5}
        
        response=$(curl -s -w "%{http_code}" --max-time $timeout "$url" -o /dev/null)
        if [ "$response" -eq "$expected_status" ]; then
            echo "✓ HTTP check passed: $url ($response)"
            return 0
        else
            echo "✗ HTTP check failed: $url (expected: $expected_status, got: $response)"
            return 1
        fi
    }
    
    # Function to check TCP port
    check_tcp() {
        local host=$1
        local port=$2
        local timeout=${3:-5}
        
        if timeout $timeout bash -c "cat < /dev/null > /dev/tcp/$host/$port"; then
            echo "✓ TCP check passed: $host:$port"
            return 0
        else
            echo "✗ TCP check failed: $host:$port"
            return 1
        fi
    }
    
    # Main health checks
    echo "Starting health checks..."
    
    # Check application health
    check_http "http://localhost:8080/health" 200 5
    
    # Check database
    check_tcp "database" 5432 3
    
    # Check Redis
    check_tcp "redis" 6379 3
    
    # Check external dependencies (non-critical)
    if ! check_http "https://api.external-service.com/health" 200 10; then
        echo "⚠ External service check failed (non-critical)"
    fi
    
    echo "All critical health checks passed"
```

---

## Advanced Deployment Strategies

### Rolling Updates - Production Configuration

#### Rolling Update with Precise Control
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rolling-update-app
  namespace: production
  labels:
    app: web-app
    version: v2.0.0
spec:
  replicas: 10
  selector:
    matchLabels:
      app: web-app
  
  # Rolling Update Strategy Configuration
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%    # Max 25% of pods can be unavailable
      maxSurge: 25%          # Max 25% additional pods during update
  
  # Deployment progress deadline
  progressDeadlineSeconds: 600  # 10 minutes timeout
  
  # Revision history limit
  revisionHistoryLimit: 5
  
  template:
    metadata:
      labels:
        app: web-app
        version: v2.0.0
      annotations:
        # Deployment annotations
        deployment.kubernetes.io/revision: "10"
        config.checksum: "sha256:1234567890abcdef"  # Forces restart on config change
    spec:
      # Graceful shutdown configuration
      terminationGracePeriodSeconds: 60
      
      containers:
      - name: web-app
        image: myapp:v2.0.0
        ports:
        - containerPort: 8080
        
        # Comprehensive probe configuration
        startupProbe:
          httpGet:
            path: /actuator/health/startup
            port: 8080
          periodSeconds: 10
          failureThreshold: 30
          timeoutSeconds: 5
        
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          periodSeconds: 5
          failureThreshold: 3
          successThreshold: 1
          timeoutSeconds: 3
        
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          periodSeconds: 30
          failureThreshold: 3
          timeoutSeconds: 10
          initialDelaySeconds: 60
        
        # Graceful shutdown handling
        lifecycle:
          preStop:
            exec:
              command:
              - /bin/sh
              - -c
              - |
                # Graceful shutdown script
                echo "Initiating graceful shutdown..."
                
                # Stop accepting new requests
                curl -X POST http://localhost:8080/actuator/shutdown-prepare
                
                # Wait for active requests to complete
                sleep 30
                
                # Final shutdown
                curl -X POST http://localhost:8080/actuator/shutdown
        
        # Resource configuration
        resources:
          requests:
            cpu: 200m
            memory: 512Mi
          limits:
            cpu: 1
            memory: 2Gi
        
        # Environment configuration
        env:
        - name: APP_VERSION
          value: "v2.0.0"
        - name: DEPLOYMENT_TIME
          value: "2024-01-15T10:30:00Z"
        - name: GRACEFUL_SHUTDOWN_TIMEOUT
          value: "30s"

---
# Service with session affinity for rolling updates
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
  namespace: production
spec:
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 8080
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 3600

---
# Pod Disruption Budget to ensure availability
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-app-pdb
  namespace: production
spec:
  minAvailable: 75%  # Ensure 75% of pods remain available
  selector:
    matchLabels:
      app: web-app
```

### Blue-Green Deployment Strategy

#### Complete Blue-Green Setup with ArgoCD Rollouts
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: blue-green-rollout
  namespace: production
spec:
  replicas: 5
  revisionHistoryLimit: 5
  
  strategy:
    blueGreen:
      # Services
      activeService: active-service      # Production traffic
      previewService: preview-service    # Preview/testing traffic
      
      # Promotion configuration
      autoPromotionEnabled: false        # Manual promotion
      scaleDownDelaySeconds: 60         # Wait before scaling down old version
      
      # Pre-promotion analysis
      prePromotionAnalysis:
        templates:
        - templateName: http-success-rate
        - templateName: response-time-p95
        args:
        - name: service-name
          value: preview-service
        - name: prometheus-server
          value: http://prometheus.monitoring.svc.cluster.local:9090
      
      # Post-promotion analysis
      postPromotionAnalysis:
        templates:
        - templateName: http-success-rate
        - templateName: response-time-p95
        args:
        - name: service-name
          value: active-service
        - name: prometheus-server
          value: http://prometheus.monitoring.svc.cluster.local:9090
      
      # Anti-affinity for blue-green pods
      antiAffinity:
        requiredDuringSchedulingIgnoredDuringExecution: {}
        preferredDuringSchedulingIgnoredDuringExecution:
          weight: 1
          podAffinityTerm:
            labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - blue-green-app
            topologyKey: kubernetes.io/hostname
  
  selector:
    matchLabels:
      app: blue-green-app
  
  template:
    metadata:
      labels:
        app: blue-green-app
    spec:
      containers:
      - name: app
        image: myapp:v3.0.0
        ports:
        - containerPort: 8080
        
        # Enhanced probe configuration for blue-green
        startupProbe:
          httpGet:
            path: /actuator/health/startup
            port: 8080
          periodSeconds: 5
          failureThreshold: 60    # 5 minutes for startup
        
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          periodSeconds: 2        # Fast readiness checks
          failureThreshold: 3
          successThreshold: 2     # Require 2 successes
        
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          periodSeconds: 30
          failureThreshold: 3
          initialDelaySeconds: 90
        
        resources:
          requests:
            cpu: 300m
            memory: 768Mi
          limits:
            cpu: 1.5
            memory: 3Gi

---
# Active Service (Production)
apiVersion: v1
kind: Service
metadata:
  name: active-service
  namespace: production
spec:
  selector:
    app: blue-green-app
  ports:
  - port: 80
    targetPort: 8080

---
# Preview Service (Testing)
apiVersion: v1
kind: Service
metadata:
  name: preview-service
  namespace: production
spec:
  selector:
    app: blue-green-app
  ports:
  - port: 80
    targetPort: 8080

---
# Analysis Templates for Blue-Green
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: http-success-rate
  namespace: production
spec:
  args:
  - name: service-name
  - name: prometheus-server
  metrics:
  - name: success-rate
    successCondition: result[0] >= 0.99
    failureLimit: 3
    interval: 30s
    count: 10
    provider:
      prometheus:
        address: "{{args.prometheus-server}}"
        query: |
          sum(rate(http_requests_total{service="{{args.service-name}}",status!~"5.."}[5m])) /
          sum(rate(http_requests_total{service="{{args.service-name}}"}[5m]))

---
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: response-time-p95
  namespace: production
spec:
  args:
  - name: service-name
  - name: prometheus-server
  metrics:
  - name: response-time-p95
    successCondition: result[0] <= 0.5  # 500ms
    failureLimit: 3
    interval: 30s
    count: 10
    provider:
      prometheus:
        address: "{{args.prometheus-server}}"
        query: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket{service="{{args.service-name}}"}[5m])) by (le)
          )
```

### Canary Deployment with Flagger

#### Sophisticated Canary Configuration
```yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: advanced-canary
  namespace: production
spec:
  # Target deployment
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: canary-app
  
  # Deployment configuration
  progressDeadlineSeconds: 60
  
  # HPA reference
  autoscalerRef:
    apiVersion: autoscaling/v2
    kind: HorizontalPodAutoscaler
    name: canary-app-hpa
  
  # Service configuration
  service:
    # Primary service
    name: canary-app
    port: 80
    targetPort: 8080
    
    # Istio traffic management
    gateways:
    - public-gateway.istio-system.svc.cluster.local
    hosts:
    - app.mycompany.com
    
    # Traffic policy
    trafficPolicy:
      tls:
        mode: DISABLE
      loadBalancer:
        simple: LEAST_CONN
      connectionPool:
        tcp:
          maxConnections: 100
        http:
          http1MaxPendingRequests: 50
          maxRequestsPerConnection: 10
    
    # CORS policy
    corsPolicy:
      allowOrigins:
      - exact: https://mycompany.com
      - regex: ".*\\.mycompany\\.com"
      allowMethods:
      - GET
      - POST
      - PUT
      - DELETE
      allowHeaders:
      - authorization
      - content-type
      maxAge: 24h
  
  # Canary analysis configuration
  analysis:
    # Traffic routing
    interval: 30s           # Metrics check interval
    threshold: 5            # Number of failed checks before rollback
    maxWeight: 50           # Maximum traffic % for canary
    stepWeight: 5           # Traffic increment per step
    stepWeights: [5, 10, 15, 25, 40, 50]  # Custom step progression
    
    # Mirror traffic percentage
    mirror: true
    mirrorWeight: 100       # Mirror 100% of traffic to canary
    
    # Metrics for canary analysis
    metrics:
    # HTTP success rate
    - name: request-success-rate
      templateRef:
        name: success-rate
        namespace: flagger-system
      thresholdRange:
        min: 99             # Minimum 99% success rate
      interval: 30s
    
    # HTTP request duration
    - name: request-duration
      templateRef:
        name: latency
        namespace: flagger-system
      thresholdRange:
        max: 500           # Maximum 500ms response time
      interval: 30s
    
    # Custom business metric
    - name: business-metric
      provider:
        prometheus:
          address: http://prometheus.monitoring.svc.cluster.local:9090
      query: |
        sum(rate(business_transactions_total{app="canary-app",status="success"}[1m])) /
        sum(rate(business_transactions_total{app="canary-app"}[1m])) * 100
      thresholdRange:
        min: 95             # Minimum 95% business success rate
      interval: 60s
    
    # Webhooks for testing
    webhooks:
    # Pre-rollout validation
    - name: "conformance-test"
      type: pre-rollout
      url: http://flagger-loadtester.test/
      timeout: 30s
      metadata:
        type: bash
        cmd: "curl -sd 'test' http://canary-app-canary.production:80/api/health | grep -q 'healthy'"
    
    # Load testing during canary
    - name: "load-test"
      type: rollout
      url: http://flagger-loadtester.test/
      timeout: 5s
      metadata:
        cmd: "hey -z 1m -q 10 -c 2 -H 'Authorization: Bearer test-token' http://app.mycompany.com/api/test"
    
    # Post-rollout verification
    - name: "integration-test"
      type: post-rollout
      url: http://test-runner.test/
      timeout: 300s
      metadata:
        type: grpc
        service: "TestRunner"
        method: "RunIntegrationTests"
        payload: '{"app": "canary-app", "version": "canary"}'

---
# Target Deployment for Canary
apiVersion: apps/v1
kind: Deployment
metadata:
  name: canary-app
  namespace: production
spec:
  replicas: 5
  selector:
    matchLabels:
      app: canary-app
  template:
    metadata:
      labels:
        app: canary-app
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9090"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: app
        image: myapp:v1.0.0  # Will be updated by Flagger
        ports:
        - containerPort: 8080
          name: http
        - containerPort: 9090
          name: metrics
        
        # Optimized probes for canary deployment
        startupProbe:
          httpGet:
            path: /health/startup
            port: 8080
          periodSeconds: 5
          failureThreshold: 30
        
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
            httpHeaders:
            - name: X-Health-Check
              value: "readiness"
          periodSeconds: 3        # Fast readiness for traffic switching
          failureThreshold: 2     # Quick failure detection
          successThreshold: 1
          timeoutSeconds: 2
        
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          periodSeconds: 30
          failureThreshold: 3
          initialDelaySeconds: 60
        
        # Enhanced resource configuration
        resources:
          requests:
            cpu: 250m
            memory: 512Mi
          limits:
            cpu: 1
            memory: 2Gi
        
        # Application configuration
        env:
        - name: DEPLOYMENT_TYPE
          value: "canary"
        - name: METRICS_ENABLED
          value: "true"
        - name: TRACING_ENABLED
          value: "true"

---
# HPA for Canary Deployment
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: canary-app-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: canary-app
  minReplicas: 3
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
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
```

### Recreate Deployment Strategy

#### Recreate Strategy for Stateful Applications
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: database-migration-app
  namespace: production
spec:
  replicas: 1  # Single instance for stateful app
  selector:
    matchLabels:
      app: migration-app
  
  # Recreate strategy for database migrations
  strategy:
    type: Recreate
  
  template:
    metadata:
      labels:
        app: migration-app
    spec:
      # Ensure graceful shutdown
      terminationGracePeriodSeconds: 120
      
      # Init container for pre-checks
      initContainers:
      - name: pre-migration-check
        image: migration-tools:v1.0.0
        command:
        - /bin/sh
        - -c
        - |
          echo "Running pre-migration checks..."
          
          # Check database connectivity
          until pg_isready -h $DATABASE_HOST -p $DATABASE_PORT -U $DATABASE_USER; do
            echo "Waiting for database..."
            sleep 5
          done
          
          # Verify migration state
          /migration-tools/verify-state.sh
          
          echo "Pre-migration checks completed"
        
        env:
        - name: DATABASE_HOST
          value: "postgres.database.svc.cluster.local"
        - name: DATABASE_PORT
          value: "5432"
        - name: DATABASE_USER
          valueFrom:
            secretKeyRef:
              name: database-secret
              key: username
      
      containers:
      - name: migration-app
        image: migration-app:v2.0.0
        ports:
        - containerPort: 8080
        
        # Startup probe with extended timeout for migrations
        startupProbe:
          httpGet:
            path: /migration/status
            port: 8080
          periodSeconds: 30
          failureThreshold: 20    # Allow 10 minutes for migration
          timeoutSeconds: 10
        
        # Readiness probe
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
          periodSeconds: 10
          failureThreshold: 3
        
        # Liveness probe (disabled during migration)
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          periodSeconds: 60
          failureThreshold: 5
          initialDelaySeconds: 300  # Wait 5 minutes after startup
        
        # Lifecycle hooks for graceful shutdown
        lifecycle:
          preStop:
            exec:
              command:
              - /bin/sh
              - -c
              - |
                echo "Starting graceful shutdown..."
                
                # Mark as shutting down
                curl -X POST http://localhost:8080/admin/shutdown-prepare
                
                # Wait for ongoing migrations to complete
                timeout=60
                while [ $timeout -gt 0 ]; do
                  if curl -s http://localhost:8080/migration/status | grep -q "idle"; then
                    echo "No active migrations, proceeding with shutdown"
                    break
                  fi
                  echo "Waiting for migrations to complete... ($timeout seconds remaining)"
                  sleep 5
                  timeout=$((timeout - 5))
                done
                
                # Final shutdown
                curl -X POST http://localhost:8080/admin/shutdown
        
        # Resource allocation for migration workload
        resources:
          requests:
            cpu: 1
            memory: 2Gi
          limits:
            cpu: 4
            memory: 8Gi
        
        # Environment configuration
        env:
        - name: MIGRATION_MODE
          value: "enabled"
        - name: SHUTDOWN_TIMEOUT
          value: "60"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: database-secret
              key: url
        
        volumeMounts:
        - name: migration-data
          mountPath: /data/migrations
        - name: backup-volume
          mountPath: /backups
      
      volumes:
      - name: migration-data
        configMap:
          name: migration-scripts
      - name: backup-volume
        persistentVolumeClaim:
          claimName: backup-pvc

---
# PVC for backup storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: backup-pvc
  namespace: production
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
  storageClassName: fast-ssd
```

---

## Advanced Interview Questions for Senior Java Developers (15+ Years Experience)

### Docker Interview Questions

#### **1. Advanced Docker Architecture and Internals**

**Q1:** Explain the difference between Docker's overlay2 storage driver and devicemapper. When would you choose one over the other for a Java application deployment?

**Expected Answer:**
- Overlay2 uses union filesystems and is more efficient for read-heavy workloads
- Devicemapper uses block-level storage and provides better write performance
- For Java applications with large JARs, overlay2 is preferred due to layer sharing
- Devicemapper might be chosen for applications with heavy write operations
- Performance implications of copy-on-write vs block-level operations

**Q2:** How would you optimize a Docker image for a Spring Boot application that uses multiple fat JARs and has a startup time requirement of under 30 seconds?

**Expected Answer:**
```dockerfile
# Multi-stage build with layer optimization
FROM openjdk:17-jdk-slim as builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src src
RUN mvn package -DskipTests

FROM openjdk:17-jre-slim
# Create app user for security
RUN groupadd -r app && useradd -r -g app app
# Install required tools
RUN apt-get update && apt-get install -y dumb-init && rm -rf /var/lib/apt/lists/*
WORKDIR /app
# Copy dependencies first (better caching)
COPY --from=builder /app/target/dependency/ ./
# Copy application JAR
COPY --from=builder /app/target/app.jar app.jar
# Optimize JVM for containers
ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0 -XX:+UseG1GC -XX:+UnlockExperimentalVMOptions -XX:+UseJVMCICompiler"
USER app
ENTRYPOINT ["dumb-init", "--"]
CMD ["java", "$JAVA_OPTS", "-jar", "app.jar"]
```

**Q3:** Describe the security implications of running Java applications in containers and how you would implement defense-in-depth security.

**Expected Answer:**
- Non-root user execution
- Read-only root filesystem
- Resource limits to prevent DoS
- Network policies and segmentation
- Image scanning for vulnerabilities
- Secrets management (not in environment variables)
- AppArmor/SELinux profiles
- Regular base image updates

#### **2. Docker Networking and Java Applications**

**Q4:** You have a microservices architecture with 20+ Java services. How would you design the Docker networking to ensure optimal performance and security?

**Expected Answer:**
- Use custom bridge networks for service isolation
- Implement service mesh (Istio) for advanced traffic management
- Configure DNS resolution for service discovery
- Use overlay networks for multi-host deployment
- Implement network policies for security
- Consider using macvlan for performance-critical services
- Load balancing strategies (round-robin, least connections)

**Q5:** Explain how you would troubleshoot a Java application that's experiencing intermittent connection timeouts in a Docker swarm environment.

**Expected Answer:**
- Check Docker daemon logs and container logs
- Analyze network connectivity using `docker exec` and networking tools
- Monitor DNS resolution issues
- Check for resource constraints (CPU/memory throttling)
- Analyze service mesh metrics if applicable
- Use distributed tracing to identify bottlenecks
- Check for port conflicts and firewall rules

#### **3. Docker Performance and Monitoring**

**Q6:** How would you monitor and optimize the performance of Java applications running in Docker containers at scale?

**Expected Answer:**
```yaml
# Docker Compose with monitoring stack
version: '3.8'
services:
  app:
    image: java-app:latest
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 4G
        reservations:
          cpus: '1.0'
          memory: 2G
    environment:
      - JAVA_OPTS=-XX:+UseContainerSupport -XX:MaxRAMPercentage=75
    labels:
      - "prometheus.io/scrape=true"
      - "prometheus.io/port=8080"
      - "prometheus.io/path=/actuator/prometheus"
  
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
  
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
```

### Kubernetes Interview Questions

#### **4. Advanced Kubernetes Architecture**

**Q7:** Explain the role of etcd in a Kubernetes cluster and how you would backup and restore it for a production Java application deployment.

**Expected Answer:**
- etcd stores all cluster data and configuration
- Backup strategies using etcdctl snapshot
- Point-in-time recovery procedures
- Disaster recovery planning
- Encryption at rest configuration
- High availability setup with multiple etcd nodes

**Q8:** How would you design a Kubernetes cluster for a Java-based e-commerce platform that needs to handle Black Friday traffic (10x normal load)?

**Expected Answer:**
```yaml
# Cluster autoscaler configuration
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
spec:
  template:
    spec:
      containers:
      - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.21.0
        name: cluster-autoscaler
        command:
        - ./cluster-autoscaler
        - --v=4
        - --stderrthreshold=info
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/ecommerce-cluster
        - --balance-similar-node-groups
        - --skip-nodes-with-system-pods=false
        - --scale-down-delay-after-add=10m
        - --scale-down-unneeded-time=10m
```

#### **5. Resource Management and Performance**

**Q9:** You have a Java application that's experiencing OOMKilled errors intermittently. Walk me through your debugging and resolution process.

**Expected Answer:**
1. **Immediate Investigation:**
   ```bash
   kubectl describe pod <pod-name>
   kubectl logs <pod-name> --previous
   kubectl top pod <pod-name>
   ```

2. **Memory Analysis:**
   ```yaml
   resources:
     requests:
       memory: "2Gi"
     limits:
       memory: "4Gi"
   env:
   - name: JAVA_OPTS
     value: "-Xmx3g -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp"
   ```

3. **Long-term Monitoring:**
   - JVM metrics via Micrometer
   - Memory profiling with tools like VisualVM
   - Application Performance Monitoring (APM)

**Q10:** How would you implement autoscaling for a Java microservice that has varying load patterns throughout the day?

**Expected Answer:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: java-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: java-app
  minReplicas: 3
  maxReplicas: 50
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
  - type: Pods
    pods:
      metric:
        name: java_app_requests_per_second
      target:
        type: AverageValue
        averageValue: "100"
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 100
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 600
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
```

#### **6. Advanced Deployment Strategies**

**Q11:** Design a deployment strategy for a critical Java payment service that requires zero-downtime deployments and instant rollback capability.

**Expected Answer:**
```yaml
# Blue-Green deployment with Argo Rollouts
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: payment-service
spec:
  strategy:
    blueGreen:
      activeService: payment-service-active
      previewService: payment-service-preview
      autoPromotionEnabled: false
      scaleDownDelaySeconds: 30
      prePromotionAnalysis:
        templates:
        - templateName: payment-success-rate
        - templateName: payment-latency
        args:
        - name: service-name
          value: payment-service-preview
      postPromotionAnalysis:
        templates:
        - templateName: payment-success-rate
        - templateName: payment-latency
        args:
        - name: service-name
          value: payment-service-active
  selector:
    matchLabels:
      app: payment-service
  template:
    metadata:
      labels:
        app: payment-service
    spec:
      containers:
      - name: payment-service
        image: payment-service:v1.0.0
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 30
```

**Q12:** How would you handle database migrations in a Kubernetes environment for a Java application without causing downtime?

**Expected Answer:**
```yaml
# Migration Job with Init Container pattern
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration-v2
spec:
  template:
    spec:
      initContainers:
      - name: migration-check
        image: migration-tool:latest
        command:
        - /bin/sh
        - -c
        - |
          # Check current schema version
          current_version=$(psql $DATABASE_URL -t -c "SELECT version FROM schema_version ORDER BY version DESC LIMIT 1;")
          target_version="2.0.0"
          
          if [ "$current_version" = "$target_version" ]; then
            echo "Migration already applied"
            exit 0
          fi
          
          # Run backward compatibility checks
          /migration-tools/compatibility-check.sh $current_version $target_version
      containers:
      - name: migration
        image: migration-tool:latest
        command:
        - /bin/sh
        - -c
        - |
          # Apply migrations
          flyway -url=$DATABASE_URL -user=$DB_USER -password=$DB_PASSWORD migrate
          
          # Verify migration success
          flyway -url=$DATABASE_URL -user=$DB_USER -password=$DB_PASSWORD validate
      restartPolicy: Never
  backoffLimit: 3
```

#### **7. Security and Compliance**

**Q13:** Implement Pod Security Standards for a Java application that processes PCI-compliant data.

**Expected Answer:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: pci-compliant
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pci-java-app
  namespace: pci-compliant
spec:
  template:
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        runAsGroup: 1000
        fsGroup: 1000
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: app
        image: pci-app:latest
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          capabilities:
            drop:
            - ALL
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: var-logs
          mountPath: /var/logs
        resources:
          limits:
            cpu: "2"
            memory: "4Gi"
          requests:
            cpu: "1"
            memory: "2Gi"
      volumes:
      - name: tmp
        emptyDir: {}
      - name: var-logs
        emptyDir: {}
```

**Q14:** How would you implement secret rotation for a Java application that connects to multiple databases and external APIs?

**Expected Answer:**
```yaml
# External Secrets Operator with AWS Secrets Manager
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secret-store
  namespace: production
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-west-2
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa

---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: java-app-secrets
  namespace: production
spec:
  refreshInterval: 15m
  secretStoreRef:
    name: aws-secret-store
    kind: SecretStore
  target:
    name: java-app-secrets
    creationPolicy: Owner
  data:
  - secretKey: database-password
    remoteRef:
      key: java-app/database
      property: password
  - secretKey: api-key
    remoteRef:
      key: java-app/external-api
      property: key
```

#### **8. Observability and Troubleshooting**

**Q15:** Design a comprehensive observability strategy for a Java microservices architecture running on Kubernetes.

**Expected Answer:**
```yaml
# Prometheus monitoring with ServiceMonitor
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: java-app-metrics
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: java-microservice
  endpoints:
  - port: metrics
    path: /actuator/prometheus
    interval: 30s

---
# Jaeger tracing configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: jaeger-config
data:
  application.yml: |
    opentracing:
      jaeger:
        service-name: java-microservice
        sampler:
          type: probabilistic
          param: 0.1
        reporter:
          log-spans: false
          flush-interval: 1000
          max-queue-size: 10000
```

**Q16:** You have a Java application that's experiencing high latency spikes. Describe your troubleshooting methodology using Kubernetes tools.

**Expected Answer:**
1. **Initial Investigation:**
   ```bash
   kubectl top nodes
   kubectl top pods -n production
   kubectl describe pod <problematic-pod>
   kubectl logs <pod> --tail=100 -f
   ```

2. **Performance Analysis:**
   ```bash
   # Check resource utilization
   kubectl exec -it <pod> -- top
   
   # Java-specific debugging
   kubectl exec -it <pod> -- jstat -gc <pid> 5s
   kubectl exec -it <pod> -- jstack <pid>
   ```

3. **Network Analysis:**
   ```bash
   # Check service endpoints
   kubectl get endpoints
   
   # Network connectivity
   kubectl exec -it <pod> -- netstat -tulpn
   kubectl exec -it <pod> -- ss -tulpn
   ```

#### **9. Advanced Scenarios and Problem Solving**

**Q17:** Your Java application cluster is experiencing cascading failures during peak traffic. How would you implement circuit breakers and bulkheads at the Kubernetes level?

**Expected Answer:**
```yaml
# Istio DestinationRule with Circuit Breaker
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: java-app-circuit-breaker
spec:
  host: java-app-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        maxRequestsPerConnection: 10
    outlierDetection:
      consecutiveErrors: 3
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
    loadBalancer:
      simple: LEAST_CONN

---
# Pod Disruption Budget for resilience
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: java-app-pdb
spec:
  minAvailable: 70%
  selector:
    matchLabels:
      app: java-app
```

**Q18:** Design a disaster recovery strategy for a stateful Java application (with persistent data) running on Kubernetes.

**Expected Answer:**
```yaml
# Velero backup configuration
apiVersion: v1
kind: Schedule
metadata:
  name: java-app-backup
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  template:
    includedNamespaces:
    - production
    includedResources:
    - persistentvolumes
    - persistentvolumeclaims
    - secrets
    - configmaps
    labelSelector:
      matchLabels:
        app: java-app
    hooks:
      resources:
      - name: postgres-backup-hook
        includedNamespaces:
        - production
        excludedNamespaces: []
        includedResources: []
        excludedResources: []
        labelSelector:
          matchLabels:
            app: postgres
        pre:
        - exec:
            container: postgres
            command:
            - /bin/bash
            - -c
            - pg_dump -h localhost -U postgres myapp > /backup/dump.sql
        post:
        - exec:
            container: postgres
            command:
            - /bin/bash
            - -c
            - rm -f /backup/dump.sql
```

#### **10. Architecture and Design Questions**

**Q19:** You need to migrate a monolithic Java application to microservices on Kubernetes. Describe your approach, including the challenges and solutions.

**Expected Answer:**
- **Strangler Fig Pattern:** Gradually replace monolith components
- **Database Decomposition:** Strategy for data separation
- **Service Mesh:** For inter-service communication
- **Distributed Tracing:** Observability across services
- **Configuration Management:** Externalized configuration
- **Gradual Rollout:** Feature flags and canary deployments

**Q20:** How would you implement multi-tenancy for a Java SaaS application on Kubernetes while ensuring security and resource isolation?

**Expected Answer:**
```yaml
# Namespace per tenant with resource quotas
apiVersion: v1
kind: Namespace
metadata:
  name: tenant-abc123
  labels:
    tenant-id: abc123
    security-profile: standard

---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: tenant-quota
  namespace: tenant-abc123
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    count/pods: "50"
    count/services: "10"

---
# Network Policy for tenant isolation
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: tenant-isolation
  namespace: tenant-abc123
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: istio-system
    - namespaceSelector:
        matchLabels:
          tenant-id: abc123
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: shared-services
  - to: []
    ports:
    - protocol: TCP
      port: 443
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
```

### Scenario-Based Problem Solving Questions

#### **21. Production Incident Response**

**Scenario:** "It's Black Friday, your e-commerce Java application is running on Kubernetes, and you're getting reports of 504 Gateway Timeout errors. The application was working fine yesterday. Walk me through your incident response."

**Expected Response Process:**

**1. Immediate Assessment (0-5 minutes)**
```bash
# Check overall cluster health
kubectl get nodes
kubectl top nodes

# Check pod status and resource usage
kubectl get pods -n production
kubectl top pods -n production --sort-by=cpu
kubectl top pods -n production --sort-by=memory

# Check service endpoints
kubectl get endpoints -n production
kubectl describe service ecommerce-app -n production

# Quick log analysis
kubectl logs -n production deployment/ecommerce-app --tail=50 --since=5m
```

**2. Triage and Stabilization (5-15 minutes)**
```bash
# Scale up immediately if resource constrained
kubectl scale deployment ecommerce-app --replicas=20 -n production

# Check ingress controller status
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller --tail=100

# Verify HPA status
kubectl describe hpa ecommerce-app-hpa -n production

# Check for any recent deployments
kubectl rollout history deployment/ecommerce-app -n production

# Emergency circuit breaker activation
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: ecommerce-emergency-circuit-breaker
  namespace: production
spec:
  host: ecommerce-app
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 50
      http:
        http1MaxPendingRequests: 20
        maxRequestsPerConnection: 5
    outlierDetection:
      consecutiveErrors: 2
      interval: 10s
      baseEjectionTime: 10s
      maxEjectionPercent: 80
EOF
```

**3. Root Cause Analysis (15-45 minutes)**
```bash
# Deep dive into metrics
kubectl port-forward -n monitoring service/prometheus 9090:9090 &
# Query Prometheus for:
# - HTTP request rate: rate(http_requests_total[5m])
# - Error rate: rate(http_requests_total{status=~"5.."}[5m])
# - Response time: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
# - JVM metrics: jvm_memory_used_bytes, jvm_gc_pause_seconds

# Check database connections
kubectl exec -n production deployment/ecommerce-app -- \
  curl -s http://localhost:8080/actuator/health/db

# Analyze JVM performance
kubectl exec -n production deployment/ecommerce-app -- \
  jstat -gc $(pgrep java) 5s 10

# Check for memory leaks
kubectl exec -n production deployment/ecommerce-app -- \
  jmap -histo $(pgrep java) | head -20

# Network connectivity tests
kubectl exec -n production deployment/ecommerce-app -- \
  curl -w "@curl-format.txt" -s -o /dev/null http://database-service:5432
```

**4. Resolution and Recovery (45+ minutes)**
```yaml
# If root cause is resource exhaustion
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecommerce-app-emergency-fix
  namespace: production
spec:
  replicas: 30
  template:
    spec:
      containers:
      - name: app
        resources:
          requests:
            cpu: "2"
            memory: "4Gi"
          limits:
            cpu: "4"
            memory: "8Gi"
        env:
        - name: JAVA_OPTS
          value: |
            -XX:+UseG1GC
            -XX:MaxGCPauseMillis=100
            -XX:+UseContainerSupport
            -XX:MaxRAMPercentage=70.0
            -XX:+UnlockDiagnosticVMOptions
            -XX:+LogVMOutput
            -XX:LogFile=/tmp/jvm.log
            -Dserver.tomcat.max-threads=400
            -Dserver.tomcat.accept-count=200
            -Dspring.datasource.hikari.maximum-pool-size=20
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          periodSeconds: 2
          failureThreshold: 2
```

**5. Post-Incident Review**
- Document timeline and root cause
- Update runbooks and monitoring
- Implement permanent fixes
- Conduct blameless post-mortem
- Update capacity planning models

#### **22. Capacity Planning**

**Scenario:** "Your Java application currently handles 1000 requests per second with 10 pods. Marketing says traffic will increase 5x in the next month. How do you prepare your Kubernetes infrastructure?"

**Expected Answer:**

**1. Current State Analysis**
```bash
# Baseline current performance
kubectl top pods -n production
kubectl get hpa -n production

# Current resource utilization
# Assuming 1000 RPS with 10 pods = 100 RPS per pod
# Current CPU: 70% (target threshold)
# Current Memory: 60%
```

**2. Resource Requirement Calculations**
```yaml
# Current Configuration
current_pods: 10
current_rps_per_pod: 100
target_total_rps: 5000  # 5x increase
target_rps_per_pod: 100  # Keep same per-pod performance

# Required pods calculation
required_pods: 50  # 5000 / 100

# Resource scaling calculation
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecommerce-app-scaled
  namespace: production
spec:
  replicas: 50
  template:
    spec:
      containers:
      - name: app
        resources:
          requests:
            cpu: "1"      # Same per pod
            memory: "2Gi" # Same per pod
          limits:
            cpu: "2"
            memory: "4Gi"

---
# Updated HPA for higher scale
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ecommerce-app-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ecommerce-app
  minReplicas: 30    # Higher baseline
  maxReplicas: 100   # Higher ceiling
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
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "80"  # Target 80 RPS per pod
```

**3. Infrastructure Scaling**
```yaml
# Cluster Autoscaler Configuration
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  template:
    spec:
      containers:
      - name: cluster-autoscaler
        command:
        - ./cluster-autoscaler
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/ecommerce-prod
        - --scale-down-delay-after-add=5m
        - --scale-down-unneeded-time=5m
        - --max-nodes-total=200      # Increased from 50
        - --cores-total=0:1000       # Increased CPU capacity
        - --memory-total=0:4000      # Increased memory capacity

---
# Node Pool Configuration for High Traffic
apiVersion: v1
kind: ConfigMap
metadata:
  name: high-traffic-node-pool
data:
  node-pool.yaml: |
    # AWS EKS Node Group
    nodeGroups:
    - name: high-performance-nodes
      instanceType: c5.2xlarge  # 8 vCPU, 16 GB RAM
      minSize: 20
      maxSize: 100
      desiredCapacity: 30
      volumeSize: 100
      labels:
        workload-type: high-traffic
      taints:
        - key: high-traffic
          value: "true"
          effect: NoSchedule
```

**4. Database Scaling Strategy**
```yaml
# Database Connection Pool Scaling
apiVersion: v1
kind: ConfigMap
metadata:
  name: database-config-scaled
  namespace: production
data:
  application.yml: |
    spring:
      datasource:
        hikari:
          maximum-pool-size: 50      # Increased from 20
          minimum-idle: 20           # Increased from 10
          connection-timeout: 20000
          idle-timeout: 300000
          max-lifetime: 900000
          leak-detection-threshold: 60000

# Read Replica Configuration
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecommerce-app-read-optimized
spec:
  template:
    spec:
      containers:
      - name: app
        env:
        - name: DATABASE_READ_URL
          value: "jdbc:postgresql://postgres-read-replica:5432/ecommerce"
        - name: DATABASE_WRITE_URL
          value: "jdbc:postgresql://postgres-primary:5432/ecommerce"
```

**5. Caching and CDN Strategy**
```yaml
# Redis Cluster for Caching
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
  namespace: production
spec:
  replicas: 6  # 3 masters, 3 slaves
  template:
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        resources:
          requests:
            cpu: "500m"
            memory: "2Gi"
          limits:
            cpu: "1"
            memory: "4Gi"

---
# Application Cache Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: cache-config
data:
  application.yml: |
    spring:
      cache:
        type: redis
        redis:
          cluster:
            nodes: redis-cluster-0:6379,redis-cluster-1:6379,redis-cluster-2:6379
          time-to-live: 300000
    
    # Cache strategy
    cache:
      product-catalog:
        ttl: 1800      # 30 minutes
        max-size: 10000
      user-sessions:
        ttl: 3600      # 1 hour
        max-size: 50000
```

**6. Load Testing and Validation**
```yaml
# Load Testing Job
apiVersion: batch/v1
kind: Job
metadata:
  name: capacity-load-test
  namespace: testing
spec:
  template:
    spec:
      containers:
      - name: load-test
        image: loadimpact/k6
        command:
        - k6
        - run
        - --vus=1000         # 1000 virtual users
        - --duration=30m     # 30 minute test
        - --rps=5000         # Target 5000 RPS
        - /scripts/load-test.js
        volumeMounts:
        - name: test-scripts
          mountPath: /scripts
      volumes:
      - name: test-scripts
        configMap:
          name: load-test-scripts

---
# Monitoring and Alerting Updates
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: high-traffic-alerts
  namespace: monitoring
spec:
  groups:
  - name: high-traffic.rules
    rules:
    - alert: HighRequestLatency
      expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 0.5
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: "High request latency detected"
    
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "Error rate above 5%"
    
    - alert: PodScalingNeeded
      expr: kube_deployment_status_replicas{deployment="ecommerce-app"} > 80
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Consider increasing max replicas"
```

#### **23. Security Incident**

**Scenario:** "You discovered that one of your Java microservices has a critical security vulnerability that could allow remote code execution. The service is running in production with 100+ pods across multiple clusters. How do you handle this?"

**Expected Response:**

**1. Immediate Containment (0-15 minutes)**
```bash
# Step 1: Identify affected services and versions
kubectl get pods -A -o wide | grep vulnerable-service:v1.2.3

# Step 2: Implement network isolation
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: emergency-isolation-vulnerable-service
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: vulnerable-service
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: system
    ports:
    - protocol: TCP
      port: 53
EOF

# Step 3: Enable additional monitoring
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: emergency-monitoring
  namespace: production
data:
  filebeat.yml: |
    filebeat.inputs:
    - type: container
      paths:
        - /var/log/containers/*vulnerable-service*.log
      processors:
      - add_kubernetes_metadata:
          host: \${NODE_NAME}
          matchers:
          - logs_path:
              logs_path: "/var/log/containers/"
              resource_type: "container"
      fields:
        security_incident: "RCE-2024-001"
        severity: "critical"
EOF
```

**2. Assessment of Blast Radius (15-30 minutes)**
```bash
# Identify all affected clusters and environments
for cluster in prod-us-east prod-us-west prod-eu-west staging; do
  echo "=== Checking cluster: $cluster ==="
  kubectl --context=$cluster get pods -A -l app=vulnerable-service -o wide
  kubectl --context=$cluster get deployments -A -l app=vulnerable-service
done

# Check service dependencies
kubectl get networkpolicies -A | grep vulnerable-service
kubectl get services -A -l app=vulnerable-service
kubectl get ingress -A | grep vulnerable-service

# Review recent access logs
kubectl logs -n production deployment/vulnerable-service --since=24h | \
  grep -E "(POST|PUT|PATCH)" | \
  grep -E "\.(jsp|php|sh|bash|cmd|exe)" > potential_exploitation.log

# Check for suspicious activities
kubectl top pods -n production -l app=vulnerable-service --sort-by=cpu
kubectl exec -n production deployment/vulnerable-service -- \
  ps aux | grep -E "(sh|bash|cmd|powershell)"
```

**3. Emergency Patching Procedures (30-60 minutes)**
```yaml
# Emergency hotfix deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vulnerable-service-hotfix
  namespace: production
  annotations:
    security.incident: "RCE-2024-001"
    patched.version: "v1.2.4-security-hotfix"
spec:
  replicas: 100
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 10%
      maxSurge: 20%
  template:
    metadata:
      labels:
        app: vulnerable-service
        version: v1.2.4-security-hotfix
    spec:
      # Enhanced security context
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        fsGroup: 10001
        seccompProfile:
          type: RuntimeDefault
      
      containers:
      - name: app
        image: vulnerable-service:v1.2.4-security-hotfix
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          capabilities:
            drop:
            - ALL
            add:
            - NET_BIND_SERVICE
        
        # Immediate hardening
        env:
        - name: JAVA_OPTS
          value: |
            -XX:+UseContainerSupport
            -XX:MaxRAMPercentage=70.0
            -Djava.security.manager
            -Djava.security.policy=/security/java.policy
            -Dcom.sun.management.jmxremote=false
            -Dlog4j2.formatMsgNoLookups=true
        
        volumeMounts:
        - name: security-policy
          mountPath: /security
          readOnly: true
        - name: tmp
          mountPath: /tmp
        - name: logs
          mountPath: /logs
      
      volumes:
      - name: security-policy
        configMap:
          name: java-security-policy
      - name: tmp
        emptyDir: {}
      - name: logs
        emptyDir: {}

---
# Enhanced monitoring for security incident
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: security-incident-monitoring
  namespace: production
spec:
  selector:
    matchLabels:
      app: vulnerable-service
  endpoints:
  - port: metrics
    interval: 10s  # Increased frequency
    path: /actuator/prometheus
```

**4. Communication Protocols**
```yaml
# Incident notification configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: security-incident-notifications
  namespace: monitoring
data:
  notification-template.json: |
    {
      "incident_id": "RCE-2024-001",
      "severity": "CRITICAL",
      "status": "ACTIVE",
      "affected_services": ["vulnerable-service"],
      "affected_clusters": ["prod-us-east", "prod-us-west", "prod-eu-west"],
      "estimated_pods_affected": 300,
      "containment_status": "IN_PROGRESS",
      "patch_status": "DEPLOYING",
      "business_impact": "LIMITED_FUNCTIONALITY",
      "estimated_resolution": "2024-11-23T18:00:00Z",
      "communication_channels": {
        "slack": "#security-incidents",
        "pagerduty": "P1-SECURITY",
        "email": "security-team@company.com"
      }
    }
```

**5. Recovery and Validation (60+ minutes)**
```bash
# Validation script for patched services
#!/bin/bash
echo "Starting security validation for RCE-2024-001 hotfix..."

# Check all pods are running patched version
for cluster in prod-us-east prod-us-west prod-eu-west; do
  echo "Validating cluster: $cluster"
  
  # Verify image versions
  kubectl --context=$cluster get pods -n production -l app=vulnerable-service \
    -o jsonpath='{.items[*].spec.containers[*].image}' | \
    grep -q "v1.2.4-security-hotfix"
  
  if [ $? -eq 0 ]; then
    echo "✓ $cluster: All pods running patched version"
  else
    echo "✗ $cluster: Some pods still running vulnerable version"
    exit 1
  fi
  
  # Test exploit mitigation
  kubectl --context=$cluster exec -n production deployment/vulnerable-service -- \
    curl -X POST -H "Content-Type: application/xml" \
    -d '<?xml version="1.0"?><!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]><foo>&xxe;</foo>' \
    http://localhost:8080/api/vulnerable-endpoint
  
  # Should return 400 Bad Request, not file contents
done

# Security audit
kubectl get networkpolicies -A | grep vulnerable-service
kubectl get podsecuritypolicies | grep vulnerable-service
kubectl auth can-i --list --as=system:serviceaccount:production:vulnerable-service

echo "Security validation completed"
```

**6. Security Audit and Hardening**
```yaml
# Post-incident security hardening
apiVersion: v1
kind: ConfigMap
metadata:
  name: security-hardening-checklist
data:
  checklist.md: |
    # Security Hardening Checklist - Post RCE-2024-001
    
    ## Immediate Actions Completed
    - [x] Network isolation implemented
    - [x] Emergency patching deployed
    - [x] Enhanced monitoring activated
    - [x] Incident communication sent
    - [x] Service validation completed
    
    ## Follow-up Actions Required
    - [ ] Comprehensive security audit of all Java services
    - [ ] Update base container images
    - [ ] Implement runtime security monitoring
    - [ ] Review and update security policies
    - [ ] Conduct penetration testing
    - [ ] Update incident response procedures
    - [ ] Security training for development teams
    
    ## Long-term Improvements
    - [ ] Implement admission controllers
    - [ ] Deploy Falco for runtime security
    - [ ] Establish security scanning in CI/CD
    - [ ] Regular vulnerability assessments
    - [ ] Security chaos engineering

---
# Runtime security monitoring with Falco
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: falco-security-monitoring
  namespace: security
spec:
  template:
    spec:
      containers:
      - name: falco
        image: falcosecurity/falco:latest
        securityContext:
          privileged: true
        volumeMounts:
        - name: dev
          mountPath: /host/dev
        - name: proc
          mountPath: /host/proc
        - name: boot
          mountPath: /host/boot
        - name: lib-modules
          mountPath: /host/lib/modules
        - name: usr
          mountPath: /host/usr
        - name: etc
          mountPath: /host/etc
        env:
        - name: FALCO_GRPC_ENABLED
          value: "true"
        - name: FALCO_GRPC_BIND_ADDRESS
          value: "0.0.0.0:5060"
      volumes:
      - name: dev
        hostPath:
          path: /dev
      - name: proc
        hostPath:
          path: /proc
      - name: boot
        hostPath:
          path: /boot
      - name: lib-modules
        hostPath:
          path: /lib/modules
      - name: usr
        hostPath:
          path: /usr
      - name: etc
        hostPath:
          path: /etc
```

**Key Response Metrics:**
- **Time to Containment**: < 15 minutes
- **Time to Patch**: < 60 minutes  
- **Time to Full Recovery**: < 2 hours
- **False Positive Rate**: < 5%
- **Service Availability**: > 99.9% maintained

### Technical Deep-Dive Questions

#### **24. JVM Optimization in Containers**

**Q:** "Explain how you would tune JVM parameters for a Java application running in Kubernetes containers, considering different workload patterns (CPU-intensive vs memory-intensive vs I/O-intensive)."

**Expected Answer:**

```yaml
# CPU-Intensive Workload Configuration
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cpu-intensive-app
spec:
  template:
    spec:
      containers:
      - name: app
        image: cpu-intensive-app:latest
        env:
        - name: JAVA_OPTS
          value: |
            -XX:+UseG1GC
            -XX:MaxGCPauseMillis=200
            -XX:+UseContainerSupport
            -XX:MaxRAMPercentage=70.0
            -XX:+UnlockExperimentalVMOptions
            -XX:+UseJVMCICompiler
            -XX:ParallelGCThreads=8
            -XX:ConcGCThreads=2
            -XX:G1HeapRegionSize=16m
            -XX:+AggressiveOpts
            -XX:+UseFastAccessorMethods
            -Djava.awt.headless=true
        resources:
          requests:
            cpu: "4"
            memory: "8Gi"
          limits:
            cpu: "8"
            memory: "12Gi"

---
# Memory-Intensive Workload Configuration
apiVersion: apps/v1
kind: Deployment
metadata:
  name: memory-intensive-app
spec:
  template:
    spec:
      containers:
      - name: app
        image: memory-intensive-app:latest
        env:
        - name: JAVA_OPTS
          value: |
            -XX:+UseG1GC
            -XX:MaxGCPauseMillis=100
            -XX:+UseContainerSupport
            -XX:MaxRAMPercentage=80.0
            -XX:G1HeapRegionSize=32m
            -XX:G1NewSizePercent=20
            -XX:G1MaxNewSizePercent=30
            -XX:G1MixedGCCountTarget=8
            -XX:InitiatingHeapOccupancyPercent=35
            -XX:+UnlockDiagnosticVMOptions
            -XX:+G1PrintRegionRememberedSetInfo
            -XX:+UseStringDeduplication
            -Djava.awt.headless=true
        resources:
          requests:
            cpu: "2"
            memory: "16Gi"
          limits:
            cpu: "4"
            memory: "24Gi"

---
# I/O-Intensive Workload Configuration
apiVersion: apps/v1
kind: Deployment
metadata:
  name: io-intensive-app
spec:
  template:
    spec:
      containers:
      - name: app
        image: io-intensive-app:latest
        env:
        - name: JAVA_OPTS
          value: |
            -XX:+UseParallelGC
            -XX:+UseContainerSupport
            -XX:MaxRAMPercentage=60.0
            -XX:ParallelGCThreads=4
            -XX:MaxGCPauseMillis=50
            -XX:GCTimeRatio=9
            -XX:NewRatio=3
            -XX:SurvivorRatio=8
            -XX:+UseNUMA
            -XX:+PreferContainerQuotaForCPUCount
            -Djava.awt.headless=true
            -Djava.net.preferIPv4Stack=true
            -Djava.security.egd=file:/dev/./urandom
        resources:
          requests:
            cpu: "1"
            memory: "4Gi"
          limits:
            cpu: "2"
            memory: "6Gi"
        volumeMounts:
        - name: high-speed-storage
          mountPath: /data
      volumes:
      - name: high-speed-storage
        persistentVolumeClaim:
          claimName: nvme-ssd-pvc
```

**Key Considerations:**
- **Container-aware JVM**: Always use `-XX:+UseContainerSupport`
- **Memory allocation**: Use `-XX:MaxRAMPercentage` instead of `-Xmx`
- **GC selection**: G1GC for low latency, ParallelGC for throughput
- **CPU optimization**: Adjust GC threads based on available CPUs
- **I/O optimization**: Use faster storage and optimize network settings

#### **25. Service Mesh Architecture**

**Q:** "Compare Istio, Linkerd, and Consul Connect for a Java microservices architecture. What factors would influence your choice?"

**Expected Answer:**

| Feature | Istio | Linkerd | Consul Connect |
|---------|--------|---------|----------------|
| **Complexity** | High | Low | Medium |
| **Performance Overhead** | 5-10ms | 0.5-1ms | 2-5ms |
| **Memory Usage** | High (150-300MB) | Low (50-100MB) | Medium (100-200MB) |
| **Traffic Management** | Excellent | Good | Good |
| **Security** | Excellent | Excellent | Good |
| **Observability** | Excellent | Good | Good |
| **Multi-cluster** | Excellent | Good | Excellent |
| **Learning Curve** | Steep | Gentle | Moderate |

```yaml
# Istio Configuration for Java Microservices
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  name: java-microservices-mesh
spec:
  values:
    pilot:
      traceSampling: 1.0
    global:
      proxy:
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
  components:
    pilot:
      k8s:
        resources:
          requests:
            cpu: 500m
            memory: 2048Mi
    ingressGateways:
    - name: istio-ingressgateway
      enabled: true
      k8s:
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
        hpaSpec:
          maxReplicas: 5
          minReplicas: 2

---
# Linkerd Configuration for Java Microservices
apiVersion: linkerd.io/v1alpha2
kind: ServiceProfile
metadata:
  name: java-service
  namespace: production
spec:
  routes:
  - name: api-endpoints
    condition:
      method: GET
      pathRegex: "/api/.*"
    responseClasses:
    - condition:
        status:
          min: 200
          max: 299
      isFailure: false
    - condition:
        status:
          min: 500
          max: 599
      isFailure: true
    timeout: 30s
    retryBudget:
      retryRatio: 0.2
      minRetriesPerSecond: 10
      ttl: 10s

---
# Consul Connect Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: consul-connect-injector
data:
  config.yaml: |
    connectInject:
      enabled: true
      default: true
      imageEnvoy: envoyproxy/envoy:v1.24.0
      resources:
        requests:
          memory: "100Mi"
          cpu: "50m"
        limits:
          memory: "200Mi"
          cpu: "100m"
```

**Decision Factors:**

1. **Choose Istio if:**
   - You need advanced traffic management (canary, A/B testing)
   - You have complex security requirements
   - You can handle the operational complexity
   - You need multi-cluster federation

2. **Choose Linkerd if:**
   - You prioritize simplicity and performance
   - You want minimal operational overhead
   - You need basic service mesh features
   - You're new to service mesh

3. **Choose Consul Connect if:**
   - You're already using HashiCorp stack
   - You need multi-platform support (VMs + K8s)
   - You want integrated service discovery
   - You need dynamic configuration

#### **26. GitOps and CI/CD Integration**

**Q:** "Design a GitOps workflow for deploying Java applications to Kubernetes with proper testing, security scanning, and rollback capabilities."

**Expected Answer:**

```yaml
# GitHub Actions CI Pipeline
name: Java Application CI/CD
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven
    
    - name: Run Tests
      run: |
        mvn clean verify
        mvn jacoco:report
    
    - name: SonarQube Analysis
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      run: mvn sonar:sonar
    
    - name: Dependency Check
      run: mvn org.owasp:dependency-check-maven:check

  build-and-scan:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
    - uses: actions/checkout@v3
    
    - name: Build Docker Image
      run: |
        docker build -t ${{ github.repository }}:${{ github.sha }} .
        docker tag ${{ github.repository }}:${{ github.sha }} ${{ github.repository }}:latest
    
    - name: Security Scan with Trivy
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: '${{ github.repository }}:${{ github.sha }}'
        format: 'sarif'
        output: 'trivy-results.sarif'
    
    - name: Upload Trivy Results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'
    
    - name: Push to Registry
      run: |
        echo ${{ secrets.REGISTRY_PASSWORD }} | docker login -u ${{ secrets.REGISTRY_USERNAME }} --password-stdin
        docker push ${{ github.repository }}:${{ github.sha }}
        docker push ${{ github.repository }}:latest
    
    - name: Update Kubernetes Manifests
      run: |
        git config --global user.email "ci@company.com"
        git config --global user.name "CI Bot"
        git clone https://${{ secrets.GITOPS_TOKEN }}@github.com/company/k8s-manifests.git
        cd k8s-manifests
        sed -i 's|image: .*|image: ${{ github.repository }}:${{ github.sha }}|' apps/java-app/deployment.yaml
        git add -A
        git commit -m "Update java-app to ${{ github.sha }}"
        git push

---
# ArgoCD Application Configuration
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: java-microservice
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  
  source:
    repoURL: https://github.com/company/k8s-manifests
    targetRevision: HEAD
    path: apps/java-app
    
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m

---
# ArgoCD Rollouts for Progressive Delivery
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: java-app-rollout
  namespace: production
spec:
  replicas: 5
  strategy:
    canary:
      maxSurge: "25%"
      maxUnavailable: 0
      steps:
      - setWeight: 10
      - pause:
          duration: 2m
      - setWeight: 20
      - pause:
          duration: 2m
      - setWeight: 40
      - pause:
          duration: 2m
      - setWeight: 60
      - pause:
          duration: 2m
      - setWeight: 80
      - pause:
          duration: 2m
      analysis:
        templates:
        - templateName: success-rate
        - templateName: latency-p99
        args:
        - name: service-name
          value: java-app-canary
        startingStep: 2
        successfulRunHistoryLimit: 5
        unsuccessfulRunHistoryLimit: 3
      canaryService: java-app-canary
      stableService: java-app-stable
      trafficRouting:
        istio:
          virtualService:
            name: java-app-vs
            routes:
            - primary
          destinationRule:
            name: java-app-dr
            canarySubsetName: canary
            stableSubsetName: stable

---
# Analysis Templates for Automated Validation
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
spec:
  args:
  - name: service-name
  metrics:
  - name: success-rate
    successCondition: result[0] >= 0.95
    interval: 60s
    count: 5
    failureLimit: 3
    provider:
      prometheus:
        address: http://prometheus.monitoring.svc.cluster.local:9090
        query: |
          sum(irate(istio_requests_total{reporter="destination",destination_service_name="{{args.service-name}}",response_code!~"5.*"}[2m])) / 
          sum(irate(istio_requests_total{reporter="destination",destination_service_name="{{args.service-name}}"}[2m]))

---
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: latency-p99
spec:
  args:
  - name: service-name
  metrics:
  - name: latency-p99
    successCondition: result[0] <= 500
    interval: 60s
    count: 5
    failureLimit: 3
    provider:
      prometheus:
        address: http://prometheus.monitoring.svc.cluster.local:9090
        query: |
          histogram_quantile(0.99,
            sum(irate(istio_request_duration_milliseconds_bucket{reporter="destination",destination_service_name="{{args.service-name}}"}[2m])) by (le)
          )

---
# Notification Configuration for GitOps Events
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  config.yaml: |
    triggers:
      - name: on-deployed
        condition: app.status.operationState.phase in ['Succeeded'] and app.status.health.status == 'Healthy'
        template: app-deployed
      - name: on-health-degraded
        condition: app.status.health.status == 'Degraded'
        template: app-health-degraded
      - name: on-sync-failed
        condition: app.status.operationState.phase in ['Error', 'Failed']
        template: app-sync-failed
    
    templates:
      - name: app-deployed
        slack:
          attachments: |
            [{
              "title": "{{ .app.metadata.name}}",
              "title_link":"{{.context.argocdUrl}}/applications/{{.app.metadata.name}}",
              "color": "#18be52",
              "fields": [
              {
                "title": "Sync Status",
                "value": "{{.app.status.sync.status}}",
                "short": true
              },
              {
                "title": "Repository",
                "value": "{{.app.spec.source.repoURL}}",
                "short": true
              },
              {
                "title": "Revision",
                "value": "{{.app.status.sync.revision}}",
                "short": true
              }
              {{range $index, $c := .app.status.conditions}}
              {{if not $index}},{{end}}
              {{if $index}},{{end}}
              {
                "title": "{{$c.type}}",
                "value": "{{$c.message}}",
                "short": true
              }
              {{end}}
              ]
            }]

---
# Backup and Rollback Strategy
apiVersion: batch/v1
kind: CronJob
metadata:
  name: gitops-backup
  namespace: argocd
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: alpine/git:latest
            command:
            - /bin/sh
            - -c
            - |
              # Backup ArgoCD applications
              kubectl get applications -n argocd -o yaml > /backup/applications-$(date +%Y%m%d).yaml
              
              # Backup git repository
              git clone https://$GITOPS_TOKEN@github.com/company/k8s-manifests.git /backup/manifests-$(date +%Y%m%d)
              
              # Upload to S3 or backup storage
              aws s3 cp /backup/ s3://company-gitops-backup/ --recursive
            
            env:
            - name: GITOPS_TOKEN
              valueFrom:
                secretKeyRef:
                  name: gitops-secret
                  key: token
            - name: AWS_ACCESS_KEY_ID
              valueFrom:
                secretKeyRef:
                  name: aws-secret
                  key: access-key
            - name: AWS_SECRET_ACCESS_KEY
              valueFrom:
                secretKeyRef:
                  name: aws-secret
                  key: secret-key
            
            volumeMounts:
            - name: backup-storage
              mountPath: /backup
          
          volumes:
          - name: backup-storage
            emptyDir: {}
          
          restartPolicy: OnFailure
```

**GitOps Workflow Components:**

1. **CI Pipeline Features:**
   - Automated testing with coverage reports
   - Security scanning (OWASP, Trivy)
   - Code quality analysis (SonarQube)
   - Container image building and scanning
   - Automated manifest updates

2. **CD Pipeline Features:**
   - ArgoCD for declarative deployments
   - Progressive delivery with Argo Rollouts
   - Automated canary analysis
   - Instant rollback capabilities
   - Multi-environment promotion

3. **Monitoring and Alerting:**
   - Prometheus-based metrics
   - Success rate and latency monitoring
   - Slack notifications for deployment events
   - Automated rollback on failures

4. **Security and Compliance:**
   - Image vulnerability scanning
   - Dependency checks
   - RBAC for GitOps access
   - Audit trails for all changes

5. **Disaster Recovery:**
   - Automated backups of applications and manifests
   - Point-in-time recovery capabilities
   - Cross-region backup storage
   - Documented rollback procedures

These interview questions are designed to assess not just theoretical knowledge but practical experience with real-world challenges that senior developers face when working with Java applications in containerized environments.

---

## Ensuring Smooth Autoscaling - Production Best Practices

### Understanding Autoscaling Components

Kubernetes provides multiple layers of autoscaling that work together:

#### **1. Horizontal Pod Autoscaler (HPA)**
Scales the number of pod replicas based on metrics.

#### **2. Vertical Pod Autoscaler (VPA)**
Adjusts CPU and memory requests/limits for pods.

#### **3. Cluster Autoscaler (CA)**
Scales the number of nodes in the cluster.

#### **4. Custom Autoscalers**
Application-specific scaling based on business metrics.

### Key Principles for Smooth Autoscaling

#### **1. Proper Resource Management**

```yaml
# Correct Resource Configuration with Detailed Explanations
apiVersion: apps/v1              # Kubernetes API version for Deployments
kind: Deployment                 # Resource type - manages pod replicas
metadata:
  name: java-app-optimized       # Unique name for this deployment
  namespace: production          # Namespace isolation for production workloads
spec:
  replicas: 3                    # Initial number of pod instances
  selector:                      # How deployment finds its pods
    matchLabels:
      app: java-app              # Must match template labels below
  template:                      # Pod template specification
    metadata:
      labels:
        app: java-app            # Labels for pod identification
    spec:
      containers:
      - name: app                # Container name within the pod
        image: java-app:v1.0.0   # Container image with specific version tag
        
        # CRITICAL: Resource requests and limits for autoscaling
        resources:
          requests:              # Guaranteed resources - used for scheduling
            cpu: "500m"          # 500 millicores = 0.5 CPU core minimum
                                # HPA uses this for CPU percentage calculations
            memory: "1Gi"        # 1 GiB minimum memory guaranteed
                                # Kubernetes schedules pod only if node has this available
          limits:                # Maximum resources - prevents resource hogging
            cpu: "2"             # Maximum 2 CPU cores (400% of request)
                                # Container throttled if it exceeds this
            memory: "4Gi"        # Maximum 4 GiB memory (400% of request)
                                # Container killed (OOMKilled) if it exceeds this
        
        # Container ports for service discovery and health checks
        ports:
        - containerPort: 8080    # Application HTTP port
          name: http             # Named port for service reference
          protocol: TCP          # Protocol type (default: TCP)
        - containerPort: 9090    # Metrics port for Prometheus scraping
          name: metrics          # Named port for monitoring
          protocol: TCP
        
        # Optimized JVM settings for containers
        env:
        - name: JAVA_OPTS        # Environment variable for JVM configuration
          value: |               # Multi-line YAML string for JVM options
            -XX:+UseContainerSupport           # JVM recognizes container limits
            -XX:MaxRAMPercentage=75.0          # Use max 75% of container memory
            -XX:+UseG1GC                       # G1 garbage collector for low latency
            -XX:MaxGCPauseMillis=200           # Target max 200ms GC pause time
            -XX:InitialRAMPercentage=50.0      # Start with 50% of container memory
            -XX:MinRAMPercentage=50.0          # Minimum 50% of container memory
            -XX:+UnlockExperimentalVMOptions   # Enable experimental JVM features
            -XX:+UseStringDeduplication        # Optimize string memory usage
            -Djava.awt.headless=true          # Disable GUI components
            -Dspring.jmx.enabled=false        # Disable JMX for security
        
        # Fast and reliable readiness probe for traffic routing
        readinessProbe:
          httpGet:                           # HTTP GET health check
            path: /actuator/health/readiness # Spring Boot readiness endpoint
            port: 8080                       # Port to check (matches containerPort)
            scheme: HTTP                     # HTTP scheme (not HTTPS)
          initialDelaySeconds: 30            # Wait 30s after container start
                                            # Allows application to initialize
          periodSeconds: 5                   # Check every 5 seconds
                                            # Fast checks for quick scaling response
          timeoutSeconds: 3                  # Timeout after 3 seconds
                                            # Prevents hanging on slow responses
          failureThreshold: 3                # 3 consecutive failures = not ready
                                            # Removes pod from service endpoints
          successThreshold: 1                # 1 success = ready for traffic
                                            # Default value, pod receives traffic
        
        # Liveness probe to restart unhealthy containers
        livenessProbe:
          httpGet:                           # HTTP GET health check
            path: /actuator/health/liveness  # Spring Boot liveness endpoint
            port: 8080                       # Same port as readiness probe
            scheme: HTTP                     # HTTP scheme
          initialDelaySeconds: 60            # Wait 60s before first check
                                            # Longer delay for full startup
          periodSeconds: 30                  # Check every 30 seconds
                                            # Less frequent than readiness
          timeoutSeconds: 10                 # Timeout after 10 seconds
                                            # Longer timeout for thorough check
          failureThreshold: 3                # 3 failures = restart container
                                            # Gives multiple chances before restart
        
        # Optional: Startup probe for slow-starting applications
        startupProbe:
          httpGet:                           # HTTP GET health check
            path: /actuator/health/startup   # Spring Boot startup endpoint
            port: 8080                       # Same port as other probes
          periodSeconds: 10                  # Check every 10 seconds
          failureThreshold: 30               # Allow up to 300s (30*10) for startup
                                            # Protects liveness probe during startup
          timeoutSeconds: 5                  # Timeout after 5 seconds

---
# Pod Disruption Budget - Ensures availability during scaling operations
apiVersion: policy/v1            # API version for Pod Disruption Budgets
kind: PodDisruptionBudget        # Resource type for availability guarantees
metadata:
  name: java-app-pdb            # Unique name for this PDB
  namespace: production         # Same namespace as the deployment
spec:
  minAvailable: 75%             # Always keep at least 75% of pods running
                               # During voluntary disruptions (updates, scaling)
                               # Alternative: minAvailable: 3 (absolute number)
  # maxUnavailable: 25%         # Alternative: max 25% can be unavailable
  selector:                     # Which pods this PDB applies to
    matchLabels:
      app: java-app             # Must match deployment labels
  
  # How PDB works:
  # - Prevents voluntary evictions if it would violate the policy
  # - Used during rolling updates, node drains, cluster scaling
  # - Does NOT prevent involuntary disruptions (node failures)
  # - Ensures application availability during maintenance
```

**Resource Configuration Explained:**

**CPU Requests vs Limits:**
- **Request (500m)**: Guaranteed CPU allocation for scheduling and HPA calculations
- **Limit (2 cores)**: Maximum CPU allowed, prevents noisy neighbor problems
- **Ratio 1:4**: Allows bursting up to 4x the baseline requirement

**Memory Requests vs Limits:**
- **Request (1Gi)**: Guaranteed memory, prevents scheduling on overcommitted nodes
- **Limit (4Gi)**: Hard limit, container killed if exceeded (OOMKilled)
- **Ratio 1:4**: Accommodates memory spikes while preventing memory leaks

**JVM Configuration:**
- **UseContainerSupport**: JVM recognizes container memory/CPU limits
- **MaxRAMPercentage=75%**: Uses 75% of container limit (3Gi out of 4Gi)
- **G1GC**: Low-latency garbage collector suitable for containerized applications

**Health Check Strategy:**
- **Startup Probe**: Protects slow-starting applications during initialization
- **Readiness Probe**: Fast checks (5s interval) for quick scaling decisions
- **Liveness Probe**: Slower checks (30s interval) for container restart decisions

#### **2. Sophisticated HPA Configuration**

```yaml
# Production-Ready HPA with Multiple Metrics - Comprehensive Explanation
apiVersion: autoscaling/v2         # HPA API version (v2 supports multiple metrics)
kind: HorizontalPodAutoscaler      # Resource type for automatic pod scaling
metadata:
  name: java-app-hpa              # Unique name for this HPA
  namespace: production           # Same namespace as target deployment
spec:
  scaleTargetRef:                 # Which deployment to scale
    apiVersion: apps/v1           # API version of target resource
    kind: Deployment              # Type of resource to scale
    name: java-app-optimized      # Name of deployment (must exist)
  
  # Scaling boundaries - Critical for cost control and availability
  minReplicas: 3                  # Minimum pods for high availability
                                 # Never scale below this number
                                 # Ensures service availability during low load
  maxReplicas: 100               # Maximum pods to prevent cost explosion
                                # Safety limit for scaling runaway scenarios
                                # Should align with cluster capacity
  
  # Multiple metrics for intelligent scaling decisions
  # HPA evaluates ALL metrics and chooses the one requiring MOST replicas
  metrics:
  
  # CPU-based scaling - Most common metric
  - type: Resource                # Uses Kubernetes resource metrics
    resource:
      name: cpu                   # Target CPU utilization
      target:
        type: Utilization         # Percentage of resource request
        averageUtilization: 70    # Scale when average CPU > 70% of request
                                 # 70% of 500m = 350m actual CPU usage
                                 # Conservative threshold prevents constant scaling
  
  # Memory-based scaling - Important for memory-intensive apps
  - type: Resource                # Uses Kubernetes resource metrics
    resource:
      name: memory                # Target memory utilization
      target:
        type: Utilization         # Percentage of resource request
        averageUtilization: 80    # Scale when average memory > 80% of request
                                 # 80% of 1Gi = 800Mi actual memory usage
                                 # Higher threshold as memory is less spiky
  
  # Custom application metrics - Business logic based scaling
  - type: Pods                    # Per-pod custom metric
    pods:
      metric:
        name: http_requests_per_second  # Custom metric name from Prometheus
      target:
        type: AverageValue        # Average across all pods
        averageValue: "50"        # Scale when RPS > 50 per pod
                                 # Ensures each pod handles reasonable load
                                 # Prevents performance degradation
  
  # Queue-based scaling for async workloads
  - type: Object                  # Object-level metric (not per-pod)
    object:
      metric:
        name: queue_length        # Queue depth metric
      describedObject:            # Which Kubernetes object provides the metric
        apiVersion: v1            # API version of metric source
        kind: Service             # Type of object (could be ConfigMap, etc.)
        name: message-queue       # Name of the object
      target:
        type: Value               # Absolute value (not per-pod)
        value: "100"              # Scale when queue length > 100 messages
                                 # Prevents queue buildup and delays
  
  # Advanced scaling behavior - Prevents thrashing and provides stability
  behavior:
    scaleUp:                      # Scaling up configuration
      stabilizationWindowSeconds: 60    # Wait 60s before scaling up again
                                       # Prevents rapid successive scale-ups
                                       # Allows metrics to stabilize
      selectPolicy: Max                 # Choose policy requiring MOST replicas
                                       # Max = most aggressive scaling
                                       # Alternatives: Min, Disabled
      policies:                         # Multiple scaling policies
      - type: Percent                   # Percentage-based scaling
        value: 50                       # Increase by max 50% of current replicas
        periodSeconds: 60               # Apply this limit every 60 seconds
                                       # If 10 pods, max increase = 5 pods
      - type: Pods                      # Absolute pod-based scaling
        value: 10                       # Add max 10 pods at once
        periodSeconds: 60               # Apply this limit every 60 seconds
                                       # Prevents massive scale-up events
      # HPA uses the MINIMUM of both policies (50% OR 10 pods, whichever is less)
    
    scaleDown:                          # Scaling down configuration
      stabilizationWindowSeconds: 300   # Wait 5 minutes before scaling down
                                       # Longer stabilization prevents thrashing
                                       # Accounts for transient load spikes
      selectPolicy: Min                 # Choose policy requiring LEAST replicas
                                       # Min = most conservative scaling down
                                       # Prevents aggressive scale-down
      policies:                         # Scaling down policies
      - type: Percent                   # Percentage-based scaling
        value: 10                       # Decrease by max 10% of current replicas
        periodSeconds: 60               # Apply every 60 seconds
                                       # Gradual scale-down for stability
      - type: Pods                      # Absolute pod-based scaling
        value: 5                        # Remove max 5 pods at once
        periodSeconds: 60               # Apply every 60 seconds
                                       # Prevents massive scale-down events
      # HPA uses the MINIMUM of both policies (10% OR 5 pods, whichever is less)
```

**HPA Configuration Explained:**

**Metric Types and Their Use Cases:**
1. **Resource Metrics (CPU/Memory)**: 
   - Uses Kubernetes metrics-server data
   - Based on resource requests in pod spec
   - CPU: Good for compute-intensive workloads
   - Memory: Good for memory-intensive applications

2. **Pod Metrics (Custom)**:
   - Application-specific metrics from Prometheus
   - Per-pod averages (RPS, response time, active connections)
   - Better reflects actual application load

3. **Object Metrics**:
   - External system metrics (queue length, database connections)
   - Single value for entire application
   - Good for async/batch processing workloads

**Scaling Behavior Explained:**
- **Scale-Up**: Fast response to handle load spikes quickly
- **Scale-Down**: Slow and conservative to prevent thrashing
- **Stabilization Windows**: Prevent rapid back-and-forth scaling
- **Policies**: Limit the rate of scaling changes

**Threshold Selection Guidelines:**
- **CPU 70%**: Conservative, allows for traffic spikes
- **Memory 80%**: Higher threshold as memory usage is steadier
- **RPS 50/pod**: Application-specific, based on performance testing
- **Queue 100**: Prevents backlog buildup in async systems

#### **3. Cluster Autoscaler Configuration**

```yaml
# Cluster Autoscaler for Node Scaling - Detailed Configuration
apiVersion: apps/v1                    # Deployment API version
kind: Deployment                       # Resource type for managing cluster autoscaler
metadata:
  name: cluster-autoscaler            # Unique name for the deployment
  namespace: kube-system              # System namespace for cluster components
  labels:
    app: cluster-autoscaler          # Label for identification
spec:
  selector:                          # How deployment finds its pods
    matchLabels:
      app: cluster-autoscaler        # Must match template labels
  template:                          # Pod template specification
    metadata:
      labels:
        app: cluster-autoscaler      # Pod labels for selection
    spec:
      priorityClassName: system-cluster-critical  # High priority for system component
                                                 # Ensures autoscaler runs even under resource pressure
      securityContext:                           # Security configuration for pod
        runAsNonRoot: true                       # Don't run as root user
        runAsUser: 65534                         # Run as nobody user (security)
        fsGroup: 65534                           # File system group for volumes
      serviceAccountName: cluster-autoscaler     # ServiceAccount with cluster permissions
      containers:
      - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.26.0  # Autoscaler image
        name: cluster-autoscaler                                 # Container name
        resources:                                               # Resource limits for autoscaler
          limits:
            cpu: 100m                                           # Max 100 millicores
            memory: 600Mi                                       # Max 600 MiB memory
          requests:
            cpu: 100m                                           # Guaranteed 100 millicores
            memory: 600Mi                                       # Guaranteed 600 MiB memory
        command:                                                # Command line arguments
        - ./cluster-autoscaler                                 # Main executable
        
        # Logging and debug configuration
        - --v=4                                                # Verbose logging level (0-10)
        - --stderrthreshold=info                              # Log info+ to stderr
        
        # Cloud provider configuration
        - --cloud-provider=aws                                # AWS as cloud provider
                                                              # Alternatives: gcp, azure, etc.
        
        # Node discovery and selection
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/production-cluster
                                                              # Auto-discover AWS Auto Scaling Groups
                                                              # Groups must have both tags:
                                                              # - k8s.io/cluster-autoscaler/enabled=true
                                                              # - k8s.io/cluster-autoscaler/production-cluster=owned
        
        # Scaling behavior configuration
        - --expander=least-waste                              # Node selection strategy
                                                              # least-waste: minimize unused resources
                                                              # Alternatives: most-pods, random, priority
        - --balance-similar-node-groups                       # Balance pods across similar node groups
                                                              # Prevents hotspots in multi-AZ deployments
        - --skip-nodes-with-local-storage=false              # Allow scaling nodes with local storage
        - --skip-nodes-with-system-pods=false                # Allow scaling nodes with system pods
        
        # Timing configurations for smooth scaling - CRITICAL for stability
        - --scale-down-delay-after-add=10m                   # Wait 10min after adding node before considering scale-down
                                                              # Prevents immediate scale-down of newly added nodes
                                                              # Allows time for pods to be scheduled
        - --scale-down-unneeded-time=10m                     # Node must be unneeded for 10min before removal
                                                              # Prevents premature node removal
                                                              # Accounts for scheduling delays
        - --scale-down-delay-after-delete=10s                # Wait 10s between node deletions
                                                              # Prevents mass node removal
                                                              # Allows time for rescheduling
        - --scale-down-delay-after-failure=3m                # Wait 3min after failed scale-down attempt
                                                              # Prevents rapid retry of failed operations
        - --scale-down-utilization-threshold=0.5             # Remove node if utilization < 50%
                                                              # 50% = sum of requests / allocatable capacity
                                                              # Conservative threshold prevents aggressive removal
        
        # Provisioning and capacity limits
        - --max-node-provision-time=15m                      # Max time to wait for new node
                                                              # Timeout for cloud provider node creation
                                                              # Prevents hanging on failed provisions
        - --max-nodes-total=100                              # Maximum total nodes in cluster
                                                              # Safety limit to prevent cost explosion
                                                              # Should align with cluster capacity planning
        - --cores-total=0:1000                               # CPU limits: min:max cores
                                                              # 0 = no minimum, 1000 = max cores
                                                              # Prevents over-provisioning
        - --memory-total=0:1000                              # Memory limits: min:max GB
                                                              # 0 = no minimum, 1000GB = max memory
        
        # Bulk operations configuration
        - --max-bulk-soft-taint-count=10                     # Max nodes to taint at once
                                                              # Limits disruption during scale-down
        - --max-bulk-soft-taint-time=3s                      # Max time between tainting nodes
                                                              # Spreads disruption over time
        
        # Additional stability options
        - --scan-interval=10s                                # How often to evaluate scaling needs
        - --scale-down-candidates-pool-ratio=0.1             # Percentage of nodes to consider for scale-down
        - --scale-down-candidates-pool-min-count=50          # Minimum nodes to consider for scale-down
        - --ignore-daemonsets-utilization                    # Don't count DaemonSet resource usage
                                                              # DaemonSets run on every node anyway
        - --ignore-mirror-pods-utilization                   # Don't count mirror pod usage
                                                              # Static pods managed by kubelet

---
# Service Account for Cluster Autoscaler
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
  name: cluster-autoscaler
  namespace: kube-system
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::ACCOUNT:role/eks-cluster-autoscaler

---
# RBAC for Cluster Autoscaler
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
- apiGroups: [""]
  resources: ["events", "endpoints"]
  verbs: ["create", "patch"]
- apiGroups: [""]
  resources: ["pods/eviction"]
  verbs: ["create"]
- apiGroups: [""]
  resources: ["pods/status"]
  verbs: ["update"]
- apiGroups: [""]
  resources: ["endpoints"]
  resourceNames: ["cluster-autoscaler"]
  verbs: ["get", "update"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["watch", "list", "get", "update"]
- apiGroups: [""]
  resources: ["namespaces", "pods", "services", "replicationcontrollers", "persistentvolumeclaims", "persistentvolumes"]
  verbs: ["watch", "list", "get"]
- apiGroups: ["extensions"]
  resources: ["replicasets", "daemonsets"]
  verbs: ["watch", "list", "get"]
- apiGroups: ["policy"]
  resources: ["poddisruptionbudgets"]
  verbs: ["watch", "list"]
- apiGroups: ["apps"]
  resources: ["statefulsets", "replicasets", "daemonsets"]
  verbs: ["watch", "list", "get"]
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses", "csinodes", "csidrivers", "csistoragecapacities"]
  verbs: ["watch", "list", "get"]
- apiGroups: ["batch", "extensions"]
  resources: ["jobs"]
  verbs: ["get", "list", "watch", "patch"]
- apiGroups: ["coordination.k8s.io"]
  resources: ["leases"]
  verbs: ["create"]
- apiGroups: ["coordination.k8s.io"]
  resourceNames: ["cluster-autoscaler"]
  resources: ["leases"]
  verbs: ["get", "update"]
```

#### **4. Vertical Pod Autoscaler (VPA) for Right-Sizing**

```yaml
# VPA for automatic resource recommendation and updates - Comprehensive Explanation
apiVersion: autoscaling.k8s.io/v1  # VPA API version
kind: VerticalPodAutoscaler         # Resource type for automatic resource sizing
metadata:
  name: java-app-vpa              # Unique name for this VPA
  namespace: production           # Same namespace as target deployment
spec:
  targetRef:                      # Which deployment to analyze and update
    apiVersion: apps/v1           # API version of target resource
    kind: Deployment              # Type of resource (Deployment, ReplicaSet, StatefulSet)
    name: java-app-optimized      # Name of deployment to optimize
  
  # Update policy - Controls how VPA applies recommendations
  updatePolicy:
    updateMode: "Auto"            # Automatically apply recommendations by recreating pods
                                 # VPA will restart pods with new resource settings
                                 # Best for applications that can handle restarts
    # updateMode: "Initial"       # Only set resources when pods are created
                                 # Good for applications where restarts are expensive
                                 # VPA won't update existing pods
    # updateMode: "Off"           # Only provide recommendations, don't apply
                                 # Useful for observation and manual decision making
                                 # Check recommendations with: kubectl describe vpa
  
  # Resource policy to constrain VPA recommendations
  resourcePolicy:
    containerPolicies:            # Per-container resource policies
    - containerName: app          # Must match container name in deployment
      
      # Minimum allowed resources - Safety bounds
      minAllowed:
        cpu: 100m                 # Never recommend less than 100 millicores
                                 # Prevents under-provisioning that could cause instability
        memory: 128Mi             # Never recommend less than 128 MiB
                                 # Ensures minimum memory for JVM startup
      
      # Maximum allowed resources - Cost and safety bounds  
      maxAllowed:
        cpu: 4                    # Never recommend more than 4 cores
                                 # Prevents cost explosion from runaway recommendations
        memory: 8Gi               # Never recommend more than 8 GiB
                                 # Limits maximum memory allocation
      
      # Which resources to control
      controlledResources: ["cpu", "memory"]  # VPA manages both CPU and memory
                                             # Could be ["cpu"] or ["memory"] only
      
      # What values to update
      controlledValues: RequestsAndLimits     # Update both requests and limits
                                             # Alternatives:
                                             # RequestsOnly: Only update requests
                                             # RequestsAndLimits: Update both (recommended)
      
      # Resource calculation mode (optional)
      # mode: Auto                            # Default: use all available data
      # mode: Off                             # Don't generate recommendations for this container
      
      # Scaling mode for resources (optional)
      # scalingMode: Auto                     # Default: scale both up and down
      # scalingMode: Off                      # Don't scale this resource
      # scalingMode: ScaleUp                  # Only scale up, never down
      # scalingMode: ScaleDown                # Only scale down, never up

---
# VPA Recommender Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: vpa-config
  namespace: kube-system
data:
  recommender-config.yaml: |
    recommenders:
    - name: default
      recommenderInterval: 1m
      checkpointsGCInterval: 10m
      prometheusAddress: "http://prometheus.monitoring.svc.cluster.local:9090"
      storage: prometheus
      historyLength: "8d"
      cpuHistogramDecayHalfLife: "24h"
      memoryHistogramDecayHalfLife: "24h"
```

### Advanced Autoscaling Strategies

#### **5. Custom Metrics Autoscaling**

```yaml
# Custom Metrics Server Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: adapter-config
  namespace: custom-metrics
data:
  config.yaml: |
    rules:
    # HTTP request rate metric
    - seriesQuery: 'http_requests_per_second{namespace!="",pod!=""}'
      resources:
        overrides:
          namespace: {resource: "namespace"}
          pod: {resource: "pod"}
      name:
        matches: "^http_requests_per_second"
        as: "http_requests_per_second"
      metricsQuery: 'avg(rate(http_requests_total{<<.LabelMatchers>>}[2m]))'
    
    # Queue length metric
    - seriesQuery: 'queue_length{namespace!="",service!=""}'
      resources:
        overrides:
          namespace: {resource: "namespace"}
          service: {resource: "service"}
      name:
        matches: "^queue_length"
        as: "queue_length"
      metricsQuery: 'max(queue_length{<<.LabelMatchers>>})'
    
    # Business-specific metrics
    - seriesQuery: 'active_users_count{namespace!="",service!=""}'
      resources:
        overrides:
          namespace: {resource: "namespace"}
          service: {resource: "service"}
      name:
        matches: "^active_users_count"
        as: "active_users_count"
      metricsQuery: 'sum(active_users_count{<<.LabelMatchers>>})'

---
# HPA using custom metrics
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: java-app-custom-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: java-app-optimized
  minReplicas: 2
  maxReplicas: 50
  metrics:
  # Custom application metric
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "100"
  
  # External metric (from external monitoring system)
  - type: External
    external:
      metric:
        name: aws_sqs_queue_length
        selector:
          matchLabels:
            queue_name: "order-processing"
      target:
        type: Value
        value: "100"
```

#### **6. Predictive Autoscaling**

```yaml
# CronJob for Predictive Scaling
apiVersion: batch/v1
kind: CronJob
metadata:
  name: predictive-scaling
  namespace: production
spec:
  schedule: "*/5 * * * *"  # Every 5 minutes
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: predictive-scaler
            image: predictive-scaler:v1.0.0
            env:
            - name: TARGET_DEPLOYMENT
              value: "java-app-optimized"
            - name: NAMESPACE
              value: "production"
            - name: PROMETHEUS_URL
              value: "http://prometheus.monitoring.svc.cluster.local:9090"
            command:
            - /bin/sh
            - -c
            - |
              #!/bin/bash
              
              # Get current time and predict next 15 minutes
              CURRENT_HOUR=$(date +%H)
              CURRENT_DAY=$(date +%u)  # 1=Monday, 7=Sunday
              
              # Historical traffic patterns (example)
              case $CURRENT_DAY in
                1|2|3|4|5)  # Weekdays
                  if [ $CURRENT_HOUR -ge 9 ] && [ $CURRENT_HOUR -le 17 ]; then
                    PREDICTED_LOAD="high"
                    MIN_REPLICAS=10
                  else
                    PREDICTED_LOAD="medium"
                    MIN_REPLICAS=5
                  fi
                  ;;
                6|7)  # Weekends
                  PREDICTED_LOAD="low"
                  MIN_REPLICAS=3
                  ;;
              esac
              
              # Query current metrics
              CURRENT_RPS=$(curl -s "$PROMETHEUS_URL/api/v1/query?query=sum(rate(http_requests_total[5m]))" | jq -r '.data.result[0].value[1]')
              CURRENT_REPLICAS=$(kubectl get deployment java-app-optimized -o jsonpath='{.spec.replicas}')
              
              # Apply predictive scaling
              if [ "$PREDICTED_LOAD" = "high" ] && [ $CURRENT_REPLICAS -lt $MIN_REPLICAS ]; then
                echo "Pre-scaling for predicted high load"
                kubectl scale deployment java-app-optimized --replicas=$MIN_REPLICAS
                
                # Update HPA minimum
                kubectl patch hpa java-app-hpa -p '{"spec":{"minReplicas":'$MIN_REPLICAS'}}'
              fi
              
              # Log prediction and action
              echo "$(date): Predicted load: $PREDICTED_LOAD, Current RPS: $CURRENT_RPS, Action: Scaled to $MIN_REPLICAS"
          
          restartPolicy: OnFailure
          serviceAccountName: predictive-scaler

---
# ServiceAccount for Predictive Scaler
apiVersion: v1
kind: ServiceAccount
metadata:
  name: predictive-scaler
  namespace: production

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: predictive-scaler
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "deployments/scale"]
  verbs: ["get", "list", "patch", "update"]
- apiGroups: ["autoscaling"]
  resources: ["horizontalpodautoscalers"]
  verbs: ["get", "list", "patch", "update"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: predictive-scaler
  namespace: production
subjects:
- kind: ServiceAccount
  name: predictive-scaler
  namespace: production
roleRef:
  kind: Role
  name: predictive-scaler
  apiGroup: rbac.authorization.k8s.io
```

### Monitoring and Troubleshooting Autoscaling

#### **7. Comprehensive Monitoring Setup**

```yaml
# Prometheus Rules for Autoscaling Monitoring - Comprehensive Alerting
apiVersion: monitoring.coreos.com/v1  # Prometheus Operator API version
kind: PrometheusRule                  # Resource type for Prometheus alerting rules
metadata:
  name: autoscaling-monitoring       # Unique name for this rule set
  namespace: monitoring              # Namespace where Prometheus is running
spec:
  groups:                            # Rule groups organize related alerts
  - name: autoscaling.rules          # Group name for autoscaling-related rules
    rules:                           # List of alerting rules
    
    # HPA scaling events - Informational alert for scaling activity
    - alert: HPAScalingEvent          # Alert name
      expr: increase(kube_hpa_status_current_replicas[5m]) > 2  # PromQL expression
                                     # Triggers when HPA increases replicas by >2 in 5 minutes
                                     # increase() calculates the increase over time window
                                     # kube_hpa_status_current_replicas = current replica count
      for: 0m                        # Fire immediately (no wait time)
                                     # 0m = instant alert for informational purposes
      labels:                        # Labels attached to the alert
        severity: info               # Severity level (info/warning/critical)
                                     # Used for routing and notification channels
      annotations:                   # Human-readable alert descriptions
        summary: "HPA {{ $labels.hpa }} scaled up by {{ $value }} replicas"
                                     # Template using alert labels and values
                                     # {{ $labels.hpa }} = HPA name from metric labels
                                     # {{ $value }} = result of the expression
    
    # HPA unable to scale - Warning when scaling is blocked
    - alert: HPAUnableToScale         # Alert name for scaling limitations
      expr: kube_hpa_status_condition{condition="ScalingLimited", status="true"} == 1
                                     # Triggers when HPA condition shows scaling limited
                                     # condition="ScalingLimited" = HPA hit min/max limits
                                     # status="true" = condition is active
      for: 2m                        # Wait 2 minutes before firing
                                     # Prevents alerts on transient scaling limits
      labels:
        severity: warning            # Warning severity (not critical)
      annotations:
        summary: "HPA {{ $labels.hpa }} unable to scale due to limits"
                                     # Indicates HPA hit configured min/max replicas
    
    # High CPU utilization but no scaling - Critical performance issue
    - alert: HighCPUNoScaling         # Alert for performance bottleneck
      expr: |                        # Multi-line PromQL expression
        (
          avg by (deployment) (      # Average CPU usage per deployment
            rate(container_cpu_usage_seconds_total{container!="POD",container!=""}[5m])
          ) * 100                    # Convert to percentage
        ) > 80                       # CPU usage > 80%
        and                          # AND condition (both must be true)
        (
          kube_deployment_status_replicas == kube_hpa_spec_max_replicas
        )                            # Deployment at maximum replicas
                                     # Indicates scaling exhausted but load still high
      for: 5m                        # Wait 5 minutes to avoid false alerts
                                     # Ensures sustained high CPU, not spikes
      labels:
        severity: warning            # Warning level (may need intervention)
      annotations:
        summary: "Deployment {{ $labels.deployment }} at max replicas but CPU still high"
                                     # Indicates need for vertical scaling or optimization
    
    # Cluster autoscaler events - Warning for autoscaler problems
    - alert: ClusterAutoscalerError   # Alert for cluster scaling issues
      expr: increase(cluster_autoscaler_errors_total[10m]) > 0
                                     # Any increase in autoscaler errors over 10 minutes
                                     # cluster_autoscaler_errors_total = cumulative error count
      for: 0m                        # Immediate alert for autoscaler errors
                                     # Errors can impact entire cluster scaling
      labels:
        severity: warning            # Warning level for operational issues
      annotations:
        summary: "Cluster autoscaler encountered {{ $value }} errors"
                                     # Shows number of errors for troubleshooting
    
    # Node scaling events - Informational alert for node changes
    - alert: NodeScalingEvent         # Alert for node addition
      expr: increase(kube_node_info[5m]) > 0
                                     # Triggers when new nodes are added
                                     # kube_node_info = node information metric
                                     # increase() detects new nodes in 5-minute window
      for: 0m                        # Immediate notification
      labels:
        severity: info               # Informational (normal operation)
      annotations:
        summary: "{{ $value }} new nodes added to cluster"
                                     # Shows how many nodes were added
        
    # Additional useful alerts for autoscaling monitoring
    
    # Pod pending for too long - Indicates scaling issues
    - alert: PodPendingTooLong        # Alert for scheduling problems
      expr: kube_pod_status_phase{phase="Pending"} == 1
                                     # Pods stuck in Pending state
      for: 5m                        # Wait 5 minutes (normal for node provisioning)
      labels:
        severity: warning
      annotations:
        summary: "Pod {{ $labels.pod }} pending for over 5 minutes"
        description: "May indicate insufficient cluster capacity or scheduling constraints"
    
    # VPA recommendation deviation - Resource right-sizing issues
    - alert: VPARecommendationDeviation  # Alert for resource misalignment
      expr: |
        abs(
          kube_pod_container_resource_requests_cpu_cores - 
          vpa_container_recommendation_target_cpu_cores
        ) / vpa_container_recommendation_target_cpu_cores > 0.5
                                     # Current CPU requests deviate >50% from VPA recommendation
      for: 30m                       # Wait 30 minutes for VPA to stabilize
      labels:
        severity: info
      annotations:
        summary: "Container {{ $labels.container }} resource requests significantly differ from VPA recommendations"

---
# Grafana Dashboard ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: autoscaling-dashboard
  namespace: monitoring
data:
  dashboard.json: |
    {
      "dashboard": {
        "id": null,
        "title": "Autoscaling Monitoring",
        "tags": ["autoscaling", "kubernetes"],
        "timezone": "browser",
        "panels": [
          {
            "id": 1,
            "title": "HPA Current vs Desired Replicas",
            "type": "graph",
            "targets": [
              {
                "expr": "kube_hpa_status_current_replicas",
                "legendFormat": "Current - {{ hpa }}"
              },
              {
                "expr": "kube_hpa_status_desired_replicas",
                "legendFormat": "Desired - {{ hpa }}"
              }
            ]
          },
          {
            "id": 2,
            "title": "Resource Utilization",
            "type": "graph",
            "targets": [
              {
                "expr": "avg(rate(container_cpu_usage_seconds_total[5m])) by (pod) * 100",
                "legendFormat": "CPU % - {{ pod }}"
              },
              {
                "expr": "avg(container_memory_usage_bytes) by (pod) / avg(container_spec_memory_limit_bytes) by (pod) * 100",
                "legendFormat": "Memory % - {{ pod }}"
              }
            ]
          },
          {
            "id": 3,
            "title": "Scaling Events",
            "type": "table",
            "targets": [
              {
                "expr": "increase(kube_hpa_status_current_replicas[5m])",
                "format": "table"
              }
            ]
          }
        ]
      }
    }
```

#### **8. Autoscaling Best Practices Checklist**

```yaml
# ConfigMap with Best Practices Checklist
apiVersion: v1
kind: ConfigMap
metadata:
  name: autoscaling-best-practices
  namespace: production
data:
  checklist.md: |
    # Autoscaling Best Practices Checklist
    
    ## Resource Configuration
    - [ ] CPU requests set to 80% of typical usage
    - [ ] Memory requests set to 90% of typical usage
    - [ ] CPU limits set to 2-3x requests
    - [ ] Memory limits set to 1.5-2x requests
    - [ ] JVM heap size set to 75% of memory limit
    
    ## Health Checks
    - [ ] Readiness probe configured with fast response time
    - [ ] Liveness probe configured with appropriate timeout
    - [ ] Startup probe configured for slow-starting applications
    - [ ] Health check endpoints optimized and lightweight
    
    ## HPA Configuration
    - [ ] Multiple metrics configured (CPU, memory, custom)
    - [ ] Appropriate scaling thresholds (70% CPU, 80% memory)
    - [ ] Scaling behavior configured to prevent thrashing
    - [ ] Min replicas ≥ 2 for high availability
    - [ ] Max replicas set based on resource limits
    
    ## Cluster Configuration
    - [ ] Cluster autoscaler enabled and configured
    - [ ] Node groups with appropriate instance types
    - [ ] Pod Disruption Budgets configured
    - [ ] Resource quotas set per namespace
    - [ ] Network policies for security
    
    ## Monitoring and Alerting
    - [ ] Prometheus metrics collection enabled
    - [ ] Grafana dashboards for autoscaling visibility
    - [ ] Alerts for scaling events and failures
    - [ ] Log aggregation for troubleshooting
    - [ ] Regular review of scaling patterns
    
    ## Application Optimization
    - [ ] Fast application startup time (< 30 seconds)
    - [ ] Graceful shutdown handling
    - [ ] Connection pooling optimized
    - [ ] Stateless application design
    - [ ] Efficient resource usage patterns
    
    ## Testing and Validation
    - [ ] Load testing with gradual traffic increase
    - [ ] Chaos engineering for scaling resilience
    - [ ] Regular scaling drill exercises
    - [ ] Performance regression testing
    - [ ] Capacity planning reviews
```

### Common Autoscaling Problems and Solutions

#### **9. Troubleshooting Guide**

```yaml
# Troubleshooting ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: autoscaling-troubleshooting
  namespace: production
data:
  troubleshooting.md: |
    # Autoscaling Troubleshooting Guide
    
    ## Problem: HPA Not Scaling
    
    ### Symptoms:
    - High CPU/memory usage but no new pods created
    - HPA shows "unknown" for metrics
    
    ### Diagnosis Commands:
    ```bash
    kubectl describe hpa <hpa-name>
    kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
    kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods
    kubectl top pods
    ```
    
    ### Common Causes & Solutions:
    1. **Missing resource requests**: Set CPU/memory requests
    2. **Metrics server not running**: Deploy metrics-server
    3. **Insufficient permissions**: Check RBAC
    4. **Wrong metric names**: Verify custom metric names
    
    ## Problem: Frequent Scaling (Thrashing)
    
    ### Symptoms:
    - Pods constantly scaling up and down
    - Resource waste and instability
    
    ### Solutions:
    ```yaml
    behavior:
      scaleUp:
        stabilizationWindowSeconds: 300
      scaleDown:
        stabilizationWindowSeconds: 600
    ```
    
    ## Problem: Slow Scaling Response
    
    ### Symptoms:
    - Long delay between load increase and scaling
    - Performance degradation during traffic spikes
    
    ### Solutions:
    1. **Reduce metric collection interval**
    2. **Optimize readiness probes**
    3. **Implement predictive scaling**
    4. **Use pre-warmed node pools**
    
    ## Problem: Cluster Autoscaler Issues
    
    ### Symptoms:
    - Pods stuck in Pending state
    - Nodes not being added/removed
    
    ### Diagnosis Commands:
    ```bash
    kubectl describe pod <pending-pod>
    kubectl logs -n kube-system deployment/cluster-autoscaler
    kubectl get events --sort-by=.metadata.creationTimestamp
    ```
    
    ### Common Solutions:
    1. **Check node group limits**
    2. **Verify IAM permissions**
    3. **Review node taints and tolerations**
    4. **Check resource quotas**
```

This comprehensive guide covers all aspects of ensuring smooth autoscaling in Kubernetes environments, from basic configuration to advanced troubleshooting techniques. The key to success is proper resource management, comprehensive monitoring, and continuous optimization based on real-world usage patterns.

---


