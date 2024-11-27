# Kubernetes Applications

This repository contains Kubernetes configuration files for deploying various self-hosted applications and services.

## Applications

Current applications include:
- ChangeDetection.io
- Firefox
- Flame dashboard
- Ghost blog
- Homarr dashboard
- LittleLink
- MySQL + MySQL Workbench
- Nextcloud
- Pi-hole
- SmokePing
- Speedtest Tracker
- TeamCity
- And more...

## Getting Started

1. Ensure you have a functioning Kubernetes cluster
2. Install kubectl command-line tool
3. Clone this repository

## Usage

Most applications can be deployed using:

```bash
kubectl apply -f <application-directory>/
```

### Common Components

- **Deployments**: Define container specs, replicas, and runtime configs
- **Services**: Expose applications within cluster
- **PersistentVolumeClaims**: Handle persistent storage
- **ConfigMaps/Secrets**: Manage application configuration
- **Ingress**: Configure external access

## Structure

Each application has its own directory containing:
- Deployment files
- Service definitions
- Storage configurations
- Additional app-specific resources

## Contributing

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

## License

MIT License