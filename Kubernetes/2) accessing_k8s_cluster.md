To access an **EKS cluster from an EC2 instance**, you need to configure the EC2 instance with the proper tools and permissions. Here's a step-by-step guide:

---

### ✅ Prerequisites

1. **EKS Cluster is already running**
2. **EC2 instance is launched in the same VPC or has network access to the EKS cluster**
3. **IAM Role or IAM user with EKS access**
4. **Security Groups allow access to the Kubernetes API server (port 443)**

---

### 🧰 Step-by-Step Setup

#### 1. **Install Required Tools on EC2**

SSH into your EC2 instance, then install:

##### a. AWS CLI:

```bash
sudo yum install -y awscli   # Amazon Linux
# OR for Ubuntu
sudo apt update && sudo apt install -y awscli
```

##### b. `kubectl` (Kubernetes CLI):

```bash
curl -o kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.29.0/2024-05-10/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

##### c. `eksctl` (optional but useful for cluster operations):

```bash
curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

---

#### 2. **Configure AWS CLI**

If using an IAM user:

```bash
aws configure
# Enter AWS Access Key, Secret Key, Region
```

If using an IAM role attached to the EC2:

* Make sure the role has the following policies:

  * `AmazonEKSClusterPolicy`
  * `AmazonEKSWorkerNodePolicy`
  * `AmazonEC2ContainerRegistryReadOnly`

---

#### 3. **Update kubeconfig for EKS Cluster**

Run:

```bash
aws eks --region us-west-2 update-kubeconfig --name my-eks-cluster
```

Replace:

* `<region>` = your AWS region (e.g., `us-east-1`)
* `<cluster-name>` = name of your EKS cluster

This updates or creates the kubeconfig at `~/.kube/config`.

---

#### 4. **Verify Access**

Run:

```bash
kubectl get nodes
```

You should see a list of nodes in the EKS cluster.

---

### ⚠️ Common Troubleshooting

* **Access Denied**: Ensure your IAM role/user has `eks:DescribeCluster` permission.
* **Timeout Errors**: Make sure EC2 can reach the EKS control plane (check VPC/Security Groups).
* **RBAC issues**: Your IAM role must be mapped in the EKS cluster's `aws-auth` ConfigMap.

You can check it with:

```bash
kubectl -n kube-system describe configmap aws-auth
```

Let me know your setup details (Amazon Linux vs Ubuntu, IAM role/user, region, etc.) if you want help debugging.
