# 🌌 Gatekeeper-Alpha

**Zero-Trust Perimeter & Mesh Ingress for the Homelab**

Gatekeeper-Alpha is a hardened, low-latency edge gateway engineered to provide a
secure, identity-based bridge between a trusted local network and the public
internet.

It replaces traditional port-forwarding with a **Zero-Trust mesh architecture**,
ensuring internal services remain invisible to the WAN while remaining securely
accessible to authorized devices anywhere.

Gatekeeper-Alpha serves as the **single ingress point** for the homelab.

---

## 🎯 Mission

Security through obscurity is a failure state.

Gatekeeper-Alpha enforces a deny-by-default perimeter model where:
- No services are exposed to the public internet
- All access is authenticated by cryptographic identity
- The internal LAN is never directly reachable from WAN

Only trusted, authenticated mesh peers are permitted access.

---

## 🧱 Architecture Overview

Gatekeeper-Alpha follows a **Defense-in-Depth** model. Each layer is minimal,
independent, and designed with a distinct failure mode.

### 1. Operating System — DietPi (Debian 12)

- Ultra-minimal footprint (<100MB RAM)
- Reduced attack surface
- Performance-focused configuration

**Volatility-first logging**
- `/var/log` mounted as `tmpfs`
- Prevents SD-card wear
- Avoids persistent forensic metadata retention

---

### 2. Mesh Layer — Tailscale (WireGuard)

- Identity-based peer authentication
- Encrypted point-to-point tunnels
- No reliance on exposed WAN ports

**Subnet routing**
- Gatekeeper-Alpha advertises the local LAN to the mesh:
