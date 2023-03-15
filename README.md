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

### GitOps with Flux

## GitOps with Flux

### Install Flux CLI

```bash
curl -s https://fluxcd.io/install.sh | sudo bash
```

### Setup git repository for flux (fleet-infra)

- Create an empty private github repository(flux-infra)
- Create a PAT token with access to contents, deploy keys for the above created repository
- Store the PAT token as an environment variable - `GITHUB_TOKEN`
- Store the GitHub username as `GITHUB_USER`

### Install flux

```bash
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=flux-infra \
  --branch=main \
  --personal
```

### Setup git repository for application and kong manifests

- Create an SSH key pair using `ssh-keygen -t ed25519 -C "changme@example.com"`
- Create a deploy key in the `kong-assessment` repository with the above created public key with read only access

### Configure flux to sync manifests

- Create a secret for the git repository - 
```bash
flux create secret git kong-assessment-auth \
    --url=ssh://git@github.com/$GITHUB_USER/kong-assessment \
    --private-key-file=./id_ed25519 # This private key is the private key created in the previous section
```
- Create a file `kong-assessment-flux.yaml` with the following contents and check it into the _flux-infra_ repository - 
```yaml
---
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: kong-assessment
  namespace: flux-system
spec:
  interval: 1m0s
  ref:
    branch: main
  secretRef:
    name: kong-assessment-auth
  url: ssh://git@github.com/avinashupadhya99/kong-assessment
---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: kong-assessment
  namespace: flux-system
spec:
  interval: 10m0s
  path: ./
  prune: true
  sourceRef:
    kind: GitRepository
    name: kong-assessment
```

