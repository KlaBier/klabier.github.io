---
title: "How to change language in M365 Portal for a synchronized identity"
date: 2021-07-03T22:01:32
layout: list

description: this setting is really hard to find

image:
  path: /MyPics/2021-06-23-ChangeLanguageinM365Portal_cover.png

#Picture licensed according: CC BY-SA-NC
---

<figure>
  <img src="/MyPics/2021-06-23-ChangeLanguageinM365Portal_cover.png" style="width:75%">
</figure>

## Puzzling why language can not be changed in M365 Portal for synced users?

Changing the language in the Azure portal instead is quite simple. Just click on the gear icon in the upper right corner and then on the language item, change it, save and you're done. And the whole thing in the Azure Portal is independent of whether you are currently working with a cloud-only identity or with a synchronized account. So far so good 😃

However, there is one issue in my daily life that I have been wondering about and putting off for quite some time. Whenever I work with a synchronized account in the M365 Admin portal and try to change the language there, the following message is shown:

if the language is in German

<figure>
  <img src="/MyPics/2021-06-23-ChangeLanguageinM365Portal_II.png" style="width:75%">
  <figcaption>Figure 1: Language Setting in German</figcaption>
</figure>

and when you have set it already in English language

<figure>
  <img src="/MyPics/2021-06-23-ChangeLanguageinM365Portal_IV.png" style="width:75%">
  <figcaption>Figure 2: Language setting in English</figcaption>
</figure>

Now the whole thing becomes clumsy at last when you are the administrator yourself, as in my case. In my daily work the language was German, it was ok and the whole thing does not belong to my main issues. But whenever I try to change the language, I wonder at which central place this setting has to be made? On the myaccount.microsoft.com page? Or in some other place in the M365 portal?

This week I didn't have the topic with the language in the M365 portal present at all and I was in completely different topics in the Active Directory and with the attributes. But after looking through the list of attributes, I noticed the **preferredLanguage** attribute

<figure>
  <img src="/MyPics/2021-06-23-ChangeLanguageinM365Portal_III.png" style="width:50%">
  <figcaption>Figure 3: Use Attribute "preferredLanguage" to set requested language</figcaption>
</figure>

I remembered the language in the Microsoft 365 portal and indeed the attribute is used to set the language in Microsoft 365 in the form of the "Language Culture Names". So **de-DE** stands for German and **en-US** for English - United States etc.. You can find an overview of the codes here:
[Table of Language Culture Names](https://docs.microsoft.com/en-us/previous-versions/commerce-server/ee825488(v=cs.20)?redirectedfrom=MSDN)

My first thought was, not really consistent from Microsoft to control the language behavior differently in two portals (Azure / M365), at least for synchronized identities. But the language is not changed too often and managing it centrally in on-premises for accounts originating there can make sense. After all, many other settings are also administered there, why not the language as well?

{% include  share.html %}