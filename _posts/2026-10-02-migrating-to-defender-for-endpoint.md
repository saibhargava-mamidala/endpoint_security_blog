---
title: "Migrating to Defender for Endpoint: removing third-party antivirus safely"
tags: [defender-for-endpoint, antivirus, migration]
series: defender
part: 1
---
Replacing an antivirus or EDR product sounds simple: install the new agent, remove the old one. In practice, the risky part is the gap between the two. Devices can end up with two active security products fighting each other, or with none protecting them at all. This post walks through the order I follow to move from a third-party agent to Microsoft Defender for Endpoint (MDE) without leaving either gap open.

The idea in one sentence: **run both side by side first, prove Defender works, and only then remove the old agent.**

## Step 1: Inventory what you have

Before touching any device, write down:

- Which security products are installed, and which versions
- Operating systems in scope: Windows clients, Windows Server, and any older server versions
- How the old product is managed and who owns its console
- Whether the old agent is protected by an uninstall password or tamper protection
- How you can deploy software today (Intune, Configuration Manager, scripts)

The uninstall password is the item people forget most often. Without it, the removal step stops dead in the middle of your change window.

## Step 2: Get licences and portal access ready

Make sure the right users have an MDE licence (Plan 1 or Plan 2, directly or through a Microsoft 365 bundle), and that the admins doing the migration can sign in to the Microsoft Defender portal with suitable roles. Create device groups for your pilot and for each later wave.

## Step 3: Set up exclusions in both directions

Two security products on one machine can slow each other down by scanning each other's files. Before onboarding, add Defender's processes and folders to the old product's exclusion list. Each vendor documents its own steps, so use theirs. Microsoft's guide covers the Defender side of the story: [Migrate to Defender for Endpoint from non-Microsoft protection](https://learn.microsoft.com/defender-endpoint/switch-to-mde-overview).

## Step 4: Build your Defender policies first

Configure the settings you want before devices switch over, not after. A typical starting point:

- Antivirus policy with cloud-delivered protection turned on
- Tamper protection enabled
- Attack surface reduction rules in **audit** mode, not block, so you can see the impact before enforcing anything
- Update settings so security intelligence stays current

[Add your own tip here: for example, the baseline you use or a setting that caused trouble in a past project.]

## Step 5: Understand passive mode

This is where clients and servers differ.

- **Windows clients:** once a device is onboarded and a third-party antivirus is registered with Windows, Defender Antivirus switches to passive mode on its own. When the old product is removed, it moves back to active mode automatically.
- **Windows Server:** Defender Antivirus doesn't step aside automatically, so you set passive mode yourself. On servers this is usually done with the `ForcePassiveMode` registry value under `HKLM\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection`. Check Microsoft Learn for the exact steps for your server version before applying it.

## Step 6: Onboard a pilot group and verify

Start with a small, varied pilot: a few clients, a few servers, and at least one of the awkward devices everyone has. After onboarding, check three things on each machine:

```powershell
Get-Service Sense
Get-MpComputerStatus | Select-Object AMRunningMode, AMServiceEnabled, RealTimeProtectionEnabled
```

You want the `Sense` service running and `AMRunningMode` showing passive mode while the old product is still installed. Then confirm the device appears in the Microsoft Defender portal and is reporting.

## Step 7: Uninstall the third-party agent in waves

Only after the pilot looks healthy:

1. Disable or unlock the old product's tamper protection in its own console.
2. Deploy the vendor's supported uninstaller through your normal tooling, one ring at a time.
3. Reboot if the vendor requires it.
4. Watch for leftovers such as services, drivers, or Windows Security Center registrations.

Pause between waves long enough to catch problems. A failed uninstall on 20 devices is a small ticket; on 2,000 it's an incident.

## Step 8: Confirm Defender took over

After the removal, check again:

```powershell
Get-MpComputerStatus | Select-Object AMRunningMode, AMProductVersion, AntivirusSignatureLastUpdated
```

`AMRunningMode` should now show normal (active) mode, and the signature date should be recent. Windows Security should list Microsoft Defender Antivirus as the active product. A safe way to test detection is the harmless EICAR test file. If a device stays in passive mode after the old agent is gone, the usual cause is a leftover registration from the old product, so clean that up before moving on.

## Mistakes worth avoiding

- Removing the old agent before checking that Defender is actually running
- Forgetting servers, which need manual passive mode
- Enabling attack surface reduction rules in block mode on day one
- Skipping the pilot because the change window is short
- Not having the uninstall password or old console access ready

[Add one real lesson from your own migrations here. A short, specific story is the most useful part of any post.]

## What's next in this series

This is part 1. Next I'll move on to Microsoft Defender for Identity, then Defender for Office 365, Defender for Cloud Apps, and vulnerability management, with each post linking to the next.

*This is an independent blog and is not affiliated with or endorsed by Microsoft. Always check the current Microsoft Learn documentation for exact settings.*
