# Azure Secure Two-Tier Network Lab

## What is this project?

> ✏️ In 2-3 sentences, explain what you built in plain English.
> Example starter: "In this project I built a..."
> Think about: What are the two VMs? What is the point of keeping one hidden?




---

## The Architecture

> ✏️ Describe the layout in your own words — what is web-vm, what is app-vm, how do they relate?
> You can keep the diagram below — just explain it in your own words above it.

```
🌍 Internet
      |
      ↓
🖥️ web-vm (web-subnet) — public facing, has public IP
      |
      ↓ internal only
🖥️ app-vm (app-subnet) — private, no public IP
```




---

## What I Used

> ✏️ Just list the Azure resources you created. You already know these — VNet, subnets, NSGs, VMs etc.

| Resource | Name | What it does |
|---|---|---|
| | | |
| | | |
| | | |




---

## Security Rules

> ✏️ In your own words, explain what each NSG does. Think of the bouncer analogy — what is each one allowing and blocking?

### web-nsg
> ✏️ What ports did you open and why?

### app-nsg
> ✏️ What is the rule that makes the vault secure? Where does traffic have to come from?




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
