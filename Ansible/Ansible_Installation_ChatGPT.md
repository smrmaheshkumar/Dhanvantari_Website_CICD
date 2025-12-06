To install **Ansible** on an **Amazon Linux 2** instance, you can follow these steps:

---

### Step 1: Update the system packages

```bash
sudo yum update -y   
```

---

### Step 2: Install EPEL repository

Ansible is available in the Extra Packages for Enterprise Linux (EPEL) repository, so you need to enable it:

```bash
sudo amazon-linux-extras install epel -y
```

---

### Step 3: Install Ansible

Now install Ansible using `yum`:

```bash
sudo yum install ansible -y
```

---

### Step 4: Verify Ansible installation

Check the installed version:

```bash
ansible --version
```

You should see output similar to:

```
ansible [core 2.xx.x]
  config file = /etc/ansible/ansible.cfg
  ...
```

---

### Optional: Install a specific Ansible version using pip (if you want latest version)

If you want the latest Ansible version instead of the repo version, you can install Python and pip, then install Ansible via pip:

```bash
sudo yum install python3 -y
python3 -m pip install --user --upgrade pip
python3 -m pip install --user ansible
```

Add `~/.local/bin` to your `PATH` if it’s not already:

```bash
export PATH=$PATH:~/.local/bin
```

Then check version:

```bash
ansible --version
```

---

If you want, I can help with writing a script or an Ansible playbook to automate this. Would you like that?
