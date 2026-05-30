# Azure Secure Two-Tier Network Project

**Author:** Awez  
**Date:** May 2026  
**Tools Used:** Microsoft Azure Portal, Windows Server 2022, PowerShell  
**Status:** ✅ Completed

---

## What Is This Project?

This project involved building a secure two-tier network inside Microsoft Azure using only the Azure Portal — no code required.

The goal was to create two virtual machines (VMs) that could talk to each other privately, but where only one of them was visible to the internet. The other one stays completely hidden — only reachable through the first.

This is a real-world architecture used by companies every day to protect their databases and backend systems from the public internet.

---

## The Architecture (Plain English)

Think of it like a building with two rooms:

- **Room 1 — Reception (web-vm):** Faces the street. The public can knock on the door. This is where admins connect in, and where a web server would eventually live.
- **Room 2 — The Vault (app-vm):** No street entrance. No public address. The only way in is through Reception. This is where sensitive data or backend services would live.

```
🌍 Internet
      |
      ↓ (RDP port 3389, HTTP port 80 only)
🐱 web-vm — Reception (public IP, web-subnet)
      |
      ↓ (internal traffic only, from 10.0.1.0/24)
😼 app-vm — The Vault (no public IP, app-subnet)
```

---

## What I Built

| Resource | Name | Purpose |
|---|---|---|
| Resource Group | project-rg | Container for all resources |
| Virtual Network | project-vnet | The building — 10.0.0.0/16 |
| Subnet 1 | web-subnet | Public-facing room — 10.0.1.0/24 |
| Subnet 2 | app-subnet | Private room — 10.0.2.0/24 |
| Network Security Group | web-nsg | Bouncer for Reception |
| Network Security Group | app-nsg | Bouncer for the Vault |
| Virtual Machine | web-vm | Windows Server, public IP, B2s |
| Virtual Machine | app-vm | Windows Server, no public IP, F1 |

---

## Security Rules Configured

### web-nsg (Reception Bouncer)
| Priority | Rule Name | Port | Action | Reason |
|---|---|---|---|---|
| 100 | Allow-RDP | 3389 | Allow | So admins can connect |
| 110 | Allow-HTTP | 80 | Allow | For future web server traffic |
| 65500 | DenyAllInBound | Any | Deny | Block everything else |

### app-nsg (Vault Bouncer)
| Priority | Rule Name | Source | Port | Action | Reason |
|---|---|---|---|---|---|
| 100 | Allow-From-WebSubnet | 10.0.1.0/24 | Any | Allow | Only Reception can enter |
| 200 | Deny-All-Inbound | Any | Any | Deny | Everyone else blocked |

---

## What I Tested and Proved

### ✅ Test 1 — RDP into web-vm from my laptop
Connected directly to web-vm using Remote Desktop. Worked because web-nsg allows port 3389 from the internet.

### ✅ Test 2 — Ping app-vm from inside web-vm
Ran `ping 10.0.2.4` from web-vm's PowerShell. Got 4 replies, 0% packet loss. Proved internal connectivity between the two VMs works.

### ✅ Test 3 — app-vm is invisible from the internet
app-vm has no public IP address. There is literally no way to connect to it directly from the internet — it doesn't exist publicly.

### ✅ Test 4 — RDP into app-vm through web-vm
Ran `mstsc /v:10.0.2.4` from inside web-vm. Successfully connected to app-vm through the internal network. The only route in is through Reception first.

---

## The Troubleshooting Journey

This project didn't go smoothly at first — and that's the most valuable part.

### Problem 1 — VM sizes showing as unavailable
**What happened:** When trying to deploy VMs, all B-series sizes (B1s, B1ms, B2s) showed as unavailable or hit quota limits.

**Why it happened:** Azure free trial accounts have very restrictive vCPU quotas. Even after upgrading to Pay As You Go, the quota didn't update immediately.

**How I fixed it:** Switched regions to West Europe where more sizes were available, and used B2s for web-vm and F1 for app-vm as alternatives. The project still worked perfectly — VM size doesn't affect networking architecture.

**What I learned:** Azure quotas are per-region and per-VM family. Free trial accounts are heavily restricted. The fix is to either request a quota increase or find an available region.

---

### Problem 2 — NSGs not appearing when trying to associate to subnets
**What happened:** When trying to attach the NSGs to the subnets, the VNet showed as "No available items" in the dropdown.

**Why it happened:** The NSGs were created in North Europe but the VNet had been moved to West Europe. Azure resources must be in the same region to work together at the subnet level.

**How I fixed it:** Deleted the old NSGs and recreated them in West Europe — the same region as the VNet. After that the association worked immediately.

**What I learned:** Region consistency is critical in Azure. NSGs, VNets, and VMs must all be in the same region. This is a real-world mistake that even experienced engineers make.

---

### Problem 3 — Ping not working between VMs
**What happened:** After setting up both VMs and connecting to web-vm, pinging app-vm returned no replies even though the NSG should allow it.

**Why it happened:** The Azure NSG was configured correctly, but Windows Server has its own built-in firewall inside the operating system. By default it blocks ICMP (ping) traffic — a separate layer of protection entirely.

**How I fixed it:** RDP'd into app-vm through web-vm, then ran this command in PowerShell as Administrator:
```
netsh advfirewall firewall add rule name="Allow ICMPv4" protocol=icmpv4:8,any dir=in action=allow
```
After that, ping worked immediately.

**What I learned:** This is Defence in Depth in practice. There are multiple independent layers of security — the Azure NSG (network layer) and the Windows Firewall (compute layer) are separate and both need to be configured. Bypassing one doesn't bypass the other.

---

## Key Concepts This Project Demonstrates

**Jump Box / Bastion Host**
web-vm acts as a jump box — the single controlled entry point into the private network. All admin access goes through it. app-vm is never directly exposed.

**Least Privilege**
app-nsg only allows traffic from web-subnet (10.0.1.0/24). Nothing else. No exceptions. This means even if someone got onto the VNet somehow, they still couldn't reach app-vm unless they came from the correct subnet.

**Defence in Depth**
Two separate firewalls protect app-vm — the Azure NSG at the network layer and Windows Firewall at the compute layer. Each is independent. Both must be configured.

**Network Segmentation**
Splitting the network into web-subnet and app-subnet means a breach in the public-facing tier doesn't automatically compromise the private tier. Each subnet has its own security boundary.

---

## Azure Concepts Covered (AZ-900 Relevant)

- Virtual Networks and Subnets
- Network Security Groups and inbound rules
- Virtual Machines and sizing
- Public vs Private IP addresses
- Resource Groups and region consistency
- Defence in Depth (multiple security layers)
- Least Privilege access (Zero Trust principle)
- Azure Advisor recommendations (appeared on web-vm after deployment)
- Cost Management (stopping VMs to preserve credits)

---

## What I Would Add Next

- **Install IIS** on web-vm to serve a real webpage on port 80
- **Replace the Jump Box with Azure Bastion** — Microsoft's built-in secure remote access service that doesn't require port 3389 to be open publicly
- **Add Azure Monitor alerts** to notify when CPU exceeds a threshold
- **Tag all resources** properly for cost tracking
- **Deploy using ARM templates** instead of the Portal for repeatability

---

## Cost Notes

Both VMs were stopped (deallocated) after testing to preserve Azure credits. Deallocated VMs do not consume compute credits. The VNet, NSGs, and other network resources are free.

Approximate cost during active use: ~£0.04/hour for both VMs combined.

---

*Project completed as part of self-directed Azure cloud engineering study alongside AZ-900 exam preparation.*
