# 🐋 What is Docker?

Docker is **not a virtual machine**.

> Docker is a daemon that orchestrates Linux kernel features to create isolated environments for processes.

❌ Not:

- A full guest OS
- A packet router
- A gatekeeper proxy

> The container thinks it is a fully independent machine, but it shares the host's exact Linux kernel.

### 🧠 Kernel Responsibilities:

- Docker configures the network and writes the firewall rules, then **steps aside**.
- The **Linux kernel** handles all the actual packet routing, NAT, and ARP.


# 🌐 How Docker Networking Works

### 🔧 Key Components:

- **veth Pair:** A virtual Ethernet cable. One end is inside the container (`eth0`), the other sits in the host's namespace.
- **Bridge (`docker0`):** A virtual network switch on the host. All container `veth` cables plug into this switch.
- **IP & MAC Addresses:** Every container gets its own unique MAC address and an IP address assigned by Docker's IPAM (not by your home router).


# 🛠️ Verification Commands

| Command | Purpose |
| --- | --- |
| `ip a` | View all interfaces in the namespace (shows `docker0`, physical cards, and host-side `veth` cables) |
| `ip route` | View the IP routing table (shows which subnet is attached to which interface) |
| `ip neigh` | View the ARP table (shows MAC addresses learned by the kernel per interface) |
| `docker inspect <name> \| grep IPAddress` | Get the IP address of a running container |


# 🌍 How does a Request Travel?

### Assumptions:

- Container IP: `172.17.0.2`
- Host IP: `192.168.1.48`
- Port mapped: `8080:80`

---

### 🧭 Scenario 1: Host to Container
When the host tries to reach a process running on the container via its internal IP and port (e.g., `172.17.0.2:3000`):

```
Host Process
   ↓
Kernel Routing Table (finds 172.17.x.x route)
   ↓
docker0 Bridge
   ↓
Kernel ARP Table (finds MAC for 172.17.0.2)
   ↓
veth Cable
   ↓
Container eth0
   ↓
Container Kernel (Socket Lookup for port 3000)
   ↓
Container Process (listening on 0.0.0.0:3000)
```
> **Note:** If the process inside the container is only bound to `127.0.0.1:3000` (localhost), the Container Kernel will reject the packet! It must be bound to `0.0.0.0` to receive external traffic from the `eth0` interface.

---

### 🧭 Scenario 2: External Device to Container
When another device on your home Wi-Fi tries to reach the mapped port on the host (`192.168.1.48:8080`):

```
External Device
   ↓
Host NIC (wlp3s0)
   ↓
Kernel iptables (DNAT rewrites to 172.17.0.2:80)
   ↓
Kernel Routing Table
   ↓
docker0 Bridge
   ↓
Kernel ARP Table
   ↓
veth Cable
   ↓
Container eth0
   ↓
Container Kernel (Socket Lookup for port 80)
   ↓
Container Process (listening on 0.0.0.0:80)
```

---

## ❓ Frequently Asked Questions (Deep Dive)

**Q: Why can't the host access the container via `localhost`?**
**A:** Because of Network Namespaces. The host and the container have completely separate loopback (`127.0.0.1`) interfaces. The container's `localhost` is entirely invisible to the host.

**Q: When reaching a container, does the system use Routing or ARP first?**
**A:** **Routing happens first.** The OS must look at the IP Routing Table to figure out *which interface* (which bridge) the packet needs to exit from. Only after it selects the interface does it use the **ARP Table** to find the specific MAC address on that wire.

**Q: If the destination is on the same local subnet, why does the OS still need a routing entry?**
**A:** A machine can have multiple network cards (Wi-Fi, Ethernet, Docker bridges). The OS doesn't magically know which card is plugged into which subnet. The routing table maps the subnet (e.g., `192.168.1.0/24`) to the specific physical or virtual card (e.g., `wlp3s0`).

**Q: If I ping `8.8.8.8`, does `8.8.8.8` get cached in my ARP table?**
**A:** **No.** ARP only works on the local network. For external internet traffic, the OS uses the "Default Route", which points to your home router. The machine ARPs for the *router's* MAC address and sends the packet there.

**Q: Where exactly is the ARP table stored? Is it on the network card?**
**A:** It is stored in the **machine's RAM**, maintained by the **Linux Kernel**. The network card is just a dumb radio/transmitter. Crucially, the Kernel tags every single entry in that table with the specific interface it belongs to.
