Here’s how you can **install Jenkins on an Amazon Linux 2 AMI** step-by-step:

### ✅ Step 1: Update the system

```bash
sudo yum update -y
```

---

### ✅ Step 2: Install Java (Jenkins requires Java 11 or 17+)

```bash
sudo amazon-linux-extras enable corretto11
sudo yum install java-11-amazon-corretto -y
sudo rpm --import https://yum.corretto.aws/corretto.key
sudo curl -L -o /etc/yum.repos.d/corretto.repo https://yum.corretto.aws/corretto.repo
sudo yum install -y java-17-amazon-corretto-devel wget -y
```

Verify Java installation:

```bash
java -version
```

---

### ✅ Step 3: Add Jenkins Repository & Maven Packages

```bash
cd /opt
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

wget https://dlcdn.apache.org/maven/maven-3/3.9.10/binaries/apache-maven-3.9.10-bin.tar.gz
tar -xvzf apache-maven-3.9.10-bin.tar.gz
mv apache-maven-3.9.10 maven
```

---

### ✅ Step 4: Install Jenkins

```bash
sudo yum install jenkins git -y
```

---

### ✅ Step 5: Start and Enable Jenkins

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

Check status:

```bash
sudo systemctl status jenkins
```

### ✅ Step 6: Unlock Jenkins

Get the initial admin password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Use this in the Jenkins web UI.

---
