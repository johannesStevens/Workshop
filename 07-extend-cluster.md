# Extend the cluster

We want to extend the cluster to become a fully functional devops center.

For this we need the following components:

- Docker Repository: Harbor 
- Database: Postgresql
- Monitoring: Prometheus
- Source Control: Gitea
- Pipeline: Drone
- Deployment: ArgoCD
- Secrets: Key Vault
- Backup: Velero
- Cache: Redis
- Centralized Logging: Loki

# Service Accounts

Service Accounts are needed for applications to interact with the cluster
This will be relevant for:
- ArgoCD
- Prometheus
- Velero
- Drone/CI

# Other Kubernetes Topics

## Components:
- ConfigMaps
- Resource Limits
- Autoscaling
- Rolling Updates
- Rollbacks
- Node Affinity
- Taints & Tolerations

## Security

- Pod Security
- Image Scanning

## Operations

- Debugging
- Node Maintenance
- Cluster upgrades
- disaster recovery
- patch resource definitions

## Advanced

- Custom Resource Definitions
- Operators
- kustomize