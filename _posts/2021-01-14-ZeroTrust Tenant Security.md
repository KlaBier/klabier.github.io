---
title: "Zero Trust in Azure Identity - Part 1: Tenant Security"
date: 2021-01-16T09:59:57
layout: list

description: Overview of generell tenant related security settings

image:
  path: /MyPics/2021-01-18-ZeroTrust_TenantSecurity_cover.jpg

# License according to CC-BY
---

* this unordered seed list will be replaced by the toc
{:toc}

## Start with inventory

One of the key challenges is to determine the current status of identity and security measures in Azure Active Directory. Did you miss anything? Are there new features in the Azure portfolio that are worth implementing? If you ask yourself these questions regularly as an administrator, this has nothing to do with you being paranoid, but is rather a normal part of the daily work of a CISO and is propagated by Microsoft again and again in training courses and also in documents on Azure security.
If you are at the beginning of the considerations, the [Zero Trust Journey Assessment Tool](https://www.microsoft.com/security/blog/2020/04/02/announcing-microsoft-zero-trust-assessment-tool/){:target="_blank"} can be helpful to get a basic feeling for the current situation.
The term "tool" may be a bit of a stretch here, but the website definitely assists in getting a feel for the possibilities. After answering a few questions regarding your current setup, you'll get a list of additional documents on what you consider critical. You don't use MFA for external users? For this and other aspects, you will find links to further Microsoft Docs pages at the end of the evaluation, which show how best to close gaps.
Based on the existing categories, you can also see here which aspects still belong to the "Zero Trust" area, of which Identity is only one part, which we will look at further in the following.

### Vulnerability analysis

The Identity Secure Score on the Azure Portal is more dynamic. In addition to the Secure Score, which you may be familiar with from the Security Center and which examines everything from virtual machines to network security, the Identity Secure Score is limited purely to identities and evaluates the current implementation against Microsoft recommendations. The result is a point scale that signals to the administrator where he stands on average. The result can be exported, sent as an e-mail and thus serves as a constant proof for responsible third parties in the company, such as the security department. You can find the identity security score in the Azure Active Directory dashboard, behind the "Security" option.
Further up, in the security dashboard, in the "Protect" category is the Security Center, which is an additional option for analysis. Here you will find a subset, so to speak, related to Identity, of the aforementioned higher-level Security Center, which examines all infrastructure resources.
Each of the mentioned dashboards has a high value for administration. Regular evaluation of the Secure Scores should be part of every Azure AD operations manual. Only then can you be sure to use the maximum of measures for security. However, there are more security aspects related to identities that you should be aware of, for example, the settings in Azure AD, the tenant settings.

<figure class="medium">
  <a href="/MyPics/2021-01-14-ZeroTrust_Tenant_Security_II.png"><img src="/MyPics/2021-01-14-ZeroTrust_Tenant_Security_II.png"></a>
  <figcaption>Identity Secure Score to analyse vulnerabilities</figcaption>
</figure>

## Common Tenant Settings

The Azure Active Directory dashboard contains all configuration elements related to the directory service. Starting from user settings to application registration. Behind just about every category are settings that affect the behavior and capabilities of the respective function. Interesting here are the possibilities when selecting the option "User settings". Here you can find settings that can significantly contribute to securing the client in context. If you navigate to the "User settings", some general settings become visible on the first pages, such as whether access to the Admin Portal is allowed for non-administrators, which is possible by default.

Things get exciting behind the inconspicuous hyperlink "Manage external collaboration settings".
<figure class="medium">
  <a href="/MyPics/2021-01-14-ZeroTrust_Tenant_Security_I.png"><img src="/MyPics/2021-01-14-ZeroTrust_Tenant_Security_I.png"></a>
  <figcaption>Tenant Security: External Collaboration Settings</figcaption>
</figure>
Here you have, among other things, the option to control what external users are allowed to do and how far this can go. If you want to prevent unnecessary guest accounts from accumulating in Azure, consider which of the settings available here are suitable for fine-tuning for your tenant. For example, you might want to question whether guests should really be allowed to invite additional guests. Or with the "Collaboration restrictions" option, which domains are allowed for user cones to be invited and which are not. Both whitelisting and blacklisting are possible here.
Familiarize yourself with all the tenant settings. There are no standard recipes here, because the security requirements of today's companies are too diverse. Knowing the possibilities of which settings apply to the tenant can contribute significantly to security.

## Emergency Access

Before you start protecting Azure AD with various tools and policies, you should consider emergency scenarios. In this case, emergency means that access to Azure AD is no longer possible with a Global Admin or other administrative users. There are a lot of reasons for this. The MFA service at Microsoft is down or the administrator has locked himself out when configuring policies, to name just two. For these scenarios, the so-called "Break Glass Accounts" or "BGAs" come into play. These are administrative accounts that you don't use for day-to-day operations, but which have the maximum user rights in Azure AD. Other aspects make the accounts stand out: they are cloud only accounts, therefore not synced from on-premises. You implement two of the accounts and preferably add them to a group. And most importantly, they are excluded from all policies. "Exclude" options are used for this accordingly when configuring the policies. These features are critical for any business. It is unthinkable if the accounts and passwords fall into the wrong hands. Therefore, you should think about how the processes around the accounts look like. Maybe it makes sense to document and store the password in two parts and to grant access to the password fragments only to a divided group of people? It is also very important to test the login with the accounts from time to time to make sure that you are prepared for an emergency. It is also necessary to monitor the Azure AD group, which serves as an exclude group, because this represents a backdoor for people who want to bypass all protection mechanisms. The same applies to a login with a BGA account. For both cases, alerts in combination with KQL queries are suitable, which we will deal with in the second part of this workshop. There are a couple of articles in the Internet, including [this one](https://docs.microsoft.com/de-de/azure/active-directory/users-groups-roles/directory-emergency-access){:target="_blank"}, that provide a step by step guide to setup up BGA Accounts.
Part 6 of this blog series "Monitoring critical roles" describes additional aspects regarding monitoring of BGA Accounts

Cover image by Gerd Altmann from Pixabay

{% include  share.html %}

