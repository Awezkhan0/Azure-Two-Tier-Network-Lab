# Azure Secure Two-Tier Network Lab

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
| 100 | Allow-From-WebSubnet | 10.0.1.0/24 | Any | Allow | Only Reception can enter |
| 200 | Deny-All-Inbound | Any | Any | Deny | Everyone else blocked |

---

## What I Tested

> ✏️ Describe each test in one sentence — what did you try, and did it work or fail?

- **Test 1 —** 
- **Test 2 —** 
- **Test 3 —** 
- **Test 4 —** 




---

## Problems I Hit and How I Fixed Them

> ✏️ This is the most important section — be honest, explain what went wrong in your own words.
> Don't worry about sounding perfect. "I kept getting quota errors because..." is great.

### Problem 1 — VM sizes unavailable
**What happened:**

**Why it happened:**

**How I fixed it:**

---

### Problem 2 — NSGs not showing when associating to subnets
**What happened:**

**Why it happened:**

**How I fixed it:**

---

### Problem 3 — Ping not working
**What happened:**

**Why it happened:**

**How I fixed it:**




---

## What I Learned

> ✏️ 3-5 bullet points in your own words. What do you now understand that you didn't before?
> Think about: regions, Defence in Depth, Jump Box, NSGs vs Windows Firewall...

- 
- 
- 
- 




---

## Screenshots

> ✏️ Add your screenshots to a /screenshots folder and reference them here like this:
> ![description](screenshots/filename.png)

**VNet Subnets with NSGs attached:**


**web-nsg rules:**


**app-nsg rules:**


**Both VMs running:**


**Ping success:**




---

## What I Would Add Next

> ✏️ What would you do to improve this project? Even one or two ideas shows you're thinking like an engineer.

- 
- 




---

*Part of my Azure cloud engineering learning journey alongside AZ-900 exam preparation.*
