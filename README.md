# 🧪 Home Server Lab

A hands-on lab to explore and understand **Docker, Linux networking, and container internals** through real experiments.

This is not a tutorial repo, it is a **learning journal + experimental lab** focused on understanding how things work under the hood.

---

## 🎯 Objective

To build a strong mental model of container networking to **design better systems**.

We aim to understand:

- How containers practically connect and isolate (Namespaces, Bridges, veth)
- The actual wiring behind Port Binding and NAT
- Network fundamentals required to architect robust microservices
- System behavior and troubleshooting through hands-on experiments

*(Note: The goal is not to get lost in deep kernel hacking, but to master the underlying network abstractions so we can confidently design and debug complex system architectures.)*

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

```text
.
├── README.md
├── setups/        # Installation and server configuration guides
├── notes/         # Theoretical concepts and deep dives (Docker, Linux)
├── cheatsheets/   # Quick reference commands and abbreviations
├── experiments/   # Step-by-step practical lab procedures
└── images/        # Proof, results, and screenshots
```
