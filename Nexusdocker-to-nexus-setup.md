This error means Docker is trying to connect via **HTTPS**, but your Nexus repository is only serving **HTTP** on port `5000`.

To fix this, you need to **explicitly tell Docker to treat your Nexus registry as an insecure HTTP registry**.

---

## 🔧 Fix: Mark Nexus as an Insecure Registry

### 🖥️ On Linux:

1. **Edit Docker daemon config**:

```bash
sudo nano /etc/docker/daemon.json
```

2. **Add this** (or merge if the file already exists):

```json
{
  "insecure-registries": ["35.90.70.193:5000"]
}
```

> ⚠️ Make sure the IP is correct and inside quotes. If there are other entries, use a comma-separated list.

3. **Restart Docker**:

```bash
sudo systemctl restart docker
```

---

## 🔁 Retry Login:

```bash
docker login 35.90.70.193:5000
```

Enter `admin` and your password again.

---

## ✅ Confirm:

If successful, proceed with tagging and pushing the image:

```bash
docker tag my-app:1.0 35.90.70.193:5000/my-app:1.0
docker push 35.90.70.193:5000/my-app:1.0
```

---

Let me know if your Nexus is behind a domain name or using HTTPS — that would need a certificate setup.
