---
title: "More Infos on Azure AD Cloud Provisioning and news from Ignite 2020"
date: 2020-09-24T10:59:57
layout: list

description: What's next wir Azure AD Cloud Sync

image:
  path:    /MyPics/2020-09-24-CP-Advanced_Cover.png
---

* Table of content below
{:toc}

Last year at Ignite 2019, Microsoft introduced the Public Preview of Azure AD Cloud Provisioning (CP), wich was available shortly after the event.

One year later, at all-digital Microsoft Ignite 2020, from September, 22-24, Microsoft announced some news in several sessions and talks about that new synchronization method, which is very interesting and make it more and more a complete solution. Microsoft called it  "Public Preview Refresh" and it will be availlable in October 2020.

## Whats in the box today?

Check my related article Cloud Provisioning, which also describes the entire Synchronization Story from Microsoft. Yes, it's not just Cloud Provisioning. There is a lot more with the Microsoft history journey along synchronization. Read the article [here](http://nothingbutcloud.net/azuread/CP/){:target="_blank"}.

The first release of Cloud Provisioning appears very promising and has a couple of valuable features, especially easy setup, central administration from the Azure Portal and disconnected forest support, only to mention three usefull functionalities.

But when we go into details,  we can see that it is not overall complete. What definitly makes sense, since it is Public Preview.

I have checked it in my KBCORP Lab and wanted to share my experiences.

## My Lab Environment

The setup of CP in my Lab was very easy. My Main Environment is the KBCORP.DE Domain which has a hybrid config with a P2 Tenant and the synchronisation is done by AAD Connect. I will not change this for testing CP and because of this I brought up a second domain KBACCOUNT.DE to play with CP. In that scenario I can also test how the new synchronization works, in a disconnected forest approach, because there is no trust between KBCORP and KBACCOUNT.

I have two servers in KBACCOUNT with the CP agent installed. How I made the setup doesn't need to be described here step-by-step, because Microsoft provides a very detailed [Configuration Guide](https://docs.microsoft.com/en-us/azure/active-directory/cloud-provisioning/tutorial-single-forest){:target="_blank"} for setting up Cloud Provisioning.

I decided to start with two servers to see how failover mechanisms are working. I tested it in a various ways, with stopping services, switching off the server etc. The sync cycle of 2 min is held and I can assure you, High Availlability works very reliable.

The accounts from both domains are synchronized into my tenant without any issues. My lab has two registered domain names KBCORP.DE and KBACCOUNT.DE. With this setup I can demonstrate a typical customer scenario, where, for example, another company is taken over or whatever the reason is. Lot's of reasons are possible here. The point is, that both Active Directory Domains are untrusted.

The existing tenant can be used from the new domain (Company) and the existing AAD Connect setup on KBCORP isn't touched. I just setup CP in the KBACCOUNT and all the accounts will go into the tenant with having the correct UPN and domain suffix.

Can you imagine what you have to do in that scenarion without CP? Just with AAD Connect? Yes, it can be manifactured and this is what customers do over years. But you must establish a trusted environment where AAD Connect can act between all the domains. Now, with CP, this is not required and the setup is very easy.

<figure>
  <img src="/MyPics/2020-09-24-CP2-IMG1.jpg" style="width:80%">
  <figcaption>KBCORP Lab from an synchronization perspective</figcaption>
</figure>

## Potential for improvement

In a daily usage of CP you find a couple of things that can be improved. I will not talk about features which are not implemented in the current version, like PTA Support or the not yet implemented Exchange hybrid support. I just want to talk about a few things that I have noticed.

Conspicuous is the Dashboard, where a lot of free space remains for more information and functionalities. It looks like a little empty, under construction. Yes, I know, Public Preview...

I checked the CP environment for logging information, but this seems to be a challenge: the server, where the agent is installed  has two new Eventlogs. **AgentUpdater** and **ProvisioningAgent** logs. Both will not provide detailed informations from the CP services.

Same with the log in the Cloud Provisioning Portal. Here the Admin will find rudimentary informations. Alltogether troubleshooting is a challenge with the logging capabilities at the moment.

At Ignite Microsoft announces improvements with the logging in Cloud Provisioning for the GA version, when this becomes availlable by the end of 2020.

## Tricky Attribute Mappings and Transformations with Microsoft Graph

One thing, that costs me multiple hours, are the Attribute Mappings and Transformations changes with Cloud Provisioning in the Public Preview

Microsoft describes the entire procedure [here](https://docs.microsoft.com/de-de/azure/active-directory/cloud-provisioning/concept-attributes#view-the-schema){:target="_blank"}. But I want to share my experiences in addition to that article, because a couple of things are not clear (at least in my opinion).

With the CP Setup an App Registation will be created in Azure. This service principal can be used to access the schema, to modify the attribute mappings.

The [Microsoft Graph Explorer](https://developer.microsoft.com/de-de/graph/graph-explorer){:target="_blank"} must be used to setup a couple of queries against the Azure AD. Don't forget to check the permissions. **Directory.ReadWrite.All** is required here.

The first query we have to use ist to get the **ServicePrincipleID**


```posh
https://graph.microsoft.com/beta/serviceprincipals?$filter=startswith(Displayname,'kbaccount')
```

<figure>
  <img src="/MyPics/2020-09-24-CP2-IMG2.png" style="width:80%">
  <figcaption>Getting ServicePrincipleID</figcaption>
</figure>


The above mentioned article explains how to query the Displayname for a string that starts with 'Active' as the ServicePrinciple Name. But keep in mind that the name here is individual and has nothing to do with 'Active Directory' for example. When you are not sure what's the name in your setup ist, check it in the App Registrations. In my case the name was the domain name kbaccount.de what works with the query, as you can see in the example above.

<figure>
  <img src="/MyPics/2020-09-24-CP2-IMG3.png" style="width:80%">
  <figcaption>Checking App registration</figcaption>
</figure>

The next step is to get the ProvisioningID. For that query the former resulting ServicePrincipleID must be used:

```posh
https://graph.microsoft.com/beta/serviceprincipals/{Service Principal id}/synchronization/jobs/
```

<figure>
  <img src="/MyPics/2020-09-24-CP2-IMG4.png" style="width:90%">
  <figcaption>Getting ProvisioningID</figcaption>
</figure>

When you have both IDs you can create the last query to get the schema.

```posh
https://graph.microsoft.com/beta/serviceprincipals/{Service Principal Id}/synchronization/jobs/{AD2AAD Provisioning id}/schema.
```

Replace the PrincipleID and the ProvisioningID in the query and after executing this the result is displayed in the Browser Window. Copy the output into a editor of your choice and then you can make the modifications in the areas with source and target attribut mappings. Check [this article](https://docs.microsoft.com/de-de/azure/active-directory/cloud-provisioning/how-to-transformation){:target="_blank"} for a step by step guide what needs to be changed here in detail. I will not re-write Microsoft Article topics here. I just want to share my experiences with that method in manipulating the attribute mapping.

- Be sure that you use the correct app registration name. It is not something like "active" as a wildcard
- the results in the queries gives you lots of IDs in the browser windows. Check the screen-output above to pick up the correct IDs
- The schema is not really handy. I put it into VSCode and saved the file and the result was a 20MB output.
  In my testing the size was **not** a problem in getting the schema and also **not** to transfer it into an editor for changing. The problem what I had was the way back with the PUT Query, like MS describes. After editing the attribute section (see the MS article that I had mentioned above) I copied the entire content into the browser section in MS Graph and I covered a couple of error messages which makes no sense in that context. **when I cleared the Browser cache it worked "sometimes"**. By the way, I experienced this on both, a MacBook and a Windows 10 client

Microsoft integrates the possibility to edit attribute mappings directly in the CP Dashboard with the Public Preview Refresh Update. This makes it easy to implement an attribute mapping.

<figure>
  <img src="/MyPics/2020-09-24-CP2-IMG5.png" style="width:90%">
  <figcaption>New Attribute Mapping Possiblities from Public Preview Refresh Update</figcaption>
</figure>

But the entire procedure of manipulating the scheme, with the information above, and the referenced Microsoft article is not obsolete, because a programatic way is sometimes the better way to make changes. One example could be, when you implement mappings in the lab and when you want to implement it in the production tenant or for other customer. If you will go that way, keep my information above in mind.



## News from Ignite 2020

At Ignite Microsoft announces those changes with the following timeline:

<u>In Oct, 2020 a Public preview refresh will be availlabe and this contains those additional features:</u>

- Improved user experience
- Supporting null value fields
- On demand provisioning of single user
- improved troubleshooting experiences
- Accidental Deletes prevention
- Attribute Mapping and transformations in the user interface

<u>GA should be availlable by the end of the year with the following features</u>

- Password writeback (SSPR)
- Synchronization of up to 150k AD Objects
- gMSA Support for agent setup
- Improved performance in the initial sync cycle
- Improved Provisioning logs and health Info in the portal
- Azure workbooks for provisioning
- Auto upgrade of provisioning agent

Especialy with the monitoring and logging functionalities CP seems to be interesting for daily usage


## Summarize

I have the CP installed in my Lab now for half a year. To be honest, I will not touch the Sync Feature every day, but whenever I work with it, I see lots of additions and improvements to Azure AD Connect. Definitly not an replacement of the AAD Connect Server but it brings many extensions to an existing hybrid environment, especially when new domains must be integrated into the  existing flow. The GA is also interesting for new setups, when the admin can live without the not included features.

{% include  share.html %}
