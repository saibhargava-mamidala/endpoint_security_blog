---
title: "Deploying Defender for Identity: Sensor v3.x for New Domain Controllers and Migrating from v2.x"
tags: [Defender-for-Identity, Active-Directory, Domain-Controllers, Identity-Security, Infrastructure]
series: defender
part: 2
image: /assets/img/social-part2.png
---
![Series](https://img.shields.io/badge/Series-Microsoft%20Defender%20XDR-0078D4?style=flat-square&logo=microsoft)
![Part](https://img.shields.io/badge/Part-2%20of%205-0078D4?style=flat-square)
[![LinkedIn](https://img.shields.io/badge/Connect-LinkedIn-0A66C2?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/saibhargavamamidala/)

In [Part 1](/endpoint_security_blog/migrating-to-defender-for-endpoint/) we moved endpoints to Defender for Endpoint. Attackers rarely stop at the endpoint, though. Sooner or later they go after identity, and in most organisations identity still runs through Active Directory.

Microsoft Defender for Identity (MDI) watches your domain controllers for the signals of that attack path. It only sees what you cover, so the deployment matters, and the way to deploy it has changed. This post covers the current approach with the v3.x sensor, how to migrate existing v2.x sensors, and a short look at the classic v2.x scenario so you recognise it in older environments.

## 🌟 Why Defender for Identity deserves a place in your stack

Most breaches don't end at the first compromised device. They end when an attacker gets hold of an identity and moves through Active Directory with it. That is the gap Defender for Identity closes.

- 🔍 **It sees the attacks that endpoint tools can miss.** It detects identity-based techniques such as DCSync, Kerberoasting, pass-the-ticket, and golden ticket activity, plus suspicious NTLM authentication, delegation changes, and Active Directory Certificate Services enumeration.
- 🧭 **It shows you where you are weak before an attacker does.** Identity posture assessments flag risks such as stale accounts, service accounts sitting in privileged groups, and passwords exposed in leaks.
- 🎯 **It tells you which identities matter most.** An identity risk score from 0 to 100 combines how likely an identity is to be compromised with how much damage it could do.
- ⚡ **It lets you act, not just watch.** From the portal you can run response actions such as disabling an account or resetting a password.
- 🔗 **It connects the whole story.** Identity signals join your endpoint, email, and cloud alerts in Microsoft Defender XDR, so one incident shows the full attack path from a phished user to a domain controller.

> **The big picture:** Endpoint protection (Part 1) tells you what happened on a device. Defender for Identity tells you who the attacker pretended to be, and where they tried to go next. Together they turn separate alerts into one investigation.

*Some capabilities depend on your licence and on Microsoft's gradual rollout, and a few are still in preview, so check what your tenant offers.*

> **The golden rule of MDI:** Every domain controller needs a sensor. A domain controller without one is a blind spot attackers can use.

---

## 📝 What changed in the console

In the Defender portal, many tenants no longer see the option to download the v2.x sensor. In practice that leaves two routes:

- **Existing v2.x sensors are migrated to v3.x**
- **New domain controllers are onboarded directly to v3.x**

Microsoft rolls changes out gradually, so what you see can differ between tenants. Check the **Sensor management** page in your own portal before you plan, and read the [What's new page](https://learn.microsoft.com/defender-for-identity/whats-new), because this area changes quickly and preview features can change before general availability.

---

## 🧭 Which route applies to you?

| Your starting point | What to do |
|---|---|
| New domain controllers on Windows Server 2019 or later | Activate the v3.x sensor directly |
| Existing v2.x sensors | Migrate them to v3.x from the portal |
| Domain controllers on older Windows Server versions | Check Microsoft's current guidance, as they may not be eligible for v3.x yet |
| AD FS, AD CS, or Entra Connect servers that are not domain controllers | v3.x, which is in preview at the time of writing |

---

## 🚀 Deploying the v3.x sensor

### What's new with v3.x

- **Activation without Defender for Endpoint:** You can activate the v3.x sensor on eligible domain controllers without onboarding them to Defender for Endpoint first. This is generally available for domain controllers on Windows Server 2019 or later, for **new** deployments only. It does not cover migrating existing v2.x sensors.
- **Automatic activation and auditing:** In organisations with a licence that includes Defender for Identity, automatic v3.x activation and automatic Windows auditing are rolling out as defaults. Existing customers see a notice in the portal first and can opt out.
- **More server roles:** v3.x supports domain controllers with any identity role, and eligible AD FS, AD CS, and Entra Connect servers that are not domain controllers are covered in preview, including automatic auditing.
- **Higher limits:** up to 1,000 sensors per workspace.

📚 Reference: [Activate the Defender for Identity sensor](https://learn.microsoft.com/defender-for-identity/deploy/activate-sensor)

### Step 1: Discover and plan

List every domain controller (including read-only ones), its operating system and update level, and any AD FS, AD CS, or Entra Connect servers. Decide who owns each one and when you can change it.

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

- **With Defender for Endpoint:** if your servers are already onboarded as in Part 1, eligible identity servers can use that connection, including automatic activation where it applies to you.
- **Without Defender for Endpoint:** activate the sensor on eligible domain controllers directly from the Defender portal, with no endpoint onboarding first.

> **💡 Before you start:** open the **Advanced features** and **Sensor management** pages and check the automatic activation setting. If you want to control when sensors appear, review it before the portal enables it for you.

### Step 4: Auditing

With v3.x, automatic Windows auditing applies the required settings and corrects misconfigurations on existing sensors. RPC auditing is enabled automatically on recent sensor versions. Watch the health alerts, especially the one for RPC audit misconfiguration, because some advanced detections depend on it.

### Step 5: Pilot, then roll out

1. Activate two or three domain controllers across different sites as a pilot.
2. Watch the Sensor management page for health issues, and check CPU and memory on those servers for a few days.
3. Roll out to the remaining domain controllers in waves.
4. Cover AD FS, AD CS, and Entra Connect servers once the domain controllers are healthy.

### Step 6: Verify coverage

Compare the sensors listed in the portal with your Step 1 inventory, and fix any health issue straight away. A sensor with a warning is still a partial blind spot.

---

## 🔄 Migrating existing v2.x sensors to v3.x

If you already have v2.x sensors, the supported route is migration from the Defender portal:

1. Open **Sensor management** and look for servers marked as **Ready for migration**.
2. For a server marked **Not ready for migration**, hover over the status to see which prerequisite is missing, fix it, and check again.
3. Migrate a pilot group first. The v2.x sensor keeps running until the v3.x sensor is ready, and a migration takes up to about 20 minutes.
4. If your v2.x setup had a group managed service account configured as the **action account** for response actions, remove that configuration in the Defender portal after migrating. Microsoft says leaving it enabled stops response actions, including attack disruption, from working on v3.x. If you never configured one, skip this step.
5. Check the health alerts after each wave, including audit-related ones, and confirm every domain controller still has exactly one healthy sensor.

Windows Server 2025 domain controllers can migrate too.

---

## 🏛️ Background: the classic v2.x scenario

You will still meet v2.x in older environments, so it helps to know what it needed and what v3.x simplifies.

| | Classic v2.x | v3.x |
|---|---|---|
| Installation | Download the installer and register it with a workspace access key | Activate the sensor from the Defender portal |
| Credentials | A directory services account, with a group managed service account recommended | No credentials entered in the portal |
| Auditing | Windows audit policies configured manually in advance | Automatic Windows auditing |
| Server role coverage | Domain controllers plus AD FS, AD CS, and Entra Connect servers | Domain controllers with any identity role, and eligible non-domain-controller servers in preview |

The practical lesson: v2.x deployments were mostly about preparing accounts, audit policies, and installers. With v3.x, most of that work moves into the portal, so your effort shifts to inventory, prerequisites, and verification.

>> MDI with v2.x is a real painful task for Security teams as the ownerships of DC's with be with Infra team and ends up most of the times with Mis-configurations leading to MDI sensor health issues. Kudos to Microsoft for Developing v3.x with controls that like Advance Auditing that can be managed directly from Security console.

---

## 🚨 Common pitfalls

- ❌ **Leaving domain controllers uncovered,** especially read-only DCs and branch sites
- ❌ **Planning a fresh v2.x install** when the v2.x download no longer appears in your portal. Use v3.x for new domain controllers
- ❌ **Treating v3.x activation without Defender for Endpoint as a migration tool,** when it only covers new deployments
- ❌ **Leaving the v2.x action account configured,** which stops response actions on v3.x
- ❌ **Ignoring time sync,** which breaks registration and detections
- ❌ **Missing the automatic activation notice,** then being surprised when sensors appear
- ❌ **Skipping health alerts,** especially audit-related ones
- ❌ **Installing without a change window** on production domain controllers

> **🛠️ Pro tip: if a migration fails or stalls**
>
> 1. **Don't remove anything by hand.** In Microsoft's migration flow, the v2.x sensor keeps running until the v3.x sensor is ready, so the domain controller should stay covered while you investigate. Check the sensor's status on the **Sensor management** page before you touch it.
> 2. **Read the reason.** For a server marked *Not ready for migration*, hover over the status to see which prerequisite is missing.
> 3. **Re-check the basics:** Windows Server 2019 or later, a current cumulative update, time synchronized within five minutes, and outbound connectivity to the Defender for Identity service. You can re-run `Test-MdiReadiness.ps1` to confirm.
> 4. **Look for leftovers.** A group managed service account from v2.x response actions, or a health alert about auditing, can get in the way.
> 5. **Retry on one domain controller first,** and only continue with the rest once that one is healthy.
> 6. **Still stuck?** Check Microsoft's [migration troubleshooting steps](https://learn.microsoft.com/defender-for-identity/deploy/migrate-to-sensor-v3#troubleshoot-not-ready-for-migration-status) and open a support case with the sensor details.
---

## ⏭️ Next in this series

- Part 1: Migrating to Defender for Endpoint
- Part 2: Deploying Microsoft Defender for Identity (this post)
- Part 3: Hardening Microsoft Defender for Office 365 (MDO) mail flow
- Part 4: Configuring Defender for Cloud Apps (MDCA) and SaaS governance
- Part 5: Operationalizing Microsoft Defender Vulnerability Management

*Disclaimer: this is an independent technical blog and is not affiliated with or endorsed by Microsoft. Always check the current Microsoft Learn documentation for sensor versions, prerequisites, and portal settings.*
