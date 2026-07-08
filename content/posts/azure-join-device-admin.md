---
title: "Adding a Local Admin to a Single Entra-Joined Device"
date: 2026-07-07
tags: ["azure", "entraid", "windows", "sysadmin"]
draft: false
---

Today I was helping my church with an issue related to admin rights on a local Windows machine. They don't use AD anymore thanks to things like Entra/Microsoft 365 and are embracing the new setup. One thing that's different about Entra joined devices is that the registering account will be just a normal non-admin user. That's fine for most scenarios, but if you have a power user that needs it, it's nice to have that option available.

## The Setup

- Device is Entra ID joined (no hybrid, no on-prem AD)
- Target user has zero admin rights on the machine currently
- I have Global Admin / Cloud Device Administrator in the tenant, so I'm a de facto local admin on any Entra-joined device, I just needed a way *in*

Through some research I confirmed the right place to do this is Computer Management, adding the user as `AzureAD\username@company.com`. My plan was to just run `runas` elevated from a command prompt and add the local user to Administrators via PowerShell. Unfortunately that doesn't really work if you have MFA enabled. `runas` is straight NTLM-style auth under the hood, so there's no way for it to pass through a Conditional Access MFA challenge. It just throws a generic "username or password is incorrect" error, which sent me down the wrong path for a bit thinking I'd fat-fingered something.

So after some troubleshooting, I ended up doing a **Switch User** instead of `runas`: Ctrl+Alt+Delete, Switch User, sign in with my M365 admin account, complete the MFA prompt like a normal interactive login. That let the original session stay open in the background and got me logged in as admin without a full logout.

From there I opened PowerShell **as administrator** (right-click, Run as administrator; just being logged in as an admin account isn't enough, the shell itself needs to be elevated too, learned that one the hard way with an "Access is denied" on my first attempt) and ran:

```powershell
Add-LocalGroupMember -Group "Administrators" -Member "AzureAD\username@company.com"
```

That did it. Probably more than one way to do this, but that's the direction that worked for me. Hope it helps you if you run across this.
