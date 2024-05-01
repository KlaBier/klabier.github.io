---
title: "Group Writeback in Entra Cloud Sync"
author: Klaus Bierschenk
date: 2024-05-01T08:00:32

layout: list

image:
  path: /MyPics/2024-05-01-GroupWriteBack_Cover.png
---

<figure>
  <img src="/MyPics/2024-05-01-GroupWriteBack_Cover.png" style="width:75%">
</figure>


* this unordered seed list will be replaced by the toc
{:toc}

Warm welcome to my new blog post!

One function that I have missed for a long time in the local Active Directory is the option of using dynamic groups. User accounts have certain characteristics and I automatically fill groups based on these characteristics. How cool is that? Unfortunately, this is not possible on-premises without additional tools. But it is possible in Entra ID and with the option of Group Writeback I can extend the feature to the local AD. With restrictions, ok, but what remains is very powerful.

Microsoft has described the topic in detail here
[MS Learn Article](https://learn.microsoft.com/en-us/entra/identity/hybrid/cloud-sync/how-to-configure-entra-to-active-directory), 
so I won't open that can of worms again here. Instead, let's take a look at a practical example of how we can map a dynamic group function in AD DS.

The Figure below shows an overview of the workflow.

<figure>
  <img src="/MyPics/2024-05-01-GroupWriteBack_1.png" style="width:75%">
  <figcaption>Figure 1: Example overview</figcaption>
</figure>

In our example, we have Entra Cloud Sync (Point 1 from pic above) and synchronize certain users to Entra ID. So far so good. Then we create a dynamic group in Entra (Point 2), with a dynamic membership rule based on the attribute “Department” = “HR”. This ensures that users from HR are in the dynamic group. In our example “SG-DG-HR”.

[![Groupwriteback_2](/MyPics/2024-05-01-GroupWriteBack_2.png)](/MyPics/2024-05-01-GroupWriteBack_2.png){:target="_blank"}
Figure 2: Dynamic membership rule
{:.figcaption}

Then we set up Group Writeback in CloudSync ...

[![Groupwriteback_3](/MyPics/2024-05-01-GroupWriteBack_3.png)](/MyPics/2024-05-01-GroupWriteBack_3.png){:target="_blank"}
Figure 3: Group Writeback Option in Entra Cloud Sync
{:.figcaption}

 ... and the group is synchronized as a universal group with the On-Premises Active Directory Domain (Point 3 from the overview picture).

[![Groupwriteback_4](/MyPics/2024-05-01-GroupWriteBack_4.png)](/MyPics/2024-05-01-GroupWriteBack_4.png){:target="_blank"}
Figure 4: AD DS OU with synchronized Entra Group
{:.figcaption}

This is set with the scoping filter in Cloud Sync

[![Groupwriteback_5](/MyPics/2024-05-01-GroupWriteBack_5.png)](/MyPics/2024-05-01-GroupWriteBack_5.png){:target="_blank"}
Figure 5: Scoping filter in Cloud Sync to define the Target OU in On-Prem AD
{:.figcaption}

The group name is formed here with a part of the ObjectID. However, this can be adjusted in the attribute mapping

[![Groupwriteback_6](/MyPics/2024-05-01-GroupWriteBack_6.png)](/MyPics/2024-05-01-GroupWriteBack_6.png){:target="_blank"}
Figure 6: CN setting for local AD Group
{:.figcaption}

That's all to implement. Now you can use the group to provide access to local resources

This example is focused on our pecific usecase. More details and more possibilities are explained in the related [Article](https://learn.microsoft.com/en-us/entra/identity/hybrid/cloud-sync/how-to-configure-entra-to-active-directory)

Let me summarize some important aspects:

* Until now, the Group WriteBack v2 option was available in Cloud Connect as public preview. This is no longer supported as of 30.06.2024. From now on, the successor described here must be used


* The users must be in Entra ID. Of course, make sense. Otherwise they cannot be a member in the Cloud Group and in scope of the membership rule 


* Users are never created in the local AD. Cloud-only accounts that are members of the dynamic group, due to their characteristics, are skipped during synchronization and do not appear in the AD Group. The rule could also be extended here and exclude synchronized users as members for example

Have fun trying it out 😀

{% include  share.html %}


