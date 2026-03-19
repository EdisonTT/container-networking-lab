
# 🧪 Experiment: Creating a Home Server
## 🎯 Objective

To convert a regular Ubuntu machine into a **headless home server** and enable **remote access via SSH** within a local network.


## 📋 Prerequisites

- A dedicated machine for the server  
  *(Old laptop is sufficient; high specs are not required)*  
- Ubuntu Desktop / Ubuntu Server installed  
- Internet access  
- A local network (home router is enough)  
- A client machine (Windows/Linux/macOS) to access the server remotely  


## 🖥️ Machines Used in This Lab

### Server Machine
- OS: Ubuntu 24.04.2 LTS  
- CPU: AMD A9-9420 Radeon R5 (2C+3G cores)  
- RAM: 4 GB  
- Storage: 1 TB HDD  

### Client Machine
- OS: Windows 11  
- CPU: Intel i5 (11th Gen)  
- RAM: 8 GB  
- Storage: 512 GB SSD  

### Network
- Home router (LAN)


## 🧠 Theory

A server does not require a graphical interface (GUI).  
Running without GUI:

- Reduces memory usage
- Improves performance
- Makes the system more stable for long-running tasks

Remote access is achieved using **SSH (Secure Shell)**:
- Allows secure communication over a network
- Enables full control of the server from another machine


## 🔬 Procedure

### Step 1: Convert Ubuntu Desktop to Headless Server  
*(Skip if using Ubuntu Server)*

Disable GUI:

```bash
sudo systemctl set-default multi-user.target
````

Reboot the system:

```bash
sudo reboot
```

✅ After reboot:

* System runs in terminal mode
* No graphical interface
* Acts like a server

### Step 2: Install SSH Server

```bash
sudo apt update
sudo apt install openssh-server -y
```

### Step 3: Start SSH Service

```bash
sudo systemctl start ssh
```

### Step 4: Enable SSH on Boot

```bash
sudo systemctl enable ssh
```

### Step 5: Verify SSH Status

```bash
sudo systemctl status ssh
```

![SSH running status](/images/server-setup/ssh-status.jpg)

### Step 6: Find Server IP Address

```bash
ip a
```

Look for:

* `wlp3s0` (WiFi) or `eth0` (Ethernet)
* Under it, find:

```
inet 192.168.x.x
```

Example:

```
inet 192.168.1.48
```

### Step 7: Connect to Server from Client

On client machine (PowerShell / Terminal):

```bash
ssh username@server-ip
```

Example:

```bash
ssh home-server@192.168.1.48
```

Enter password → Access granted


## 📊 Observations

* GUI removal significantly reduced memory usage (~500MB observed)
* SSH service runs as a background process
* Server becomes accessible within the local network
* IP address is assigned by router using **DHCP**

## 🔍 Results & Findings

* A normal Ubuntu machine can be converted into a server without reinstalling OS
* Headless systems are more efficient for server use
* SSH enables full remote control over the network
* DHCP assigns IP dynamically, but often reuses the same IP for known devices


## 🧾 Key Takeaways

* Servers do not need GUI → saves resources
* SSH is the standard way to manage remote systems
* Local network (LAN) is enough to build a home lab
* Understanding IP allocation (DHCP) is important for stable access

## 💡 Notes

* IP address may change over time (DHCP behavior)
* Routers can be configured for static IP assignment (if supported)
* MAC address is used by router to identify devices
* In this setup, IP remained stable throughout experiments


## 🤔 Why Ubuntu Desktop Instead of Ubuntu Server?

* Existing system already had data
* Avoided reinstalling OS
* Disabled GUI to achieve server-like behavior


## 🔮 Next Steps

* Static IP configuration
* Understanding DHCP in detail
* Docker installation on server
* Container networking experiments
