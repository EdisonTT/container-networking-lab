# 🧪 Home Server Lab

A hands-on lab to explore and understand **Docker, Linux networking, and container internals** through real experiments.

This is not a tutorial repo, it is a **learning journal + experimental lab** focused on understanding how things work under the hood.

---

## 🎯 Objective

To deeply understand:

- How containers actually work (not just how to run them)
- Linux networking fundamentals behind Docker
- Kernel-level concepts like namespaces, veth, and iptables
- Port binding, NAT, and container communication
- System behavior through real experiments

---

## 🧠 What This Repo Covers

- Docker fundamentals
- Container lifecycle
- Network namespaces
- veth pairs (virtual ethernet)
- Docker bridge network
- Port mapping & NAT (iptables)
- Binding (`0.0.0.0` vs `localhost`)
- Container isolation vs communication
- Host vs container networking behavior

---

## 🧪 Approach

Each concept is learned through **experiments**, not just theory.

Every experiment includes:

- 📘 Theory
- 🔬 Procedure
- 📊 Observations
- ✅ Results
- 🧾 Key findings

---
## 📂 Repo Structure

```
│
├── README.md
├── server-setup/
├── images/

├── experiments/
├── notes/
└── diagrams/
├── veth.png
├── docker-bridge.png
```

