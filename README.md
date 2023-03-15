# Kong Assessment

## Kong on Docker Compose

> Read the detailed README at [docker-compose/README.md](docker-compose/README.md)

```bash
cd docker-compose
docker-compose up -d
# Bring up all kong components with configuration
deck sync --headers "Kong-Admin-Token:password"
```

### GitOps with decK

```bash
# Dump kong config
deck dump --headers "Kong-Admin-Token:password"
```

- GitHub workflow brings up the kong environment with docker-compose.
- Run decK sync against the docker compose environment brought up earlier (Ideally, it would point to the actual kong instance's Admin API)

## Kong on Kubernetes

> Read the detailed README at [kubernetes/README.md](kubernetes/README.md)

```bash
# Create cluster
eksctl create cluster -f eks-config.yaml # Or create/use any k8s cluster

# Install kong
helm repo add kong https://charts.konghq.com
helm repo update
helm install kong kong/kong -n kong --create-namespace

# Install Kong components, k8s services, ingress
# Replace the host with the loadbalancer IP of kong-proxy
kubectl apply -f kubernetes/
```