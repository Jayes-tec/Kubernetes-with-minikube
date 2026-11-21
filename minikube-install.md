# Minikube and kubectl Installation Guide on WSL2 (Ubuntu)

## 1. Introduction

This guide explains how to set up a local Kubernetes learning environment using **Minikube**, **kubectl**, and **Docker Desktop** on Windows Subsystem for Linux 2 (WSL2) with Ubuntu.

The objective is to install the required tools, create a local Kubernetes cluster, configure Kubernetes access, and verify that the cluster is operational.

### 1.1 What is Minikube?

Minikube is a tool that runs a local Kubernetes cluster on a developer's machine.

It is primarily used for:

- Learning Kubernetes.
- Developing and testing Kubernetes applications.
- Practicing Kubernetes resources such as Pods, Deployments, and Services.
- Testing Kubernetes YAML manifests locally.
- Experimenting with cluster configurations.

Minikube can create single-node and multi-node Kubernetes clusters using supported drivers.

### 1.2 What is kubectl?

`kubectl` is the Kubernetes command-line interface (CLI).

It allows users to communicate with a Kubernetes cluster through the Kubernetes API Server.

Examples:

```bash
kubectl get nodes
kubectl get pods
kubectl get deployments
kubectl describe pod <pod-name>
kubectl apply -f deployment.yaml
```

### 1.3 What is Docker Desktop's role?

Docker Desktop provides the container runtime environment used by Minikube's Docker driver.

In this setup:

- Windows runs Docker Desktop.
- WSL2 provides the Ubuntu development environment.
- Minikube creates and manages the local Kubernetes cluster.
- Docker runs the Minikube node container.
- kubectl communicates with the Kubernetes API Server.

### 1.4 Architecture of our local setup

```text
Windows 11
    |
    +-- Docker Desktop
    |       |
    |       +-- Docker Engine
    |               |
    |               +-- Minikube Node Container
    |                       |
    |                       +-- Kubernetes Components
    |                               |
    |                               +-- API Server
    |                               +-- etcd
    |                               +-- Scheduler
    |                               +-- Controller Manager
    |                               +-- Kubelet
    |
    +-- WSL2
            |
            +-- Ubuntu
                    |
                    +-- Minikube CLI
                    +-- kubectl CLI
                    +-- Kubernetes Configuration
```

---

## 2. System Requirements

Before installing Minikube, verify that the system meets the basic requirements.

### 2.1 Required Software

| Software | Purpose |
|---|---|
| Windows 11 | Host operating system |
| WSL2 | Linux environment on Windows |
| Ubuntu | Linux distribution |
| Docker Desktop | Container runtime |
| Minikube | Local Kubernetes cluster manager |
| kubectl | Kubernetes command-line client |

### 2.2 Minimum resources

Minikube's documented starting requirements include:

- 2 CPUs or more.
- 2 GB of free memory.
- 20 GB of free disk space.
- Internet connectivity.
- A supported container or virtual machine driver.

For a comfortable learning environment, additional memory and disk space are recommended.

### 2.3 Verify WSL and Ubuntu

Open the Ubuntu terminal and execute:

```bash
uname -a
```

This displays Linux kernel and system information.

Check the Ubuntu version:

```bash
lsb_release -a
```

Check available memory:

```bash
free -h
```

Check available CPU cores:

```bash
nproc
```

Check disk availability:

```bash
df -h /
```

## 3. Docker Desktop Setup

Minikube needs a supported driver to create and run its Kubernetes node.

We will use the Docker driver.

### 3.1 Install Docker Desktop

Download Docker Desktop for Windows:

https://www.docker.com/products/docker-desktop/

Install Docker Desktop and start it.

### 3.2 Enable WSL2 Backend

Open Docker Desktop:

1. Go to Settings.
2. Open General.
3. Enable **Use the WSL 2 based engine**.
4. Click Apply and restart if required.

### 3.3 Enable Ubuntu WSL Integration

Open:

Settings → Resources → WSL Integration

Enable integration for your Ubuntu distribution.

Apply the changes.

This allows Ubuntu to access Docker Desktop's Docker Engine through the WSL integration.

**Important:** Do not install a second Docker Engine inside Ubuntu when using Docker Desktop's WSL integration. Mixing separate Docker daemons can create configuration and context confusion.

### 3.4 Verify Docker Installation

Open Ubuntu and run:

```bash
docker version
```

This displays both:

- Docker Client information.
- Docker Server/Engine information.

Check running containers:

```bash
docker ps
```

Check all containers, including stopped containers:

```bash
docker ps -a
```

Check Docker information:

```bash
docker info
```

**Expected result:**

The Docker client and server should both return their version and engine information without connection errors.

If the server section is missing or Docker reports a connection error, check that Docker Desktop is running and Ubuntu WSL integration is enabled.

---

## 4. Install Minikube

We will install the Linux AMD64 Minikube binary.

### 4.1 Download Minikube

Run:

```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
```

**What does this command do?**

| Part | Explanation |
|---|---|
| `curl` | Downloads data from a URL |
| `-L` | Follows HTTP redirects |
| `-O` | Saves the file using its original filename |
| URL | Official Minikube GitHub release download |

The downloaded file is:

```text
minikube-linux-amd64
```

### 4.2 Install Minikube

Run:

```bash
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

**Why `/usr/local/bin`?**

`/usr/local/bin` is a standard location for locally installed executable programs.

It is commonly included in the system's PATH, allowing us to execute Minikube from any directory.

The `install` command copies the binary to the destination and installs it with executable permissions.

### 4.3 Remove the downloaded binary

After successful installation:

```bash
rm minikube-linux-amd64
```

We no longer need the downloaded installer file in our home directory.

### 4.4 Verify Minikube

Run:

```bash
minikube version
```

Expected output format:

```text
minikube version: vX.Y.Z
commit: <commit-hash>
```

The exact version changes with new releases.

**Installation is successful when the Minikube version is displayed.**

---

## 5. Install kubectl

`kubectl` is required to interact with the Kubernetes cluster.

We will install the latest stable Linux AMD64 client.

### 5.1 Download kubectl

Run:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

**How does this command work?**

The inner command:

```bash
curl -L -s https://dl.k8s.io/release/stable.txt
```

Retrieves the latest stable Kubernetes release version.

The outer command uses that version to construct the download URL.

For example, if the stable version is `v1.xx.x`, the resulting URL becomes:

```text
https://dl.k8s.io/release/v1.xx.x/bin/linux/amd64/kubectl
```

The binary is downloaded as:

```text
kubectl
```

### 5.2 Install kubectl

Run:

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Explanation:

| Argument | Purpose |
|---|---|
| `-o root` | Sets file owner to root |
| `-g root` | Sets file group to root |
| `-m 0755` | Sets executable permissions |
| `/usr/local/bin/kubectl` | Installation destination |

### 5.3 Remove downloaded binary

```bash
rm kubectl
```

### 5.4 Verify kubectl

Run:

```bash
kubectl version --client
```

Expected output format:

```text
Client Version: vX.Y.Z
Kustomize Version: vX.Y.Z
```

**Important:** This verifies only the kubectl client installation. It does not verify that a Kubernetes cluster exists or is running.

### 5.5 Kubernetes version compatibility

The kubectl version should be within one minor version of the Kubernetes cluster version.

For example:

- kubectl v1.36 with cluster v1.37: supported.
- kubectl v1.37 with cluster v1.37: supported.
- kubectl v1.38 with cluster v1.37: supported.

Always verify the actual cluster version before diagnosing compatibility problems.

---

## 6. Verify All Installed Tools

Run:

```bash
minikube version
kubectl version --client
docker version
```

At this stage:

- Minikube should be installed.
- kubectl should be installed.
- Docker Client and Server should be accessible.

The Kubernetes cluster has not necessarily been created yet.

---

## 7. Create the Kubernetes Cluster

Now we will create our first local Kubernetes cluster using Minikube.

### 7.1 Start Minikube with Docker Driver

Run:

```bash
minikube start --driver=docker --cpus=2 --memory=3072
```

Explanation:

| Option | Meaning |
|---|---|
| `minikube start` | Creates or starts the Minikube cluster |
| `--driver=docker` | Uses Docker as the infrastructure driver |
| `--cpus=2` | Allocates 2 CPUs to the Minikube node |
| `--memory=3072` | Allocates 3072 MB of memory |

### 7.2 What happens internally?

During the first startup, Minikube generally:

1. Checks the selected Docker driver.
2. Downloads the required Kubernetes node image.
3. Creates the Minikube node container through Docker.
4. Configures the Kubernetes node.
5. Starts the Kubernetes control plane components.
6. Configures kubelet and the container runtime.
7. Configures networking and required certificates.
8. Writes or updates Kubernetes access information in kubeconfig.
9. Configures the Minikube kubectl context.

The first startup can take several minutes because images and other required components may need to be downloaded.

### 7.3 Check Docker containers

Run:

```bash
docker ps
```

You should see a Minikube-related container after successful cluster creation.

Inspect its details:

```bash
docker inspect minikube
```

The exact container name can differ if a different Minikube profile is used.

---

## 8. Verify the Kubernetes Cluster

After Minikube starts successfully, verify the cluster.

### 8.1 Check Minikube Status

```bash
minikube status
```

This reports the status of the Minikube profile and its components, such as:

- Host.
- Kubelet.
- API Server.
- Kubeconfig.

### 8.2 Check Kubernetes Context

```bash
kubectl config get-contexts
```

This displays the available Kubernetes contexts.

Check the currently selected context:

```bash
kubectl config current-context
```

The default Minikube context is normally:

```text
minikube
```

### 8.3 Check Cluster Nodes

```bash
kubectl get nodes
```

This lists the Kubernetes nodes.

Expected output format:

```text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...   vX.Y.Z
```

The node should eventually reach `Ready` status.

### 8.4 Check Cluster Information

```bash
kubectl cluster-info
```

This displays the Kubernetes control-plane endpoint and other available service information.

### 8.5 Check Pods Across All Namespaces

```bash
kubectl get pods -A
```

`-A` means `--all-namespaces`.

This command lists Pods from all namespaces, including system Pods.

### 8.6 Check Namespaces

```bash
kubectl get namespaces
```

This displays the namespaces configured in the cluster.

### 8.7 Check Cluster API Readiness

```bash
kubectl get --raw='/readyz?verbose'
```

This queries the API Server's readiness endpoint and provides detailed checks.

A successful readiness response indicates that the API Server considers itself ready.

---

## 9. Understanding kubeconfig

Kubernetes uses a configuration file called `kubeconfig` to store connection and authentication information.

The default location is:

```text
~/.kube/config
```

Check the file:

```bash
ls -la ~/.kube/
```

View configured contexts:

```bash
kubectl config get-contexts
```

View the current context:

```bash
kubectl config current-context
```

View the cluster names:

```bash
kubectl config get-clusters
```

View the configuration:

```bash
kubectl config view
```

**Security note:** Kubeconfig can contain authentication information and certificates. Never commit private kubeconfig files or credentials to a public GitHub repository.

Minikube normally configures its own context when a cluster is started.

---

## 10. Common Minikube and kubectl Commands

### Cluster Management

```bash
minikube start
minikube stop
minikube status
minikube delete
minikube profile list
minikube ip
```

### Node Management

```bash
kubectl get nodes
kubectl describe node minikube
```

### Pod Management

```bash
kubectl get pods
kubectl get pods -A
kubectl get pods -o wide
```

### Namespace Management

```bash
kubectl get namespaces
kubectl create namespace development
kubectl delete namespace development
```

### Context Management

```bash
kubectl config get-contexts
kubectl config current-context
kubectl config use-context minikube
```

### Cluster Information

```bash
kubectl cluster-info
kubectl version
```

---

## 11. Troubleshooting

### Problem 1: kubectl connects to localhost:8080 and fails

Example:

```text
The connection to the server localhost:8080 was refused
```

**Possible reason:**

- Kubernetes cluster has not been created.
- Minikube has not started.
- kubectl does not have a valid current context.
- Kubeconfig is missing or incorrect.

**Diagnosis:**

```bash
minikube status
kubectl config get-contexts
kubectl config current-context
```

If no Minikube profile exists, start the cluster:

```bash
minikube start --driver=docker
```

If the profile exists but is stopped:

```bash
minikube start
```

### Problem 2: Docker daemon is not running

Example:

```text
Cannot connect to the Docker daemon
```

**Possible reason:**

Docker Desktop is stopped or WSL integration is disabled.

**Solution:**

1. Start Docker Desktop.
2. Enable the WSL2 engine.
3. Enable Ubuntu under WSL Integration.
4. Restart the Ubuntu terminal.
5. Verify:

```bash
docker version
docker ps
```

### Problem 3: Minikube profile not found

Example:

```text
Profile "minikube" not found
```

**Meaning:**

No Minikube profile with that name currently exists.

**Solution:**

```bash
minikube start --driver=docker
```

### Problem 4: Kubernetes node is NotReady

Check:

```bash
kubectl get nodes
kubectl describe node minikube
kubectl get pods -A
```

Inspect relevant Pod logs:

```bash
kubectl logs <pod-name> -n <namespace>
```

The `describe` output and Pod events help identify scheduling, networking, image-pull, or resource problems.

### Problem 5: Insufficient memory

Symptoms may include:

- Minikube startup failure.
- Pods stuck in Pending.
- Containers being terminated.
- Out-of-memory errors.

Check:

```bash
free -h
docker stats
kubectl describe nodes
```

Reduce cluster resource allocation or stop unnecessary workloads if the machine is under memory pressure.

### Problem 6: WSL networking issues

With Minikube's Docker driver in WSL, the Minikube node IP may not be directly reachable from the Windows host.

For NodePort services, use:

```bash
minikube service <service-name> --url
```

For LoadBalancer services, use:

```bash
minikube tunnel
```

Refer to Minikube's WSL Docker driver documentation for current networking limitations.

---

## 12. Stop, Restart and Delete the Cluster

### Stop Minikube

```bash
minikube stop
```

This stops the local cluster while retaining its profile and configuration.

### Start it again

```bash
minikube start
```

### Delete the cluster

```bash
minikube delete
```

This deletes the Minikube cluster and its associated resources.

**Warning:** Deleting a cluster removes its Kubernetes workloads and cluster-local data. Do not use this command if you need to preserve your workloads or persistent data.

---

## 13. Final Verification Checklist

After completing the setup, verify each item.

- [ ] WSL2 is installed and working.
- [ ] Ubuntu is accessible.
- [ ] Docker Desktop is installed.
- [ ] Docker WSL integration is enabled.
- [ ] Docker Client and Server are working.
- [ ] Minikube is installed.
- [ ] kubectl is installed.
- [ ] Minikube cluster is created.
- [ ] Minikube status is healthy.
- [ ] kubectl context is configured.
- [ ] Kubernetes node reaches Ready status.
- [ ] System Pods are running.
- [ ] Kubernetes API Server is accessible.

---

## 14. Official Documentation and References

Use official documentation for updated installation instructions and compatibility information.

| Resource | Documentation |
|---|---|
| Minikube Official Website | https://minikube.sigs.k8s.io/docs/ |
| Minikube Installation | https://minikube.sigs.k8s.io/docs/start/ |
| Minikube Docker Driver | https://minikube.sigs.k8s.io/docs/drivers/docker/ |
| Minikube WSL Docker Setup | https://minikube.sigs.k8s.io/docs/tutorials/wsl_docker_driver/ |
| Kubernetes Official Documentation | https://kubernetes.io/docs/ |
| kubectl Installation | https://kubernetes.io/docs/tasks/tools/ |
| Docker Desktop WSL Integration | https://docs.docker.com/desktop/features/wsl/ |

---

## 15. Installation Environment Used

The following environment was used while preparing and testing this guide.

| Component | Version / Configuration |
|---|---|
| Host OS | Windows 11 |
| Linux Environment | WSL2 |
| Distribution | Ubuntu 26.04 LTS |
| Architecture | Linux AMD64 |
| CPU | 8 logical CPUs |
| Available WSL Memory | Approximately 6.7 GiB |
| Docker Desktop | 4.82.0 |
| Docker Engine | 29.6.1 |
| Minikube | v1.39.0 |
| kubectl | v1.37.2 |
| Minikube Driver | Docker |

**Note:** Versions shown above represent the environment used during this setup. They are not fixed requirements. Always check the latest supported releases and compatibility before installing.

---

