# kind-cluster

3-Node kind cluster with Cilium CNI.

## Pre-requisite

#### 1. [Install `kind`](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)

```
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

## Procedure

#### 1. Prepare Kubernetes Cluster

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

```
kind create cluster --config=kind-config.yaml
```

Kind will create the cluster and will configure an associated kubectl context. Confirm your new kind cluster is the default kubectl context:

```
kubectl config current-context
```
Now you should be able to use kubectl and the Cilium CLI tool and interact with your newly minted kind cluster.

```
kubectl get nodes
```

![image](https://github.com/ShubhamKumar89/kind-cluster/assets/97805339/e1db918a-512a-4855-a6d9-66e5337a5573)

> **Note**: 
> Because you have created the cluster without a default CNI, the Kubernetes nodes are in a NotReady state.
