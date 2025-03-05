---
title: "Want to switch off ADFS? Looking for some content?"
date: 2022-01-09T23:06:32
layout: list

description: summary of ADFS decommissioning aspects

image:
  path: /MyPics/2022-01-09-GoodBye-ADFS-Linklist_Cover.png
---

<figure>
  <img src="/MyPics/2022-01-09-GoodBye-ADFS-Linklist_I.png" style="width:75%">
  <figcaption>Figure 1: Federation Services Splash Screen</figcaption>
</figure>

Don't get me wrong, ADFS is great. But it is also a technology with a long history and can (in some cases) be replaced by a better and easier technology in our modern Azure AD World.

But where to start when you have a bunch of applications relying in ADFS?

Or when you might have a relying party to M365 with your federated Azure AD Domain?

Then there are a couple of activities waiting for you ....

I attended some sessions last year on shutting down ADFS, and in migrating Relying Party Trusts from ADFS to Azure AD Enterprise Applications, for enduser SSO experiences. From those events and talks I got my own short archive with a list of articles and videos from Microsoft which covers related topics.

In some cases it can make sense to get rid of the ADFS deployment and use PTA or PHS instead and also to move your relying party trusts to Azure AD Enterprise Applications. With this approach you can use the entire stack of Zero Trust features whilst accessing your applications. Conditional access e.g.

Therefore it is a good time to start with migration considerations ... if not done already

| Description | Link/Reference | Comment |
| ------ | ------- | ----- |
| Upgrade from Active Directory Federation Services (AD FS) | [Show Article](https://www.microsoft.com/en-us/security/business/identity-access-management/upgrade-adfs){:target="_blank"} | Overview of the entire situation! Start reading here when you plan to upgrade from ADFS to Azure AD. Contains comparison between ADFS and Azure AD Features |
| Migrate from federation to PHS or PTA | [Show Article](https://docs.microsoft.com/en-us/azure/active-directory/hybrid/migrate-from-federation-to-cloud-authentication){:target="_blank"} | Good article with lots of considerations in planning and required pre-work on the entire migration process flow. Also covers step-by-step guidance for implementation. Provides links to more additional articles for topics which are not covered |
| Migrate application authentication to Azure Active Directory | [Show Article](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/migrate-application-authentication-to-azure-active-directory){:target="_blank"} | Bring time to read this one (30min read). Sophisticated article with details reagarding application migration from ADFS to Azure AD |
| ADFS Activity Report | [Show Article](https://aka.ms/migrateapps/ADFSactivity){:target="_blank"} | Explains how a ADFS activity report can be created to discover AD FS applications that can be migrated (or not). Covers additional claim rule support aswell |
| Videos with topics to upgrade your apps authentication from ADFS to Azure AD | [Get Videos](http://aka.ms/upgradeadfstoaad){:target="_blank"} | Short Videos on YouTube (5-7 min each) from MS Security |
| Resources for migrating applications to Azure Active Directory | [Show Article](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/migration-resources){:target="_blank"} | Links with resources for migration (deployment plan etc.) |
| ADFS to AAD App Migration tool | [Open Github](https://aka.ms/migrateapps/ADFStools){:target="_blank"} | Powershell based tools box with cmdlets to collect and analyze ADFS apps. Includes Excel export and cmdlets for migration |

{% include  share.html %}