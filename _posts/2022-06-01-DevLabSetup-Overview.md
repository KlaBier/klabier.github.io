---
title: "Hybrid Identity Testlab: Experiences from the field - Overview"
date: 2023-06-04T22:01:32
#
layout: list

image:
  path: /MyPics/2022-06-01-DevLabSetup-Overview_cover.png

# License according to CC-BY
---

* this unordered seed list will be replaced by the toc
{:toc}

A test environment is an important part of my daily work to test new features and technologies, illustrate content to customers and so on. There are many scenarios.
Normally, the installation of a test lab, for example based on a developer tenant, is long-term and does not need to be renewed.
In the past, however, it was unfortunately the case that I repeatedly found myself in the situation of having to rebuild my test environment. Most recently, I had a sponsored lab and after a job change, I moved to a Developer Tenant. In the process, I had to rebuild the hybrid setup, with all the components, with everything.
There are other situations that may require a re-install and the effort is always the same and there are lots of pitfalls waiting for you.
Since we are looking at a typical hybrid setup here, i.e. with leading Active Dierctory On-Prem and AAD, and not a in comparison, simple Azure IaaS setup, just to have a quick look at something, we do have aspects here that are quite specific and that can cause headaches during a reinstall:

Some examples:

- Registered domain names
- transferring subscriptions from one tenant to another
- Synchronization that needs to be set up again
- Transfer of other security settings such as conditional access policies

This series of articles describes my experiences when I moved my hybrid testlab to another tenant. The topics presented in the series are also applicable to other scenarios, for example, where a test tenant is to be implemented and parallel operation is desired.

## Overview of the article series:
**Part 1:**
Commissioning of an E5 Developer Tenant

**Part 2**
Transferring subscriptions between tenants without mail access or similar.

**Part 3:**
Synchronization and important aspects of reinstallation

**Part 4**
Transfer of security settings from tenant and also from M365 with M365DSC

I hope you enjoy reading the series 😃


{% include  share.html %}

