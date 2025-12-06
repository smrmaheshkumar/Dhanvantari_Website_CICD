Thanks for the update. Since you've:

* Enabled `PasswordAuthentication yes`
* Confirmed `PubkeyAuthentication yes`
* Created the `.ssh` and `authorized_keys` files correctly
* Still get `Permission denied (publickey)`

Let’s resolve it step by step.

---

## ✅ Final Fix Checklist

Run these steps **on the target server (`172.31.40.246`)**:

---

### 🔹 1. **Confirm the SSH config values**

Run this command:

```bash
grep -Ei 'password|pubkey' /etc/ssh/sshd_config | grep -v '^#'
```

You should see:

```bash
PasswordAuthentication yes
PubkeyAuthentication yes
```

---

### 🔹 2. **Restart SSH service**

```bash
sudo systemctl restart ssh
```

---

### 🔹 3. **Set password for `ansadmin` (if not done)**

```bash
sudo passwd ansadmin
```

---

### 🔹 4. **Check correct home directory for `ansadmin`**

Run:

```bash
getent passwd ansadmin
```

Sample output should look like:

```
ansadmin:x:1001:1001::/home/ansadmin:/bin/bash
```

Verify that the home is `/home/ansadmin`.

If it’s **not**, then create `.ssh` in the correct home directory.

---

### 🔹 5. **Fix `.ssh` directory and permissions**

```bash
sudo mkdir -p /home/ansadmin/.ssh
sudo touch /home/ansadmin/.ssh/authorized_keys
sudo chown -R ansadmin:ansadmin /home/ansadmin/.ssh
sudo chmod 700 /home/ansadmin/.ssh
sudo chmod 600 /home/ansadmin/.ssh/authorized_keys
```

---

### 🔹 6. **Manually copy the key if `ssh-copy-id` fails**

On the **source (your terminal/MobaXterm)**, run:

```bash
cat ~/.ssh/id_rsa.pub
```

Copy the whole output (it starts with `ssh-rsa` or `ecdsa`).

Then on the **target server**:

```bash
sudo nano /home/ansadmin/.ssh/authorized_keys
```

Paste the key there. Save and exit (Ctrl+O, Enter, Ctrl+X).

---

### 🔹 7. **Try SSH again from source:**

```bash
ssh ansadmin@172.31.40.246
```

---

### 🔍 If it still fails:

Run this **on the source machine** and share output:

```bash
ssh -vvv ansadmin@172.31.40.246
```

This will print detailed debug logs showing **why the key is rejected** — I'll help interpret it.

Let me know how far you get!
