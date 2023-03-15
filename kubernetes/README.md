# Kong on Kubernetes

## Create cluster

```bash
eksctl create cluster -f ../eks-config.yaml
```

## Install Kong Ingress Controller

```bash
helm repo add kong https://charts.konghq.com
helm repo update
helm install kong kong/kong -n kong --create-namespace
```

## Rate limiting per consumer

### Enable key authentication plugin

```bash
kubectl apply -f key-auth-plugin.yaml
```

### Create consumers

```bash
kubectl apply -f consumer1.yaml
kubectl apply -f consumer2.yaml
```

### Create API Keys for the consumers

```bash
kubectl apply -f key1.yaml
kubectl apply -f key2.yaml
```

### Enable rate limiting plugin

```bash
kubectl apply -f user1-rate-limiting.yaml
kubectl apply -f user2-rate-limiting.yaml
```

### Create Kubernetes service for echo

```bash
kubectl apply -f echo-service.yaml
```

### Get ingress domain

```bash
export PROXY_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].hostname}" service -n kong kong-kong-proxy)
```

### Create ingress

```bash
# Replace the host with the loadbalancer IP of kong-proxy
kubectl apply -f kong-ingress.yaml
```

### Verify rate limting per consumer

```bash
curl $PROXY_IP/echo -H "apikey:key1"
curl $PROXY_IP/echo -H "apikey:key2"
```