# Production Infra Recommendations

## 1. Infrastructure Overview

For production environments, we recommend a dedicated infrastructure setup that separates Devtron microservices to ensure optimal performance, reliability, and cost efficiency.

### Node Group Structure

| Node Group                | Purpose                             | Instance Type  | Configuration                  |
| ------------------------- | ----------------------------------- | -------------- | ------------------------------ |
| **Devtron Microservices** | Hosts all Devtron core components   | On-Demand      | 3 nodes (4 CPU, 16GB RAM each) |
| **CI Workers**            | Handles build jobs and CI processes | Spot Instances | Auto-scaling based on workload |

### Autoscaling Configuration

* **Devtron Node Group**: Minimum 1 node, Maximum 5 nodes (Autoscaled)
* **CI Node Group** (Tainted): Configure based on build volume and concurrency requirements

### Blob Storage

Refer Blob Storage Guide to know more.

* **Cache-Storage**: Used to store the build cache to reduce build time of your Application.
* **CI Logs Bucket**: Used to store the build logs of you Application.

***

## 2. Cloud-Specific Setup Guidelines

Depending on your choice of cloud provider, refer the below instructions/scripts to set up a cluster.

### AWS (EKS)

You can follow the [Readme](https://github.com/devtron-labs/utilities/tree/main/eksctl-configs#creating-a-cluster-for-devtron-setup) for setting up the EKS cluster for Devtron.

### GCP (GKE)

Use our provided [Terraform scripts](https://github.com/devtron-labs/utilities/tree/main/terraform/terraform-gke) to set up GKE Cluster for Devtron.

### Azure (AKS)

Use our provided [Terraform scripts](https://github.com/devtron-labs/utilities/tree/main/terraform/terraform-aks) to set up AKS Cluster for Devtron.

{% hint style="success" %}
#### Next Step

Proceed to Devtron Installation on your cluster
{% endhint %}

***

## 3. Microservice Resource Recommendations

### Core Components

| Component      | CPU Requests | CPU Limits | Memory Requests | Memory Limits |
| -------------- | ------------ | ---------- | --------------- | ------------- |
| **Devtron**    | 1            | 1          | 1536Mi          | 1536Mi        |
| **Kubelink**   | 1            | 1          | 1536Mi          | 1536Mi        |
| **Dashboard**  | 100m         | 100m       | 150Mi           | 150Mi         |
| **Lens**       | 200m         | 200m       | 100Mi           | 100Mi         |
| **Git-sensor** | 800m         | 1          | 1510Mi          | 1510Mi        |
| **Kubewatch**  | 200m         | 300m       | 600Mi           | 1000Mi        |

{% hint style="info" %}
#### Need a YAML template to add resources?

You can create a resources file similar to this [YAML file](https://github.com/devtron-labs/devtron/blob/main/charts/devtron/resources-small.yaml) and add resources according to your load and requirements for any service that you want and remove those whose you won't want to modify.

Run the below command once the file is ready:

```bash
helm upgrade devtron devtron/devtron-operator -n devtroncd --reuse-values -f resources-values-file.yaml
```
{% endhint %}

***

## 4. Kubernetes Configuration

### CI Isolation with Taints and Tolerations

To ensure CI workloads run exclusively on the dedicated CI nodes you need to specify the taints and labels to the node,for the CI build pods the you can add the tolerations and node selectors in the `devtron-custom-cm` of `devtroncd` namespace inside these keys. These would automatically get propagated to the CI workloads whenever they get created.

If you are following our Cloud-Specific Setup Guidelines then set the below values for the keys in `devtron-custom-cm`:

```bash
CI_NODE_LABEL_SELECTOR: purpose=ci
CI_NODE_TAINTS_KEY: dedicated
CI_NODE_TAINTS_VALUE: ci
```
