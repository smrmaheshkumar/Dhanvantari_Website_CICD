To set a **default namespace** for `kubectl` commands so you don’t have to specify `-n <namespace>` each time, you can do it in **two main ways**:

---

### ✅ Option 1: Set Namespace in Your Current Context

Use this command:

```bash
kubectl config set-context --current --namespace=<your-namespace>
```

🔁 Replace `<your-namespace>` with your actual namespace.

📌 This sets the default namespace **only for the current context**, which is typically what you want.

**Example:**

```bash
kubectl config set-context --current --namespace=dev
```

Now you can run:

```bash
kubectl get pods
```

…and it will default to the `dev` namespace.

---

### ✅ Option 2: Set Namespace in a Specific Context (Optional for Advanced Use)

If you have multiple contexts and want to set the namespace for one of them:

```bash
kubectl config set-context <context-name> --namespace=<your-namespace>
```

Check available contexts with:

```bash
kubectl config get-contexts
```

---

### 🧪 Verify

```bash
kubectl config view --minify | grep namespace:
```

You should see:

```
namespace: <your-namespace>
```

---

Let me know if you want to set this globally or in a script!
