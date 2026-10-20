---
title: "Hardening Defender for Office 365: A Practical Rollout Order from Email Authentication to Attack Simulation"
tags: [Defender-for-Office-365, Email-Security, Phishing, Exchange-Online, Infrastructure]
series: Defender
part: 3
image: /assets/img/social-part3.png
---
![Series](https://img.shields.io/badge/Series-Microsoft%20Defender%20XDR-0078D4?style=flat-square&logo=microsoft)
![Part](https://img.shields.io/badge/Part-3%20of%205-0078D4?style=flat-square)
[![LinkedIn](https://img.shields.io/badge/Connect-LinkedIn-0A66C2?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/saibhargavamamidala/)

In [Part 1](/endpoint_security_blog/migrating-to-defender-for-endpoint/) we covered endpoints, and in [Part 2](/endpoint_security_blog/deploying-defender-for-identity/) we covered Identity. Email is the third door attackers keep knocking on, and for most organisations it is still where the first click happens.

Microsoft Defender for Office 365 (MDO) adds protection on top of the built-in email security that comes with Exchange Online. The catch is that a lot of it is **not switched on by default**. This post walks through a practical order of work, so you end up with protection that is actually active instead of merely licensed.

> **The golden rule of MDO:** start with the Standard preset security policy for everyone, and make exceptions only where the business genuinely needs them.

---

## 🌟 Why MDO deserves a place in your stack

- 🔗 **Safe Links** checks URLs when a user clicks, not just when the message arrives
- 📎 **Safe Attachments** detonates suspicious files before they reach the inbox
- 🎭 **Impersonation protection** catches emails pretending to be your executives or your partners
- 🧑‍🏫 **Attack simulation training** (Plan 2) tests users with realistic phishing and trains them
- 🔎 **Investigation tools** show who received a message, who clicked, and what to clean up

The best part: Defender for Office 365 shares the same portal and incident view as the rest of Defender XDR, so an email threat connects to the endpoint and identity story from Parts 1 and 2.

---

## 🪜 Plan 1 vs Plan 2: what do you actually get?

Defender for Office 365 always includes Exchange Online Protection (EOP). Plan 1 expands on prevention and detection, and Plan 2 adds investigation and response, including automation.

| Capability | Plan 1 | Plan 2 |
|---|---|---|
| Safe Links, Safe Attachments, anti-phishing with impersonation protection | ✔ | ✔ |
| Seeing detections | Real-time detections | Threat Explorer |
| Automated investigation and response (AIR) | | ✔ |
| Campaigns and threat trackers | | ✔ |
| Attack simulation training | | ✔ |
| Custom user tags | | ✔ |

Plan 1 is included in Microsoft 365 Business Premium. Plan 2 is included in Office 365 E5, Microsoft 365 E5, and Microsoft 365 E5 Security. Everything is managed in the Microsoft Defender portal, because access to the old Security & Compliance Center ended in 2022.

> **📚 Good companions:** Microsoft's [Defender for Office 365 setup guide](https://setup.cloud.microsoft/defender/office-365-setup-guide) and the automated setup guide in the Microsoft 365 admin center walk through the same steps based on your own tenant.

---

## 🛡️ What is on by default, and what is not

This is the point many teams miss:

| Protection | Default state |
|---|---|
| Built-in protection (basic Safe Links and Safe Attachments) | On for all recipients |
| Default policies (anti-spam, anti-malware, anti-phishing) | On |
| **Standard** preset security policy | **Off until you turn it on** |
| **Strict** preset security policy | **Off until you turn it on** |
| User and domain impersonation protection | No entries until you configure them |

Doing nothing gives you a baseline. Doing the steps below gives you the recommended protection.

---

## 🗺️ Rollout at a glance

| Phase | What happens |
|---|---|
| 1. Foundations | Email authentication (SPF, DKIM, DMARC) for every domain |
| 2. Policies | Turn on Standard (and Strict for high-risk users), configure impersonation entries |
| 3. People | Roles, priority accounts, and user reporting |
| 4. Operations | Block and allow discipline, Teams, SharePoint and OneDrive coverage, ongoing review |

---

## 📋 Step 1: Email authentication

Set up SPF, DKIM, and DMARC, in that order, for **every** custom domain you use for mail. That includes parked domains and subdomains that never send email, because attackers spoof those too.

- Aim for a DMARC policy that quarantines or rejects, and set report destinations so you can see who is sending as you.
- If a third-party service changes inbound mail before it reaches Microsoft 365, add it as a trusted ARC sealer where it supports that, so legitimate messages don't fail authentication.

> **⚠️ Tip:** move DMARC from monitoring to enforcement gradually. Reports often reveal forgotten senders such as marketing tools and old applications.

---

## ⚙️ Step 2: Choose your policy strategy

You have four broad options:

| Option | Best for |
|---|---|
| Standard preset for everyone | Most organisations, and the recommended starting point |
| Standard for everyone, Strict for high-risk users | Executives, finance, and other likely targets |
| Presets plus custom policies | When some users need settings presets don't offer |
| Custom policies only | Special requirements, with regular review against the recommended baseline |

Microsoft recommends starting with the Standard preset for all users unless you have a compelling reason not to, and preset settings are updated automatically as threats change.

**Watch the order of precedence.** Strict wins over Standard, which wins over custom policies, which win over the defaults. To avoid surprises:

- Use clear, non-overlapping groups for each level
- Add the people who need custom handling as exceptions to the presets
- Remember that a user covered by a custom policy can't also get the preset settings for that feature

> **💡 Rollout tip:** don't flip Strict on for everyone on a Monday morning. Start with a small pilot group, watch quarantine and user feedback, then widen it. Tune anti-spam where false positives hurt most, such as teams that depend on time-sensitive external email.

---

## 🎭 Step 3: Configure impersonation protection

In the Standard and Strict presets, almost everything is locked to the recommended values. What you **do** have to configure yourself are the user and domain impersonation entries, and they are empty until you fill them in.

- Add the executives and key people attackers would pretend to be
- Add your own domains and important partner or supplier domains
- Review the list when leadership or partners change

---

## 🔑 Step 4: Permissions

Don't leave every admin as a Global Administrator. Give people the role that matches the job: **Security Administrator** to configure policies, **Security Operator** to work alerts and incidents, and **Security Reader** for read-only access, for example for auditors or managers. Check in the Defender portal what each role can do before you assign it.
I would prefer custom RBAC roles from console rather than the global Entra roles.

---

## ⭐ Step 5: Priority accounts and user tags

Mark your high-value users as **priority accounts** (up to 250) so they are easy to spot in reports and investigations, and check which priority account protections your plan includes. With Plan 2 you can also create custom **user tags** to group other sets of users.

---

## 📢 Step 6: User reporting

Make sure users have a way to report suspicious and wrongly blocked email, such as the built-in Report button in Outlook.

- Decide where reports go: a dedicated mailbox, Microsoft, or both
- The default mailbox for reports is the first admin's mailbox, so create a dedicated one instead
- Submitting reports to Microsoft helps the filters learn

---

## 🚫 Step 7: Block and allow discipline

Use the **Tenant Allow/Block List** and the **Submissions** page, and keep this principle in mind: blocks are safe, allows are risky.

- Blocking a sender, file, or URL is straightforward
- Allow entries override filtering for bulk, spam, and some phishing verdicts, so keep them rare and temporary
- You can't directly allow malware or high-confidence phishing. Submit the item and confirm it's clean first

> **🛠️ Pro tip: if a legitimate message gets blocked,** submit it from the Submissions page and use the temporary allow it creates. Don't weaken a whole policy because of one sender.

---

## 💬 Step 8: Cover Teams, SharePoint, and OneDrive

Safe Attachments for SharePoint, OneDrive, and Teams is a **global setting that presets don't control**. It's easy to forget, so turn it on separately. If you use Teams heavily, review Microsoft's Teams protection guidance as well.

---

## 🏢 Hybrid and on-premises mailboxes

Not every mailbox lives in Exchange Online. In a hybrid setup, Defender for Office 365 can protect a mix of cloud and on-premises mailboxes, with Exchange Online Protection filtering the inbound mail.

- Check how inbound mail is routed, so it passes through Microsoft 365 first
- Confirm which mailboxes your policies and licences actually cover
- Pilot with one mailbox in each location before you widen the rollout

---
## 📬 Mail flow and delivery rules

Policies only work well when mail reaches Microsoft 365 the right way. Check these before you tune anything.

- **Mail routing:** your MX record should point to Exchange Online Protection, or to a gateway that relays inbound mail straight to it. Confirm your main domain is set up correctly in Exchange Online.
- **Third-party gateways:** if mail passes through another service first, Defender only sees that service's IP address. Turn on **Enhanced filtering** on the inbound connector so it can find the real sender, which keeps SPF, DKIM, and DMARC checks accurate and reduces false positives.
- **Advanced delivery:** use it sparingly for two cases: a security operations mailbox that needs to receive suspicious messages unfiltered for analysis, and the sources of approved phishing simulations.
- **Quarantine policies:** decide what users may do with quarantined messages. Let the help desk review and release mail with the **Quarantine Administrator** role instead of full security admin rights, which follows least privilege.
- **Licensing:** make sure every mailbox covered by your policies is licensed, including shared mailboxes.

---
## ✅ Step 9: Verify, then keep checking

- Open the **Configuration analyzer** in the Defender portal (Email & collaboration, Policies & rules, Threat policies) and compare your policies against the Standard and Strict recommendations
- Fix anything weaker than the recommended settings, and watch the drift history to see if settings change over time
- Re-check after product updates, for example every quarter
- With Plan 2, launch **attack simulation training** to see how users really behave
- Use Microsoft's Security Operations Guide for monitoring and investigation

---

## 🔗 Connect it to the rest of your stack

Email threats rarely stand alone. In Defender XDR, an email alert can join endpoint and identity alerts (Parts 1 and 2) in a single incident, so you investigate one story instead of three.

- Plan 2 adds automated investigation and response for common email threats
- If you use Microsoft Sentinel or another SIEM, plan the connection early and test alert quality before go-live

---

## 📝 Document it

If only one admin knows how everything is configured, your response plan is fragile. Keep a simple policy register: where each policy lives, what it controls, who owns it, and what to do during an incident. Review it together with your quarterly policy check.

---

## 🚨 Common pitfalls

- ❌ **Assuming Standard and Strict are already on,** when only built-in protection is
- ❌ **Skipping DMARC enforcement,** leaving your domains easy to spoof
- ❌ **Leaving impersonation lists empty**
- ❌ **Overlapping groups,** so users land in the wrong policy
- ❌ **Broad allow entries** that quietly bypass filtering
- ❌ **Forgetting Safe Attachments for SharePoint, OneDrive, and Teams**
- ❌ **Never reviewing,** so old custom policies drift below the recommended baseline
- ❌ **Locking everything down on day one,** then watching users find workarounds
- ❌ **Skipping user training and an easy way to report,** so warnings get ignored
- ❌ **Leaving all the knowledge with one admin,** with no documentation
- ❌ **Enabling it without an integration plan** for Defender XDR, Sentinel, or your SIEM


---

## ⏭️ Next in this series

- Part 1: Migrating to Defender for Endpoint
- Part 2: Deploying Microsoft Defender for Identity
- Part 3: Hardening Microsoft Defender for Office 365 (this post)
- Part 4: Configuring Defender for Cloud Apps (MDCA) and SaaS governance
- Part 5: Operationalizing Microsoft Defender Vulnerability Management

*Disclaimer: this is an independent technical blog and is not affiliated with or endorsed by Microsoft. Features depend on your licence (Plan 1 or Plan 2), so check Microsoft Learn for the current prerequisites and settings.*
