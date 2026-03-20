# 🌐 What is a Server?

A server is **not a machine**.

> A server is a process that listens on a port and responds to incoming network requests.

❌ Not:

- A powerful computer
- A cloud VM
- Ubuntu Server edition
- A machine without GUI

> If a process is listening and responding, the machine is acting as a server.


### 🔧 Example: SSH Server

When `openssh-server` is installed and started:

```bash
sudo systemctl start ssh
```

It starts a process called `sshd`.

### What `sshd` does:

- Binds to port `22` (default SSH port)
- Listens for incoming TCP connections
- Authenticates users
- Spawns shell sessions



# 🔌 What does "Listening on a Port" mean?

When an application starts a server:

```ts
app.listen(3000, "0.0.0.0")
```

Internally, it interacts with the kernel:

```c
bind(socket, IP, port)
listen(socket)
```

### Kernel responsibilities:

- Checks if the port is available
- Registers the socket
- Marks it as **LISTENING**

Now the kernel knows:

> If a packet arrives at `IP:Port` → deliver it to this process


### ⚠️ Important

> The application does NOT watch the network directly.  
> The **kernel handles all network traffic** and forwards relevant data to the process.

# 🌍 How does a Request Travel? (LAN)

### Assumptions:

- Server bound to `0.0.0.0:3000`
- Server IP: `192.168.1.48`
- Client and server are in the same subnet

Client sends request:

```bash
curl http://192.168.1.48:3000
```

---

### 🧭 Step 1 - Packet Creation (Client)

Client creates:

- TCP segment (destination port `3000`)
- Wrapped inside an IP packet
- Wrapped inside a WiFi/Ethernet frame (with MAC addresses)

### 🧭 Step 2 - ARP (IP → MAC Resolution)

Client checks:

> Is `192.168.1.48` in my subnet?

If yes:

- Sends ARP broadcast:
  
  ```
  Who has 192.168.1.48?
  ```
  
- Target device responds with its MAC address
- Client stores mapping in ARP table:

```
192.168.1.48 → MAC address
```

### 🧭 Step 3 - Frame Delivery

Client sends frame:

- Destination MAC → Server MAC
- Source MAC → Client MAC

Router acts as:

- Layer 2 bridge (access point)

> No routing happens because both devices are in the same subnet.

### 🧭 Step 4 - Server NIC

- Server network interface receives the frame
- Passes it to the kernel


### 🧭 Step 5 - Kernel Network Stack

Kernel:

- Unwraps frame → IP → TCP
- Verifies destination IP
- Passes to firewall (if enabled)

### 🧭 Step 6 - Firewall

- DROP → timeout
- REJECT → connection refused
- ALLOW → continue

### 🧭 Step 7 - Socket Lookup

Kernel checks:

```
192.168.1.48:3000
```

Matches if:

```
0.0.0.0:3000
```

Does NOT match if:

```
127.0.0.1:3000
```

If no match → TCP RST → connection refused

### 🧭 Step 8 - Process Handles Request

- Kernel establishes TCP connection
- Hands over to process
- Process generates response
- Response travels back (reverse path)

### 🧭 End-to-End Flow

```
Client App
   ↓
Client Kernel
   ↓
Client NIC
   ↓
Access Point / Switch
   ↓
Server NIC
   ↓
Server Kernel
   ↓
Firewall
   ↓
Socket Lookup
   ↓
Server Process
```
# 🧠 Layer Responsibilities

| Layer | Responsibility |
| --- | --- |
| Link | MAC delivery |
| Network | IP addressing & routing |
| Transport | TCP/UDP & ports |
| Kernel | Socket mapping |
| Application | Business logic |

# ❗ Why Errors Happen

| Error | Cause |
| --- | --- |
| Network unreachable | Routing issue |
| Connection timed out | Firewall DROP / no response |
| Connection refused | No process listening / port closed |
| Permission denied | Application-level issue |


# ✅ What Makes a Machine a Server?

A machine acts as a server when:

1. A process is running
2. It binds to a port
3. Kernel marks it as LISTENING
4. It responds to incoming requests

> Everything else (firewall, router, NAT) only controls reachability.