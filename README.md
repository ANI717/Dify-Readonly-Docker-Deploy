# Dify Readonly Docker Deploy

Secure and production-ready deployment of **Dify AI components** inside **read-only, hardened Docker containers**.  
This repository demonstrates how to turn Dify services into **immutable, locked-down, and tamper-resistant containers** for enterprise and air-gapped environments.

> **Goal:** Prevent runtime file modifications, reduce attack surface, and guarantee predictable execution using minimal-privilege + readonly FS models.

---

## 🔐 Features

- ✅ Read-only root filesystem (`--read-only`)
- ✅ Strict filesystem isolation (`tmpfs`, minimal writable paths)
- ✅ Rootless / non-root container execution
- ✅ Hardened Dockerfiles for production security
- ✅ Compatible with offline / air-gapped setups
- ✅ Modular deployment of individual Dify components
- ✅ Optional Kubernetes manifests (WIP)

---

## 📦 Included Components

| Component | Status | Notes |
|----------|--------|------|
| Dify API | ✅ Ready | Hardened Dockerfile + readonly runtime |
| Dify Worker | ✅ Ready | Queue + tasks execution locked down |
| Vector DB / Redis | ⚙️ Configurable | External state only |
| Dify Web UI | ✅ Ready | Static asset-optimized |
| Plugin Daemon | ✅ Ready | With FS restrictions |

---

## 🧠 Why Read-Only Containers?

Immutable containers provide:

- Strong security posture
- Zero drift / reproducible infra
- Reduced supply-chain exposure
- Compliant deployment for regulated environments
- Perfect fit for **MLOps**, **LLM Ops**, and **enterprise AI platforms**

If a service doesn't need to write — it shouldn't.
