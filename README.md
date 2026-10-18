# ☸️ Terraform AWS EKS Platform

Infrastructure as Code for **Amazon EKS** and Kubernetes application deployment. This project provisions a production-style EKS cluster with Terraform and deploys a containerized application with Kubernetes manifests or a Helm chart.

![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)

---

## 📌 What this demonstrates

- **Amazon EKS** cluster + managed node group provisioned with the community `terraform-aws-modules`
- Multi-AZ **VPC** with public/private subnets and NAT gateway
- Cluster access configuration and core add-ons (CoreDNS, kube-proxy, VPC CNI, EBS CSI)
- **ECR** repository for application images
- Application deployment via **plain Kubernetes manifests** *or* a **Helm chart**
- AWS Load Balancer Controller–ready Ingress annotations

## 🏗️ Architecture

```
                     ┌────────────────────────────────────────┐
                     │               AWS VPC                    │
                     │                                          │
     Internet ──────▶│  Public subnets (ALB)                    │
                     │        │                                 │
                     │        ▼                                 │
                     │  Private subnets                         │
                     │   ┌───────────────────────────────┐     │
                     │   │        EKS Cluster            │     │
                     │   │  ┌─────────┐  ┌─────────┐     │     │
                     │   │  │ node    │  │ node    │ ... │     │
                     │   │  └────┬────┘  └────┬────┘     │     │
                     │   └───────┼────────────┼──────────┘     │
                     │           └──────┬─────┘                 │
                     │            ┌─────▼──────┐                │
                     │            │    ECR     │                │
                     │            └────────────┘                │
                     └────────────────────────────────────────┘
```

## 📁 Repository Structure

```
terraform-aws-eks-platform/
├── infra/
│   ├── providers.tf           # AWS, kubernetes, helm providers
│   ├── vpc.tf                 # VPC module (public + private subnets)
│   ├── eks.tf                 # EKS cluster + managed node group
│   ├── ecr.tf                 # Application image repository
│   ├── variables.tf
│   ├── outputs.tf
│   └── terraform.tfvars.example
├── k8s/
│   ├── deployment.yaml        # Plain Kubernetes manifests
│   ├── service.yaml
│   └── ingress.yaml
├── helm/
│   └── app/                   # Helm chart (templates + values)
├── .github/workflows/terraform.yml
└── README.md
```

## ✅ Prerequisites

- Terraform >= 1.5
- AWS CLI v2 and `kubectl`
- (Optional) Helm 3+

## 🚀 Usage

### 1. Provision the cluster

```bash
cd infra
terraform init
cp terraform.tfvars.example terraform.tfvars
terraform apply -auto-approve
```

### 2. Configure kubectl

```bash
$(terraform output -raw configure_kubectl)
kubectl get nodes
```

### 3. Build & push the application image

```bash
ECR=$(terraform output -raw ecr_repository_url)
docker build -t $ECR:latest .
aws ecr get-login-password --region ap-south-1 | \
  docker login --username AWS --password-stdin ${ECR%%/*}
docker push $ECR:latest
```

### 4. Deploy the app

With plain manifests (replace `<ECR_REPOSITORY_URL>` first):

```bash
kubectl apply -f k8s/
```

Or with Helm:

```bash
helm install demo helm/app --set image.repository=$ECR --set image.tag=latest
```

## ⚙️ Key Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `aws_region` | AWS region | `ap-south-1` |
| `project_name` | Resource name prefix | `eks-platform` |
| `cluster_version` | Kubernetes version | `1.30` |
| `az_count` | Availability zones | `3` |
| `node_instance_types` | Node group instance types | `["t3.medium"]` |
| `node_desired_size` / `min` / `max` | Node sizing | `2` / `1` / `4` |

## 🔐 Security Notes

- Worker nodes run in **private subnets**; only the ALB is internet-facing
- Cluster creator is granted admin access via EKS access entries
- Enable IRSA (IAM Roles for Service Accounts) for least-privilege pod access in production

## 📄 License

Released under the MIT License.

---

⭐ If you find this useful, please consider giving it a star.
