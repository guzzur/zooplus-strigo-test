# Lab 1: Using mirrord

In this lesson you will:
- Install **mirrord** in the lab environment
- Deploy the sample **httpbin** application
- Use `mirrord exec` to run local `curl` against the Kubernetes pod
- Call an internal cluster service without port‑forwarding

---

## 1. Install mirrord in the lab

Run in the terminal:

```bash
curl -fsSL https://raw.githubusercontent.com/metalbear-co/mirrord/main/scripts/install.sh | bash
```

---

## 2. Deploy `httpbin`

Create `values.yaml`:

```bash
cat <<EOF2 >> values.yaml
app: httpbin
appVersion: 0.0.1
stage: dev

image:
  repository: bin.private.zooplus.net/simonkowallik/httpbin@sha256
  tag: f176fb37b6a0a242153d3ee08163d1596f640384643dcc5ab82791895896fb9d

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi

routeTable:
  enabled: true
  hosts:
  - httpbin-${ZOOPLUS_USER_NAME}.zdemyk8sd.int.aws.zooplus.io
EOF2
```

Deploy:

```bash
helm repo update
helm upgrade --install ${ZOOPLUS_USER_NAME}-zp-app zhelm/zp-app \
  -n product-zoodemy-k8s-training-6 \
  -f values.yaml \
  --wait --timeout 15m
```

Verify:

```bash
curl https://httpbin-${ZOOPLUS_USER_NAME}.zdemyk8sd.int.aws.zooplus.io/headers
```

---

## 3. Run mirrord – local curl to pod localhost

```bash
mirrord exec --target deployment/${ZOOPLUS_USER_NAME}-zp-app-httpbin \
  -n product-zoodemy-k8s-training-6 -- \
  curl http://localhost:80/headers
```

You should see headers where `Host` is `localhost` – your curl runs in the lab, but traffic goes through the pod.

---

## 4. Access an internal service via mirrord

```bash
mirrord exec --target deployment/${ZOOPLUS_USER_NAME}-zp-app-httpbin \
  -n product-zoodemy-k8s-training-6 -- \
  curl http://alertmanager-operated.core-stack:9093/-/healthy
```

You should get `OK` from the internal Alertmanager service – no port‑forwarding needed.
