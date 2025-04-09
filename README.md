# aws-puppet-webserver

Simple **Puppet Project on EC2** you can try to demonstrate **configuration management** using **Puppet Master and Puppet Agent** setup on Amazon EC2 instances.

---

## 🎯 **Project Goal:**
Set up a **Puppet Server (Master)** and a **Puppet Agent** on EC2 instances to **automatically install and manage Apache web server**.

---

## 🧱 **Infrastructure Setup**

### 1. **Launch EC2 Instances:**
- Launch **2 Amazon Linux 2** or **Amazon Linux 2023** EC2 instances:
  - `puppet-master`
  - `puppet-agent`
- Ensure both are in the **same VPC** and **security group** with:
  - Port **8140** open (Puppet communication)
  - Port **22** for SSH
  - Port **80** for HTTP (for Apache)

---

## 🧰 **Step-by-Step Setup**

### 🔧 On Puppet Master (EC2: `puppet-master`)
```bash
sudo hostnamectl set-hostname puppet-master
sudo yum update -y
sudo amazon-linux-extras enable epel
sudo yum install epel-release -y
sudo yum install puppetserver -y
```

### ✅ Configure Puppet Master
Edit Puppet configuration:
```bash
sudo nano /etc/puppetlabs/puppet/puppet.conf
```
Add:
```ini
[main]
certname = puppet-master
server = puppet-master
environment = production
runinterval = 1h

[agent]
server = puppet-master
```

Set Java memory limits:
```bash
sudo nano /etc/sysconfig/puppetserver
```
Update:
```bash
JAVA_ARGS="-Xms512m -Xmx512m"
```

Start and enable the Puppet Server:
```bash
sudo systemctl start puppetserver
sudo systemctl enable puppetserver
```

---

### 🔧 On Puppet Agent (EC2: `puppet-agent`)
```bash
sudo hostnamectl set-hostname puppet-agent
sudo yum update -y
sudo amazon-linux-extras enable epel
sudo yum install epel-release -y
sudo yum install puppet-agent -y
```

Configure Puppet Agent:
```bash
sudo nano /etc/puppetlabs/puppet/puppet.conf
```
Add:
```ini
[main]
server = puppet-master
```

Start the agent:
```bash
sudo /opt/puppetlabs/bin/puppet resource service puppet ensure=running enable=true
```

Run agent manually to request cert:
```bash
sudo /opt/puppetlabs/bin/puppet agent --test
```

---

### 🛡️ Sign Certificate on Puppet Master
```bash
sudo /opt/puppetlabs/bin/puppetserver ca list
sudo /opt/puppetlabs/bin/puppetserver ca sign --all
```

---

## 📦 Create a Puppet Manifest (Apache install)

On `puppet-master`:
```bash
sudo mkdir -p /etc/puppetlabs/code/environments/production/manifests
sudo nano /etc/puppetlabs/code/environments/production/manifests/site.pp
```

Paste this:
```puppet
node 'puppet-agent' {
  package { 'httpd':
    ensure => installed,
  }

  service { 'httpd':
    ensure => running,
    enable => true,
  }

  file { '/var/www/html/index.html':
    ensure  => file,
    content => "<h1>Puppet Managed Web Server</h1>",
  }
}
```

---

## 🚀 Test It
On `puppet-agent`:
```bash
sudo /opt/puppetlabs/bin/puppet agent --test
```

Then visit `http://<puppet-agent-public-ip>` in a browser – you should see the Apache welcome message.

---

## 📘 Optional Enhancements
- Add firewall rules to allow only Puppet traffic.
- Use Puppet modules for better structure.
- Automate signing using autosign.conf for scale.
- Store manifests in Git for GitOps.

---
