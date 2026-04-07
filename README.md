# ArgoCD Kubernetes Infrastructure

A production-ready GitOps infrastructure repository for deploying and managing Kubernetes applications using ArgoCD. This repository contains declarative configurations for automated deployment, monitoring, and load balancing of containerized applications.

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white)

## Features

### GitOps with ArgoCD
- **Automated Deployment**: Continuous deployment with ArgoCD's GitOps methodology
- **Self-Healing Applications**: Automatic synchronization and recovery of application state
- **Declarative Configuration**: Infrastructure and applications defined as code
- **Multi-Application Management**: Centralized control of multiple services
- **Static IP Assignment**: LoadBalancer with fixed IP (10.10.90.51) for ArgoCD server

### Monitoring Stack
- **Prometheus Integration**: Metrics collection and alerting with kube-prometheus-stack
- **Grafana Dashboards**: Visual monitoring and analytics interface
- **Alertmanager**: Intelligent alert routing and notification management
- **Custom Scrape Configs**: Monitor external hosts (K3s control plane at 10.10.90.151:9100)
- **LoadBalancer Access**: Grafana accessible via static IP (10.10.90.50)

### Load Balancing & Networking
- **MetalLB Integration**: Bare-metal load balancer for on-premises Kubernetes
- **IP Address Pool**: Dedicated IP range (10.10.90.30-10.10.90.140)
- **L2 Advertisement**: Layer 2 networking for seamless service exposure
- **Static IP Assignment**: Predictable service endpoints for external access

### Weather Service Deployment
- **High Availability**: Dual deployment strategy (primary + backup)
- **Horizontal Pod Autoscaling**: Auto-scaling from 2 to 10 replicas based on CPU utilization (50% threshold)
- **Health Checks**: Readiness probes for zero-downtime deployments
- **Resource Management**: Defined CPU and memory limits for optimal performance
- **LoadBalancer Service**: External access via static IP (10.10.90.52)

## Getting Started

### Prerequisites

- **Kubernetes Cluster** (K3s, K8s, or any Kubernetes distribution)
- **kubectl** - Kubernetes command-line tool
- **ArgoCD** - GitOps continuous delivery tool
- **Git** - Version control system
- **MetalLB** (optional) - For bare-metal load balancing

### Initial Setup

1. **Clone the repository**
```bash
   git clone https://github.com/TropoMetrics/ArgoCI-CD.git
   cd ArgoCI-CD
   ```

2. **Create required namespaces**
```bash
   kubectl apply -f namespaces/argocd-ns.yaml
   kubectl apply -f namespaces/metallb-ns.yaml
   ```

3. **Install ArgoCD** (if not already installed)
```bash
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

4. **Apply ArgoCD static LoadBalancer service**
```bash
   kubectl apply -f argocd-static
   ```

5. **Install MetalLB** (for bare-metal clusters)
```bash
   kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml
   kubectl apply -f metallb/metallb-conf.yaml
   ```

6. **Get ArgoCD admin password**
```bash
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
   ```

7. **Access ArgoCD UI**
   - Navigate to `http://10.10.90.51` or `https://10.10.90.51`
   - Username: `admin`
   - Password: (from step 6)

### Deploying Applications

#### Deploy Weather Service
```bash
kubectl apply -f automator.yaml
```

#### Deploy Monitoring Stack
```bash
kubectl apply -f monitoring-app.yaml
```

ArgoCD will automatically sync and deploy the applications from this repository.

## Technology Stack

### Infrastructure & Orchestration
- **Kubernetes** - Container orchestration platform
- **ArgoCD** - GitOps continuous delivery tool
- **MetalLB** - Bare-metal load balancer
- **K3s** - Lightweight Kubernetes distribution

### Monitoring & Observability
- **Prometheus** - Metrics collection and alerting
- **Grafana** - Visualization and analytics platform
- **Alertmanager** - Alert routing and management
- **kube-prometheus-stack** - Complete monitoring solution (v61.3.0)
- **Node Exporter** - Hardware and OS metrics exporter

### Application Deployment
- **Helm** - Kubernetes package manager
- **HorizontalPodAutoscaler** - Automatic pod scaling
- **Docker** - Container runtime

## Project Structure

```
ArgoCI-CD/
├── argocd-static              # ArgoCD LoadBalancer service configuration
├── automator.yaml             # ArgoCD application for weather service
├── monitoring-app.yaml        # ArgoCD application for monitoring stack
├── metallb/
│   └── metallb-conf.yaml     # MetalLB configuration with IP pool
├── monitoring/
│   ├── Chart.yaml            # Helm chart for monitoring stack
│   └── values.yaml           # Prometheus, Grafana, Alertmanager config
├── namespaces/
│   ├── argocd-ns.yaml        # ArgoCD namespace
│   └── metallb-ns.yaml       # MetalLB namespace
├── weerservice/
│   ├── deployment.yaml       # Weather service deployments & HPA
│   └── service.yaml          # LoadBalancer service
└── README.md                 # This file
```

## Configuration

### IP Address Allocation

The infrastructure uses the following static IP assignments:

| Service | IP Address | Port | Purpose |
|---------|-----------|------|---------|
| **ArgoCD Server** | 10.10.90.51 | 80, 443 | GitOps management UI |
| **Grafana** | 10.10.90.50 | 80 | Monitoring dashboards |
| **Weather Service** | 10.10.90.52 | 80 | Weather application API |
| **IP Pool Range** | 10.10.90.30-140 | - | Available LoadBalancer IPs |

### MetalLB Configuration

Edit `metallb/metallb-conf.yaml` to adjust the IP address pool:

```yaml
spec:
  addresses:
    - 10.10.90.30-10.10.90.140  # Your IP range
```

### Monitoring Configuration

#### Grafana Access
- **URL**: `http://10.10.90.50`
- **Username**: `admin`
- **Password**: `prom-operator`

#### Custom Prometheus Scrape Targets

Add additional scrape targets in `monitoring/values.yaml`:

```yaml
additionalScrapeConfigs:
  - job_name: 'custom-target'
    scrape_interval: 15s
    static_configs:
      - targets: ['<host>:<port>']
        labels:
          nodename: 'custom-node'
```

### Weather Service Scaling

Adjust autoscaling parameters in `weerservice/deployment.yaml`:

```yaml
spec:
  minReplicas: 2          # Minimum pods
  maxReplicas: 10         # Maximum pods
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50  # Scale at 50% CPU
```

## Monitoring & Observability

### Available Metrics

- **Node Metrics**: CPU, memory, disk, network from K3s control plane
- **Application Metrics**: Weather service performance and health
- **Kubernetes Metrics**: Cluster state, pod status, resource utilization
- **Custom Scrapes**: External host monitoring (10.10.90.151:9100)

### Alertmanager Configuration

Alerts are configured with the following defaults:
- **Group Wait**: 30s
- **Group Interval**: 5m
- **Repeat Interval**: 12h
- **Resolve Timeout**: 5m

Customize alert routing in `monitoring/values.yaml` under `alertmanager.config`.

## GitOps Workflow

1. **Make changes** to YAML files in this repository
2. **Commit and push** to the `main` branch
3. **ArgoCD detects** changes automatically
4. **Auto-sync** deploys updates to the cluster
5. **Self-healing** ensures desired state is maintained

### Manual Sync (if needed)

```bash
argocd app sync weerservice
argocd app sync monitoring
```

### Check Application Status

```bash
argocd app get weerservice
argocd app get monitoring
```

## Troubleshooting

### ArgoCD Not Accessible

```bash
# Check ArgoCD pods
kubectl get pods -n argocd

# Check service
kubectl get svc -n argocd argocd-server

# Verify LoadBalancer IP
kubectl get svc -n argocd argocd-server -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

### MetalLB Issues

```bash
# Check MetalLB pods
kubectl get pods -n metallb-system

# Verify IP pool
kubectl get ipaddresspool -n metallb-system

# Check L2 advertisement
kubectl get l2advertisement -n metallb-system
```

### Application Sync Failures

```bash
# View sync status
argocd app get <app-name>

# Check logs
kubectl logs -n argocd <argocd-application-controller-pod>

# Force refresh
argocd app sync <app-name> --force
```

### Weather Service Not Responding

```bash
# Check pods
kubectl get pods -l app=weerservice

# Check HPA status
kubectl get hpa weerservice-hpa

# View pod logs
kubectl logs <pod-name>

# Check service
kubectl get svc weerservice
```

## Contributing

Contributions are welcome! Here's how you can help:

1. **Fork the repository**
2. **Create a feature branch**
```bash
   git checkout -b feature/amazing-feature
   ```
3. **Commit your changes**
```bash
   git commit -m 'Add some amazing feature'
   ```
4. **Push to the branch**
```bash
   git push origin feature/amazing-feature
   ```
5. **Open a Pull Request**

### Development Guidelines

- Follow Kubernetes best practices
- Use declarative YAML configurations
- Test changes in a development cluster first
- Document IP address changes
- Keep ArgoCD applications in sync
- Use meaningful commit messages

## Additional Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [MetalLB Documentation](https://metallb.universe.tf/)
- [Prometheus Operator](https://prometheus-operator.dev/)
- [Grafana Documentation](https://grafana.com/docs/)

## License

This project is for educational purposes.

---

**Authors:** Max Blaauw, Arne Jansonius, Kai Diemel & Ole Spiegelenberg
**Organization:** HHS (The Hague University of Applied Sciences)
**Domain:** hhs.nl