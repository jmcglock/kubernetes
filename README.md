# Kubernetes Application Manifests

This repository contains a collection of Kubernetes manifests for deploying self-hosted applications in a homelab or production environment. Each application is configured with security best practices, proper resource management, and modern deployment patterns.

## Table of Contents

- [Applications](#applications)
- [Prerequisites](#prerequisites)
- [Architecture](#architecture)
- [Installation](#installation)
- [Application Details](#application-details)
- [Security Considerations](#security-considerations)
- [Customization](#customization)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Applications

This repository includes manifests for the following applications:

### Networking & Infrastructure
- **Traefik**: Modern edge router and reverse proxy with automatic HTTPS
- **Pi-hole**: Network-wide ad blocking and DNS resolution
- **AdGuard Home**: Alternative DNS-based ad and tracker blocker

### Monitoring & Observability
- **Uptime Kuma**: Simple uptime monitoring and alerting
- **SmokePing**: Network latency monitoring and visualization
- **Speedtest Tracker**: Automated internet speed monitoring

### Productivity & Web Services
- **Nextcloud**: Self-hosted productivity platform with file sharing
- **Ghost Blog**: Modern publishing platform
- **MySQL & MySQL Workbench**: Database and management tools
- **Firefox**: Containerized browser for self-hosted applications

### Dashboards & Home Pages
- **Flame Dashboard**: Minimalist application dashboard
- **Homarr**: Customizable homepage for self-hosted services
- **LittleLink**: Simple homepage with social media links

### Utilities
- **ChangeDetection.io**: Monitor website changes and get notifications
- **TeslaMate**: A powerful, self-hosted data logger for your Tesla

## Prerequisites

- A functioning Kubernetes cluster (single or multi-node)
- kubectl configured to access your cluster
- Helm installed (for Traefik and other chart-based deployments)
- Persistent storage provider (Longhorn recommended)
- Basic understanding of Kubernetes concepts

## Architecture

This repository follows a consistent pattern for each application:

- **Namespace**: Dedicated namespace per application
- **Security Contexts**: Non-root users, restricted capabilities
- **Network Policies**: Proper network isolation
- **Resource Limits**: CPU and memory specifications
- **Persistent Storage**: Based on Longhorn storage class
- **Traefik Integration**: Modern IngressRoute objects with TLS

### Directory Structure

Each application has its own directory containing:
```
application-name/
├── application-namespace.yml      # Dedicated namespace
├── application-deployment.yml     # Main application deployment
├── application-pvc.yml            # Persistent volume claims
├── application-svc.yml            # Service definitions 
├── application-ingress.yml        # IngressRoute configuration
├── application-configmap.yml      # Configuration files
├── application-secret.yml         # Sensitive data (credentials)
└── application-netpolicy.yml      # Network policies
```

## Installation

### 1. Clone the Repository
```bash
git clone https://github.com/your-username/kubernetes.git
cd kubernetes
```

### 2. Set Up Traefik First
Traefik serves as the ingress controller for all applications:

```bash
kubectl create namespace traefik
kubectl apply -f traefik/traefik-dashboard-auth.yml
kubectl apply -f traefik/traefik-dashboard-basicauth-middleware.yml
kubectl apply -f traefik/default-headers.yml

# Install Traefik using Helm
helm repo add traefik https://traefik.github.io/charts
helm repo update
helm install traefik traefik/traefik --namespace traefik -f traefik/values.yml

# Apply Traefik dashboard IngressRoute
kubectl apply -f traefik/traefik-dashboard-ingress-route-with-auth.yml
```

### 3. Deploy Applications
Deploy applications in their dependency order:

```bash
# Deploy database first if needed
kubectl apply -f mysql/

# Then deploy applications
kubectl apply -f application-name/
```

Before deploying, customize the following in each application:
- Domain names in IngressRoute objects
- Storage sizes in PVC objects
- Credentials in Secret objects

## Application Details

### Traefik
- **Features**: Automatic HTTPS with Let's Encrypt, metrics, access logs
- **Security**: Non-root execution, secure headers, TLS enforcement
- **Dashboard**: Available at `traefik.yourdomain.com` with authentication

### Nextcloud
- **Purpose**: Self-hosted file storage and collaboration
- **Components**: Web app, database, Redis cache
- **Volumes**: Separate volumes for app data and database

### Pi-hole
- **Purpose**: Network-wide ad blocking
- **Features**: DNS server, DHCP capabilities, statistics dashboard
- **Configuration**: Uses ConfigMap for customization

### Speedtest Tracker
- **Purpose**: Monitor your internet connection speed
- **Components**: Web app and PostgreSQL database
- **Scheduling**: Configurable speed tests on a regular schedule

### Uptime Kuma
- **Purpose**: Simple uptime monitoring and alerting
- **Features**: Status pages, notification integrations
- **Storage**: Persistent storage for monitoring history

### SmokePing
- **Purpose**: Network latency monitoring and visualization
- **Features**: Ping statistics, visual charts, historical data
- **Configuration**: Customizable targets via ConfigMap

## Security Considerations

This repository implements several security best practices:

1. **Pod Security**:
   - Non-root user execution
   - Restricted capabilities
   - Read-only root filesystem where possible
   - Proper seccomp profiles

2. **Network Security**:
   - Network policies to restrict communication
   - HTTPS everywhere with proper TLS configuration
   - Secure headers for all web applications

3. **Secret Management**:
   - Kubernetes Secrets for credentials
   - No hardcoded passwords
   - Base64 encoding (consider a proper secret management solution for production)

4. **Resource Control**:
   - CPU and memory limits for all containers
   - Prevent resource exhaustion

## Customization

### Domain Names
Replace `yourdomain.com` in all IngressRoute objects with your actual domain:

```yaml
routes:
  - match: Host(`app.yourdomain.com`)
```

### Storage Sizes
Adjust PVC storage sizes based on your needs:

```yaml
resources:
  requests:
    storage: 10Gi  # Change based on requirements
```

### Application Versions
Update container image versions in deployments:

```yaml
containers:
- name: application
  image: application/image:version  # Update to specific version
```

## Troubleshooting

### Check Application Status
```bash
kubectl get pods -n application-namespace
kubectl describe pod pod-name -n application-namespace
kubectl logs pod-name -n application-namespace
```

### Verify Ingress Routes
```bash
kubectl get ingressroutes -A
kubectl describe ingressroute ingressroute-name -n application-namespace
```

### Storage Issues
```bash
kubectl get pvc -n application-namespace
kubectl describe pvc pvc-name -n application-namespace
```

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/new-app`
3. Commit your changes: `git commit -am 'Add new application'`
4. Push to the branch: `git push origin feature/new-app`
5. Submit a pull request
