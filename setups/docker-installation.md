## Environment

- Old Ubuntu machine (Home Lab Server)
- Accessed remotely via SSH
- Docker installed natively on Linux
- No Docker Desktop
- Tested from another laptop on same LAN

This setup mirrors real production servers.

---

# 1️⃣ Install Docker (Ubuntu – Clean Method)

Remove old versions (if any):

```bash
sudo apt remove docker docker-engine docker.io containerd runc
```

Install prerequisites:

```bash
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release
```

Add Docker GPG key:

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Add repository:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Install Docker:

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Start and enable:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

Test:

```bash
sudo docker run hello-world
```

---

# 2️⃣ Understanding What Docker Really Is

Docker is NOT:

- A VM
- A packet router
- A gatekeeper proxy

Docker IS:

- A daemon (`dockerd`)
- A configurator of Linux kernel networking
- A namespace and cgroup manager

It configures:

- Network namespaces
- veth pairs
- Linux bridge (`docker0`)
- iptables NAT rules

After setup, the **Linux kernel handles all packet flow**.

---

# 3️⃣ Default Docker Network (Bridge Mode)

When Docker starts (no containers yet):

```bash
ip a
```

You will see:

```
docker0
inet 172.17.0.1/16
state DOWN
```

This is the Docker bridge.

It becomes active only when a container attaches.

---

# 4️⃣ Run First Container (No Port Mapping)

```bash
docker run -d --name webtest nginx
```

Now check:

```bash
ip a
```

You should see:

- `docker0` state UP
- A `vethXXXX` interface attached to docker0

Check container IP:

```bash
docker inspect webtest | grep IPAddress
```

Example:

```
"IPAddress": "172.17.0.2"
```

Test direct routing (from same machine):

```bash
curl 172.17.0.2
```

✅ Works  
❌ No NAT involved  
✔ Pure bridge routing

---

# 5️⃣ Inspect Routing Table

```bash
ip route
```

You should see:

```
172.17.0.0/16 dev docker0
```

Meaning:

Any traffic for 172.17.x.x goes through docker0.

That is why `curl 172.17.0.2` works locally.

---

# 6️⃣ Run Container With Port Mapping (DNAT)

```bash
docker run -d -p 8080:80 --name web2 nginx
```

Check:

```bash
docker ps
```

You will see:

```
0.0.0.0:8080->80/tcp
```

Meaning:

Host port 8080 → Container port 80

---

# 7️⃣ Inspect NAT Rules

```bash
sudo iptables -t nat -L -n
```

You will see something like:

```
DNAT tcp -- anywhere anywhere tcp dpt:8080 to:172.17.0.3:80
```

This means:

If destination port = 8080  
Rewrite destination → 172.17.0.3:80

This is **DNAT (Destination NAT)**.

Docker did not proxy traffic.  
It only inserted this kernel rule.

---

# 8️⃣ Full Packet Flow (Incoming)

When you access from laptop:

```bash
curl http://<server-ip>:8080
```

Packet flow:

```
Laptop
  ↓
Server network interface (wlp3s0)
  ↓
iptables PREROUTING (DNAT happens here)
  ↓
Routing table decision
  ↓
docker0 bridge
  ↓
veth pair
  ↓
Container
```

Docker daemon is NOT in this path.

Kernel handles everything.

---

# 9️⃣ Verify Using tcpdump

Install:

```bash
sudo apt install tcpdump
```

Watch external interface:

```bash
sudo tcpdump -i wlp3s0 port 8080
```

You will see traffic to 8080.

Watch container side:

```bash
sudo tcpdump -i vethXXXX port 80
```

You will see:

- Destination changed to 172.17.x.x
- Port changed to 80
  This proves DNAT is working.

---

# 🔟 Outbound Traffic (MASQUERADE / SNAT)

Check NAT table:

```bash
sudo iptables -t nat -L -n
```

You will see:

```
MASQUERADE  all  --  172.17.0.0/16  anywhere
```

This means:

When container sends traffic to internet:

Source IP:

```
172.17.0.3
```

Gets rewritten to:

```
192.168.1.xx
```

So router can reply correctly.

This is **SNAT (Source NAT)** via MASQUERADE.

---

# 1️⃣1️⃣ Key Concepts Learned

### Containers Are Just Processes

- With isolated network namespaces
- Connected via veth pairs
- Attached to a Linux bridge

### Docker Does NOT Route Packets

Docker:

- Creates namespace
- Creates bridge
- Writes iptables rules
- Steps aside

Linux kernel:

- Performs DNAT
- Performs routing
- Performs SNAT
- Handles connection tracking (conntrack)

---

# 1️⃣2️⃣ Mental Model (Final)

Without port mapping:

```
Host → docker0 → veth → Container
```

With port mapping:

```
External → Host:8080
        → DNAT → Container:80
```

Outbound:

```
Container → MASQUERADE → Host IP → Router
```

---

# 1️⃣3️⃣ Clean Reset (If Needed)

Stop and remove containers:

```bash
docker rm -f webtest web2
```

Check bridge:

```bash
ip a
```

It may return to DOWN state.

You are back to baseline.

---

# Final Understanding

You are not learning “Docker commands”.

You are learning:

- Linux routing
- Network namespaces
- Bridge networking
- DNAT
- SNAT
- Conntrack
  Docker is just the tool that exposed it.
