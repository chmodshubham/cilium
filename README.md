# kind-cluster

3-Node kind cluster with Cilium CNI + Hubble enabled

## Cilium Architecture

![image](https://github.com/ShubhamKumar89/kind-cluster/assets/97805339/9b315e92-8a6b-4efe-b624-6bc3559dc430)

## Installation Methods

Cilium supports two methods of installation:

#### 1. Cilium CLI tool
The CLI tool makes it easy to get started with Cilium, especially when you’re first learning about it. It uses the Kubernetes API directly to examine the cluster corresponding to an existing kubectl context and choose appropriate install options for the Kubernetes implementation detected. We’ll be using the Cilium CLI install method for most of the labs in the course.

#### 2. Helm chart
The Helm chart method is meant for advanced installation and production environments where you want granular control of your Cilium installation. It requires you to manually select the best datapath and IPAM mode for your particular Kubernetes environment. You can learn more about the Helm chart installation method in the Cilium documentation resources. We’ll use the Helm chart install method in a later chapter when getting familiar with some advanced capabilities.

## Pre-requisite

### 1. Install `kind`

We will need a Kubernetes cluster appropriately configured and ready for an external CNI to be installed. We were using `kind` cluster for this. [Install kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.19.0/kind-linux-amd64
sudo chmod +x ./kind
sudo mv ./kind /usr/local/bin
```

To verify:

```
which kind
kind version
```

![image](https://github.com/ShubhamKumar89/kind-cluster/assets/97805339/e0e4a400-1e43-4d2e-9e04-42907ba74bd6)

> **Note**:
> kind version >= `v0.7.0`

### 2. Install `kubectl`

`kind` does not require `kubectl`, but we will not be able to perform some of the operations without it. [Install Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

To verify:

```bash
kubectl version --client
```

## Procedure

### 1. Prepare Kubernetes Cluster

Here is the YAML configuration file for a 3-node kind cluster with default CNI disabled. Save this locally to your workstation as `kind-config.yaml` with the contents:

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
networking:
  disableDefaultCNI: true
```

Now create a new kind cluster using this configuration:

```bash
kind create cluster --config=kind-config.yaml
```

Kind will create the cluster and will configure an associated `kubectl` context. Confirm your new kind cluster is the default `kubectl` context:

```bash
kubectl config current-context
```
Now you should be able to use kubectl and the Cilium CLI tool and interact with your newly minted kind cluster.

```bash
kubectl get nodes
```

![image](https://github.com/ShubhamKumar89/kind-cluster/assets/97805339/e1db918a-512a-4855-a6d9-66e5337a5573)

> **Note**: 
> Because you have created the cluster without a default CNI, the Kubernetes nodes are in a NotReady state.

###  2. Install the Cilium CLI Tool

[Download and install](https://docs.cilium.io/en/v1.13/gettingstarted/k8s-install-default/#install-the-cilium-cli) the Cilium CLI 

```bash
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/master/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
```

To verify:

```bash
cilium version
```

![image](https://github.com/ShubhamKumar89/kind-cluster/assets/97805339/b06679e0-2509-4bb7-a5dc-c45488643d99)

We’ll be installing the default Cilium image into the Kubernetes cluster we’ve prepared.

### 3. Install Cilium

```bash
cilium install
```

To verify: 

```bash
cilium status --wait
```

![image](https://github.com/ShubhamKumar89/kind-cluster/assets/97805339/41b8a21b-8c8d-48dd-8cb3-f8c8c6071167)

### 4. Enable Hubble

```bash
cilium hubble enable --ui
```

To verify:

```bash
cilium status
```

![image](https://github.com/ShubhamKumar89/kind-cluster/assets/97805339/6b0b59fb-da81-4d2f-904c-02c172c158bb)

### 5. Validate Cilium Operation

The Cilium CLI tool also provides a command to install a set of connectivity tests in a dedicated Kubernetes namespace. We can run these tests to validate that the Cilium install is fully operational:

```bash
cilium connectivity test --request-timeout 30s --connect-timeout 10s
```

> **Note**:
> The connectivity tests require at least two worker nodes to successfully deploy in a cluster. The connectivity test pods will not be scheduled on nodes operating in the control-plane role. If you did not provision your cluster with two worker nodes, the connectivity test command may stall waiting for the test environment deployments to complete.

### 6. Examine Cluster with kubectl

With Cilium now installed, we can use kubectl to confirm that the nodes are now ready and the required Cilium operational components are present in the cluster:

```bash
kubectl get nodes
kubectl get daemonsets --all-namespaces
kubectl get deployments --all-namespaces
```

![image](https://github.com/ShubhamKumar89/kind-cluster/assets/97805339/2da14535-18fe-4431-836d-da418ef31c64)

The cilium daemonset is running on all 3 nodes in the cluster, and the `cilium-operator` deployment is running on a single node.

Now, `Cilium` is successfully installed.
