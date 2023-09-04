---
title: "Azure AD renamed"
author: Klaus Bierschenk
date: 2023-10-01T08:00:32

layout: list

image:
  path: /MyPics/2023-08-10-StaledAutomation_Cover.png
---

<figure>
  <img src="/MyPics/2023-08-10-StaledAutomation_Cover.png" style="width:75%">
  <figcaption>Niran Kasri auf Pixabay</figcaption>
</figure>


* this unordered seed list will be replaced by the toc
{:toc}

## Overview

Microsoft hat angekündigt, dass das Azure Active Directory umbenannt wird in Microsoft Entra ID

[Microsoft Artikel: Neuer Name für Azure Active Directory](https://learn.microsoft.com/de-de/azure/active-directory/fundamentals/new-name)

Namen sind bestimmt Geschmackssache. Wie viele andere Dinge im Leben. Und nicht jede Namensänderung in der Microsoft Produktpalette ist nachvollziehbar und sinnvoll. Aber bei der Änderung des Cloud Verchnisdienstes liegt Microsoft goldrichtig.

Warum? Ich fang mit einer Frage an: Was ist das Active Directory?
Hierbei handelt es sich um eine Produktpalette, die im Grunde genommen aus voneinander unabhängigen Windows Server Rollen besteht. Das Hauptprodukt, und wenn von einem AD die Rede ist, ist
**ADDS**
Active Directory Domain Service
Ein hierarchischer x.500 und LDAP basierender Verzeichnisdienst.

Es gibt aber noch mehr Serverrollen (Technologien):
**ADFS**
Active Directory Federation Services
**ADCS**
Active Directory Certificate Services
**ADLDS**
Active Directory Lightwight Directory Services
**ADRMS**
Active Directory Rights Management Services

Der Name ***Azure AD*** impliziert, dass es sich dabei um einen Domain Service in der Microsoft Cloud handelt. Dem ist aber nicht so. Du hast in Azure keine Domänencontroller. AAD ist ein flacher Dienst, ohne hierarchische Struktur. Es gibt keine Gruppenrichtlinien. Keine OUs (Organisationseinheiten). Usw. 
Azure AD war nie ein Active Directory in der Cloud.

Das heisst, der Name hat nie representiert was das Feature ist

Active Directory ist eine Windows Server Rolle und eigentlch eine Familie

Wir haben ADDS
ADCS
ADFS
ADRMS
ADLDS
d on AD


Many roles which are in that familie

AAD was never build on AD

Doesnt speak ldap, NTLM

komplet different, AAD ist flach, keint

Azure AD ist kein Azure Dienst, wie der name vielleicht vermuten lässt

Warum wurde Azure AD so benannt? Es ist kein Azure Dienst aund auch kein AD

The name is inaccurate of what it is

Lots of confusion for people. Wo sind meine DOmänen Controller

Brauche ich eine Azure Subscriptio um Azure AD zu benutzen? Nein.

Wichtig ist, das es keine änderungen an der Funktionalität gibt

Cover image Niran Kasri from Pixabay

{% include  share.html %}


