---
title: "Deploying Defender for Identity: The Latest v3.x Approach and the Classic v2.x Scenario"
tags: [Defender-for-Identity, Active-Directory, Domain-Controllers, Identity-Security, Infrastructure]
series: defender
part: 2
image: /assets/img/social-part2.png
---
![Series](https://img.shields.io/badge/Series-Microsoft%20Defender%20XDR-0078D4?style=flat-square&logo=microsoft)
![Part](https://img.shields.io/badge/Part-2%20of%205-0078D4?style=flat-square)
[![LinkedIn](https://img.shields.io/badge/Connect-LinkedIn-0A66C2?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/saibhargavamamidala/)

In [Part 1](/endpoint_security_blog/migrating-to-defender-for-endpoint/) we moved endpoints to Defender for Endpoint. Attackers rarely stop at the endpoint, though. Sooner or later they go after identity, and in most organisations identity still runs through Active Directory.

Microsoft Defender for Identity (MDI) watches your domain controllers for the signals of that attack path. It only sees what you cover, so the deployment matters. The way to deploy it has changed a lot recently, so this post covers both: the **latest v3.x approach** and the **classic v2.x scenario** you will still meet in existing environments.

> **The golden rule of MDI:** every domain controller needs a sensor. A domain controller without one is a blind spot attackers can use.

---

## 🧭 Which path are you on?

| Your situation | Path |
|---|---|
| New deployment, domain controllers on Windows Server 2019 or later | **Path A:** sensor v3.x |
| Existing environment with v2.x sensors | **Path B:** classic v2.x, then plan the migration to v3.x |
| Older domain controllers, or features that still need v2.x | **Path B:** classic v2.x, if your tenant still offers it |
| AD FS, AD CS, or Entra Connect servers that are not domain controllers | v3.x (preview at the time of writing) or v2.x |

> **📝 Latest console update:** in the Defender portal, many tenants no longer see the option to download the v2.x sensor. In practice that leaves two routes: **existing v2.x sensors are migrated to v3.x**, and **new domain controllers are onboarded directly to v3.x**. Microsoft rolls changes out gradually, so what you see can differ between tenants. Check the Sensor management page in your own portal before you plan.

Microsoft's release notes change often, so check the [What's new page](https://learn.microsoft.com/defender-for-identity/whats-new) before you plan. Preview features can change before they reach general availability.

---

## 🚀 Path A: the latest approach (sensor v3.x)

### What changed

- **Activation without Defender for Endpoint:** you can now activate the v3.x sensor on eligible domain controllers without first onboarding them to Defender for Endpoint. This is generally available for domain controllers on Windows Server 2019 or later, for **new** v3.x deployments only. It does not cover migrating existing v2.x sensors.
- **Automatic activation and auditing:** in organisations with a licence that includes Defender for Identity, automatic v3.x activation and automatic Windows auditing are rolling out as defaults. New Defender for Endpoint customers get a workspace created automatically when an eligible identity server is found. Existing Defender for Identity customers see a notice in the portal first and can opt out.
- **More server roles:** v3.x supports domain controllers with any identity role, and eligible AD FS, AD CS, and Entra Connect servers that are not domain controllers are covered in preview. Automatic auditing also reaches those servers.
- **Higher limits:** up to 1,000 sensors per workspace.

📚 Reference: [Activate the Defender for Identity sensor](https://learn.microsoft.com/defender-for-identity/deploy/activate-sensor)

### Step 1: Discover and plan

List every domain controller (including read-only ones), their operating system and update level, and any AD FS, AD CS, or Entra Connect servers. Decide who owns each one and when you can change it.

### Step 2: Check the prerequisites

- A Microsoft Entra tenant with at least one Security administrator, and a licence that includes Defender for Identity
- Domain controllers on Windows Server 2019 or later with a current cumulative update
- No existing Defender for Identity sensor on the server
- Time synchronized within five minutes across servers and domain controllers
- Enough capacity, using Microsoft's sizing guidance
- Outbound HTTPS connectivity to the Defender for Identity service

Then run Microsoft's readiness script to find gaps early:

```powershell
# Download Test-MdiReadiness.ps1 from Microsoft's documentation, then:
.\Test-MdiReadiness.ps1
```

### Step 3: Choose how the sensor gets activated

You now have two routes:

- **With Defender for Endpoint:** if your servers are already onboarded as in Part 1, eligible identity servers can use that connection, including automatic activation where it applies to you.
- **Without Defender for Endpoint:** activate the sensor on eligible domain controllers directly from the Defender portal, with no endpoint onboarding first.

> **💡 Before you start:** open the **Advanced features** and **Sensor management** pages in the Defender portal and check the automatic activation setting. If you want to control when sensors appear, review it before the portal enables it for you.

### Step 4: Auditing

With v3.x, automatic Windows auditing applies the required auditing settings and corrects misconfigurations on existing sensors. RPC auditing is enabled automatically on recent sensor versions. Keep an eye on the health alerts, especially the one for RPC audit misconfiguration, because some advanced detections depend on it.

### Step 5: Pilot, then roll out

1. Activate two or three domain controllers across different sites as a pilot.
2. Watch the Sensor management page for health issues and check CPU and memory on those servers for a few days.
3. Roll out to the remaining domain controllers in waves.
4. Cover AD FS, AD CS, and Entra Connect servers once the domain controllers are healthy.

### Step 6: Verify coverage

Compare the sensors listed in the portal with your inventory from Step 1, and fix any health issue straight away. A sensor with a warning is still a partial blind spot.

---

## 🏛️ Path B: the classic scenario (sensor v2.x)

Many environments still run v2.x, and Microsoft continues to ship updates for it. If your portal no longer offers the v2.x download, use this path mainly to understand and migrate the sensors you already have, and put new domain controllers on v3.x. Where the download is still available, the classic deployment follows these steps.

### Step 1: Prerequisites

- A Microsoft Entra tenant with a Security administrator to create the workspace
- Windows Server 2016 or later on the server, with at least 2 cores, 6 GB of RAM, and 6 GB of free disk (10 GB recommended)
- Time synchronized within five minutes
- Outbound HTTPS from each sensor to the Defender for Identity cloud service (`*.atp.azure.com`), directly or through a proxy
- Name resolution and directory ports open as documented in Microsoft's prerequisites

### Step 2: Directory services account

The v2.x sensor needs a directory services account to read from Active Directory. A group managed service account (gMSA) is the recommended choice, with every sensor server allowed to retrieve its password. Create it before installing.

### Step 3: Auditing

Configure the Windows audit policies and any required event settings on the domain controllers before the sensors go in. Use a dedicated policy rather than editing the built-in Domain Controllers policy.

### Step 4: Install

1. Create the workspace in the Defender portal and download the sensor installer, if your tenant still shows that option.
2. Install it on a pilot domain controller during a maintenance window, using the workspace access key.
3. For many servers, use the silent installation through your deployment tooling.
4. Confirm the sensor service is running and the sensor appears healthy in the portal.

### Step 5: Plan the move to v3.x

Microsoft now supports migrating v2.x sensors to v3.x from the Defender portal. Eligible servers show as ready for migration, the v2.x sensor keeps running until v3.x is ready, and the migration takes up to about 20 minutes. Windows Server 2025 domain controllers can migrate too. Remove any gMSA that was configured only for response actions on v2.x, because leaving it in place stops response actions from working on v3.x.

[Add your own tip here: for example, your pilot approach or a migration problem you hit.]

---

## 🚨 Common pitfalls

- ❌ **Leaving domain controllers uncovered,** especially read-only DCs and branch sites
- ❌ **Mixing sensor versions on one server** instead of using the supported migration
- ❌ **Planning a fresh v2.x install** when the v2.x download no longer appears in your portal. Use v3.x for new domain controllers
- ❌ **Treating v3.x activation without Defender for Endpoint as a migration tool,** when it only covers new deployments
- ❌ **Ignoring time sync,** which breaks registration and detections
- ❌ **Missing the automatic activation notice,** then being surprised when sensors appear
- ❌ **Skipping health alerts,** especially audit-related ones
- ❌ **Installing without a change window** on production domain controllers

[Add one real lesson from your own MDI deployments here.]

---

## ⏭️ Next in this series

- Part 1: Migrating to Defender for Endpoint
- Part 2: Deploying Microsoft Defender for Identity (this post)
- Part 3: Hardening Microsoft Defender for Office 365 (MDO) mail flow
- Part 4: Configuring Defender for Cloud Apps (MDCA) and SaaS governance
- Part 5: Operationalizing Microsoft Defender Vulnerability Management

*Disclaimer: this is an independent technical blog and is not affiliated with or endorsed by Microsoft. Always check the current Microsoft Learn documentation for sensor versions, prerequisites, and portal settings.*
