# Getting Started

### Running the Application

```
./gradlew bootRun
```

Open [http://localhost:8080](http://localhost:8080) in your browser.

### Building the Application

```
./gradlew bootJar
```

### Running the Application as a Docker Container

Build the image (see `Dockerfile`) or run the JAR locally:

```
./gradlew bootJar
java -jar ./build/libs/cloud-app-0.0.1-SNAPSHOT.jar
```

### Kubernetes on a cloud provider (internet-accessible)

For requirements **4** and **5**, the control plane should accept connections from your network (or the public internet where appropriate), and workloads should be reachable according to your ingress design. On **DigitalOcean** and **GKE**, the default is a **public API endpoint**; use private endpoints or a control-plane firewall only if you intentionally lock down access and still have a path for `kubectl` (VPN, bastion, or allowed IPs).

**Common managed options:** [Amazon EKS](https://docs.aws.amazon.com/eks/), [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine/docs), [Azure Kubernetes Service (AKS)](https://learn.microsoft.com/azure/aks/), [DigitalOcean Kubernetes (DOK)](https://docs.digitalocean.com/products/kubernetes/).

**Simplest for a portfolio or demo:** **DigitalOcean Kubernetes** (small clusters, straightforward UI and CLI) or **GKE Autopilot** (Google runs the nodes; you do not pick a fixed node count—capacity scales automatically; use **GKE Standard** if you need an explicit 2–3 node pool).

#### Install kubectl

Install a client version within [one minor version](https://kubernetes.io/releases/version-skew-policy/) of your cluster.

- **Official install guides:** [Install Tools](https://kubernetes.io/docs/tasks/tools/) (Windows, macOS, Linux).
- **Windows (winget):** `winget install Kubernetes.kubectl`
- **macOS (Homebrew):** `brew install kubectl`
- **Verify:** `kubectl version --client`

#### Create a cluster (2–3 nodes, public API)

Use at least **two nodes** for scheduling spread; **three** is typical for demos and tolerates one node failure.

**DigitalOcean Kubernetes**

1. Install [doctl](https://docs.digitalocean.com/reference/doctl/how-to/install/) and run `doctl auth init`.
2. Create the cluster with **three nodes** in the default pool (`--count 2` is valid if you only need two). The API server is **public by default**; skip `--enable-control-plane-firewall` unless you will allow your own IP ranges.

```
doctl kubernetes cluster create cloud-app-demo \
  --region nyc1 \
  --size s-2vcpu-4gb \
  --count 3 \
  --auto-upgrade=true \
  --maintenance-window saturday=04:00
```

3. `doctl` usually merges kubeconfig automatically (`--update-kubeconfig` defaults to true). To save or refresh credentials explicitly: `doctl kubernetes cluster kubeconfig save cloud-app-demo`

**GKE Standard** (explicit nodes; public endpoint by default—omit private-cluster-only flags)

```
gcloud config set project YOUR_PROJECT_ID
gcloud container clusters create cloud-app-demo \
  --region us-central1 \
  --num-nodes 3 \
  --machine-type e2-medium \
  --release-channel regular
gcloud container clusters get-credentials cloud-app-demo --region us-central1
```

**GKE Autopilot** (no fixed node count; public control plane)

```
gcloud container clusters create-auto cloud-app-demo --region us-central1
gcloud container clusters get-credentials cloud-app-demo --region us-central1
```

For **EKS** and **AKS**, use the respective console or CLI (`eksctl`, `az aks create`) and choose a public (non-private-only) endpoint unless you intentionally use a bastion or VPN.

### Requirements

1. This project should be made to run as a Docker image.
2. Docker image should be published to a Docker registry.
3. Docker image should be deployed to a Kubernetes cluster.
4. Kubernetes cluster should be running on a cloud provider.
5. Kubernetes cluster should be accessible from the internet.
6. Kubernetes cluster should be able to scale the application.
7. Kubernetes cluster should be able to update the application without downtime.
8. Kubernetes cluster should be able to rollback the application to a previous version.
9. Kubernetes cluster should be able to monitor the application.
10. Kubernetes cluster should be able to autoscale the application based on the load.
12. Application logs should be stored in a centralised logging system (Loki, Kibana, etc.)
12. Application should be able to send metrics to a monitoring system.
13. Database should be running on a separate container.
14. Storage should be mounted to the database container.
