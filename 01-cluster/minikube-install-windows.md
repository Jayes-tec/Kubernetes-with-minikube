# Minikube and kubectl Installation Guide on Windows

## 1. Introduction

This guide explains how to install and configure Minikube and kubectl on Windows 10/11 using PowerShell and Docker Desktop.

The objective is to create a local Kubernetes environment for learning, development, testing, and practicing Kubernetes administration.

We will use:

- **Windows 11:** Host operating system.
- **PowerShell:** Command-line interface.
- **Docker Desktop:** Container runtime.
- **Minikube:** Local Kubernetes cluster manager.
- **kubectl:** Kubernetes command-line client.

At the end of this guide, we will have a working local Kubernetes cluster accessible through kubectl.

---

## 2. Understanding the Tools

### 2.1 What is Minikube?

Minikube is a tool that creates and manages local Kubernetes clusters.

It allows developers to run Kubernetes on their personal computers without requiring a separate physical server or cloud-based Kubernetes cluster.

Minikube is useful for:

- Learning Kubernetes.
- Testing Kubernetes manifests.
- Developing containerized applications.
- Practicing Kubernetes administration.
- Experimenting with cluster configurations.
- Testing applications before deploying them to a remote cluster.

### 2.2 What is kubectl?

`kubectl` is the official Kubernetes command-line client.

It communicates with the Kubernetes API Server to manage cluster resources.

Examples:

```powershell
kubectl get nodes
kubectl get pods
kubectl get deployments
kubectl describe pod <pod-name>
kubectl apply -f deployment.yaml
```

### 2.3 What is Docker Desktop?

Docker Desktop provides a Docker Engine that Minikube can use through its Docker driver.

In this guide, Docker Desktop will run Linux containers, including the Minikube node container.

### 2.4 Architecture of the Windows setup

```text
Windows 11
    |
    +-- PowerShell
    |       |
    |       +-- Minikube CLI
    |       +-- kubectl CLI
    |
    +-- Docker Desktop
            |
            +-- WSL2 Backend (recommended)
            |
            +-- Docker Engine
                    |
                    +-- Minikube Node Container
                            |
                            +-- Kubernetes Components
                                    |
                                    +-- API Server
                                    +-- etcd
                                    +-- Scheduler
                                    +-- Controller Manager
                                    +-- Kubelet
                                    +-- Container Runtime
```

**Important:** Docker Desktop's WSL2 backend is an implementation detail of Docker Desktop. In this guide, the commands are executed from Windows PowerShell, not from the Ubuntu WSL terminal.

---

## 3. System Requirements

Before installing the tools, verify the system requirements.

### 3.1 Required Software

| Software | Purpose |
|---|---|
| Windows 10/11 64-bit | Host operating system |
| PowerShell | Execute installation commands |
| Docker Desktop | Container runtime |
| Minikube | Local Kubernetes cluster |
| kubectl | Kubernetes CLI |
| Internet connection | Download binaries and container images |

### 3.2 Hardware Requirements

Minikube's documented starting requirements include:

- At least 2 CPUs.
- At least 2 GB free memory.
- Approximately 20 GB free disk space.
- Internet connectivity.
- A supported driver.

More resources may be required depending on the number of Pods and workloads.

### 3.3 Check Windows Version

Open PowerShell and run:

```powershell
winver
```

This displays the Windows version and build.

You can also run:

```powershell
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsArchitecture
```

### 3.4 Check CPU Information

```powershell
Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores, NumberOfLogicalProcessors
```

### 3.5 Check Memory

```powershell
Get-CimInstance Win32_ComputerSystem | Select-Object TotalPhysicalMemory
```

### 3.6 Check Disk Space

```powershell
Get-PSDrive -PSProvider FileSystem
```

Verify that sufficient space is available on the drive where Docker Desktop stores its data.

---

## 4. Install Docker Desktop

Minikube will use Docker as its driver.

### 4.1 Download Docker Desktop

Official download:

https://www.docker.com/products/docker-desktop/

Download the Windows installer and complete the installation.

### 4.2 Configure Docker Desktop

Open Docker Desktop.

Recommended configuration:

1. Open Settings.
2. Go to General.
3. Enable **Use the WSL 2 based engine** if available and appropriate for your Windows setup.
4. Apply the changes.
5. Restart Docker Desktop if requested.

For this setup, Docker Desktop must use **Linux containers**, not Windows containers.

### 4.3 Verify Docker Installation

Open a new PowerShell terminal.

Check Docker version:

```powershell
docker version
```

This displays Docker Client and Server information.

Check running containers:

```powershell
docker ps
```

Check all containers:

```powershell
docker ps -a
```

Check Docker information:

```powershell
docker info
```

Check the container operating system:

```powershell
docker info --format '{{.OSType}}'
```

Expected output:

```text
linux
```

**Important:** If Docker Server information is missing or the command reports a connection error, make sure Docker Desktop is running.

Do not continue until Docker is working.

---

## 5. Install Minikube on Windows

There are multiple installation methods available.

We will use Windows Package Manager (winget) as the recommended installation method.

### 5.1 Method 1: Install Using winget (Recommended)

Open PowerShell.

Check whether winget is installed:

```powershell
winget --version
```

Install Minikube:

```powershell
winget install -e --id Kubernetes.minikube
```

Explanation:

| Argument | Purpose |
|---|---|
| `winget` | Windows Package Manager |
| `install` | Installs a package |
| `-e` | Uses an exact package ID match |
| `--id` | Specifies the package identifier |

The package identifier is:

```text
Kubernetes.minikube
```

After installation, close PowerShell and open a new terminal so that updated PATH information is loaded.

### 5.2 Verify Minikube

```powershell
minikube version
```

Expected output format:

```text
minikube version: vX.Y.Z
commit: <commit-hash>
```

The exact version changes as new releases become available.

### 5.3 Method 2: Manual Binary Installation

Use this method if winget is unavailable or you prefer manual installation.

Open PowerShell.

Create an installation directory:

```powershell
New-Item -Path 'C:\' -Name 'minikube' -ItemType Directory -Force
```

Download the Minikube Windows AMD64 executable:

```powershell
$ProgressPreference = 'SilentlyContinue'

Invoke-WebRequest `
  -OutFile 'C:\minikube\minikube.exe' `
  -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe'
```

Add the installation directory to the machine PATH.

**Run PowerShell as Administrator for this step.**

```powershell
$oldPath = [Environment]::GetEnvironmentVariable(
    'Path',
    [EnvironmentVariableTarget]::Machine
)

if ($oldPath.Split(';') -inotcontains 'C:\minikube') {
    [Environment]::SetEnvironmentVariable(
        'Path',
        "$oldPath;C:\minikube",
        [EnvironmentVariableTarget]::Machine
    )
}
```

Close and reopen PowerShell.

Verify:

```powershell
minikube version
```

**Note:** Use either winget or manual installation. Avoid maintaining multiple Minikube installations in different directories because PATH precedence can cause version confusion.

---

## 6. Install kubectl on Windows

We will use winget for installation.

### 6.1 Method 1: Install Using winget (Recommended)

Run:

```powershell
winget install -e --id Kubernetes.kubectl
```

This installs the Kubernetes command-line client.

Close and reopen PowerShell after installation if necessary.

### 6.2 Verify kubectl

```powershell
kubectl version --client
```

Expected output format:

```text
Client Version: vX.Y.Z
Kustomize Version: vX.Y.Z
```

### 6.3 Method 2: Manual Binary Installation

If winget is unavailable, download the Windows AMD64 kubectl executable.

First, retrieve the latest stable Kubernetes version:

```powershell
$K8sVersion = (Invoke-RestMethod `
  -Uri 'https://dl.k8s.io/release/stable.txt').Trim()
```

Create an installation directory:

```powershell
New-Item -ItemType Directory -Force -Path 'C:\kubectl'
```

Download kubectl:

```powershell
Invoke-WebRequest `
  -Uri "https://dl.k8s.io/release/$K8sVersion/bin/windows/amd64/kubectl.exe" `
  -OutFile 'C:\kubectl\kubectl.exe'
```

### 6.4 Verify the Download (SHA256)

Download the checksum:

```powershell
Invoke-WebRequest `
  -Uri "https://dl.k8s.io/release/$K8sVersion/bin/windows/amd64/kubectl.exe.sha256" `
  -OutFile 'C:\kubectl\kubectl.exe.sha256'
```

Compare the downloaded executable's SHA256 hash with the published checksum:

```powershell
$ActualHash = (Get-FileHash `
  -Algorithm SHA256 `
  'C:\kubectl\kubectl.exe').Hash.ToLower()

$ExpectedHash = (Get-Content `
  'C:\kubectl\kubectl.exe.sha256').Trim().ToLower()

$ActualHash -eq $ExpectedHash
```

Expected result:

```text
True
```

If the result is `False`, do not use the downloaded binary. Investigate the download or checksum mismatch.

### 6.5 Add kubectl to PATH

Open PowerShell as Administrator.

```powershell
$oldPath = [Environment]::GetEnvironmentVariable(
    'Path',
    [EnvironmentVariableTarget]::Machine
)

if ($oldPath.Split(';') -inotcontains 'C:\kubectl') {
    [Environment]::SetEnvironmentVariable(
        'Path',
        "$oldPath;C:\kubectl",
        [EnvironmentVariableTarget]::Machine
    )
}
```

Close and reopen PowerShell.

Verify:

```powershell
kubectl version --client
```

**Important:** Docker Desktop may provide its own kubectl executable. If multiple versions are installed, inspect which executable PowerShell is using:

```powershell
Get-Command kubectl -All
```

Check the executable path:

```powershell
where.exe kubectl
```

---

## 7. Verify All Installed Tools

Run:

```powershell
minikube version
kubectl version --client
docker version
```

At this stage, the following should be available:

- Minikube CLI.
- kubectl CLI.
- Docker Client.
- Docker Server.

The Kubernetes cluster may not exist yet.

---

## 8. Create the Kubernetes Cluster

Now we will create our first local Kubernetes cluster.

### 8.1 Start Minikube with Docker Driver

Run:

```powershell
minikube start --driver=docker --cpus=2 --memory=3072
```

Explanation:

| Argument | Meaning |
|---|---|
| `minikube start` | Creates or starts a Minikube cluster |
| `--driver=docker` | Uses Docker as the infrastructure driver |
| `--cpus=2` | Allocates 2 CPUs |
| `--memory=3072` | Allocates 3072 MB memory |

Resource values should be adjusted according to the available system resources.

### 8.2 What Happens Internally?

During startup, Minikube generally performs these operations:

1. Detects and validates the Docker driver.
2. Downloads the required Kubernetes node image.
3. Creates a Docker container for the Minikube node.
4. Configures the Kubernetes node environment.
5. Starts the Kubernetes control plane.
6. Configures kubelet and the container runtime.
7. Configures cluster networking and certificates.
8. Updates Kubernetes access configuration.
9. Configures the Minikube context for kubectl.

The first startup may take several minutes because the required images need to be downloaded.

### 8.3 Verify the Minikube Container

```powershell
docker ps
```

To inspect all containers:

```powershell
docker ps -a
```

To inspect the Minikube container:

```powershell
docker inspect minikube
```

The container name can differ when using a custom Minikube profile.

---

## 9. Verify the Kubernetes Cluster

After Minikube starts successfully, verify the cluster.

### 9.1 Check Minikube Status

```powershell
minikube status
```

This reports the status of:

- Host.
- Kubelet.
- API Server.
- Kubeconfig.

### 9.2 Check Kubernetes Contexts

```powershell
kubectl config get-contexts
```

This lists the configured Kubernetes contexts.

Check the current context:

```powershell
kubectl config current-context
```

The default Minikube context is normally:

```text
minikube
```

### 9.3 Check Cluster Nodes

```powershell
kubectl get nodes
```

Expected output format:

```text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...   vX.Y.Z
```

The node should eventually reach `Ready` status.

### 9.4 Check Cluster Information

```powershell
kubectl cluster-info
```

This displays Kubernetes control-plane endpoint information.

### 9.5 Check All Pods

```powershell
kubectl get pods -A
```

`-A` means `--all-namespaces`.

It displays Pods from all namespaces, including Kubernetes system Pods.

### 9.6 Check Namespaces

```powershell
kubectl get namespaces
```

### 9.7 Check API Server Readiness

```powershell
kubectl get --raw='/readyz?verbose'
```

This queries the API Server readiness endpoint.

A successful response indicates that the API Server considers itself ready.

---

## 10. Understanding kubeconfig on Windows

Kubernetes uses a configuration file called `kubeconfig` to store cluster connection and authentication information.

The default Windows location is:

```text
%USERPROFILE%\.kube\config
```

In PowerShell:

```powershell
$env:USERPROFILE
```

Check the Kubernetes configuration directory:

```powershell
Get-ChildItem "$HOME\.kube"
```

List contexts:

```powershell
kubectl config get-contexts
```

Check the current context:

```powershell
kubectl config current-context
```

List configured clusters:

```powershell
kubectl config get-clusters
```

View the configuration:

```powershell
kubectl config view
```

### What is a Kubernetes Context?

A context defines which cluster, user, and namespace kubectl should use.

For example:

```text
minikube
```

When you execute:

```powershell
kubectl get nodes
```

kubectl uses the currently selected context to determine which Kubernetes cluster should receive the request.

**Security Warning:** Kubeconfig may contain certificates, tokens, and authentication information. Never commit a private kubeconfig file to a public GitHub repository.

---

## 11. Common Minikube and kubectl Commands

### Cluster Management

```powershell
minikube start
minikube stop
minikube status
minikube delete
minikube profile list
minikube ip
```

### Node Management

```powershell
kubectl get nodes
kubectl describe node minikube
```

### Pod Management

```powershell
kubectl get pods
kubectl get pods -A
kubectl get pods -o wide
```

### Namespace Management

```powershell
kubectl get namespaces
kubectl create namespace development
kubectl delete namespace development
```

### Context Management

```powershell
kubectl config get-contexts
kubectl config current-context
kubectl config use-context minikube
```

### Cluster Information

```powershell
kubectl cluster-info
kubectl version
```

---

## 12. Accessing Applications Running in Minikube

After deploying an application, we may need to access it from a Windows browser.

### 12.1 NodePort Services

Run:

```powershell
minikube service <service-name> --url
```

This provides a URL for accessing the service.

Depending on the selected driver and networking configuration, keep the terminal session open while using the URL.

### 12.2 LoadBalancer Services

Run:

```powershell
minikube tunnel
```

Keep the tunnel process running in a separate terminal while accessing the LoadBalancer service.

**Note:** The exact networking behavior depends on the Minikube driver and Windows networking configuration.

---

## 13. Troubleshooting

### Problem 1: Docker daemon is not running

Example:

```text
Cannot connect to the Docker daemon
```

Possible causes:

- Docker Desktop is not running.
- Docker Engine has not started.
- Docker is using Windows containers instead of Linux containers.

Check:

```powershell
docker version
docker info
docker info --format '{{.OSType}}'
```

Expected container OS:

```text
linux
```

Solution:

1. Start Docker Desktop.
2. Verify that Docker Engine is running.
3. Switch to Linux containers if necessary.
4. Run `docker version` again.

### Problem 2: Minikube command is not recognized

Example:

```text
minikube : The term 'minikube' is not recognized
```

Possible cause:

The Minikube installation directory is missing from PATH.

Check:

```powershell
Get-Command minikube -All
where.exe minikube
```

For a manual installation, verify:

```powershell
Test-Path 'C:\minikube\minikube.exe'
```

Ensure the directory is included in PATH, then reopen PowerShell.

### Problem 3: kubectl command is not recognized

Check:

```powershell
Get-Command kubectl -All
where.exe kubectl
```

Verify the installation directory and PATH.

For manual installation:

```powershell
Test-Path 'C:\kubectl\kubectl.exe'
```

### Problem 4: Connection refused on localhost:8080

Example:

```text
The connection to the server localhost:8080 was refused
```

Possible causes:

- The Kubernetes cluster has not been created.
- Minikube is stopped.
- kubectl has no valid current context.
- Kubeconfig is missing or incorrect.

Diagnosis:

```powershell
minikube status
kubectl config get-contexts
kubectl config current-context
```

If no Minikube profile exists:

```powershell
minikube start --driver=docker
```

If the profile exists but is stopped:

```powershell
minikube start
```

### Problem 5: Kubernetes node is NotReady

Check:

```powershell
kubectl get nodes
kubectl describe node minikube
kubectl get pods -A
```

Inspect Pod logs:

```powershell
kubectl logs <pod-name> -n <namespace>
```

Use Pod events and node conditions to investigate scheduling, networking, image-pull, and resource problems.

### Problem 6: Insufficient memory or CPU

Symptoms may include:

- Cluster startup failure.
- Pods stuck in Pending.
- Containers restarting.
- Out-of-memory errors.

Check available memory in Windows:

```powershell
Get-CimInstance Win32_OperatingSystem |
    Select-Object TotalVisibleMemorySize, FreePhysicalMemory
```

Check Docker resource usage:

```powershell
docker stats
```

Reduce Minikube resource allocation or stop unnecessary workloads when the system is under resource pressure.

### Problem 7: kubectl is using the wrong executable

Docker Desktop may install its own kubectl executable.

Check:

```powershell
Get-Command kubectl -All
where.exe kubectl
```

Inspect executable paths and version output.

Make sure the intended kubectl installation is first in PATH.

---

## 14. Stop, Restart and Delete the Cluster

### 14.1 Stop Minikube

```powershell
minikube stop
```

This stops the local cluster but retains its profile.

### 14.2 Start the Cluster Again

```powershell
minikube start
```

### 14.3 Delete the Cluster

```powershell
minikube delete
```

This deletes the Minikube cluster and its associated resources.

**Warning:** Deleting a cluster removes its Kubernetes workloads and cluster-local data. Back up anything important before deleting it.

---

## 15. Final Verification Checklist

After completing the setup, verify each item:

- [ ] Windows PowerShell is working.
- [ ] Docker Desktop is installed.
- [ ] Docker Desktop Engine is running.
- [ ] Docker is using Linux containers.
- [ ] `docker version` displays Client and Server information.
- [ ] Minikube is installed.
- [ ] kubectl is installed.
- [ ] Minikube cluster is created.
- [ ] Minikube status is healthy.
- [ ] Kubernetes context is configured.
- [ ] Kubernetes node reaches Ready status.
- [ ] Kubernetes system Pods are running.
- [ ] Kubernetes API Server is accessible.

---

## 16. Installation Environment Used

Record the actual environment used to test this guide.

| Component | Version / Configuration |
|---|---|
| Host OS | Windows 11 |
| Architecture | AMD64 / x86_64 |
| Terminal | PowerShell |
| Container Platform | Docker Desktop |
| Docker Backend | WSL2 (if enabled) |
| Docker Driver | Docker |
| Minikube | Record tested version |
| kubectl | Record tested version |

**Note:** These are environment details, not fixed software requirements. Update the version fields after testing the guide on the target machine.

---

## 17. Official Documentation and References

| Resource | Official Documentation |
|---|---|
| Minikube | https://minikube.sigs.k8s.io/docs/ |
| Minikube Installation | https://minikube.sigs.k8s.io/docs/start/ |
| Minikube Docker Driver | https://minikube.sigs.k8s.io/docs/drivers/docker/ |
| Kubernetes Documentation | https://kubernetes.io/docs/ |
| kubectl Installation on Windows | https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/ |
| Docker Desktop for Windows | https://docs.docker.com/desktop/setup/install/windows-install/ |
| Docker Desktop WSL2 | https://docs.docker.com/desktop/features/wsl/ |

---
