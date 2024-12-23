# AKS-Workshop

![Kubernetes Iceberg](https://blog.palark.com/wp-content/uploads/2022/05/kubernetes-iceberg.png)

## Introduction
This repository contains comprehensive information about various Kubernetes add-ons, tools, and security practices. It serves as a guide for implementing and understanding key components in Azure Kubernetes Service (AKS) environments.

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
  - [Azure CLI](#azure-cli)
  - [kubectl](#kubectl)
  - [kubectx and kubens](#kubectx-and-kubens)
  - [Helm](#helm)
  - [jq](#jq)
  - [kubens](#kubens)
  - [k9s](#k9s)
  - [stern](#stern)
  - [kustomize](#kustomize)
- [Authenticating with AKS Cluster](#authenticating-with-aks-cluster)
  - [Step 1: Login to Azure](#step-1-login-to-azure)
  - [Step 2: Get AKS Credentials](#step-2-get-aks-credentials)
  - [Step 3: Verify Connection](#step-3-verify-connection)
- [Public FQDN](#public-fqdn)
- [Kubernetes Add-ons](#kubernetes-add-ons)
  - [Custom Resource Definitions (CRDs)](#custom-resource-definitions-crds)
  - [Cluster Autoscaler](#cluster-autoscaler)
  - [KEDA](#keda)
  - [Kyverno](#kyverno)
  - [External Secrets - Key Vault](#external-secrets---key-vault)
  - [Polaris](#polaris)
  - [Karpenter](#karpenter)
  - [Cilium](#cilium)
  - [Reloader](#reloader)
  - [Argo](#argo)
  - [Grafana and Prometheus](#grafana-and-prometheus)
- [Kubernetes Security](#kubernetes-security)
  - [Repositories / Tools](#repositories--tools)
  - [Learning](#learning)
  - [Attacking](#attacking)
  - [Defending](#defending)
- [Live Demo Links](#live-demo-links)
  - [Application Links](#application-links)
  - [Monitoring & Management Tools](#monitoring--management-tools)

## Prerequisites

Before starting with the AKS-Workshop, ensure you have the following tools installed on your local machine:

### Azure CLI
Azure CLI is a command-line tool for managing Azure resources.

**Installation:**
```sh
# On macOS
brew install azure-cli

# On Windows
winget install Microsoft.AzureCLI

# On Linux
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

### kubectl
kubectl is the command-line tool for interacting with Kubernetes clusters.

**Installation:**
```sh
# On macOS
brew install kubectl

# On Windows
choco install kubernetes-cli

# On Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### kubectx and kubens
kubectx and kubens are tools for switching between Kubernetes contexts and namespaces.

**Installation:**
```sh
# On macOS
brew install kubectx

# On Windows
choco install kubectx

# On Linux
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
```

### Helm
Helm is a package manager for Kubernetes.

**Installation:**
```sh
# On macOS
brew install helm

# On Windows
choco install kubernetes-helm

# On Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### jq
jq is a lightweight and flexible command-line JSON processor.

**Installation:**
```sh
# On macOS
brew install jq

# On Windows
choco install jq

# On Linux
sudo apt-get install jq
```

### kubens
kubens is a tool for switching between Kubernetes namespaces.

**Installation:**
```sh
# On macOS
brew install kubens

# On Windows
choco install kubens

# On Linux
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
```

### k9s
k9s provides a terminal UI to interact with your Kubernetes clusters.

**Installation:**
```sh
# On macOS
brew install derailed/k9s/k9s

# On Windows
choco install k9s

# On Linux
curl -sS https://webinstall.dev/k9s | bash
```

### stern
stern allows you to tail multiple pods on Kubernetes and aggregate their logs.

**Installation:**
```sh
# On macOS
brew install stern

# On Windows
choco install stern

# On Linux
curl -Lo stern https://github.com/wercker/stern/releases/download/1.11.0/stern_linux_amd64
chmod +x stern
sudo mv stern /usr/local/bin
```

### kustomize
kustomize lets you customize Kubernetes YAML configurations.

**Installation:**
```sh
# On macOS
brew install kustomize

# On Windows
choco install kustomize

# On Linux
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
```

Ensure all these tools are installed and properly configured before proceeding with the workshop.
## Authenticating with AKS Cluster

To interact with your Azure Kubernetes Service (AKS) cluster, you need to authenticate using the Azure CLI. Follow these steps to authenticate and configure access to your AKS cluster:

### Step 1: Login to Azure
First, log in to your Azure account using the Azure CLI:
```sh
az login
```
This command will open a web browser window for you to sign in with your Azure credentials. If you are using a service principal, use the following command instead:
```sh
az login --service-principal --username <appId> --password <password> --tenant <tenant>
```

### Step 2: Get AKS Credentials
Next, get the credentials for your AKS cluster. Replace `<resource-group>` and `<cluster-name>` with your resource group and cluster name:
```sh
az aks get-credentials --resource-group <resource-group> --name <cluster-name>
```
This command configures `kubectl` to use the credentials for your AKS cluster.

### Step 3: Verify Connection
Verify that you can connect to your AKS cluster by running:
```sh
kubectl get nodes
```
This command should return a list of nodes in your AKS cluster, confirming that you are authenticated and connected.

## Public FQDN

The Public Fully Qualified Domain Name (FQDN) is a unique domain name assigned to your AKS cluster. It allows you to access the cluster's API server from the internet. The FQDN is typically in the format:
```
<cluster-name>.<region>.azmk8s.io
```
You can find the FQDN of your AKS cluster using the Azure CLI:
```sh
az aks show --resource-group <resource-group> --name <cluster-name> --query "fqdn" -o tsv
```
This command will output the FQDN of your AKS cluster, which you can use to access the cluster's API server or configure DNS settings for your applications.

Ensure that your network security rules and firewall settings allow access to the AKS API server from your desired locations to use the FQDN effectively.

## Kubernetes Add-ons

### Custom Resource Definitions (CRDs)
Custom Resource Definitions (CRDs) extend the Kubernetes API by defining custom resources. They are essential building blocks for most Kubernetes add-ons, allowing them to introduce new object types into your cluster.

**Why use CRDs?**
- Define custom objects with specific schemas
- Extend Kubernetes functionality without modifying core code
- Enable declarative management of application-specific resources
- Allow operators to automate complex application management

The Cloud Native Computing Foundation (CNCF) serves as the vendor-neutral home for many of these projects, fostering collaboration between the industry's top developers, end users, and vendors. CNCF hosts critical components of the global technology infrastructure, including Kubernetes, Prometheus, and many other cloud-native tools.

[CNCF Website](https://www.cncf.io/)
## Key Objectives of CNCF
The Cloud Native Computing Foundation has several key objectives:
- Foster and sustain open source, vendor-neutral projects
- Create high-quality patterns for cloud native software
- Promote cloud native technologies and practices
- Support collaborative development among contributors
- Make cloud native ubiquitous and accessible to all


**Example CRD:**
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
    name: backends.myapp.example.com
spec:
    group: myapp.example.com
    names:
        kind: Backend
        plural: backends
        singular: backend
    scope: Namespaced
    versions:
        - name: v1
            served: true
            storage: true
            schema:
                openAPIV3Schema:
                    type: object
                    properties:
                        spec:
                            type: object
                            properties:
                                host:
                                    type: string
                                port:
                                    type: integer
```

## What are Kubernetes controllers?

Kubernetes controllers are control loops that track your resource clusters and alter them to match the desired state in a continuous cycle. For example, if you need to roast a chicken at 400 degrees Fahrenheit, the control loop in this case would monitor the oven temperature to keep it as close to your desired temperature as possible. If the temperature goes below or above 400, the control loop automatically adjusts the oven controls to fix it.

This is exactly how Kubernetes controllers work — they monitor the resources and conditions an admin has requested in a desired state and implement them automatically. Controllers ensure that the desired and actual states are in constant alignment.

## What are Kubernetes operators?

Kubernetes operators are a subcategory of controllers that use API extensions — or custom resources — to complete tasks. Operators are typically constructed as a set of independent controllers, each responsible for its own subset of tasks and resources pertaining to the managed application.

While an operator shares similar functions with a controller, it exclusively utilizes custom resources and focuses on one domain.

On the other hand, controllers work without custom resources or API extensions and don’t need to connect to a specific domain. Operators are well-suited to meet operational needs for a specific application or platform, but they do not accommodate generic resource cluster states as well as controllers.
### Cluster Autoscaler
Cluster Autoscaler automatically adjusts the size of the Kubernetes cluster when there are insufficient resources.
[GitHub Repository](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)

**Configuring Cluster Autoscaler with Azure CLI**

You can configure the Cluster Autoscaler using the Azure CLI. This method allows you to set various parameters and flags to customize the scaling behavior.

**Example Command:**
```sh
az aks nodepool update \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --name myNodePool \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 5
```

**Flags Explanation:**
- `--resource-group`: The name of the resource group containing the AKS cluster.
- `--cluster-name`: The name of the AKS cluster.
- `--name`: The name of the node pool to update.
- `--enable-cluster-autoscaler`: Enables the Cluster Autoscaler for the specified node pool.
- `--min-count`: The minimum number of nodes for the node pool.
- `--max-count`: The maximum number of nodes for the node pool.
- `--scale-down-delay-after-add`: Time to wait after adding a node before considering scale down (e.g., `10m` for 10 minutes).
- `--scale-down-unneeded-time`: Time a node should be unneeded before it is eligible for scale down (e.g., `10m` for 10 minutes).
- `--scale-down-utilization-threshold`: Node utilization level below which a node can be considered for scale down (e.g., `0.5` for 50% utilization).

You can adjust these flags based on your specific requirements. This command helps ensure that your node pool scales automatically within the defined limits.

For more detailed information on configuring profiles, refer to the [Azure CLI documentation](https://docs.microsoft.com/en-us/cli/azure/manage-azure-subscriptions-azure-cli).

### KEDA
KEDA (Kubernetes Event-driven Autoscaling) is a Kubernetes-based event-driven autoscaler that can scale applications based on the number of events needing to be processed. It supports various event sources, including message queues, databases, and more.

#### Load Testing Tools
To validate KEDA's autoscaling capabilities, you can use these popular load testing tools:

**hey** - [GitHub Repository](https://github.com/rakyll/hey):
```bash
# Install hey
go install github.com/rakyll/hey@latest

# Basic load test with 200 requests, 20 concurrent
hey -n 200 -c 20 http://your-service

# Load test with custom duration
hey -z 30s http://your-service
```

**siege** - [GitHub Repository](https://github.com/JoeDog/siege):
```bash
# Install siege
apt-get install siege

# Basic load test
siege -c 25 -t 30S http://your-service

# Detailed load test with report
siege -c 50 -t 1M -v http://your-service
```

These tools help demonstrate KEDA's ability to scale based on incoming load.
[GitHub Repository](https://github.com/kedacore/keda)

![KEDA Architecture](https://keda.sh/img/keda-arch.png)

**Installation:**
```sh
helm repo add kedacore https://kedacore.github.io/charts
helm install keda kedacore/keda
```

**Example ScaledObject:**
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
    name: example-scaledobject
    namespace: default
spec:
    scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: example-deployment
    minReplicaCount: 1
    maxReplicaCount: 10
    triggers:
    - type: azure-queue
        metadata:
            queueName: my-queue
            connectionFromEnv: QUEUE_CONNECTION_STRING
```

### Kyverno
Kyverno is a policy engine designed for Kubernetes. It allows you to manage, validate, and mutate Kubernetes resources using policies written in YAML.
[GitHub Repository](https://github.com/kyverno/kyverno)

![Kyverno Architecture](https://lh7-us.googleusercontent.com/docsz/AD_4nXcnEgVtDJ15EG_PzAZ5agKSzfFcxiU8wi1KCaoFkeCk1W0-0zkSRU8nsZyCZiDI0ZAS2im579UT5m9MLPs7BUIpXAmwG585lFPSzyBGYDsY9iSdzj3Wj3swlqbswq9cFHeGZF55fluvpf1Cn4mOubCZDl2Q?key=IQaL61ZX0_O9f9P2CJV79g)

### How Kyverno Works

Kyverno operates as a Kubernetes admission controller, intercepting requests to the Kubernetes API server and applying policies to these requests. It can validate, mutate, and generate configurations based on the policies defined. Kyverno policies are written in YAML and can be used to enforce best practices, ensure compliance, and automate configuration management.

**Key Features:**
- **Validation:** Ensures that resources comply with specified policies before they are created or updated.
- **Mutation:** Modifies resource configurations to match desired state or add default values.
- **Generation:** Automatically creates or updates resources based on other resources.

**Example Use Cases:**
- Enforcing label standards on all resources.
- Ensuring security contexts are set on all pods.
- Automatically adding network policies to namespaces.

Kyverno simplifies Kubernetes policy management by using familiar YAML syntax and integrating seamlessly with Kubernetes native resources.

**Installation:**
```sh
kubectl create -f https://raw.githubusercontent.com/kyverno/kyverno/main/definitions/release/install.yaml
```

**Example Policy:**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: forbid-cpu-limits
  annotations:
    policies.kyverno.io/title: Forbid CPU Limits
    policies.kyverno.io/category: Other
    policies.kyverno.io/subject: Pod
    kyverno.io/kyverno-version: 1.10.0
    kyverno.io/kubernetes-version: "1.26"
    policies.kyverno.io/description: >-
      Setting of CPU limits is a debatable poor practice as it can result, when defined, in potentially starving
      applications of much-needed CPU cycles even when they are available. Ensuring that CPU limits are not
      set may ensure apps run more effectively. This policy forbids any container in a Pod from defining CPU limits.      
spec:
  background: true
  validationFailureAction: Enforce
  rules:
  - name: check-cpu-limits
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: Containers may not define CPU limits.
      pattern:
        spec:
          containers:
          - (name): "*"
            =(resources):
              =(limits):
                X(cpu): null
```

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-latest-tag
  annotations:
    policies.kyverno.io/title: Disallow Latest Tag
    policies.kyverno.io/category: Best Practices
    policies.kyverno.io/minversion: 1.6.0
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      The ':latest' tag is mutable and can lead to unexpected errors if the
      image changes. A best practice is to use an immutable tag that maps to
      a specific version of an application Pod. This policy validates that the image
      specifies a tag and that it is not called `latest`.      
spec:
  validationFailureAction: enforce
  background: true
  rules:
  - name: require-image-tag
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "An image tag is required."
      pattern:
        spec:
          containers:
          - image: "*:*"
  - name: validate-image-tag
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "Using a mutable image tag e.g. 'latest' is not allowed."
      pattern:
        spec:
          containers:
          - image: "!*:latest"
```

### Kyverno and Grafana Integration

Kyverno can be integrated with Grafana to provide visual monitoring and analytics of policy enforcement. This integration allows administrators to track policy violations, audit events, and overall compliance status through customizable dashboards.

![Kyverno Grafana Dashboard](https://kyverno.io/docs/monitoring/bonus-grafana-dashboard/dashboard-example-1.png)

**Key Features:**
- Real-time policy violation monitoring
- Historical compliance trends
- Resource admission statistics
- Policy enforcement metrics
- Automated alerts for violations

To set up monitoring:
1. Configure Prometheus metrics in Kyverno
2. Import Kyverno dashboards in Grafana
3. Set up alerting rules for policy violations

Sample metrics available:
- `kyverno_policy_results_total`
- `kyverno_policy_execution_duration_seconds`
- `kyverno_policy_changes_total`

### External Secrets - Key Vault
External Secrets allows you to integrate external secret management systems like Azure Key Vault with Kubernetes.
[GitHub Repository](https://github.com/external-secrets/kubernetes-external-secrets)

**Installation:**
```sh
helm repo add external-secrets https://external-secrets.github.io/kubernetes-external-secrets/
helm install external-secrets external-secrets/kubernetes-external-secrets
```

**Example Secret:**
```yaml
apiVersion: kubernetes-client.io/v1
kind: ExternalSecret
metadata:
  name: my-secret
spec:
  backendType: azureKeyVault
  data:
    - key: secret-key
      name: secret-name
```

### Polaris
Polaris is a tool for auditing and enforcing best practices in Kubernetes clusters.
[GitHub Repository](https://github.com/FairwindsOps/polaris)

**Installation:**
```sh
kubectl apply -f https://github.com/FairwindsOps/polaris/releases/latest/download/dashboard.yaml
```

**Example Configuration:**
```yaml
polaris:
  checks:
    - name: "hostIPC"
      severity: "warning"
      category: "Security"
```

### Cluster Autoscaler
Cluster Autoscaler automatically adjusts the size of the Kubernetes cluster when there are insufficient resources.
[GitHub Repository](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)

**Installation:**
```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/azure/examples/cluster-autoscaler-autodiscover.yaml
```

**Example Configuration:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-autoscaler-config
  namespace: kube-system
data:
  cluster-autoscaler-config: |
    scale-down-delay-after-add: 10m
    scale-down-unneeded-time: 10m
```

### Karpenter
Karpenter is an open-source node provisioning project built for Kubernetes.

### Features Overview
The AKS Karpenter Provider enables node autoprovisioning using Karpenter on your AKS cluster. Karpenter improves the efficiency and cost of running workloads on Kubernetes clusters by:

- Watching for pods that the Kubernetes scheduler has marked as unschedulable
- Evaluating scheduling constraints (resource requests, node selectors, affinities, tolerations, and topology-spread constraints) requested by the pods
- Provisioning nodes that meet the requirements of the pods
- Removing the nodes when they are no longer needed
- Consolidating existing nodes onto cheaper nodes with higher utilization per node

### Node Auto Provisioning (NAP) vs. Self-hosted
Karpenter provider for AKS can be used in two modes:

- **Node Auto Provisioning (NAP) mode (preview):** Karpenter is run by AKS as a managed addon similar to managed Cluster Autoscaler. This is the recommended mode for most users. Follow the instructions in Node Auto Provisioning documentation to use Karpenter in that mode.
- **Self-hosted mode:** Karpenter is run as a standalone deployment in the cluster. This mode is useful for advanced users who want to customize or experiment with Karpenter's deployment. The rest of this page describes how to use Karpenter in self-hosted mode.

### Known limitations
- Only AKS clusters with Azure CNI Overlay + Cilium networking are supported.
- Only Linux nodes are supported.

[GitHub Repository](https://github.com/Azure/karpenter-provider-azure)


**Installation:**
```sh
helm repo add karpenter https://charts.karpenter.sh
helm install karpenter karpenter/karpenter
```

**Example Provisioner:**
```sh
cat <<EOF | kubectl apply -f -
---
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: general-purpose
  annotations:
    kubernetes.io/description: "General purpose NodePool for generic workloads"
spec:
  template:
    spec:
      requirements:
        - key: kubernetes.io/arch
          operator: In
          values: ["amd64"]
        - key: kubernetes.io/os
          operator: In
          values: ["linux"]
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["on-demand"]
        - key: karpenter.azure.com/sku-family
          operator: NotIn
          values: ["D", "E", "F"]
      nodeClassRef:
        name: default
  limits:
    cpu: 100
  disruption:
    consolidationPolicy: WhenUnderutilized
    expireAfter: Never
---
apiVersion: karpenter.azure.com/v1alpha2
kind: AKSNodeClass
metadata:
  name: default
  annotations:
    kubernetes.io/description: "General purpose AKSNodeClass for running Ubuntu2204 nodes"
spec:
  imageFamily: Ubuntu2204
EOF
```

### Cilium
Cilium is an open-source software for providing and transparently securing network connectivity and load balancing between application workloads.
[GitHub Repository](https://github.com/cilium/cilium)

![Cilium Architecture](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*K0swBsKe9-Rlivex.png)

**Installation:**
```sh
helm repo add cilium https://helm.cilium.io/
helm install cilium cilium/cilium --version 1.10.4
```

**Example Network Policy:**
```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: allow-all
spec:
  endpointSelector:
    matchLabels: {}
  ingress:
  - fromEndpoints:
    - matchLabels: {}
```

### Reloader
Reloader is a Kubernetes controller to watch changes in ConfigMap and Secrets and do rolling upgrades on Pods with their associated Deployments.
[GitHub Repository](https://github.com/stakater/Reloader)

**Installation:**
```sh
helm repo add stakater https://stakater.github.io/stakater-charts
helm install reloader stakater/reloader
```

**Example Annotation:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  annotations:
    configmap.reloader.stakater.com/reload: "my-configmap"
spec:
  replicas: 1
  template:
    metadata:
      annotations:
        configmap.reloader.stakater.com/reload: "my-configmap"
```

### Argo
Argo is a set of Kubernetes-native tools for running and managing jobs and applications on Kubernetes.
[GitHub Repository](https://github.com/argoproj/argo-workflows)

![ArgoCD Architecture](https://argo-cd.readthedocs.io/en/stable/assets/argocd_architecture.png)

GitOps is a declarative approach to continuous delivery that uses Git as the single source of truth for infrastructure and applications. It extends DevOps best practices by using Git pull requests to automatically manage infrastructure provisioning and deployment.

**Key Benefits:**
- Version Control: All changes are tracked and can be rolled back
- Declarative Configuration: Infrastructure defined as code
- Improved Security: Git's authentication and access control
- Automated Synchronization: Continuous reconciliation between Git and cluster state
- Enhanced Collaboration: Standard Git workflows for infrastructure changes

![GitOps with ArgoCD](https://learn.microsoft.com/en-us/azure/architecture/example-scenario/gitops-aks/media/gitops-argo-cd.png)

**Components:**
- Git Repository: Stores desired state of the system
- GitOps Operator: Monitors repository and reconciles cluster state
- Kubernetes Cluster: Target environment for deployments
- CI/CD Pipeline: Automates testing and deployment processes

GitOps improves reliability, security, and velocity of infrastructure and application deployments by leveraging familiar Git workflows and automation.

**Installation:**
```sh
kubectl create namespace argo
kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo-workflows/stable/manifests/install.yaml
```

**Example Workflow:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: hello-world-
spec:
  entrypoint: whalesay
  templates:
  - name: whalesay
    container:
      image: docker/whalesay:latest
      command: [cowsay]
      args: ["hello world"]
```

### Grafana and Prometheus
Grafana and Prometheus are popular open-source tools used for monitoring and observability. Prometheus is a powerful metrics collection and alerting system, while Grafana provides a rich visualization and dashboarding capability.
[Grafana GitHub Repository](https://github.com/grafana/grafana)
[Prometheus GitHub Repository](https://github.com/prometheus/prometheus)

![Grafana and Prometheus Architecture](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*IYysgfotmdrgPx15.png)

**Installation:**
```sh
# Prometheus
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/bundle.yaml

# Grafana
kubectl apply -f https://raw.githubusercontent.com/grafana/helm-charts/main/charts/grafana/templates/deployment.yaml
```

**Example Prometheus Rule:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
    name: example-rule
    namespace: monitoring
spec:
    groups:
    - name: example
        rules:
        - alert: HighRequestLatency
            expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5
            for: 10m
            labels:
                severity: "critical"
            annotations:
                summary: "High request latency"
```

## Kubernetes Security

### Repositories / Tools
[Awesome Kubernetes Security](https://github.com/magnologan/awesome-k8s-security)

#### Learning
- kubectl
- krew
- Bust-a-Kube
- kube-goat
- Kubernetes Goat
- Kubernetes Networking Labs for KubeCon EU 2020 Talk
- CNCF Security Audits
- Kube Security Lab: Learn from Kubernetes attacks using Ansible and KinD

#### Attacking
- [kdigger](https://github.com/quarkslab/kdigger) - A context discovery tool for Kubernetes penetration testing.
- [kube-hunter](https://github.com/aquasecurity/kube-hunter) - Hunt for security weaknesses in Kubernetes clusters.
- [kubeletctl](https://github.com/ricardojba/kubeletctl) - A tool to exploit kubelet API.
- [kubesploit](https://github.com/cyberark/kubesploit) - A tool for testing Kubernetes security.
- [Peirates](https://github.com/inguardians/peirates) - Kubernetes penetration testing tool.

#### Defending
- [KubeArmor](https://github.com/kubearmor/KubeArmor) - Cloud-native runtime protection.
- [Kubescape](https://github.com/armosec/kubescape) - Kubernetes security according to NSA-CISA and MITRE ATT&CK® frameworks.
- [KubiScan](https://github.com/cyberark/KubiScan) - Scan Kubernetes for risky permissions.
- [Kubernetes Audit by Trail of Bits](https://github.com/trailofbits/kubernetes-audit) - Security audit for Kubernetes clusters.
- [kubeaudit](https://github.com/Shopify/kubeaudit) - Audit Kubernetes clusters for security issues.
- [Deepfence ThreatMapper](https://github.com/deepfence/ThreatMapper) - Runtime vulnerability scanner for Kubernetes.
- [falco](https://github.com/falcosecurity/falco) - Runtime security monitoring for Kubernetes.
- [kubesec](https://github.com/controlplaneio/kubesec) - Security risk analysis for Kubernetes resources.
- [kube-bench](https://github.com/aquasecurity/kube-bench) - Checks Kubernetes clusters against CIS benchmarks.
- [trivy](https://github.com/aquasecurity/trivy) - Comprehensive security scanner for vulnerabilities.
- [MKIT](https://github.com/darkbitio/mkit) - Kubernetes security assessment toolkit.
- [kubetap](https://github.com/soluble-ai/kubetap) - Intercept and capture Kubernetes traffic.
- [kube-forensics](https://github.com/liggitt/kube-forensics) - Forensic tools for Kubernetes clusters.
- [k8s-security-dashboard](https://github.com/k8s-security-dashboard) - Dashboard for Kubernetes security.
- [CIS Kubernetes Benchmark - InSpec Profile](https://github.com/dev-sec/cis-kubernetes-benchmark) - InSpec profile for CIS Kubernetes Benchmark.
- [Kube PodSecurityPolicy Advisor](https://github.com/sysdiglabs/kube-psp-advisor) - Advisor for Kubernetes PodSecurityPolicy.
- [Inspektor Gadget](https://github.com/kinvolk/inspektor-gadget) - Collection of tools for debugging and inspecting Kubernetes applications.
- [Starboard](https://github.com/aquasecurity/starboard) - Kubernetes-native security toolkit.
- [Advocacy Site for Kubernetes RBAC](https://github.com/alcideio/rbac-tool) - Tools for managing Kubernetes RBAC.
- [Helm-Snyk](https://github.com/snyk-labs/helm-snyk) - Helm plugin for Snyk security scanning.
- [Krane](https://github.com/appvia/krane) - Kubernetes RBAC static analysis tool.
- [rakkess](https://github.com/corneliusweig/rakkess) - Show Kubernetes access rules for subjects.
- [kubectl-who-can](https://github.com/aquasecurity/kubectl-who-can) - Show who has permissions to perform actions on Kubernetes resources.
- [Kubernetes Security - Best Practice Guide](https://github.com/freach/kubernetes-security-best-practice) - Best practices for securing Kubernetes.
- [External Secrets](https://github.com/external-secrets/kubernetes-external-secrets) - Integrate external secret management systems with Kubernetes.
- [kubescape](https://github.com/armosec/kubescape) - Kubernetes security scanner.
- [KubeLinter](https://github.com/stackrox/kube-linter) - Linter for Kubernetes YAML files.
- [Open Policy Agent](https://github.com/open-policy-agent/opa) - Policy engine for Cloud Native environments.
- [Gatekeeper](https://github.com/open-policy-agent/gatekeeper) - Policy controller for Kubernetes.
- [Kyverno](https://github.com/kyverno/kyverno) - Kubernetes native policy management.
- [Kubewarden](https://github.com/kubewarden) - Kubernetes policy engine using WebAssembly.
- [KICS](https://github.com/Checkmarx/kics) - Keeping Infrastructure as Code Secure.
- [cnspec](https://github.com/mondoohq/cnspec) - Cloud-native security and policy project.
- [M9sweeper](https://github.com/m9sweeper/m9sweeper) - Kubernetes security platform.

## Live Demo Links

### Application Links
- **Store Demo**: [http://store-demo.lacoyart.online/](http://store-demo.lacoyart.online/)
    - Frontend
    - Order Service: [/order-service](http://store-demo.lacoyart.online/order-service)
    - Product Service: [/product-service](http://store-demo.lacoyart.online/product-service)

- **CRUD Car Demo**: [http://crud-car.lacoyart.online/](http://crud-car.lacoyart.online/)
    - Cars UI: [/cars](http://crud-car.lacoyart.online/cars)
    - Swagger API Documentation: [/swagger](http://crud-car.lacoyart.online/swagger)

### Monitoring & Management Tools
- **ArgoCD**: [http://argocd.lacoyart.online/](http://argocd.lacoyart.online/)
- **Goldilocks**: [http://goldilocks.lacoyart.online/](http://goldilocks.lacoyart.online/)
- **Hubble UI**: [http://hubble.lacoyart.online/](http://hubble.lacoyart.online/)
- **Grafana**: [http://grafana.lacoyart.online/](http://grafana.lacoyart.online/)
- **Polaris**: [http://polaris.lacoyart.online/](http://polaris.lacoyart.online/)