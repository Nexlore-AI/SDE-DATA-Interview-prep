# Kubernetes & Docker — Interview Q&A

---

## Docker

### 1. What is Docker and why is it used?

Docker is a **containerization platform** that packages an application and its dependencies into a lightweight, portable unit called a **container**.

**Why:**
- **Consistency:** "Works on my machine" problem solved — same environment everywhere
- **Isolation:** Each container has its own filesystem, network, and process space
- **Lightweight:** Shares the host OS kernel (unlike VMs which need a full OS)
- **Fast startup:** Containers start in seconds (VMs take minutes)
- **Portability:** Build once, run anywhere (laptop, cloud, CI/CD)

### 2. What is the difference between a Docker image and a container?

| Image | Container |
|-------|-----------|
| Blueprint / template | Running instance of an image |
| Immutable (read-only layers) | Mutable (has a writable layer) |
| Built from a Dockerfile | Created from an image (`docker run`) |
| Stored in a registry (Docker Hub, ECR, GCR) | Runs on a host machine |

**Analogy:** Image = Class, Container = Object/Instance.

### 3. Explain the layers in a Docker image.

Each instruction in a Dockerfile creates a **layer**. Layers are cached and shared.

```dockerfile
FROM python:3.11-slim        # Base layer
COPY requirements.txt .      # Layer 2
RUN pip install -r requirements.txt  # Layer 3 (cached if requirements.txt unchanged)
COPY . .                     # Layer 4
CMD ["python", "app.py"]     # Metadata (no layer)
```

**Key concept:** Docker caches layers. If `requirements.txt` hasn't changed, layer 3 is reused → faster builds.

**Best practice:** Put frequently changing instructions (COPY source code) last, and rarely changing instructions (install dependencies) first.

### 4. What is the difference between CMD and ENTRYPOINT?

| Feature | CMD | ENTRYPOINT |
|---------|-----|------------|
| Purpose | Default command/args | Fixed executable |
| Override | Easily overridden by `docker run` args | Requires `--entrypoint` flag to override |
| Common use | Default arguments | Set the main process |

**Best practice combo:**
```dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080"]
```
User can override args: `docker run myapp --port 9090`

### 5. What is a multi-stage build and why use it?

Multi-stage builds use multiple `FROM` statements to create smaller final images:

```dockerfile
# Stage 1: Build
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Stage 2: Runtime (much smaller)
FROM alpine:3.18
COPY --from=builder /app/myapp /myapp
CMD ["/myapp"]
```

**Benefits:**
- Final image only has the binary, not build tools (Go SDK, compiler, etc.)
- Image size: ~1 GB → ~10 MB
- Fewer vulnerabilities (less software in the image)

### 6. What is Docker Compose?

Docker Compose defines and runs **multi-container applications** using a YAML file:

```yaml
services:
  web:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - db
      - redis
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
  redis:
    image: redis:7-alpine

volumes:
  pgdata:
```

`docker compose up` starts everything. Great for local development.

### 7. What is the difference between Docker volumes and bind mounts?

| Feature | Volume | Bind Mount |
|---------|--------|------------|
| Managed by | Docker | Host filesystem |
| Location | `/var/lib/docker/volumes/` | Anywhere on host |
| Portability | Easy to backup/migrate | Host-dependent |
| Use case | Database data, persistent storage | Development (mount source code) |
| Performance | Optimized by Docker | Depends on host FS |

### 8. Docker networking modes?

| Mode | Description | Use Case |
|------|-------------|----------|
| **bridge** (default) | Containers get their own network, communicate via bridge | Standard isolated containers |
| **host** | Container shares host's network stack | Performance-critical (no NAT overhead) |
| **none** | No networking | Security-sensitive workloads |
| **overlay** | Multi-host networking | Docker Swarm / distributed apps |

---

## Kubernetes

### 9. What is Kubernetes and why do we need it?

Kubernetes (K8s) is a **container orchestration platform** that automates deployment, scaling, and management of containerized applications.

**Why not just Docker?**
- Docker runs containers on one machine. K8s manages containers across a **cluster of machines**.
- **Auto-scaling:** Scale pods up/down based on load
- **Self-healing:** Restart crashed containers, replace unhealthy nodes
- **Service discovery:** Containers find each other by name
- **Rolling updates:** Deploy new versions with zero downtime
- **Load balancing:** Distribute traffic across pods

### 10. Explain the Kubernetes architecture.

```
┌─────────── Control Plane ───────────┐
│  API Server ─── etcd (state store)  │
│  Scheduler ─── Controller Manager   │
└─────────────────────────────────────┘
         ↓              ↓
   ┌── Node 1 ──┐  ┌── Node 2 ──┐
   │  kubelet    │  │  kubelet    │
   │  kube-proxy │  │  kube-proxy │
   │  ┌──────┐   │  │  ┌──────┐   │
   │  │ Pod  │   │  │  │ Pod  │   │
   │  │ Pod  │   │  │  │ Pod  │   │
   │  └──────┘   │  │  └──────┘   │
   └─────────────┘  └─────────────┘
```

**Control Plane:**
- **API Server:** All communication goes through here (kubectl, other components)
- **etcd:** Distributed key-value store holding cluster state
- **Scheduler:** Assigns pods to nodes based on resources
- **Controller Manager:** Runs controllers (ReplicaSet, Deployment, etc.)

**Worker Node:**
- **kubelet:** Ensures containers are running in pods
- **kube-proxy:** Network proxy for service routing
- **Container Runtime:** Docker, containerd, or CRI-O

### 11. What is a Pod? Why not just run containers directly?

A **Pod** is the smallest deployable unit in K8s. It's a wrapper around one or more containers that share:
- **Network namespace** (same IP, can communicate via localhost)
- **Storage volumes**
- **Lifecycle** (created/destroyed together)

**Why pods?**
- **Sidecar pattern:** Main app container + logging/monitoring sidecar in one pod
- **Init containers:** Run setup tasks before the main container starts
- **Shared storage:** Containers in a pod can share a volume

**Most common:** One container per pod.

### 12. What is the difference between a Deployment, ReplicaSet, and StatefulSet?

| Resource | Purpose | Use Case |
|----------|---------|----------|
| **ReplicaSet** | Ensures N replicas of a pod are running | Rarely used directly |
| **Deployment** | Manages ReplicaSets + rolling updates, rollbacks | Stateless apps (web servers, APIs) |
| **StatefulSet** | Like Deployment but with stable identity + ordered provisioning | Databases, Kafka, ZooKeeper |

**Deployment** is the most common. It creates a ReplicaSet under the hood.

**StatefulSet guarantees:**
- Stable network identity (pod-0, pod-1, pod-2)
- Ordered startup/shutdown
- Stable persistent storage per pod

### 13. Explain Kubernetes Services (ClusterIP, NodePort, LoadBalancer, Ingress).

| Type | Accessible From | Use |
|------|-----------------|-----|
| **ClusterIP** (default) | Inside cluster only | Internal service-to-service communication |
| **NodePort** | External via `<NodeIP>:<Port>` (30000-32767) | Dev/testing |
| **LoadBalancer** | External via cloud LB | Production external access |
| **Ingress** | External via HTTP(S) routing rules | Route by hostname/path to multiple services |

```
Internet → Ingress Controller → Service (ClusterIP) → Pods
           (nginx / traefik)
           
           /api → api-service → api-pods
           /web → web-service → web-pods
```

### 14. What are ConfigMaps and Secrets?

| | ConfigMap | Secret |
|---|---|---|
| Purpose | Non-sensitive config (env vars, config files) | Sensitive data (passwords, API keys, certs) |
| Storage | Plain text in etcd | Base64 encoded in etcd (not encrypted by default!) |
| Use | Mount as env vars or volumes | Same, but with restricted access |

**Best practice for Secrets:** Enable etcd encryption at rest, use external secret managers (Vault, AWS Secrets Manager), limit RBAC access.

### 15. Explain the difference between Horizontal Pod Autoscaler (HPA) and Vertical Pod Autoscaler (VPA).

| Feature | HPA | VPA |
|---------|-----|-----|
| Scales | Number of pods (horizontal) | CPU/memory per pod (vertical) |
| Based on | CPU, memory, or custom metrics | Historical resource usage |
| Disruption | Adds/removes pods (no restarts) | Restarts pods to resize |
| Use case | Web servers, APIs (stateless) | Batch jobs, databases |

**HPA example:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### 16. What is a Kubernetes namespace?

Namespaces are **virtual clusters** within a physical cluster for resource isolation:

- **default:** Where resources go if no namespace specified
- **kube-system:** K8s system components
- **kube-public:** Publicly accessible resources

**Use cases:** Separate environments (dev/staging/prod), team isolation, resource quotas per namespace.

### 17. What is a DaemonSet?

A DaemonSet ensures that **one pod runs on every node** (or a subset of nodes).

**Use cases:**
- Log collectors (Fluentd, Filebeat)
- Monitoring agents (Prometheus node exporter, Datadog)
- Network plugins (Calico, Cilium)
- Storage daemons (GlusterFS)

### 18. Explain Rolling Updates and Rollbacks.

**Rolling Update:** Gradually replaces old pods with new ones.
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1        # Extra pod during update
    maxUnavailable: 0   # Zero downtime
```

**Rollback:**
```bash
kubectl rollout undo deployment/myapp                  # Rollback to previous
kubectl rollout undo deployment/myapp --to-revision=3  # Rollback to specific version
kubectl rollout history deployment/myapp               # View history
```

### 19. What are Liveness, Readiness, and Startup Probes?

| Probe | Purpose | On Failure |
|-------|---------|------------|
| **Liveness** | Is the container alive? | Restart the container |
| **Readiness** | Can the container serve traffic? | Remove from service (no traffic) |
| **Startup** | Has the container finished starting? | Keep trying until timeout, then kill |

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  periodSeconds: 3
```

### 20. What is Helm?

Helm is a **package manager for Kubernetes** — like apt/yum but for K8s manifests.

- **Chart:** A package of K8s YAML templates
- **Values:** Configuration to customize a chart
- **Release:** An installed instance of a chart

```bash
helm install my-redis bitnami/redis --set auth.password=secret
helm upgrade my-redis bitnami/redis --set replica.replicaCount=3
helm rollback my-redis 1
```

**Why:** Instead of managing 10+ YAML files, one Helm chart packages everything with configurable values.
