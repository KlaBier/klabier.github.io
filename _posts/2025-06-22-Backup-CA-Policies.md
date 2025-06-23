---
title: "Protecting your Conditional Access Policies: Lean Backup Strategies for Entra ID"
author: Klaus Bierschenk
date: 2025-06-22T08:00:32

layout: list

image:
  path: /MyPics/2025-06-22-CAPolicy-Cover.png
---

<figure>
  <img src="/MyPics/2025-06-22-CAPolicy-Cover.png" style="width:75%">
</figure>


* this unordered seed list will be replaced by the toc
{:toc}

Welcome to my new blog post!
## Conditional Access Policies: A Core Component of Zero-Trust
Conditional Access Policies are frequently discussed in posts and articles. This is hardly surprising, as they are a central component of modern zero-trust strategies.

Sometimes there’s a new signal that can be added to the scope. Sometimes a fancy feature that, when used correctly, brings even more security. All of that is good, important, and absolutely necessary.

I’m a big fan of the level of security that this great technology in Entra ID provides us. In my projects, I secretly get excited when the topic of Conditional Access comes up, simply because it’s really fun to see the enormous leverage administrators have here.

**The Risk of Changes: What happens when things go wrong?**

What is often overlooked amidst all the enthusiasm around Conditional Access Policies: they are vulnerable to changes. And those can happen quickly.

You come into the office in the morning, suddenly something doesn’t work as usual, the cause is unclear. A look at the Conditional Access Policies, and you think: “Wait … that looked different from yesterday?”

Who changed what?

**Audit Logs: Helpful but limited**

Sure, the audit logs help with tracking down changes, but analyzing errors that way is rarely fun. Especially if you’re working with just the built-in tools. Because then the logs are often the only chance to understand where and when something went wrong.

**No Built-In Backup Function — A responsibility for Admins**

Unfortunately, the Entra Admin Center does not have a built-in backup function that allows you to comfortably save Conditional Access Policies (or other settings).

Microsoft hasn’t forgotten this, though; rather, they clearly place the responsibility in the hands of administrators and provide thorough documentation.

Conditional Access Policies can be reliably exported and imported again if needed via the Microsoft Graph API.

**Roles and Human Error**

Admittedly: Not everyone is allowed to make changes to Conditional Access Policies. You need at least a role like Conditional Access Admin, Security Admin, or Global Admin.

But who holds these roles? Exactly — people. And people make mistakes.

**Versioning without DevOps: Simple Backup Methods**

If you’re one of the admins who doesn’t work with DevOps or other configuration-as-code mechanisms, there are still ways to backup and version your CA-Policies.

With a few simple tools, you can create backups, archive them, and even compare them with previous versions, all without commercial tools.

At [Experts Live in Leipzig](https://expertslive.de/) in 2025, I had the opportunity to give a talk about backups in Entra. If you’re interested, feel free to check out my  [conference session history](https://nothingbutcloud.net/talks/2025-04-11-ExpertsLive_25/). In the slide deck you can find supporting material on deleted objects in Entra ID:

- Which objects offer soft delete?
- Which ones get hard deleted immediately?

**The Challenge of Hard-Deleted Objects**

An exciting topic, especially in the context of Conditional Access Policies, since these belong to the “hard deleted” objects. That means: If a CA policy is deleted, it’s gone immediately and permanently, without any recovery option.

Changes can’t simply be rolled back either. There is no button to “restore last working state.”

It’s actually critical, considering how important Conditional Access are for the security of a tenant.

## Manual Import/Export via the Graph API

The easiest way to back up a Conditional Access Policy is through the Microsoft Graph API.

Use the following endpoint to retrieve a complete list of your CA policies via a GET request:

``` Powershell
GET https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies
``` 

[![CABackup_1](/MyPics/2025-06-22-CABackup-1.png)](/MyPics/2025-06-22-CABackup-1.png){:target="_blank"}
Figure 1: Graph Explorer to get Conditional Access Policies
{:.figcaption}

You can also retrieve a specific policy, simply append the ID to the endpoint:

``` Powershell
GET https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies/{policy-id}
```

[![CABackup_2](/MyPics/2025-06-22-CABackup-2.png)](/MyPics/2025-06-22-CABackup-2.png){:target="_blank"}
Figure 2: Graph Explorer to get a specific CA via Policy-ID
{:.figcaption}

The result is a complete JSON containing all the details of the respective policy.

**Save as JSON-File**

You can copy this JSON output via the clipboard into a file for later use or you can copy it in a local Git repository or on GitHub with version control. A simple PowerShell example can be found further below.

[![CABackup_3](/MyPics/2025-06-22-CABackup-3.png)](/MyPics/2025-06-22-CABackup-3.png){:target="_blank"}
Figure 3: JSON Policy File in Notepad++
{:.figcaption}

**Upload a policy back into the Entra Admin Center**

The way back is just as straightforward:
In the Entra Admin Center, you’ll find the function "upload policy file ..." in the Conditional Access Policies section.

**➔ Important:**

Use “Review + Create” directly when importing, not “Save”, as the latter may fail when processing read-only attributes.

[![CABackup_4](/MyPics/2025-06-22-CABackup-4.png)](/MyPics/2025-06-22-CABackup-4.png){:target="_blank"}
Figure 4: Create CA Policy from JSON-Export-File
{:.figcaption}

## EntraExporter – Easily back up CA Policies (and more...) via CLI

I’m a big fan of this tool, thanks to Merill Fernando (link at the end of this article) for creating it and dedicating so much time to providing tools like this one!

With EntraExporter, you get a lightweight CLI tool that exports all configurations from Entra ID, including, of course, Conditional Access Policies, without unnecessary complexity.

**What does EntraExporter do?**
- Exports various settings from Entra ID
- Saves the results as neatly structured JSON files in a directory tree
- Can be easily integrated into a local Git repository

The screenshot below shows the Entra Exporter embedded into my local Git script.

[![CABackup_5](/MyPics/2025-06-22-CABackup-5.png)](/MyPics/2025-06-22-CABackup-5.png){:target="_blank"}
Figure 5: Entra Exporter to Export configutation settings like CA-Policies
{:.figcaption}

This gives you a machine-readable backup right away, allowing you to easily compare, archive, or restore configurations later.

**Importing CA Policies using the Exporter’s JSON**

The import works just like the process described in the previous section using Graph Explorer:
- Open the Conditional Access Policy dashboard
- Upload the Exporter’s JSON file using “Upload policy file”
- Go directly to “Review + Create” to avoid conflicts with read-only attributes

This works because the JSON format generated by EntraExporter matches the expected import format.

**Finding Policy Names**

One small catch: the exported files use the policy ID as the default filename.

If you need more descriptive names, a simple PowerShell command like this can help:

``` Powershell
Get-MgIdentityConditionalAccessPolicy | Select-Object Id, DisplayName
```
This allows you to quickly build a mapping table or adjust your exporter workflow accordingly.

[![CABackup_6](/MyPics/2025-06-22-CABackup-6.png)](/MyPics/2025-06-22-CABackup-6.png){:target="_blank"}
Figure 5: Sample Entra Exporter Export Folder Structure
{:.figcaption}

In my lab, I store all configuration settings in a local Git repository on my workstation. I use two PowerShell scripts for this: one feeds the local Git with backups, each timestamped, and the other shows a list of all saved versions. This way, you can easily select and restore an specific older version.

You then import the restored JSON from the desired day just as described above. Admittedly, the process is manual, but it gives you maximum flexibility and independence from external tools. Without such backups, you have nothing in case of an emergency. That’s why it’s worth running the EntraExporter regularly to be prepared for the worst-case scenario.

## Microsoft 365 DSC
Last but not least, I’d like to mention Microsoft365DSC. This open-source PowerShell module can do much more than what we’ve covered so far in this post. With it, you can generate configuration reports, compare policies in the form of reports, or even keep multiple tenants automatically synchronized. Microsoft365DSC is also nicely rounded off with DevOps integration.

As the name suggests, the module is based on PowerShell DSC and comes with several dependencies on other modules. Solid knowledge of PowerShell and DSC is definitely an advantage here. The community puts a lot of effort into the project, which means the module and its documentation are always up to date (https://microsoft365dsc.com). So I won’t re-explain everything here. Feel free to check out the official docs if the tool interests you.

To wrap things up, here’s an example of what an export with M365DSC might look like.

[![CABackup_7](/MyPics/2025-06-22-CABackup-7.png)](/MyPics/2025-06-22-CABackup-7.png){:target="_blank"}
Figure 7: M365DSC Sample Cmdlets
{:.figcaption}

The export then needs to be converted into a JSON format and can either be imported manually (as shown in the previous examples) or by using the import features provided by the Desire State mechanisms.

Considering that M365DSC is a community-driven open-source solution and therefore available free of charge, the range of features it offers is truly impressive.

## Own Local Git for file content

In the example with EntraExporter, we save the exported files in a local Git repository. Of course, you can also do this with GitHub, but here I want to show how simple a local solution can be that works entirely without GitHub access.

In my repository, I have two PowerShell scripts:

**- EntraExporter_WriteLocalGit.ps1** – This creates a versioned local folder with a CA backup. A screenshot above shows the script in action.

**- EntraExporter_ReadLocalGit.ps1** – This script lists all versions stored in the archive. In true retro style, you enter a number to select which backup you want to restore. The result is then placed in the specified target folder, including the JSON file you need for the import.

[![CABackup_8](/MyPics/2025-06-22-CABackup-8.png)](/MyPics/2025-06-22-CABackup-8.png){:target="_blank"}
Figure 7: list and read backup from local GIT
{:.figcaption}

Admittedly, it’s all pretty old-school — but honestly: how often do you actually need to restore a policy?

## Protected Actions – Keeping an Eye on the Admin

At the beginning of this post, I mentioned that only CA Admins and higher roles are allowed to make changes to Conditional Access Policies. Nevertheless, you can further reduce the risk of human errors by setting up so-called Protected Actions.

These ensure that with every action on CA Policies, whether creating, deleting, or modifying, an additional layer of security is applied. To do this, you create a special CA Policy that activates as soon as a change to the CA Policies is about to be made.

Within this policy, you can use the full range of Conditional Access signals to precisely control from where and with which devices or browsers changes are permitted. For example, you can exclude unauthorized devices, insecure browsers, or certain locations from making edits.

This is especially helpful to prevent damage upfront that would otherwise require restoring the policies later.

## Summary
	
- Entra does not provide built-in mechanisms to automatically back up CA policies or other settings.
- Microsoft offers APIs and documentation that every administrator should know and use.
- There are simple ways to back up CA policies—even without expensive tools.

**➔ Very important:** Regardless of which method you choose, regularly test your backup and restore processes so that you’re well prepared in case of an emergency.

## Further reading

- [EntraExporter GitHub](https://github.com/microsoft/EntraExporter)
- [Microsoft365DSC GitHub](https://microsoft365dsc.com/)
- [Recover from deletions – Microsoft Learn](https://learn.microsoft.com/en-en/entra/architecture/recover-from-deletions)
- [Shared responsibility between Micosoft and Organizations - Microsoft Learn](https://learn.microsoft.com/en-us/entra/architecture/recoverability-overview#shared-responsibility)
- [Protected Actions - Microsoft Learn](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/protected-actions-overview)

Enjoy trying it out 😀

... and let me know, when you have hints, problems, questions, using the e-mail option below

{% include  share.html %}


