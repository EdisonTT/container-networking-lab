# 🐧 Linux Networking & System Commands

### 🌐 Network Interfaces & Routing
| Command | What it does |
| --- | --- |
| `ip a` | View all network interfaces (physical cards, bridges, veth cables) and their IP addresses. |
| `ip route` | View the IP routing table. Shows which subnet is attached to which interface. |
| `ip neigh` | View the ARP table. Shows MAC addresses learned by the kernel for each interface. |
| `ping <ip>` | Test network reachability to a specific IP address. |

### 🔌 Ports & Sockets
| Command | What it does |
| --- | --- |
| `sudo ss -tulpn` | View all listening TCP/UDP sockets, their ports, and the Process IDs (PIDs) bound to them. |
| `sudo lsof -i :<port>` | Find exactly which process is listening on a specific port (e.g., `sudo lsof -i :3000`). |

### ⚙️ System Services (systemd)
| Command | What it does |
| --- | --- |
| `sudo systemctl status <service>` | Check if a background service (like `ssh` or `docker`) is running. |
| `sudo systemctl start <service>` | Start a background service immediately. |
| `sudo systemctl enable <service>` | Set a service to start automatically when the server boots. |
