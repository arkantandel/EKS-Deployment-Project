<div align="center">

# ☸️ AWS EKS — Amazon Elastic Kubernetes Service

### A Complete Technical Guide: From Zero to Production-Ready Cluster

[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.28+-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io)
[![AWS](https://img.shields.io/badge/Amazon_AWS-Cloud-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com)
[![Docker](https://img.shields.io/badge/Docker-Containerization-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://docker.com)
[![IAM](https://img.shields.io/badge/AWS_IAM-Identity-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/iam)
[![VPC](https://img.shields.io/badge/Amazon_VPC-Networking-8C4FFF?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/vpc)
[![ECR](https://img.shields.io/badge/Amazon_ECR-Registry-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/ecr)

---

> **EKS** is a fully managed Kubernetes service by AWS that eliminates the operational overhead of running your own control plane.  
> AWS handles upgrades, patching, high availability, and scaling — you focus on your applications.

</div>

---

## 📋 Table of Contents

- [🏗️ EKS Architecture Overview](#-eks-architecture-overview)
- [⚖️ EKS vs. Self-Managed Kubernetes](#️-eks-vs-self-managed-kubernetes)
- [🔧 Chapter 1 — AWS Environment Setup](#-chapter-1--aws-environment-setup)
  - [1.1 Creating an AWS Account & IAM Users](#11-creating-an-aws-account--iam-users)
  - [1.2 Configuring the AWS CLI & kubectl](#12-configuring-the-aws-cli--kubectl)
  - [1.3 Networking & Security Groups](#13-networking--security-groups)
- [🚀 Chapter 2 — Launching Your EKS Cluster](#-chapter-2--launching-your-eks-cluster)
  - [2.1 Create via the AWS Console](#21-create-via-the-aws-console)
  - [2.2 Launch via AWS CLI](#22-launch-via-aws-cli)
  - [2.3 Authenticate with the Cluster](#23-authenticate-with-the-cluster)
- [📦 Chapter 3 — Deploying Applications](#-chapter-3--deploying-applications)
  - [3.1 Containerizing with Docker](#31-containerizing-with-docker)
  - [3.2 Kubernetes Deployment YAMLs](#32-kubernetes-deployment-yamls)
  - [3.3 Step-by-Step Deployment Guide](#33-step-by-step-deployment-guide)
- [📊 Flow & Network Diagrams](#-flow--network-diagrams)
- [🔑 Quick Reference Cheatsheet](#-quick-reference-cheatsheet)

---

## 🏗️ EKS Architecture Overview

```mermaid
graph TB
    subgraph INTERNET["🌐 Internet / External Traffic"]
        USER["👤 Developer<br/>kubectl CLI"]
        CICD["🔄 CI/CD Pipeline<br/>GitHub Actions / Jenkins"]
        WEB["🌐 Web Traffic"]
    end

    subgraph EDGE["⚡ AWS Edge Layer"]
        R53["🔵 Route 53<br/>DNS Routing"]
        ALB["⚖️ Application<br/>Load Balancer"]
        WAF["🛡️ AWS WAF<br/>+ Shield"]
    end

    subgraph CTRL["☸️ EKS Control Plane — Managed by AWS"]
        API["🖥️ kube-apiserver<br/>REST API Gateway"]
        ETCD["📦 etcd<br/>Cluster State Store"]
        SCHED["📅 kube-scheduler<br/>Pod Placement Engine"]
        CM["🎮 Controller Manager<br/>Reconciliation Loop"]
    end

    subgraph DATA["🖧 Data Plane — Your Worker Nodes"]
        subgraph AZ1["📍 AZ-1 (us-east-1a)"]
            N1["🖧 Worker Node 1<br/>kubelet + kube-proxy"]
            P1["📦 Pod A"]
            P2["📦 Pod B"]
        end
        subgraph AZ2["📍 AZ-2 (us-east-1b)"]
            N2["🖧 Worker Node 2<br/>kubelet + kube-proxy"]
            P3["📦 Pod C"]
            P4["📦 Pod D"]
        end
        subgraph AZ3["📍 AZ-3 (us-east-1c)"]
            N3["🖧 Worker Node 3<br/>kubelet + kube-proxy"]
            P5["📦 Pod E"]
            P6["📦 Pod F"]
        end
    end

    subgraph AWS_SVC["🔧 Supporting AWS Services"]
        IAM["🔑 IAM<br/>Auth & Authorization"]
        VPC["🌐 VPC<br/>Networking"]
        ECR["📦 ECR<br/>Container Registry"]
        CW["📊 CloudWatch<br/>Monitoring & Logs"]
        EBS["💾 EBS<br/>Persistent Storage"]
        SG["🔒 Security Groups<br/>Firewall Rules"]
    end

    USER -->|kubectl commands| API
    CICD -->|deploy via AWS CLI| API
    WEB --> WAF --> ALB --> R53

    ALB --> P1
    ALB --> P3
    ALB --> P5

    API <--> ETCD
    API <--> SCHED
    API <--> CM

    SCHED -->|schedules pods| N1
    SCHED -->|schedules pods| N2
    SCHED -->|schedules pods| N3

    N1 --- P1 & P2
    N2 --- P3 & P4
    N3 --- P5 & P6

    CTRL <-->|Auth| IAM
    DATA <-->|Network| VPC
    N1 & N2 & N3 -->|pull images| ECR
    CTRL & DATA -->|metrics & logs| CW
    P1 & P3 & P5 -->|storage| EBS
    DATA <-->|traffic rules| SG

    style CTRL fill:#0D2137,stroke:#00C6FF,stroke-width:2px,color:#fff
    style DATA fill:#0A1E2E,stroke:#10B981,stroke-width:2px,color:#fff
    style AWS_SVC fill:#1A1A0D,stroke:#F59E0B,stroke-width:2px,color:#fff
    style INTERNET fill:#0D0D1A,stroke:#8B5CF6,stroke-width:1px,color:#fff
    style EDGE fill:#1A0D0D,stroke:#EF4444,stroke-width:1px,color:#fff
```

---

## ⚖️ EKS vs. Self-Managed Kubernetes

```mermaid
quadrantChart
    title EKS vs Self-Managed — Effort vs Control
    x-axis Low Operational Effort --> High Operational Effort
    y-axis Low Control --> High Control
    quadrant-1 Expert Territory
    quadrant-2 Ideal Sweet Spot
    quadrant-3 Avoid
    quadrant-4 Managed Services
    Amazon EKS: [0.25, 0.45]
    EKS Fargate: [0.10, 0.20]
    Self-Managed K8s: [0.85, 0.90]
    kOps on EC2: [0.75, 0.75]
    ECS Fargate: [0.08, 0.10]
    GKE Autopilot: [0.15, 0.35]
```

### Detailed Feature Comparison

```mermaid
flowchart LR
    subgraph EKS["☸️ Amazon EKS"]
        direction TB
        E1["✅ Managed Control Plane"]
        E2["✅ Auto Kubernetes Updates"]
        E3["✅ Multi-AZ HA by Default"]
        E4["✅ IAM Native Auth"]
        E5["✅ CloudWatch Integration"]
        E6["✅ Compliance Certified"]
        E7["❌ Higher Cost"]
        E8["❌ Less Infra Control"]
    end

    subgraph SELF["⚙️ Self-Managed"]
        direction TB
        S1["✅ Spot Instance Savings"]
        S2["✅ Full K8s Config Control"]
        S3["✅ Experimental Features"]
        S4["✅ AWS Services Compatible"]
        S5["❌ Manual Updates & Patches"]
        S6["❌ DIY High Availability"]
        S7["❌ Complex Setup"]
        S8["❌ Security Config Overhead"]
    end

    CHOOSE{"Which to Choose?"}
    TEAM["Team Expertise?"] --> CHOOSE
    BUDGET["Budget?"] --> CHOOSE
    SCALE["Scale?"] --> CHOOSE

    CHOOSE -->|"Fast Delivery<br/>Reliability Priority"| EKS
    CHOOSE -->|"Max Control<br/>Cost Optimization"| SELF

    style EKS fill:#0D2137,stroke:#00C6FF,stroke-width:2px
    style SELF fill:#1A1A0D,stroke:#94A3B8,stroke-width:2px
    style CHOOSE fill:#1A0D1A,stroke:#8B5CF6
```

---

## 🔧 Chapter 1 — AWS Environment Setup

### 1.1 Creating an AWS Account & IAM Users

```mermaid
sequenceDiagram
    actor DEV as 👤 Developer
    participant AWS as 🌐 AWS Website
    participant ROOT as 🔑 Root Account
    participant IAM as 🛡️ IAM Service
    participant MFA as 📱 MFA Device
    participant USER as 👤 IAM User

    DEV->>AWS: Register at aws.amazon.com
    AWS-->>DEV: Verification email sent
    DEV->>AWS: Confirm email & add payment
    AWS-->>ROOT: Root account created ✅

    DEV->>ROOT: Log in as root
    ROOT->>IAM: Navigate to IAM service
    IAM->>MFA: Enable MFA on root
    MFA-->>ROOT: Root account secured 🔒

    ROOT->>IAM: Create IAM Admin User
    IAM-->>USER: User created with credentials
    USER->>IAM: Attach AdministratorAccess policy
    IAM-->>USER: Permissions granted ✅

    DEV->>USER: Log out root, log in as IAM User
    USER-->>DEV: Access Keys (ID + Secret) generated
    Note over DEV,USER: Root credentials locked away forever 🔐
```

**Required IAM Policies:**

| Role | Policy Name | Purpose |
|------|------------|---------|
| EKS Cluster Role | `AmazonEKSClusterPolicy` | Control plane operations |
| EKS Cluster Role | `AmazonEKSServicePolicy` | EKS service interactions |
| Worker Node Role | `AmazonEKSWorkerNodePolicy` | Node registration to cluster |
| Worker Node Role | `AmazonEKS_CNI_Policy` | Pod networking (VPC CNI) |
| Worker Node Role | `AmazonEC2ContainerRegistryReadOnly` | Pull images from ECR |

---

### 1.2 Configuring the AWS CLI & kubectl

```mermaid
flowchart TD
    START([🚀 Start Setup]) --> INSTALL_CLI

    subgraph CLI["📥 AWS CLI Installation"]
        INSTALL_CLI["Install AWS CLI v2\ncurl + installer"]
        CONFIG_CLI["aws configure\nEnter: Key ID, Secret, Region, Format"]
        VERIFY_CLI["aws sts get-caller-identity\nVerify identity ✅"]
        INSTALL_CLI --> CONFIG_CLI --> VERIFY_CLI
    end

    subgraph KUBECTL["📥 kubectl Installation"]
        INSTALL_K8S["Download kubectl binary\ndl.k8s.io/release/stable"]
        CHMOD["chmod +x kubectl\nmv to /usr/local/bin"]
        VERIFY_K8S["kubectl version\nVerify install ✅"]
        INSTALL_K8S --> CHMOD --> VERIFY_K8S
    end

    subgraph CONNECT["🔗 Connect to EKS"]
        UPDATE_CONFIG["aws eks update-kubeconfig\n--name cluster-name --region"]
        TEST["kubectl get nodes\nVerify connection ✅"]
        UPDATE_CONFIG --> TEST
    end

    VERIFY_CLI --> INSTALL_K8S
    VERIFY_K8S --> UPDATE_CONFIG
    TEST --> DONE([✅ Tools Configured])

    style CLI fill:#0D1F35,stroke:#00C6FF
    style KUBECTL fill:#0D1F35,stroke:#10B981
    style CONNECT fill:#0D1F35,stroke:#F59E0B
```

```bash
# Step 1: Configure AWS CLI
aws configure
# AWS Access Key ID:       AKIA...
# AWS Secret Access Key:   ****
# Default region name:     us-east-1
# Default output format:   json

# Step 2: Verify identity
aws sts get-caller-identity

# Step 3: Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/

# Step 4: Connect to cluster after creation
aws eks update-kubeconfig --name my-cluster --region us-east-1
kubectl get nodes
```

---

### 1.3 Networking & Security Groups

```mermaid
graph TB
    subgraph VPC["🌐 Amazon VPC — 10.0.0.0/16"]
        subgraph PUBLIC["🌍 Public Subnets — Internet-facing"]
            PUB_1A["Subnet 10.0.1.0/24\nus-east-1a\n⚖️ ALB / NAT Gateway"]
            PUB_1B["Subnet 10.0.2.0/24\nus-east-1b\n⚖️ ALB Replica"]
        end

        subgraph PRIVATE["🔒 Private Subnets — Worker Nodes"]
            PRIV_1A["Subnet 10.0.10.0/24\nus-east-1a\n🖧 Worker Nodes"]
            PRIV_1B["Subnet 10.0.20.0/24\nus-east-1b\n🖧 Worker Nodes"]
            PRIV_1C["Subnet 10.0.30.0/24\nus-east-1c\n🖧 Worker Nodes"]
        end

        IGW["🌐 Internet Gateway"]
        NAT["🔄 NAT Gateway\n(Outbound Only)"]
        SG_CP["🔒 Security Group:\nControl Plane\nPort 443 Inbound"]
        SG_WN["🔒 Security Group:\nWorker Nodes\n10250, 443 Inbound"]
    end

    INTERNET["🌐 Internet"] --> IGW
    IGW --> PUB_1A & PUB_1B
    PUB_1A --> NAT
    NAT --> PRIV_1A & PRIV_1B & PRIV_1C
    SG_CP --> PRIVATE
    SG_WN --> PRIVATE

    style VPC fill:#0A1628,stroke:#2E75B6,stroke-width:2px
    style PUBLIC fill:#0D2137,stroke:#F59E0B,stroke-width:1px
    style PRIVATE fill:#0A1E2E,stroke:#10B981,stroke-width:1px
```

**Security Group Rules:**

| Direction | Port | Protocol | Source | Purpose |
|-----------|------|----------|--------|---------|
| Inbound | 443 | HTTPS | Control Plane SG | API server comms |
| Inbound | 10250 | TCP | Control Plane SG | kubelet API |
| Inbound | 22 | SSH | Bastion Host IP | Admin access |
| Outbound | All | All | 0.0.0.0/0 | Pull images, call AWS APIs |

> ⚠️ **Tag your subnets!** EKS requires specific tags for load balancer auto-discovery:
> - Public subnets: `kubernetes.io/role/elb = 1`
> - Private subnets: `kubernetes.io/role/internal-elb = 1`

---

## 🚀 Chapter 2 — Launching Your EKS Cluster

### 2.1 Cluster Creation Flow

```mermaid
stateDiagram-v2
    [*] --> CREATING : aws eks create-cluster

    CREATING : 🔄 CREATING
    CREATING : AWS provisions control plane
    CREATING : API server, etcd, scheduler
    CREATING : Multi-AZ deployment in progress

    CREATING --> ACTIVE : ~10-15 minutes

    ACTIVE : ✅ ACTIVE
    ACTIVE : Control plane is ready
    ACTIVE : Ready for node groups

    ACTIVE --> ADDING_NODES : create-nodegroup

    ADDING_NODES : 🔄 ADDING NODES
    ADDING_NODES : EC2 instances launching
    ADDING_NODES : kubelet bootstrapping
    ADDING_NODES : Nodes registering to cluster

    ADDING_NODES --> READY : All nodes Ready

    READY : 🚀 CLUSTER READY
    READY : kubectl get nodes → Ready
    READY : Deploy workloads!

    READY --> [*]
```

### 2.2 Launch via AWS CLI

```bash
# ── Create the EKS Cluster ──────────────────────────────────────
aws eks create-cluster \
  --name production-cluster \
  --kubernetes-version 1.28 \
  --role-arn arn:aws:iam::ACCOUNT_ID:role/EKSClusterRole \
  --resources-vpc-config \
    subnetIds=subnet-abc123,subnet-def456,subnet-ghi789,\
    securityGroupIds=sg-controlplane,\
    endpointPrivateAccess=true,\
    endpointPublicAccess=true \
  --logging '{"clusterLogging":[{"types":["api","audit","authenticator","controllerManager","scheduler"],"enabled":true}]}'

# ── Wait for cluster to be Active ──────────────────────────────
aws eks wait cluster-active --name production-cluster

# ── Create Worker Node Group ───────────────────────────────────
aws eks create-nodegroup \
  --cluster-name production-cluster \
  --nodegroup-name main-workers \
  --node-role arn:aws:iam::ACCOUNT_ID:role/EKSNodeRole \
  --subnets subnet-priv-1a subnet-priv-1b subnet-priv-1c \
  --instance-types t3.medium \
  --scaling-config minSize=2,maxSize=10,desiredSize=3 \
  --disk-size 20 \
  --ami-type AL2_x86_64
```

### 2.3 Authenticate with the Cluster

```bash
# Connect kubectl to EKS
aws eks update-kubeconfig \
  --region us-east-1 \
  --name production-cluster

# Verify nodes are Ready
kubectl get nodes -o wide

# Check cluster components
kubectl get pods -n kube-system

# View cluster details
kubectl cluster-info
```

---

## 📦 Chapter 3 — Deploying Applications

### 3.1 Containerizing with Docker

```mermaid
flowchart LR
    CODE["💻 Application\nSource Code"] -->|Dockerfile| BUILD

    subgraph BUILD["🐳 Docker Build Process"]
        direction TB
        STAGE1["Stage 1: Builder\nFROM node:18-alpine\nInstall dependencies\nnpm ci --only=production"]
        STAGE2["Stage 2: Runtime\nFROM node:18-alpine\nCopy from builder\nMinimal final image"]
        STAGE1 --> STAGE2
    end

    BUILD -->|docker push| ECR

    subgraph ECR["📦 Amazon ECR"]
        direction TB
        IMG1["my-app:latest"]
        IMG2["my-app:v1.0.0"]
        IMG3["my-app:v1.0.1"]
    end

    ECR -->|image pull| K8S

    subgraph K8S["☸️ EKS Cluster"]
        direction TB
        POD1["📦 Pod 1\ncontainer running"]
        POD2["📦 Pod 2\ncontainer running"]
        POD3["📦 Pod 3\ncontainer running"]
    end

    style BUILD fill:#0D2137,stroke:#2496ED
    style ECR fill:#1A1A0D,stroke:#FF9900
    style K8S fill:#0A1E2E,stroke:#326CE5
```

**Production Dockerfile:**

```dockerfile
# ── Stage 1: Build ──────────────────────────────────────
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# ── Stage 2: Runtime (minimal image) ─────────────────────
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

```bash
# Authenticate Docker to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS \
  --password-stdin ACCOUNT.dkr.ecr.us-east-1.amazonaws.com

# Build, tag and push
docker build -t my-app:latest .
docker tag my-app:latest ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
docker push ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
```

---

### 3.2 Kubernetes Deployment YAMLs

```mermaid
graph LR
    subgraph K8S_OBJECTS["☸️ Kubernetes Objects"]
        DEP["📋 Deployment\nManages ReplicaSet\nRolling Updates\nSelf-healing"]
        RS["🔁 ReplicaSet\nEnsures 3 Pods\nAlways Running"]
        SVC["🌐 Service\nLoad Balancing\nDNS Resolution\nPort Mapping"]
        HPA["📈 HPA\nAuto-scales pods\nbased on CPU/Memory"]
        POD1["📦 Pod 1"] 
        POD2["📦 Pod 2"] 
        POD3["📦 Pod 3"]
    end

    subgraph AWS_INFRA["☁️ AWS Infrastructure"]
        ALB["⚖️ AWS ALB\nExternal Traffic"]
        ECR_IMG["📦 ECR Image"]
        CW_LOG["📊 CloudWatch\nLogs & Metrics"]
    end

    DEP --> RS
    DEP --> HPA
    RS --> POD1 & POD2 & POD3
    SVC --> POD1 & POD2 & POD3
    ALB --> SVC
    ECR_IMG --> POD1 & POD2 & POD3
    POD1 & POD2 & POD3 --> CW_LOG

    style K8S_OBJECTS fill:#0A1628,stroke:#326CE5,stroke-width:2px
    style AWS_INFRA fill:#1A1A0D,stroke:#FF9900,stroke-width:2px
```

**deployment.yaml:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-application
  labels:
    app: my-application
    version: v1.0.0
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-application
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0        # Zero-downtime deployments
  template:
    metadata:
      labels:
        app: my-application
    spec:
      containers:
      - name: my-application
        image: ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
        ports:
        - containerPort: 3000
        resources:             # ALWAYS define these!
          requests:
            cpu: "250m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
        readinessProbe:        # Pod only receives traffic when ready
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:         # Restart unhealthy pods automatically
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  type: LoadBalancer           # Provisions AWS ALB automatically
  selector:
    app: my-application
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-application
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

### 3.3 Step-by-Step Deployment Guide

```mermaid
sequenceDiagram
    actor DEV as 👤 Developer
    participant GIT as 📁 Git Repository
    participant CICD as 🔄 CI/CD Pipeline
    participant ECR as 📦 Amazon ECR
    participant EKS as ☸️ EKS Cluster
    participant POD as 📦 Application Pods
    participant ALB as ⚖️ Load Balancer

    DEV->>GIT: git push origin main
    GIT-->>CICD: Webhook trigger

    CICD->>CICD: docker build -t my-app .
    CICD->>ECR: docker push (tagged image)
    ECR-->>CICD: Image stored ✅

    CICD->>EKS: kubectl apply -f deployment.yaml
    EKS-->>CICD: Deployment created

    EKS->>POD: Schedule Pod 1 on Node 1
    EKS->>POD: Schedule Pod 2 on Node 2
    EKS->>POD: Schedule Pod 3 on Node 3

    POD->>ECR: Pull container image
    ECR-->>POD: Image downloaded ✅

    POD->>POD: Container starts
    POD->>POD: Readiness probe passes ✅

    EKS->>ALB: Configure target group
    ALB-->>DEV: kubectl get service → EXTERNAL-IP

    DEV->>ALB: curl http://EXTERNAL-IP
    ALB->>POD: Route request (round-robin)
    POD-->>DEV: 200 OK — App is live! 🎉
```

**Complete deployment commands:**

```bash
# ── 1. Apply manifests ──────────────────────────────────────
kubectl apply -f deployment.yaml

# ── 2. Watch rollout progress ───────────────────────────────
kubectl rollout status deployment/my-application

# ── 3. Verify pods are running ──────────────────────────────
kubectl get pods -l app=my-application

# ── 4. Get LoadBalancer endpoint ────────────────────────────
kubectl get service my-app-service
# NAME              TYPE           CLUSTER-IP     EXTERNAL-IP
# my-app-service    LoadBalancer   10.100.0.123   abc.us-east-1.elb.amazonaws.com

# ── 5. Stream live logs ─────────────────────────────────────
kubectl logs -l app=my-application --tail=100 -f

# ── 6. Scale manually if needed ─────────────────────────────
kubectl scale deployment/my-application --replicas=5

# ── 7. Rollback if something goes wrong ─────────────────────
kubectl rollout undo deployment/my-application
```

---

## 📊 Flow & Network Diagrams

### Request Flow — Internet to Pod

```mermaid
flowchart LR
    USER["👤 User\nBrowser / App"] -->|"HTTPS :443"| R53
    R53["🔵 Route 53\nabc.example.com"] -->|"DNS A Record"| ALB
    ALB["⚖️ ALB\nssl-termination"] -->|"HTTP :80"| TG
    TG["🎯 Target Group\nRegistered Pods"] -->|"Port 3000"| SVC
    SVC["🌐 K8s Service\nClusterIP"] -->|"Round Robin"| POD

    subgraph POD["📦 Application Pod"]
        CONT["Node.js Container\nPort 3000"]
    end

    style USER fill:#1A0D1A,stroke:#8B5CF6
    style ALB fill:#1A0D0D,stroke:#EF4444
    style SVC fill:#0D2137,stroke:#00C6FF
    style POD fill:#0A1E2E,stroke:#10B981
```

### Cluster Autoscaling Flow

```mermaid
flowchart TD
    TRAFFIC["📈 Traffic Spike"] --> HPA
    HPA["📊 HPA\nDetects CPU > 70%"] -->|"Scale pods"| DEPLOY
    DEPLOY["📋 Deployment\nRequests more pods"] -->|"Pending pods"| CA
    CA["⚙️ Cluster Autoscaler\nor Karpenter"] -->|"Provision nodes"| ASG
    ASG["🖧 Auto Scaling Group\nLaunches EC2 instances"] -->|"Nodes join cluster"| NODES
    NODES["✅ New Worker Nodes\nReady state"] -->|"Pods scheduled"| PODS
    PODS["📦 New Pods\nRunning"] -->|"Traffic served"| DONE["✅ Load Handled"]

    style HPA fill:#0D2137,stroke:#00C6FF
    style CA fill:#0A1E2E,stroke:#10B981
    style ASG fill:#1A1A0D,stroke:#F59E0B
```

---

## 🔑 Quick Reference Cheatsheet

### Essential kubectl Commands

```bash
# ─── Cluster Info ─────────────────────────────────────────────
kubectl cluster-info                          # Cluster endpoint info
kubectl get nodes -o wide                     # All nodes + IPs
kubectl describe node <node-name>             # Node details

# ─── Pods ─────────────────────────────────────────────────────
kubectl get pods -A                           # All pods, all namespaces
kubectl get pods -l app=my-app                # Filter by label
kubectl describe pod <pod-name>               # Pod details + events
kubectl logs <pod-name> -f                    # Stream logs
kubectl exec -it <pod-name> -- /bin/sh        # Shell into pod

# ─── Deployments ──────────────────────────────────────────────
kubectl get deployments                       # List deployments
kubectl rollout status deployment/<name>      # Check rollout
kubectl rollout history deployment/<name>     # View history
kubectl rollout undo deployment/<name>        # Rollback
kubectl scale deployment/<name> --replicas=5  # Scale

# ─── Services ─────────────────────────────────────────────────
kubectl get services                          # List services
kubectl get service <name> -o yaml            # Full service spec

# ─── Debugging ────────────────────────────────────────────────
kubectl get events --sort-by='.lastTimestamp' # Recent events
kubectl top nodes                             # CPU/Memory by node
kubectl top pods                              # CPU/Memory by pod
```

### AWS EKS Commands

```bash
# ─── Cluster Management ───────────────────────────────────────
aws eks list-clusters                         # List all clusters
aws eks describe-cluster --name my-cluster    # Cluster details
aws eks update-kubeconfig --name my-cluster   # Connect kubectl

# ─── Node Groups ──────────────────────────────────────────────
aws eks list-nodegroups --cluster-name my-cluster
aws eks describe-nodegroup --cluster-name my-cluster --nodegroup-name workers
aws eks update-nodegroup-config \             # Scale node group
  --cluster-name my-cluster \
  --nodegroup-name workers \
  --scaling-config minSize=3,maxSize=15,desiredSize=5

# ─── Upgrades ─────────────────────────────────────────────────
aws eks update-cluster-version \              # Upgrade K8s version
  --name my-cluster --kubernetes-version 1.29
```

---

<div align="center">

## 🏆 Technologies Used

| Technology | Version | Role |
|:---:|:---:|:---|
| ![Kubernetes](https://img.shields.io/badge/-Kubernetes-326CE5?logo=kubernetes&logoColor=white) | v1.28+ | Container orchestration platform |
| ![AWS](https://img.shields.io/badge/-Amazon_EKS-FF9900?logo=amazon-aws&logoColor=white) | Latest | Managed Kubernetes control plane |
| ![Docker](https://img.shields.io/badge/-Docker-2496ED?logo=docker&logoColor=white) | Latest | Container build & packaging |
| ![ECR](https://img.shields.io/badge/-Amazon_ECR-FF9900?logo=amazon-aws&logoColor=white) | - | Private container image registry |
| ![IAM](https://img.shields.io/badge/-AWS_IAM-232F3E?logo=amazon-aws&logoColor=white) | - | Authentication & authorization |
| ![VPC](https://img.shields.io/badge/-Amazon_VPC-8C4FFF?logo=amazon-aws&logoColor=white) | - | Network isolation & routing |
| ![CloudWatch](https://img.shields.io/badge/-CloudWatch-FF4F8B?logo=amazon-aws&logoColor=white) | - | Monitoring, metrics & logs |

---

**Built for portfolio & resume by a Cloud Engineer passionate about Kubernetes**

[![AWS Certified](https://img.shields.io/badge/AWS-Solutions_Architect-FF9900?style=flat-square&logo=amazon-aws)](https://aws.amazon.com/certification/)
[![CKA](https://img.shields.io/badge/Certified-Kubernetes_Administrator-326CE5?style=flat-square&logo=kubernetes)](https://www.cncf.io/certification/cka/)

</div>
