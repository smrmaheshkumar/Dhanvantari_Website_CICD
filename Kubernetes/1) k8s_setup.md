## Prerequisites

1. **Amazon Linux 2 EC2 instance** with proper IAM permissions.
2. **AWS CLI v2** installed and configured (`aws configure`).
3. **kubectl** installed.
4. **eksctl** installed — the easiest way to create and manage EKS clusters.

---

## Step 1: Update Amazon Linux 2 and Install Dependencies

```bash
sudo yum update -y
sudo yum install -y curl jq
```

---

## Step 2: Install AWS CLI v2 (if not installed)

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```
## Now create an IAM Role and attach this role to an EC2 instance

   IAM user should have access to the following services

   IAM, EC2, CloudFormation & AdministorAccess
   
   Note: Check eksctl documentaiton for [Minimum IAM policies](https://eksctl.io/usage/minimum-iam-policies/)

---

## Step 3: Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

---

## Step 4: Install eksctl

`eksctl` is a CLI tool by AWS to create/manage EKS clusters easily.

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

---

## Step 5: Create the EKS Cluster

Basic example to create a cluster named `my-cluster` in `us-east-1` region with 2 nodes.

```bash
eksctl create cluster \
  --name my-eks-cluster \
  --region us-west-2 \
  --nodegroup-name linux-nodes \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed
```

This command will:

* Create an EKS cluster control plane.
* Create a managed node group with EC2 instances running Amazon Linux 2.
* Configure `kubectl` context automatically.

---

## Step 6: Verify your cluster

Check nodes:

```bash
kubectl get nodes
```

---

## Step 7: Cleanup

To delete the cluster when done:

```bash
eksctl delete cluster --name my-eks-cluster --region us-west-2
```

---

If you want, I can also help with creating the cluster using CloudFormation or Terraform or do a custom manual setup. Let me know!




   #### Deploying Nginx pods on Kubernetes
1. Deploying Nginx Container
    ```sh
    kubectl create deployment nginx-deployment --image=nginx --replicas=4 --port=80
    ```
    ### By using the above syntax we will deploy our own developed application as follows
    ```sh
    kubectl create deployment mydeployment --image=dhanvantarisoftware/dhanvantari-image --replicas=4 --port=8080
    kubectl get all
    kubectl get pod
   ```

1. ### Expose the deployment as service. This will create an ELB in front of those 4 containers and allow us to access publicly.
   ```sh
   kubectl expose deployment nginx-deployment --port=80 --type=LoadBalancer
   kubectl expose deployment mydeployment --port=8080 --type=LoadBalancer
   kubectl get services -o wide
   ```

### Kubectl Troubleshooting

The error `-bash: kubectl: command not found` means `kubectl` is not installed or not in your system's PATH.

Let's fix that step-by-step on Amazon Linux 2.

---

### Install `kubectl` on Amazon Linux 2

Run these commands:

```bash
# Download kubectl binary (latest stable release)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make it executable
chmod +x kubectl

# Move it to a directory in your PATH
sudo mv kubectl /usr/local/bin/

# Verify installation
kubectl version --client
```

---

If this succeeds, you should see the kubectl client version output.

---

If you get a "permission denied" or similar error, make sure `/usr/local/bin` is in your PATH:

```bash
echo $PATH
```

If not, you can add it by:

```bash
export PATH=$PATH:/usr/local/bin
```

Then try running `kubectl version --client` again.

---

Let me know if you hit any error!

