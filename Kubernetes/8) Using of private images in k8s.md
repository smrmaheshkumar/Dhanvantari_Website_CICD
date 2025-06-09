To use **private container images** in **Kubernetes**, you need to **authenticate your cluster with the image registry**, typically by creating a **Secret** of type `docker-registry` and referencing it in your pod or deployment.

---

## 🔐 Step-by-Step: Use Private Images in Kubernetes

### 1. **Create a Docker Registry Secret**

Use the `kubectl create secret docker-registry` command:

```bash
kubectl create secret docker-registry mydockerregistrysecret \
  --docker-server=index.docker.io \
  --docker-username=smrmaheshkumar \
  --docker-password=Dars@130324 \
  --docker-email=smr.maheshkumar@gmail.com
```

🔁 Replace:

* `myregistrykey`: name of your secret
* `REGISTRY_URL`: e.g. `index.docker.io` (for Docker Hub) or `ghcr.io` (for GitHub Container Registry)
* `USERNAME`, `PASSWORD`, `EMAIL`: your credentials

---

### 2. **Reference the Secret in a Pod/Deployment**

Update your `Deployment` or `Pod` YAML to use the secret under `imagePullSecrets`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dhanvantari-deployment
  labels:
     app: website

spec:
  replicas: 2
  selector:
    matchLabels:
      app: website

  template:
    metadata:
      labels:
        app: website
    spec:
      containers:
      - name: dhanvantari-container
        image: smrmaheshkumar/dhanvantari-image
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
      imagePullSecrets:
      - name: mydockerregistrysecret

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

---

### 3. ✅ (Optional) Use Default ImagePullSecrets for a Namespace

To avoid adding `imagePullSecrets` to every pod spec, set it at the **namespace level**:

```bash
kubectl patch serviceaccount default \
  -n your-namespace \
  -p '{"imagePullSecrets": [{"name": "myregistrykey"}]}'
```

---

## 🐳 Supported Registries

This works with most container registries:

* Docker Hub
* GitHub Container Registry (`ghcr.io`)
* GitLab Container Registry
* AWS ECR (requires token-based auth)
* Google Container Registry (GCR)
* Azure Container Registry (ACR)

Let me know which registry you're using, and I can give you an exact example.
