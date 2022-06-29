---
title: "Zero Trust in Azure Identity - Part 4: Access Reviews"
date: 2021-01-19T13:22:32
layout: list

description: Automatic membership reminder

image:
  path: /MyPics/2021-01-19-ZeroTrust_AR_cover.png

# Licensed according to CC BY-NC-ND

---

* this unordered seed list will be replaced by the toc
{:toc}

## Goodbye forgotten group memberships

Who hasn't had this happen to them? A user account is added to a group and then you forget to remove the member? Unfortunate if the group is a security-sensitive group or role, and there are quite a few of those in Azure. One remedy for this is the "Access Reviews", or access checks, in German portal language. Access reviews are a means of putting groups, roles or enterprise applications to the test and validating the members assigned or contained.
The settings in the Azure AD portal can be found in the objects for which you want to create a check. This means in the area of the groups or enterprise applications. The exception to this are the roles, whose "Access Reviews" are not administered in Azure AD in the roles, but in the PIM Dashboard. Somewhat hidden after selecting the "Manage" option. By the way, do not confuse this with the item "Review Access", which is located in PIM directly above, under "Tasks". This is used to process any pending reviews for the user account that is currently navigating in the PIM portal and is stored as a reviewer.

## Access Review using an example

To explain the options available when reviewing roles, let's take a look at an example of the settings you may encounter. Let's say you want to review the members of the Global Reader role on a weekly basis and remove any user accounts you no longer need that you know should not be members there. To do this, we navigate to the "Manage" section in the PIM Dashboard and select the "Azure AD Roles" option, from where we go directly to the access checks. Many of the setting options are self-explanatory, such as the name or even the start date. One of the elementary options here is the time frame, notably the "Duration (in days)" option. This defines the period for which an "Access Review" is available and reviewers can make entries. The number of days possible here again depends on whether and which "Frequency" is selected, as the duration must not be longer than the number of days in the selected period, otherwise time overlaps will occur during the reviews. For our example, we select the weekly interval and 1 day for the duration. You can specify more than one role, but for our example we will limit ourselves to the role "Global Reader".
Further down in the dialog you specify the already mentioned reviewers. If there are several decision makers in your department, it may make sense to specify all of them here. If one of them starts the check, the list will later show for which user accounts the membership was approved or denied and also who decided this.

## Fine tuning for access control

At the very bottom of the dialog there is still the possibility of "fine tuning" via some advanced settings. Here you can specify whether accounts are actually removed or not. If no, this is only logged, which is useful for a test phase, for example. Especially worth mentioning at this point is the possibility to define what should happen if the reviewer(s) do not react. "No changes", "Remove access", "Approve access" or "Follow recommendations". In the case of the latter, the system decides on the continued existence of the membership. For example, has a user not logged in for a period of 30 days? In that case, the role membership would be removed. Also hidden, but possibly very useful, is the possibility to select "Members (self)" in the checker. As the name already conveys, no other persons are requested to check, but the members themselves. This kind of check is basically a "knot in the handkerchief" that the administrator makes himself here to be reminded of the membership. An administratively probably rather rare scenario, but if the admin wants it, the option can certainly provide valuable services.

## Access Reviews vs. Dynamic Groups

When using access reviews for Azure AD groups, it is worth mentioning the possibility of dynamic groups, which permanently fill the security groups with members based on query rules. This can replace access checks and may be a more elegant way to deal with members, as the admin does not have to worry about anything else with a well-defined query. Of course, this only applies to groups where dynamic membership is appropriate and not to roles and enterprise applications, as there is no dynamic assignment here. Here, it is important to plan well and make an approach for access reviews or a dynamic group in advance.
Access reviews also show up in various other places in Azure AD, such as access packages. The methodology behind this is always the same and analogous to that described here.

{% include  share.html %}
