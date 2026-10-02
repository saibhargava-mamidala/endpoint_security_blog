---
title: "Migrating to Defender for Endpoint: Removing Third-Party Antivirus Safely"
tags: [Defender-for-Endpoint, Antivirus, EDR, Migration, Infrastructure, GreenFieldDeployment]
series: defender
part: 1
image: /assets/img/social-part1.png
---
![Series](https://img.shields.io/badge/Series-Microsoft%20Defender%20XDR-0078D4?style=flat-square&logo=microsoft)
![Part](https://img.shields.io/badge/Part-1%20of%205-0078D4?style=flat-square)
[![LinkedIn](https://img.shields.io/badge/Connect-LinkedIn-0A66C2?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/saibhargavamamidala/)

Replacing an Antivirus or EDR product sounds deceptively simple: *install the new agent, remove the old one.*

In practice, the biggest risk is the gap between the two. Devices can end up with two active security products competing for the same resources, which causes performance problems or instability. Or they end up unprotected if the legacy agent is removed before Defender for Endpoint (MDE) is fully working.

> **The golden rule of co-existence:** run both solutions side by side first, confirm Defender telemetry in the portal, and only then remove the legacy agent.

---

## 🗺️ Migration lifecycle overview

A migration follows three phases: **Prepare → Set up → Onboard and switch**.

| Phase | What happens |
|---|---|
| 1. Prepare | Inventory devices and uninstall passwords, get licences and portal roles, set mutual exclusions |
| 2. Set up | Deploy Defender baselines (ASR rules in Audit mode, Network Protection in Block mode, SmartScreen enabled), then handle passive mode (automatic on clients, manual on servers) |
| 3. Onboard and switch | Onboard a pilot group, verify telemetry, disable legacy tamper protection, uninstall the legacy agent, confirm Defender is active (ASR and Network Protection start enforcing now), run an EICAR test, then move ASR rules from Audit to Block |

---

## 📋 Step 1: Inventory and pre-flight checks

Before deploying a single policy or onboarding script, gather this baseline:

| Area | What to validate |
|---|---|
| Existing security stack | Vendor, agent version, installed modules (AV, EDR, firewall, web filter) |
| Operating systems in scope | Windows 10/11, Windows Server (2012 R2, 2016, 2019, 2022, 2025), Linux, macOS |
| Management consoles | Console URLs, admin access, who controls policy overrides |
| Tamper protection | Uninstall passwords or keys, and the central tamper protection status |
| Deployment mechanism | Intune, Microsoft Configuration Manager, Group Policy, or custom scripts |

> **⚠️ The number one blocker in field migrations:** the legacy agent's uninstall password. If you don't retrieve it, or disable protection centrally, before your change window, your uninstall scripts will fail on every endpoint.

---

## 🔑 Step 2: Licensing and portal role readiness

Verify that users and devices are covered by a suitable licence (Defender for Endpoint Plan 1 or Plan 2, or a Microsoft 365 bundle that includes it).

Then prepare your admin scope in the Microsoft Defender portal (security.microsoft.com):

- Assign appropriate roles, for example Security Administrator in Entra ID or the matching Defender roles
- Pre-create device groups for the pilot ring, later waves, and servers

---

## 🔄 Step 3: Configure mutual exclusions

Running two real-time agents together can slow machines down, because both try to scan the same file activity.

- **In the legacy product:** exclude Defender for Endpoint's processes and folders. Each vendor documents its own steps, and Microsoft publishes the list of what to exclude.
- **In Defender:** if needed during the co-existence window, add exclusions for the legacy agent's paths through Intune or Group Policy.

📚 Reference: [Migrate to Defender for Endpoint from non-Microsoft protection](https://learn.microsoft.com/defender-endpoint/switch-to-mde-overview) on Microsoft Learn.

---

## ⚙️ Step 4: Build Defender policies first

Configure your target baseline before onboarding devices, not after:

- 🛡️ **Antivirus policy:** turn on cloud-delivered protection and set update schedules
- 🔒 **Tamper protection:** enable it tenant-wide through Intune or the Defender portal
- ⚡ **Attack surface reduction (ASR) rules:** deploy in **Audit** mode first to monitor telemetry and evaluate false-positive impact before switching to Block mode
- 🌐 **Network Protection:** set to **Block** mode. It only works while Defender Antivirus is the active antivirus, so it starts protecting on day one for greenfield devices, or the day the legacy agent is removed on migrated ones. It blocks connections to malicious domains, IP addresses, and phishing URLs, and it enforces your custom indicators of compromise (IoCs) for URLs, domains, and IPs
- 🛡️ **Microsoft Defender SmartScreen:** enable it for Edge and Windows to protect users against malicious downloads and untrusted sites
- 🔄 **Security intelligence updates:** make sure devices can reach Microsoft update endpoints or your WSUS/MECM fallback

> **⏳ Important timing point:** ASR rules and Network Protection both need Microsoft Defender Antivirus running as the primary antivirus in **active** mode. They do not work while Defender Antivirus is passive next to the legacy product, so your custom IoC blocking in Defender is also inactive during co-existence. Keep the legacy product's own protection in place until cutover, and stage these policies before onboarding so they take effect the moment Defender becomes active. ASR audit data also only arrives **after** the legacy agent is removed.

> **💡 Architect tip:** keep ASR rules in Audit mode long enough to see your real business applications in the ASR reports. I recommend at least 14 days after Defender Antivirus becomes active. Network Protection is different: in my projects I enable it in **Block** mode from the first day Defender Antivirus is active, because blocking known-bad domains, IPs, and URLs, including your own IoCs, is essential from the first day of protection.

---

## 🔀 Step 5: Understand passive mode

Clients and servers behave differently:

```text
+-------------------+---------------------------+---------------------------+
| OS platform       | Legacy AV installed       | Legacy AV removed         |
+-------------------+---------------------------+---------------------------+
| Windows clients   | Automatic passive mode    | Automatic active mode     |
| Windows servers   | Manual passive mode (reg) | Remove the setting        |
+-------------------+---------------------------+---------------------------+
```

- **Windows 10/11:** once onboarded alongside a registered third-party antivirus, Defender Antivirus steps down to passive mode on its own. When the legacy product is removed, it returns to active mode automatically.
- **Windows Server:** Defender Antivirus does not step aside on its own, so you set passive mode yourself with the `ForcePassiveMode` registry value:

```powershell
# Set passive mode on Windows Server (verify the path and steps for your OS version on Microsoft Learn)
$key = "HKLM:\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection"
if (-not (Test-Path $key)) { New-Item -Path $key -Force | Out-Null }
New-ItemProperty -Path $key -Name "ForcePassiveMode" -Value 1 -PropertyType DWord -Force
```

When the legacy product is gone, remove this value or set it back to 0 so Defender Antivirus can become active.

---

## 🧪 Step 6: Onboard a pilot group and validate

Onboard a representative cross-section, for example 5 to 10 percent of the fleet, including IT workstations, regular users, and non-critical servers.

Run these checks on the pilot machines:

```powershell
# 1. Is the Defender for Endpoint sensor service running?
Get-Service -Name Sense | Select-Object Name, Status, StartType

# 2. Which mode is Defender Antivirus in?
Get-MpComputerStatus | Select-Object AMRunningMode, AMServiceEnabled, RealTimeProtectionEnabled
```

**Expected result:** `Sense` is Running and `AMRunningMode` shows Passive while the legacy product is still installed. Confirm the device shows as onboarded in security.microsoft.com.

---

## 📦 Step 7: Remove third-party agents in waves

Once the pilot looks healthy:

1. **Disable legacy tamper protection** in the vendor's own console.
2. **Run a silent uninstall** with the vendor's supported command through Intune, MECM, or scripts, one ring at a time.
3. **Handle reboots** if the vendor requires one to remove filter drivers.
4. **Pause between rings** and check for leftover services, registry keys, or WMI registrations.

---

## ✅ Step 8: Verify active protection

After the legacy agent is gone, confirm Defender has taken over:

```powershell
Get-MpComputerStatus | Select-Object AMRunningMode, AMProductVersion, AntivirusSignatureLastUpdated, RealTimeProtectionEnabled
```

You should see something like:

```text
AMRunningMode                 : Normal
RealTimeProtectionEnabled     : True
AntivirusSignatureLastUpdated : <a recent date>
```

> **🧪 Validation test:** download the harmless EICAR test file to confirm real-time detection and that the alert appears in the Defender portal.

If a device stays in passive mode after the old agent is gone, a leftover registration from the old product is the usual cause.

---

## 🚦 Step 9: Move ASR rules from Audit to Block

Now that Defender Antivirus is active, the ASR Audit policies start producing real data:

1. Wait through your audit period and review the ASR reports in the Defender portal.
2. Add exclusions or fix the business applications you find, rather than leaving rules off.
3. Move rules to Block one at a time, starting with the low-impact ones and the pilot group.
4. Keep a rollback plan so a blocked line-of-business app can be restored quickly.

---

## 🚨 Common migration pitfalls

- ❌ **Removing the legacy AV too early,** before confirming the Sense service reports to the cloud
- ❌ **Forgetting server passive mode,** which needs explicit configuration
- ❌ **Enforcing ASR rules on day one,** which can break business scripts
- ❌ **Expecting ASR rules or Network Protection to work during co-existence,** when Defender Antivirus is passive
- ❌ **Skipping the pilot** and rolling straight out to production
- ❌ **Missing uninstall keys** when launching automated uninstall jobs

---

## ⏭️ Next in this series

- Part 1: Migrating to Defender for Endpoint (this post)
- Part 2: Deploying Microsoft Defender for Identity (MDI) on domain controllers
- Part 3: Hardening Microsoft Defender for Office 365 (MDO) mail flow
- Part 4: Configuring Defender for Cloud Apps (MDCA) and SaaS governance
- Part 5: Operationalizing Microsoft Defender Vulnerability Management

*Disclaimer: this is an independent technical blog and is not affiliated with or endorsed by Microsoft. Always check the current Microsoft Learn documentation for portal settings and prerequisites.*
