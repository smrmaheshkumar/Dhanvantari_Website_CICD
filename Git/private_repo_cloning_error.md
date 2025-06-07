The error you're seeing in Jenkins is:

```
No ECDSA host key is known for github.com and you have requested strict checking.
Host key verification failed.
fatal: Could not read from remote repository.
```

This means Jenkins (or rather the system Jenkins is running on) **has not verified the SSH fingerprint of GitHub**, so it is refusing to connect due to strict host key checking.

---

## ✅ Fix This Error in 3 Steps:

### 🔧 **1. Manually Trust GitHub's SSH Host Key**

Log in to the Jenkins server (EC2 or whatever host it's running on), then run:

```bash
ssh -T git@github.com
```

You’ll likely see a message like:

```
The authenticity of host 'github.com (IP_ADDRESS)' can't be established.
Are you sure you want to continue connecting (yes/no)?
```

Type `yes` and press Enter. This will add GitHub's key to `~/.ssh/known_hosts`.

> ⚠️ This step is necessary **only once** to trust GitHub's identity.

---

### ✅ **2. Ensure Jenkins User Has Access**

Jenkins runs as the `jenkins` user (not your login user). So:

* Either switch to the Jenkins user:

  ```bash
  sudo su - jenkins
  ```

  Then run:

  ```bash
  ssh -T git@github.com
  ```
* Or **copy the `known_hosts` and private key** to the Jenkins user:

  ```bash
  sudo mkdir -p ~jenkins/.ssh
  sudo cp ~/.ssh/known_hosts ~jenkins/.ssh/
  sudo cp ~/.ssh/id_rsa ~jenkins/.ssh/   # or use credential manager instead
  sudo chown -R jenkins:jenkins ~jenkins/.ssh
  sudo chmod 600 ~jenkins/.ssh/id_rsa
  sudo chmod 644 ~jenkins/.ssh/known_hosts
  ```

---

### 🧪 **3. Rebuild the Jenkins Job**

* Go back to Jenkins.
* Use SSH URL like:

  ```
  git@github.com:smrmaheshkumar/Dhanvantari_Website_CICD.git
  ```
* Make sure the **SSH credential** (from Jenkins credentials manager) is selected.

Click **Save → Build Now**, and the cloning should succeed.

---

Let me know if you'd like me to help troubleshoot SSH key config or switch to using Personal Access Token via HTTPS.

