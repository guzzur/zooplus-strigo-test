# Lab 1: Using mirrord

Mirrord Local Development Setup

## Local Setup

Please refer to the [official installation](https://metalbear.com/mirrord/docs/overview/quick-start#cli-tool) guide for detailed instructions for your local workstation:

---

## Install mirrord to the lab env

```bash
curl -fsSL https://raw.githubusercontent.com/metalbear-co/mirrord/main/scripts/install.sh | bash
```

---

## Deploy httpbin App

### 1. Create a `values.yaml` file

Generate `values.yaml` with the minimal required settings:

   ```yaml
   ---
   classes:
     - lab_resources:
         - display_name: "AWS EC2 Instance"
           platform_type: "lab"
           image_platform: "linux"
           view_interface: "desktop"
           cloud_provider: "aws"
           aws_vm_definition:
             machine_size: "t2.micro"
             ami_region_mapping:
               eu-west-1:
                 image_id: "ami-0b32f4a77e48fcb24"
       exercises:
         - file: "../hello-world/hello.md"
           title: "Hello!"
         - file: "../hello-world/world.md"
           title: "World!"
   ```


### 2. Deploy

```bash
helm repo update
helm upgrade --install ${ZOOPLUS_USER_NAME}-zp-app zhelm/zp-app \
  -n product-zoodemy-k8s-training-6 \
  -f values.yaml \
  --wait --timeout 15m
```

### 3. Check if the service is accessible

```bash
curl https://httpbin-${ZOOPLUS_USER_NAME}.zdemyk8sd.int.aws.zooplus.io/headers
```

You should see a response similar to:

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin-stanislav-ogarkov.zdemyk8sd.int.aws.zooplus.io",
    "Traceparent": "00-6465e9af438ccc6092479a95947530d5-cb07d2bc6b068729-00",
    "Tracestate": "",
    "User-Agent": "curl/7.68.0",
    "X-Envoy-Attempt-Count": "1",
    "X-Envoy-External-Address": "10.152.0.97",
    "X-Forwarded-Client-Cert": "By=spiffe://zoodemy-k8s-dev.zdemyk8sd.int.aws.zooplus.io/ns/product-zoodemy-k8s-training-6/sa/stanislav-ogarkov-zp-app-stanislav-ogarkov-http;Hash=d7626d055f2d22c74bcfd5394fddebf6dcd29f34e54687a0db359a6167ed7c02;Subject=\"\";URI=spiffe://zoodemy-k8s-dev.zdemyk8sd.int.aws.zooplus.io/ns/istio-system/sa/ingressgateway-private-service-account"
  }
}
```

---

## Run mirrord – local curl against pod localhost

Execute local curl against the pod’s localhost:

```bash
mirrord exec --target deployment/${ZOOPLUS_USER_NAME}-zp-app-httpbin \
  -n product-zoodemy-k8s-training-6 -- \
  curl http://localhost:80/headers
```

Expected output:

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "localhost",
    "User-Agent": "curl/7.68.0"
  }
}
```

You have used curl from the **lab environment** against **localhost** of the pod running in Kubernetes.

---

## Execute local curl vs a cluster‑internal service

When your application depends on a service that is only accessible inside the cluster (for example, a Tier‑3 network or an internal API), you would normally use:

```bash
kubectl port-forward ...
```

With mirrord, this step is no longer needed. When your local process runs through mirrord, it automatically inherits the same network, DNS, and environment variables as the target pod inside Kubernetes.  
Your local application effectively behaves as if it were running inside the cluster — with direct access to internal services.

Run:

```bash
mirrord exec --target deployment/${ZOOPLUS_USER_NAME}-zp-app-httpbin \
  -n product-zoodemy-k8s-training-6 -- \
  curl http://alertmanager-operated.core-stack:9093/-/healthy
```

Expected output (trimmed):

```text
* Running command: curl http://alertmanager-operated.core-stack:9093/-/healthy
* mirrord will target: deployment/…-zp-app-httpbin, no configuration file was loaded
...
OK
```

This shows that your curl is reaching a **cluster‑internal** service without any port‑forwarding or redeploy.

---

## Steal traffic from a remote service to your local process

Imagine you’ve deployed a feature branch to Kubernetes and the application logs an unclear error — and even worse, the error only happens when another service in the cluster calls yours.  
The old debugging cycle is:

- Change the code to add debug logging  
- Commit the change  
- Wait for the build and deploy  
- Reproduce the issue and inspect the logs  

With mirrord you don’t need to do all that. You can run your application locally (even from your IDE) and redirect traffic from the cluster to your local process. No compilation, no Docker build, no deploy — **instant iteration**.

### Create `mirrord.json`

```bash
cat <<EOF >> mirrord.json
{
  "feature": {
    "network": {
      "incoming": {
        "mode":"steal",
        "port_mapping": [[ 8080, 80 ]]
      },
      "outgoing": true
    },
    "fs": "read",
    "env": true
  }
}
EOF
```

### Run local Python server with mirrord

```bash
mirrord exec -f mirrord.json \
  --target deployment/${ZOOPLUS_USER_NAME}-zp-app-httpbin \
  -n product-zoodemy-k8s-training-6 \
  python3 -- -m http.server 8080
```

Once Python reports that the web server has started, you can call the remote service from another terminal:

![mirrord traffic stealing](mirrod.png)

```bash
curl https://httpbin-${ZOOPLUS_USER_NAME}.zdemyk8sd.int.aws.zooplus.io/
```

In the output, you should see the **directory listing** instead of the normal httpbin UI – this confirms that mirrord is stealing the traffic and sending it to your local process.

---

## Cleanup

```bash
helm delete ${ZOOPLUS_USER_NAME}-zp-app -n product-zoodemy-k8s-training-6
```
