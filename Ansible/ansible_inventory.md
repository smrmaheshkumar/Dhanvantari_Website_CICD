If you want to configure an **Ansible inventory** file for **password authentication** (instead of SSH key authentication), you'll need to:

1. Specify the **hosts**.
2. Provide the **username** and **password**.
3. Optionally disable host key checking and key-based auth.

Here's an example of an **INI-style inventory** with variables for **password authentication**:

---

### 📝 Example: `inventory.ini`

```ini
[webservers]
192.168.1.10
192.168.1.11

[webservers:vars]
ansible_user=your_username
ansible_ssh_pass=your_password
ansible_port=22
ansible_connection=ssh
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

---

### 🔐 Best Practice for Passwords

**Avoid putting plain-text passwords in inventory files.** Use [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) to encrypt sensitive data:

```bash
ansible-vault encrypt_string 'your_password' --name 'ansible_ssh_pass'
```

Example result in inventory:

```ini
ansible_ssh_pass=!vault |
          $ANSIBLE_VAULT;1.1;AES256
          333465313636353034353163653031653339...
```

---

### ✅ Run Example

To run a playbook with this inventory:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

---

Would you like a YAML-style inventory (`inventory.yml`) or a dynamic inventory version as well?
