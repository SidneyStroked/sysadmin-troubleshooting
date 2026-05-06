# RDP File Security Warnings (2024–2026) – Explanation and Options

## Overview

Recent Windows updates introduced stricter security behavior when opening `.rdp` files.  
Users may now see warnings such as:

- “Unknown publisher”
- “This RDP file is from an untrusted source”
- Additional prompts before connection

This change is **not related to the server’s TLS certificate** (e.g. Let’s Encrypt).  
Even with a valid and trusted server certificate, warnings may still appear.

---

## Root Cause

The issue is caused by how Windows now treats `.rdp` files:

- `.rdp` files are considered **potentially unsafe launch configurations**
- They can be used in phishing or lateral movement attacks
- Windows now requires stronger trust validation (code signing)

### Important distinction

| Component | Purpose |
|----------|--------|
| TLS certificate (e.g. Let’s Encrypt) | Secures the RDP connection |
| `.rdp` file | Launches/configures the connection |

The warning is triggered by the **`.rdp` file**, not the TLS layer.

---

## Why Microsoft Changed This

This behavior was introduced due to real-world attacks abusing `.rdp` files.

### Example (Google / Mandiant research)

Attackers distributed malicious `.rdp` files that:

- Automatically connected victims to attacker-controlled servers
- Accessed clipboard, drives, or user sessions
- Looked legitimate (sometimes even signed)

Reference:
https://cloud.google.com/blog/topics/threat-intelligence/windows-rogue-remote-desktop-protocol

---

## Official Microsoft Documentation

Microsoft explains the security model and warnings here:

- https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/remotepc/understanding-security-warnings  
- https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/clients/remote-desktop-file-signing  

Key takeaway:
> Untrusted or unsigned `.rdp` files will trigger warnings by design.

---

## Available Solutions

### 1. Do not use `.rdp` files (Recommended)

Users connect manually:

mstsc → server.ppc.monster


**Pros**
- No warnings
- No additional infrastructure
- Fully compliant with Microsoft security model

**Cons**
- Slightly less convenient for users

---

### 2. Sign `.rdp` files with a Code Signing Certificate

Use a certificate from a trusted Certificate Authority (CA), e.g.:

- DigiCert
- Sectigo
- GlobalSign

Then sign:

rdpsign.exe /sha1 <thumbprint> file.rdp


**Pros**
- No warnings
- Enterprise-grade solution

**Cons**
- Paid (typically $100–$500/year or more)
- Certificate lifecycle management required
- Requires secure key storage (USB token / HSM)

---

### 3. Use internal CA (Active Directory Certificate Services)

- Create internal code signing certificate
- Distribute root CA to all clients (GPO)

**Pros**
- No external cost
- Works in controlled environments

**Cons**
- Not suitable for external users/customers
- Requires domain infrastructure

---

### 4. User acceptance (not recommended)

Users manually accept warnings:

- Click “Connect”
- Optionally store trust decision

**Pros**
- No cost

**Cons**
- Poor UX
- Security risk
- Not scalable

---

### 5. Policy / Registry overrides (not recommended)

Warnings can be reduced or disabled via GPO or registry.

**Pros**
- Quick workaround

**Cons**
- Weakens security posture
- Not recommended by Microsoft
- May be reverted by future updates

---

## What Does NOT Work

### Let’s Encrypt for signing `.rdp`

Let’s Encrypt certificates:
- Are issued for **TLS (Server Authentication)**
- Do **not** include **Code Signing** usage

Therefore:
- Cannot be used with `rdpsign`
- Will not remove warnings

---

## Summary

- The issue is caused by **new Windows security behavior**
- It is **not a misconfiguration**
- It is **intentional and security-driven**

### Recommended approach

- Avoid `.rdp` files for external users
- Use direct connection (`mstsc`)  
- Consider code signing only if necessary

---

## Final Note

Microsoft’s changes reflect a broader trend:

> Launchable configuration files (like `.rdp`) are treated as executable content and must be trusted accordingly.

Attempting to bypass this without proper signing introduces security risks and is discouraged.


