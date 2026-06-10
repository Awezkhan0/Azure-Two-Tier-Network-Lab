# Azure Secure Network Lab — Jump Box Architecture
> A hands-on Azure networking lab demonstrating secure two-tier architecture, 
> jump box access, and Defence in Depth using NSGs and Windows Firewall.

## What is this project?
In this project I built a secure two-tier network using Azure Portal. I created a private layer that has no public IP and can only be accessed through a jump box without the use of Bastion. The purpose of this was to have a private VM that cannot be directly reached from the internet, simulating how a real backend server or database would be protected in a production environment.

## The Architecture
**Web-VM (web-subnet)**  —  Accessible via RDP (port 3389). Used as a security gateway and jump box to access the private backend server.


**App-VM (app-subnet)** — Private backend server with no public IP. The only route in is through web-vm on the internal network.


```
🌍 Internet → 🖥️ web-vm (web-subnet) → internal only → 🖥️ app-vm (app-subnet)
  (public)         has public IP                            no public IP
```
---

## What I Used

- **Resource Group** (project-rg)
  - **Virtual Network** (project-vnet) — 10.0.0.0/16
    - **Subnet** (web-subnet) — 10.0.1.0/24
      - **Virtual Machine** (web-vm) — Windows Server 2022, public IP, jump box
      - **NSG** (web-nsg) — allows RDP (3389) and HTTP (80)
    - **Subnet** (app-subnet) — 10.0.2.0/24
      - **Virtual Machine** (app-vm) — Windows Server 2022, no public IP, private backend
      - **NSG** (app-nsg) — allows traffic from web-subnet only

---

## Security Rules 

### web-nsg (For Public layer)
| Priority | Rule Name | Port | Action | Reason |
|---|---|---|---|---|
| 100 | Allow-RDP | 3389 | Allow | So admins can connect |
| 110 | Allow-HTTP | 80 | Allow | For future web server traffic |
| 65500 | DenyAllInBound | Any | Deny | Block everything else |

### app-nsg (For Private layer)
| Priority | Rule Name | Source | Port | Action | Reason |
|---|---|---|---|---|---|
| 100 | Allow-From-WebSubnet | 10.0.1.0/24 | Any | Allow | Only web-subnet can access this VM |
| 200 | Deny-All-Inbound | Any | Any | Deny | Everyone else blocked |

---

## What I Tested
- ☑ **Test 1 —** Connected directly to web-vm using Remote Desktop. Worked because web-nsg allows port 3389 from the internet.
- ☑ **Test 2** — Pinged app-vm's private IP from web-vm. Initially failed because Windows Server blocks ICMP by default. Fixed by enabling ICMP via PowerShell on app-vm: `netsh advfirewall firewall add rule name="Allow ICMPv4" protocol=icmpv4:8,any dir=in action=allow`
- ❌ **Test 3** — Tried to RDP directly into app-vm from my laptop. Failed — as expected. app-vm has no public IP so it cannot be reached from the internet.

- ☑ **Test 4** — RDP'd into app-vm through web-vm using `mstsc /v:10.0.2.4` from PowerShell. Worked because the connection came from web-subnet which app-nsg allows.




---

## Problems I Hit and How I Fixed Them

### Problem 1 — VM sizes unavailable
**What happened:** When trying to deploy VMs, all B-series sizes (B1s, B1ms, B2s) showed as unavailable or hit quota limits.

**Why it happened:** Azure free trial accounts have very restrictive vCPU quotas. Even after upgrading to Pay As You Go, the quota didn't update immediately.

**How I fixed it:** Switched regions to West Europe where more sizes were available, and used B2s for web-vm and F1 for app-vm as alternatives. The project still worked perfectly — VM size doesn't affect networking architecture.

**What I learned:** Azure quotas are per-region and per-VM family. Free trial accounts are heavily restricted. The fix is to either request a quota increase or find an available region.

---

### Problem 2 — NSGs not showing when associating to subnets
**What happened:** When trying to attach the NSGs to the subnets, the VNet showed as "No available items" in the dropdown.

**Why it happened:** The NSGs were created in North Europe but the VNet had been moved to West Europe. Azure resources must be in the same region to work together at the subnet level.

**How I fixed it:** Deleted the old NSGs and recreated them in West Europe — the same region as the VNet. After that the association worked immediately.

**What I learned:** Region consistency is critical in Azure. NSGs, VNets, and VMs must all be in the same region.

---

### Problem 3 — Ping not working
**What happened:** After setting up both VMs and connecting to web-vm, pinging app-vm returned no replies even though the NSG should allow it.

**Why it happened:** The Azure NSG was configured correctly, but Windows Server has its own built-in firewall inside the operating system. By default it blocks ICMP (ping) traffic — a separate layer of protection entirely.

**How I fixed it:** RDP'd into app-vm through web-vm, then ran this command in PowerShell as Administrator:
```
netsh advfirewall firewall add rule name="Allow ICMPv4" protocol=icmpv4:8,any dir=in action=allow
```
After that, ping worked immediately.

**What I learned:** This is Defence in Depth in practice. There are multiple independent layers of security — the Azure NSG (network layer) and the Windows Firewall (compute layer) are separate and both need to be configured. Bypassing one doesn't bypass the other.




---

## What I Learned
**Jump Box / Bastion Host**
web-vm acts as a jump box — the single controlled entry point into the private network. All admin access goes through it. app-vm is never directly exposed.

**Least Privilege**
app-nsg only allows traffic from web-subnet (10.0.1.0/24). Nothing else. No exceptions. This means even if someone got onto the VNet somehow, they still couldn't reach app-vm unless they came from the correct subnet.

**Defence in Depth**
Two separate firewalls protect app-vm — the Azure NSG at the network layer and Windows Firewall at the compute layer. Each is independent. Both must be configured.

**Network Segmentation**
Splitting the network into web-subnet and app-subnet means a breach in the public-facing tier doesn't automatically compromise the private tier. Each subnet has its own security boundary.



---

## Screenshots


**VNet Subnets with NSGs attached:**
![VNet Subnets](https://github.com/Awezkhan0/Azure-Two-Tier-Network-Lab/blob/main/Screenshots/01-vnet-subnets.png?raw=true)

**web-nsg Inbound Rules:**
![web-nsg rules](https://github.com/Awezkhan0/Azure-Two-Tier-Network-Lab/blob/main/Screenshots/02-web-nsg-rules.png?raw=true)

**app-nsg Inbound Rules:**
![app-nsg rules](https://github.com/Awezkhan0/Azure-Two-Tier-Network-Lab/blob/main/Screenshots/03-app-nsg-rules.png?raw=true)

**VM Status**
![VM Status](https://github.com/Awezkhan0/Azure-Two-Tier-Network-Lab/blob/main/Screenshots/04-vm-status.png?raw=true)

**Jump Box in Action — RDP into app-vm through web-vm:**
![Jump box](https://github.com/Awezkhan0/Azure-Two-Tier-Network-Lab/blob/main/Screenshots/05-jump-box.png?raw=true)

**Ping Success from web-vm to app-vm:**
![Ping success](https://github.com/Awezkhan0/Azure-Two-Tier-Network-Lab/blob/main/Screenshots/06-ping-success.png?raw=true)

---

## Phase 2 — Upgrading to Azure Bastion
In Phase 1 I deliberately built the jump box manually without Bastion to understand the underlying architecture. Having completed that, I identified that port 3389 being open publicly is a security risk. In Phase 2 I upgraded the lab to use Azure Bastion, removing all public RDP exposure.

## What Changed?
Web-vm no longer has a public IP address. Connection is now made through the browser via HTTPS (port 443) rather than downloading an RDP file over port 3389

## What I Tested?
- ☑ Connected to web-vm through Azure Bastion via the browser — no RDP file, no public IP
- ☑ From inside web-vm, RDP'd into app-vm using `mstsc /v:10.0.2.4` — proved connectivity still works through Bastion

## What I Learned?
Azure Bastion is more secure than a public RDP connection because it uses HTTPS (port 443) rather than RDP (port 3389). RDP is actively targeted by attackers scanning the internet. With Bastion, the VM has no public IP at all

## Screenshots
![Phase 2 Bastion](https://github.com/Awezkhan0/Azure-Two-Tier-Network-Lab/blob/main/Screenshots/Phase%202%20Bastion/01-vnet-bastionsubnet.png?raw=true)

![Create Bastion](https://github.com/Awezkhan0/Azure-Two-Tier-Network-Lab/blob/main/Screenshots/Phase%202%20Bastion/02-vnet-createbastion.png?raw=true)

![ipassociate](https://github.com/Awezkhan0/Azure-Two-Tier-Network-Lab/blob/main/Screenshots/Phase%202%20Bastion/03-webvm-ipdissociate.png?raw=true)

![connect bastian](https://github.com/Awezkhan0/Azure-Two-Tier-Network-Lab/blob/main/Screenshots/Phase%202%20Bastion/04-webvm-connectviabastion.png?raw=true)

![webvm bastian connected](https://github.com/Awezkhan0/Azure-Two-Tier-Network-Lab/blob/main/Screenshots/Phase%202%20Bastion/05-webvm-connected.png?raw=true)

---

*Part of my Azure cloud engineering learning journey alongside AZ-104 exam preparation.*

