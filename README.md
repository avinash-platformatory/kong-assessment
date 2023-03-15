# Kong Assessment

## Kong on Docker Compose

> Read the detailed README at [docker-compose/README.md](docker-compose/README.md)

```bash
cd docker-compose
docker-compose up -d
# Bring up all kong components with configuration
deck sync --headers "Kong-Admin-Token:password"
```

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
kubectl apply -f kubernetes/
```