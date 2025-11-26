# Introduction & Goal of Training

## Introduction & Goal of Training

This self-paced workshop is designed to equip you with both the hands-on skills and the conceptual understanding needed to use **mirrord** effectively for local development and debugging in Kubernetes environments.

Modern cloud-native development often involves microservices running in remote clusters — making it difficult to debug or test changes locally without rebuilding and redeploying your entire application. **mirrord** solves this problem by letting you run your local code as if it were inside the cluster, while still using your local IDE, debugger, and tools.

## Training Objectives

By the end of this training, you will be able to:

- **Understand and install mirrord** – Learn what mirrord is, how it works under the hood, and how to install and configure it on your local workstation.
- **Run local applications in a cluster context** – Use mirrord to proxy your local application’s network traffic, environment variables, and filesystem access through a live Kubernetes pod — without needing to commit, build, or redeploy code.
- **Deploy and test a sample application (httpbin)** – Deploy a sample httpbin application into your Kubernetes cluster using Helm, and connect to it locally through mirrord to simulate real production interactions.
- **Execute targeted mirrord sessions** – Identify specific pods, launch `mirrord exec` sessions, and validate that your local code behaves exactly like it would inside the cluster.

## What is mirrord?

mirrord is an open-source tool (by MetalBear) that acts as a bridge between your local environment and your remote Kubernetes cluster.

It enables you to:

- Run local code "inside" the cluster context – mirrord injects your local process into the network namespace of a chosen pod.
- Reuse the cluster’s configuration – your local process inherits environment variables, secrets, and service endpoints from the remote pod.
- Intercept traffic – mirrord can forward a portion or all of the pod’s incoming traffic to your local app, allowing you to test your code against live workloads.
- Debug safely – run locally but observe real traffic, databases, and dependencies from the remote environment without affecting other users.

### Common Use Cases

- **Debugging in real cluster context** – Test and debug your new feature locally using live cluster data and dependencies — without redeploying.
- **Validating service interactions** – Ensure your service communicates correctly with upstream or downstream microservices in the cluster.
- **Testing environment configurations** – Quickly confirm your app behaves as expected with the cluster’s secrets, config maps, and environment variables.
- **Speeding up the dev loop** – Replace the build–push–deploy cycle with an instant local iteration process.

