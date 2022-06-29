---
title: "Zero Trust in Azure Identity - Part 5: Simple Monitoring Break Glass Accounts"
date: 2021-01-22T22:01:32
layout: list

description: keep an eye on it :-)

image:
  path: /MyPics/2021-01-22-ZeroTrust Monitoring_cover.png
---

* this unordered seed list will be replaced by the toc
{:toc}

## Monitoring and Dealing with critical user accountss


Microsoft recommends so-called Break Glass accounts for emergency access. These have maximum permissions and are exempt from security measures such as "Conditional Access" policies. If something goes wrong with any policy or with the connection to on-premises, it is possible to access the tenant with these unaffected Break Glass accounts. In the first part of this post, we looked at more details about Break Glass accounts.
However, it is not only important to be prepared for an emergency. It is equally important to monitor activities around these accounts. For simplicity, the administrator has created a group with the accounts stored in the "Excludes" in the policies. But what if someone cheats his way into this group with his account? The person could work unhindered with wide rights past all policies and thus represents a high risk. Since all activities are documented in the Azure logs, it is easy based on these logs to react to entries using KQL (Kusto Query Language) queries and, for example, send a mail and an SMS to a sender circle when changes are made to the BGA group. The following is necessary for this.

## Keep AD logs perfectly

First of all, a log analytics workspace is needed to which Azure automatically sends the AD logs. This makes sense even without our example with the BGA accounts, since the monitoring and logon logs are kept on a limited basis. For Azure AD P1/P2 license 30 days otherwise 7 days.
The Log Analytics workspace is quickly created in the Azure Portal in the dashboard of the same name. In Azure AD, the workspace is added to the diagnostic settings. Thus, the entries end up in the storage account of the workspace, for which the retention period can be defined as desired. Step-by-step instructions for this can be found in the Azure Docs with the corresponding keyword search.

## Automated alerts

The alerts view for a Log Analytics workspace lists the alert rules that the administrator has created here over time. If you create a new alert rule, the essential elements are a condition and an action group. Both can be created directly at this point. In the same way, you can also use existing conditions or action groups and reuse them as you wish.
If a new action group is created, it may either contain notifications or actions or both. For a notification as in our example, a message definition is sufficient. To do this, we define both a mail and SMS send type in the group, which justifies a change to a critical group, such as the group containing BGA accounts. Note that action groups can be used in a variety of ways, including triggering a webhook or starting an automation runbook.
The second part in the warning rule, the condition, is essentially based on a KQL query that can bring to light from the workspace anything stored in the logs there. Besides our area, the "audit log", which we need to specify as the source here, there are other areas in a Log Analytics workspace. This ultimately depends on what data sources are connected to the Log Analytics.
Regardless of the alert rules, you can reach the KQL Editor in the Log Analytics workspace via the "Logs" option. Here, several examples are waiting to be tried out and modified for your own purposes. The whole thing invites you to try it out and it is not critical either. KQL queries are the foundation for warnings and the administrative expertise required for this wants to be learned through trial and error.

Back to our warning rule: figure (KQL group change.png) shows the basic statement and the KQL editor with the explorer.

<figure class="medium">
  <a href="/MyPics/2021-01-22-ZeroTrust Monitoring_I.png"><img src="/MyPics/2021-01-22-ZeroTrust Monitoring_I.png"></a>
  <figcaption>Exclude BGAs from CAs. A "must" in CA deployment</figcaption>
</figure>

The statement is stripped down to the bare essentials and could still be drilled down, for example to include a temporal filter. In the lower area of the results you can see which values the result still provides, which are therefore part of the log entry. In addition to the account that the BGA group has changed, the account that has been added to the group is also visible here.
The KQL statement is inserted into the condition and is thus part of the signal logic (Figure: Warning_Rule_Condition_SignalLogic.png). A number greater than 0 in the case means that the KQL statement was found, thus a change to the group has taken place and this in turn entails that as part of the warning rule the condition is met and the mail and SMS are sent. This is the principle of warnings in Log Analytics. It is recommended in the context of Break Glass accounts to also generate a warning when a BGA account logs in, which can then be acted upon in the manner described. If we assume that all BGA user accounts start with "BG", so for example the UPN is "BGAx@kbcorp.de" with sequential number x. The KQL statement for this looks like the following:

```posh
SigninLogs
| where OperationName == "Sign-in activity" | where UserPrincipalName startswith "BG". 
```

## Be careful with Access Reviews  

Actually, access reviews lend themselves to the control of BGA groups, but caution is advised in that context. Access Reviews also always allow members of a group to be removed, be it because an admin on his iPhone, on the go, mistakenly selects "deny", the BGA accounts fall out of the group and are thus unsuitable for emergency use. Admittedly a contrived example but the possibility exists and with sensitive accounts such as the BGA accounts you should leave nothing to chance and fare far better with the warning rules against Access Reviews in the event of controls on the break Glass accounts.

{% include  share.html %}