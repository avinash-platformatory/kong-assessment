# Kong on Docker Compose

```bash
export KONG_LICENSE_DATA='' # Set license data
docker-compose up -d
```

## Upstreams

### Create an Upstream

```bash
curl -H "Kong-Admin-Token:password" -X POST http://localhost:8001/upstreams \
  --data name=httpbin-upstream \
  --data healthchecks.active.healthy.interval=5 \
  --data healthchecks.active.unhealthy.interval=5
```

### Create Targets

```bash
curl -H "Kong-Admin-Token:password" -X POST http://localhost:8001/upstreams/httpbin-upstream/targets \
  --data target='httpbin:80'
curl -H "Kong-Admin-Token:password" -X POST http://localhost:8001/upstreams/httpbin-upstream/targets \
  --data target='httpbin.org:80'
```

### Create Service

```bash
curl -H "Kong-Admin-Token:password" -X POST http://localhost:8001/services \
  --data name=httpbin-service \
  --data host='httpbin-upstream' \
  --data path='/anything'
```

### Create Route

```bash
curl -H "Kong-Admin-Token:password" -X POST http://localhost:8001/services/httpbin-service/routes \
  --data 'paths[]=/echo' \
  --data name=echo-route
```

### Verify route creation

```bash
curl localhost:8000/echo
```

### Mark target as unhealthy

```bash
curl -H "Kong-Admin-Token:password" -X POST http://localhost:8001/upstreams/httpbin-upstream/targets/httpbin:80/unhealthy
```

### Verify routing to the healthy target

```bash
curl localhost:8000/echo
```

### Mark target as healthy

```bash
curl -H "Kong-Admin-Token:password" -X POST http://localhost:8001/upstreams/httpbin-upstream/targets/httpbin:80/healthy
```

### Verify routing to both targets

```bash
curl localhost:8000/echo
```

## Rate limiting per consumer

### Enable key authentication plugin

```bash
curl -H "Kong-Admin-Token:password" -X POST http://localhost:8001/routes/echo-route/plugins \
  --data "name=key-auth"  \
  --data "config.key_names=apikey"
```

### Create consumers

```bash
curl -H "Kong-Admin-Token:password" -X POST http://localhost:8001/consumers \
  --data "username=user1"
curl -H "Kong-Admin-Token:password" -X POST http://localhost:8001/consumers \
  --data "username=user2"
```

### Create API Keys for the consumers

```bash
curl -H "Kong-Admin-Token:password" -X POST http://localhost:8001/consumers/user1/key-auth \
  --data "key=key1"
curl -H "Kong-Admin-Token:password" -X POST http://localhost:8001/consumers/user2/key-auth \
  --data "key=key2"
```

### Enable rate limiting plugin

```bash
curl -H "Kong-Admin-Token:password" -X POST http://localhost:8001/consumers/user1/plugins \
  --data "name=rate-limiting"  \
  --data "config.minute=5" \
  --data "config.policy=local"
curl -H "Kong-Admin-Token:password" -X POST http://localhost:8001/consumers/user2/plugins \
  --data "name=rate-limiting"  \
  --data "config.minute=10" \
  --data "config.policy=local"
```

### Verify rate limting per consumer

```bash
curl localhost:8000/echo -H "apikey:key1"
curl localhost:8000/echo -H "apikey:key2"
```
