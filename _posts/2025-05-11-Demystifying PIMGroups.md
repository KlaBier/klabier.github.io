---
title: Demystifying Assignment Strategies with "PIM for Groups"
author: Klaus Bierschenk
date: 2025-05-11T08:00:32

layout: list

image:
  path: /MyPics/2025-05-11-PIM-Groups-Cover.png
---

<figure>
  <img src="/MyPics/2025-05-11-PIM-Groups-Cover.jpg" style="width:75%">
</figure>


* this unordered seed list will be replaced by the toc
{:toc}

Welcome to my new blog post!

**PIM for Groups** is a brilliant feature that offers several advantages. It reduces the burden on administrators while simultaneously increasing the overall level of security. However, there are challenges and considerations when using this feature

## Let’s start with a bit of background …

Using PIM for Groups helps you with two following scenarios

**Multiple configurations for multiple assignment scenarios**

Imagine you have an administrative grouping within your organization that requires a specific approval process or a custom assignment duration. These settings are defined directly at the role level and are configured once.
This works well as long as you only have one consistent role scenario.

However, if multiple stakeholders have different requirements, the centralized configuration at the role level becomes a limitation.

This is where groups become valuable.
Groups can be associated with roles and maintain their own configuration settings—just like roles do.

If you have a variety of administrative permission requirements, you can assign multiple groups, each with its own tailored settings, to meet those distinct needs.

**Activating multiple roles with a single approval**

In scenarios where, for example, a helpdesk employee needs access to multiple roles, such as Group Administrator in Entra and Collaboration Administrator in Microsoft 365, you can assign both roles to a single group.
With just one eligible assignment to that group, the user can activate all associated roles at once.

This provides a highly granular and efficient way to assign access, enabling precise control over permissions.
It aligns perfectly with the principles of Zero Trust.

## What are the options?

Now let us keep an eye to the Options, when you want to implement “PIM for Groups”.

Microsoft outlines two strategies:

**➔ Option A:**
Users are active members of a PIM-enabled group, and the group has eligible assignments to one or more administrative roles.

**➔ Option B:**
Admins have eligible member assignments to PIM-enabled groups, and these groups have active assignments to administrative roles.


Both models work — so what’s the difference?

With **Option A**, the user sees all associated roles in PIM as eligible and can activate each role individually.
This is functionally similar to assigning roles directly to the user, except that they are bundled via the group.

The key advantage: each role’s individual configuration (e.g. approval, MFA, protected actions) is fully enforced during activation.


**Option B**  is powerful.
The admin only needs to activate a single eligible group assignment and all roles attached to that group become active at once.

I personally prefer Option B for its simplicity — but it does come with important security considerations.


## Caution: Security risk!

You might assume the biggest risk is someone modifying group memberships but that’s actually minor:
PIM-enabled groups are protected, meaning only Global Administrators or Privileged Role Administrators can make changes.

The real risk lies elsewhere:
Since the user is technically a standard user (not directly assigned a role), they are not considered a protected admin identity.

If someone, like a User Administrator, can reset that user’s password, they could potentially hijack the account.
And once that identity is activated via PIM, the attacker could escalate to, for example, Global Administrator, simply because the group has that role assigned (in theory).

[![PIM_1](/MyPics/2025-05-11-PIM-Groups-I.png)](/MyPics/2025-05-11-PIM-Groups-I.png){:target="_blank"}
Figure 1: who can reset PW for which user or admins?
{:.figcaption}

To mitigate these risks:

* In addition to assigning roles via groups, explicitly assign users to roles as well.
This ensures they are treated as privileged identities, and prevents unauthorized users (e.g. User Admins, see table above) from resetting their passwords.

* Always require strong MFA for activating groups that hold multiple active role assignments.
This adds a critical layer of protection.

These are my two cents on this specific topic. If you want to dig deeper into the full solution, check out the [Microsoft documentation](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/concept-pim-for-groups)

And I hope this helps you decide which approach to take when using PIM for Groups.
It’s definitely a brilliant feature — but its implementation should be carefully planned, as its scope and impact can be significant.

Enjoy trying it out 😀

... and let me know, when you have hints, problems, questions, using the e-mail option below

{% include  share.html %}


