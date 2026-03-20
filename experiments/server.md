# 🧪 Experiment: Observing Server Behavior & Kernel Networking


## 🎯 Objective

To observe how a server process interacts with the Linux kernel by:

- Starting a Node.js server
- Tracking port binding
- Observing process ↔ port mapping
- Monitoring network activity before and after server start
- Understanding how a request flows through the system

👉 Detailed theory: [Server Notes](/notes/server.md)


## 🧠 Concepts Covered

- Server = process + port
- Kernel socket management
- Port binding
- Process ↔ port mapping
- TCP connections
- Request flow (client → kernel → process)

## 🛠️ Tools Used

| Tool | Purpose |
| --- | --- |
| `ip a` | View network interfaces |
| `ss -ltnp` | View listening ports & processes |
| `lsof -i` | Map ports to processes |
| `netstat -tulnp` | Alternative to view ports |
| `ps aux` | View running processes |
| `tcpdump` | Capture network packets |
| `curl` | Send request to server |

## 🔬 Procedure

### Step 1 - Create `server.js` on Home Server (No GUI)

Since the home server runs without a GUI, we create/edit files using **VS Code Remote SSH** from the client machine.

**Setup:**

- Install VS Code extension: `ms-vscode-remote.remote-ssh`
- Connect to the server via SSH from VS Code
- Open a folder on the server

![VS Code Remote](/images/server/vs-code-remote-connection.jpg)

Create a file named `server.js` on the server with the following content:

```javascript
const http = require("http");

const server = http.createServer((req, res) => {
  if (req.method === "GET" && req.url === "/") {
    res.writeHead(200, { "Content-Type": "text/plain" });
    res.end("Hello World");
  } else {
    res.writeHead(404, { "Content-Type": "text/plain" });
    res.end("Not Found");
  }
});

server.listen(3000, () => {
  console.log("Server listening on port 3000");
});
```

Alternative (terminal-only):

```bash
nano server.js
# or use vim
```

### Step 2 - Verify Node.js Installation

```bash
node --version && npm --version
```

![Node version](/images/server/node-version.jpg)

### Step 3 - Observe Initial State

```bash
ss -ltnp
ps aux | grep node
lsof -i
```

👉 Note:

- Existing services (e.g., SSH)
- Possible Node processes (VS Code remote)

> ⚠️ Ignore unrelated Node processes (VS Code server)

![Before Node Server](/images/server/status-before-node-server.jpg)

### Step 4 - Start the Server

```bash
node server.js
```

Re-run all the previous commands. Check the new entries.

![After Node Server](/images/server/status-after-node-server.jpg)


### Step 5 - Verify Server Response

#### From server:

```bash
curl localhost:3000
```

#### From another device:

```bash
curl http://192.168.1.48:3000
```

👉 Both should work

### Step 6 — Observe Network Traffic

```bash
sudo tcpdump -i any port 3000
```

Trigger a request and observe:

- SYN → SYN-ACK → ACK (TCP handshake)
- Data packets

### Step 7 — Stop Server

```bash
CTRL + C
```

Check again:

```bash
ss -ltnp
```

👉 Port 3000 disappears

Try:

```bash
curl http://192.168.1.48:3000
```

👉 Result: Connection failed

## 📊 Observations

- Port appears only when server runs
- Kernel maps port → process
- Requests flow through kernel
- Killing process removes port instantly

## 🔍 Results & Findings

- A server exists only when:
  
  - Process is running
  - Port is bound
  - Kernel manages:
    - IP:PORT → PROCESS
- "Connection refused" = no process listening
  
## 🔥 Bonus Experiment - Binding Behavior

### Step 1 — Modify Server

```javascript
server.listen(3000, "127.0.0.1");
```

### Step 2 — Test Access

From client machine:

```bash
curl http://192.168.1.48:3000
```

❌ Fails


### Step 3 — Debug

```bash
ss -ltnp
```

Output:

```
127.0.0.1:3000
```

---

## 🧠 Why It Failed

- 127.0.0.1 → localhost only
- Server accepts requests only from same machine

Works:

```bash
curl http://localhost:3000
```

Fails:

```bash
curl http://192.168.1.48:3000
```

---

## 🔍 Compare with Previous State

Earlier:

```
*:3000
```

Equivalent to:

```
0.0.0.0:3000
```

👉 Accepts:

- Local requests ✅
- Network requests ✅

---

## 📌 Final Takeaway

| Binding | Accessibility |
| --- | --- |
| 127.0.0.1 | Local machine only |
| 0.0.0.0 / * | All network interfaces |

👉 Binding address controls who can reach your server