To install Docker on **Amazon Linux 2**, here’s a quick step-by-step guide you can run manually or automate with a script or Ansible:

---

## Manual Docker Installation on Amazon Linux 2

1. **Update packages:**

   ```bash
   sudo yum update -y
   ```

2. **Enable Docker via Amazon Linux Extras:**

   ```bash
   sudo amazon-linux-extras enable docker
   ```

3. **Install Docker:**

   ```bash
   sudo yum install docker -y
   ```

4. **Start Docker service:**

   ```bash
   sudo systemctl start docker
   ```

5. **Enable Docker to start on boot:**

   ```bash
   sudo systemctl enable docker
   ```

6. **Add your user to the docker group (optional):**

   ```bash
   sudo usermod -aG docker $USER
   ```

7. **Verify Docker installation:**

   ```bash
   docker --version
   docker run hello-world
   ```

---

If you want, I can help create an **Ansible playbook** or **bash script** to automate this installation. Just let me know!
