---
title: "The dilemma with unused objects in Azure AD"
author: Klaus Bierschenk
date: 2023-03-22T08:00:32

layout: list

image:
  path: /MyPics/2023-03-22-FindStaledObjectsInAAD_Cover.jpg
---

<figure>
  <img src="/MyPics/2023-03-22-FindStaledObjectsInAAD_Cover.jpg" style="width:75%">
  <figcaption>mohamed_hassan auf Pixabay</figcaption>
</figure>


* this unordered seed list will be replaced by the toc
{:toc}


## Overview
It is very easy to add new users or computer objects in Azure Active Directory. Depending on the tenant settings, this can also be done quite extensively by end users themselves. Invited guest users 
or BYOD scenarios, for example. This is very convenient and timely in terms of user experience, but brings the disadvantage that over time more and more unused identities accumulate, which may no longer be needed. Guest user accounts that were used by external project members during a project, but have been forgotten afterwards. The same is true for devices registered in Azure AD. Registration is easy, even for end users, but "out of the box" no mechanism in Azure AD ensures that device objects that are no longer used are removed as silently as they were added. Smartphones, computers are swapped or lost and users soon use another device for their work. The administrator has to keep an eye on it himself, either manually and on a regular basis, as part of various "housekeeping" activities that ensure good hygiene in the tenant, or the Admin establishes automatisms that generate reports of unused objects.

Whatever the reason for these holdovers, they are unnecessary and pose a security risk.

### Device objects
In the best case, devices should be de-registered again via a lifecycle process, for example when a device is replaced. Or if a user reports a device as lost. Companies that use mobile device management (e.g., Microsoft Intune) can define policies that react in various ways to device objects that are no longer in use, such as deactivating or deleting them directly.

Each time a user accesses a cloud application with his device, the attribute "ApproximateLastLogonTimestamp" is provided with the current timestamp. Using the delta from today to the last time used, it is easy to find out when the device was last used against the tenant.

#### Azure AD Dashboard

This is very easy to do in the Azure AD Dashboard and the filter options there in the view of the devices.

<figure>
  <img src="/MyPics/2023-03-22-FindStaledObjectsInAAD_1.png" style="width:100%">
  <figcaption>Figure 1: List with devices in the Azure AD Dashboard</figcaption>
</figure>

This is helpful to look for individual devices or also with a smaller number of devices. With a high number of objects, however, this is hardly possible without the Powershell.

#### Powershell examples
Here, a list can be generated in no time at all, which can be sent by mail or otherwise serve as a basis for cleanup work. The starting point for this is the cmdlet

`Get-AzureADDevice`
{: .text-center}

For example, a list with all devices can be created like this

```posh
Get-AzureADDevice -All:$true | select-object -Property AccountEnabled, DeviceId, DeviceOSType, DeviceOSVersion, DisplayName, DeviceTrustType, ApproximateLastLogonTimestamp | Format-Table 
```

Some examples how to filter the output for a certain period of time and how to generate an Excel list directly can be found in [my Powershell repository](https://github.com/KlaBier/Powershell/FindUnusedObjects)

The result in Excel looks like this:

<figure>
  <img src="/MyPics/2023-03-22-FindStaledObjectsInAAD_2.png" style="width:100%">
  <figcaption>Figure 2: List with devices exported to Excel</figcaption>
</figure>

With this list it is easy to implement a reaction to these detected devices, best also directly via Powershell. But be careful with deleting device objects. There is no recycle bin here from which objects can be recycled, as is the case with user objects, for example. Also, the Bitlocker Keys of the respective device are stored in Azure AD. These are lost when an object is deleted.

It should also be noted that a distinction is made between Hybrid AAD Joined, AAD Joined and AAD Registered for the devices. When deleting from the AAD, the state on the respective device is not changed. Depending on which type of registration the device has, different aspects are significant here. This only applies to cases where devices may still be in use. Microsoft provides some Docs articles that describe what the differences are and what to look out for.

Every company should have a policy that describes how unused device objects are handled:
- what is the time period after which a device is considered a stale device? 
- Is Bitlocker used? Are the keys no longer needed or do recovery keys exist?
- A two-step process is suitable for removing unused devices from a tenant:

  * Step1: Deactivation of the device.
  * Step2: final deletion of the device after a defined period of time.


### Identifying unused user accounts
Even more critical than device objects are orphaned user accounts. These are easier to abuse and they come with permissions and roles.

#### Azure AD Dashboard
The Azure AD User Dashboard is only partially suitable for an investigation of identities that are no longer used. If it does get used, the column with the last login time must be part of the view. This is not visible by default. It can be easily added via "Manage view". After that, filtering and customizing the view can be done to the best of your ability.

<figure>
  <img src="/MyPics/2023-03-22-FindStaledObjectsInAAD_3.png" style="width:100%">
  <figcaption>Figure 3: Azure AD Dashboard showing device objects</figcaption>
</figure>

### Powershell vs. Graph API
To get information about the last logon time via script, we have to take a little detour. This is not quite as simple for users as shown above for devices, or as it may also be known from ADDS On-Premises via the property "LastLogonTimestamp". At least in Azure AD it does not work directly with the "Get-AzureADUser" cmdlet because the "lastSignInDateTime" property is not stored with the user object. We get this information through the signInActivity recource type of the Graph API. For this to work well graph permissions are necessary. What these are, and what other dependencies there are, is described by Microsoft in this [article](https://learn.microsoft.com/en-us/azure/active-directory/reports-monitoring/howto-manage-inactive-user-accounts)

The following cmdlets show how to access user information:

```posh
# Login/Connect
Connect-MgGraph -Scopes 'User.Read.All', "Auditlog.Read.All"

# Get list with all users
Get-MgUser -All | Format-Table  ID, DisplayName, Mail, UserPrincipalName
```

The result will look like this, for example:

<figure>
  <img src="/MyPics/2023-03-22-FindStaledObjectsInAAD_4.bmp" style="width:100%">
  <figcaption>Figure 4: List device objects with Powershell</figcaption>
</figure>

More comprehensive options like filtering, or exporting users who last logged in before a certain date, I have also in my [Powershell Repository ](https://github.com/KlaBier/Powershell/FindUnusedObjects)

#### Access Reviews zum Ermitteln unbenutzter Konten

Access Reviews are also a possibility to detect inactive users and to delete them automatically after a certain period of time. With the intervention of an administrator (approval) or without. Only for guest users or for all user accounts. So even those that are not external. All this and more can be done with Access Reviews. So you already have all the stuff that you need to build when you use Powershell. You get along completely without code. This sounds very tempting and indeed I recommend to look into this possibility if not already done.
Access Reviews have been around for a while. They were introduced at the time to review memberships in groups or to review and renew or revoke access to applications if necessary. Newly added in May 2022([Link](https://techcommunity.microsoft.com/t5/microsoft-entra-azure-ad-blog/review-and-remove-aad-inactive-users-in-public-preview/ba-p/3290632)) the described possibility to identify unused user activities.

The following figure shows the elementary part of an access review that defines a target, in this case guest accounts.

<figure>
  <img src="/MyPics/2023-03-22-FindStaledObjectsInAAD_5.png" style="width:100%">
  <figcaption>Figure 5: Access Reviews to find unused User Objects</figcaption>
</figure>

By the way, besides the direct assignment of "Approvers", it is possible to specify that automatically the respective manager of the user account or the owner of the respective group that is in review becomes Approver. There is also the possibility to define multiple approvers, who can reassign previously reviewed decisions, etc. And since it can also be that employees need to edit the results of an Access Review, but for security reasons are not allowed to have access to the Azure AD portal (project office employees, for example), there is the possibility to display the Access Reviews and administer the results via "myaccess.microsoft.com". This is perfect in times of Zero Trust.

<figure>
  <img src="/MyPics/2023-03-22-FindStaledObjectsInAAD_6.png" style="width:100%">
  <figcaption>Figure 6: "mayaccess" website for non admins to administer Access Reviews</figcaption>
</figure>

We've only scratched the surface here on what Access Reviews can do. Microsoft has good articles in Microsoft Docs that describe Access Reviews in detail, including videos. With the appropriate keyword search you will quickly find what you are looking for. Have fun trying it out 😀

# Deutsche Version


## Übersicht
Es ist sehr einfach im Azure Active Directory neue Benutzer oder Computerobjekte hinzuzufügen. Je nach Tenant Einstellungen können dies auch recht umfangreich Endbenutzer selbst bewerkstelligen. Eingeladene Gastbenutzer 
oder BYOD Szenarien beispielsweise. Das ist im Sinne der Benutzererfahrung sehr komfortabel und zeitgemäß, bringt aber den Nachteil mit sich, das im Laufe der Zeit mehr und mehr ungenutzte Identitäten ansammeln, die evtl. nicht mehr benötigt werden. Gastbenutzerkonten, die während eines Projektes von externen Projektmitarbeitern genutzt wurden, danach aber in Vergessenheit geraten sind. Das gleiche gilt für im Azure AD registrierte Geräte. Die Registrierung ist einfach, auch für Endbenutzer, aber "out of the box" sorgt kein Mechanismus im Azure AD dafür, das nicht mehr benutzte Geräteobjekte so geräuschlos entfernt werden, wie sie hinzugefugt wurden. Smartphones, Computer werden getauscht oder gehen verloren und Anwender benutzen alsbald ein anderes Gerät für ihre Arbeit. Der Administrator muss selber ein Auge darauf werfen, sei es manuell und turnusmäßig, als Teil von diversen "Housekeeping" Aktivitäten, die für eine gute Hygiene im Tenant sorgen, oder er etabliert Automatismen, die Reports unbenutzter Objekte generieren.

Was auch immer der Grund für diese Überbleibsel ist, sie sind überflüssig und stellen ein Sicherheitsrisiko dar.
Schauen wir uns einige Möglichkeiten an, wie sich damit umgehen lässt

### Computerkonten
Im besten Fall sollten Geräte über einen Lifecycle Prozess wieder deregestriert werden, zum Beispiel bei einem Gerätetausch. Oder sei es, dass ein Anwender ein Gerät als verloren meldet. Unternehmen, die ein "Mobile Device Management" (z.B. Microsoft Intune) einsetzen, können hierfür Policies definieren, die verschiedenartig auf nicht mehr benutzte Geräteobjekte reagieren, diese beispielsweise deaktivieren oder direkt löschen.

Jedes mal wenn ein Anwender mit seinem Gerät auf eine Cloud Anwendung zugreift, wird das Attribut "ApproximateLastLogonTimestamp" mit dem aktuellen Zeitstempel versehen. Über das Delta von heute zu dem letzten verwendeten Zeitpunkt ist es leicht herauszufinden, wann das Gerät zuletzt gegen den Tenant benutzt wurde.

#### Azure AD Dashboard

Sehr einfach geht das im Azure AD Dashboard und den dortigen Filtermöglichkeiten in der Ansicht der Geräte.

<figure>
  <img src="/MyPics/2023-03-22-FindStaledObjectsInAAD_1.png" style="width:100%">
  <figcaption>Figure 1: List with devices in the Azure AD Dashboard</figcaption>
</figure>

Das ist hilfreich um nach einzelnen Geräten zu schauen oder auch bei einer geringeren Anzahl an Devices. Bei einer hohen Anzahl an Objekten geht dies aber kaum ohne die Powershell.

#### Powershell Beispiele
Hier lässt sich im Handumdrehen eine Liste generieren, die per Mail verschickt oder anderweitig als Grundlage für Aufräumarbeiten dienen kann. Ausgangspunkt hierfür ist das Cmdlet 

`Get-AzureADDevice`
{: .text-center}

Eine Liste mit allen Geräten lässt sich zum Beispiel so erstellen

```posh
Get-AzureADDevice -All:$true | select-object -Property AccountEnabled, DeviceId, DeviceOSType, DeviceOSVersion, DisplayName, DeviceTrustType, ApproximateLastLogonTimestamp | Format-Table 
```

 Einige Beispiele wie sich die Ausgabe für ein bestimmten Zeitraum filtern lässt und wie direkt eine Excelliste generiert wird habe ich [hier zusammengestellt. ](https://github.com/KlaBier/Powershell/FindUnusedObjects)

Das Ergebnis in Excel sieht dann beispielsweise so aus:

<figure>
  <img src="/MyPics/2023-03-22-FindStaledObjectsInAAD_2.png" style="width:100%">
  <figcaption>Figure 2: List with devices exported to Excel</figcaption>
</figure>

Mit dieser Liste ist es einfach eine Reaktion auf diese ermittelten Geräte zu implementieren, am besten auch direkt per Powershell. Aber Vorsicht mit dem Löschen von Geräteobjekten. Es gibt hier keinen Papierkorb, aus dem sich Objekte recyclen lassen, wie dies  beispielsweise bei Benutzerobjekten der Fall ist. Auch werden die Bitlocker Keys des jeweiligen Geräters im Azure AD gespeichert. Diese sind verloren, wenn ein Objekt gelöscht wird.

Zu beachten ist noch, dass bei den Geräten unterschieden wird zwischen Hyrid AAD joined, AAD Joined und AAD Registered. Beim Löschen aus dem AAD wird der Zustand auf dem jeweiligen Gerät nicht geändert. Je nach dem welche Art der Registrierung das Gerät hat, sind hier verschiedene Aspekte bedeutend. Dies betrifft nur die Fälle, in denen Geräte eventuell doch noch benutzt werden. Microsoft liefert einige Docs Artikel, die Beschreiben worin die Unterschiede liegen und worauf dabei zu achten ist.

Jedes Unternehmen sollte eine Richtlinie haben, die beschreibt wie mit ungenutzten Geräteobjekten umgegangen wird:
- was ist der Zeitraum, ab dem ein Gerät als Staled Device gilt? 
- kommt Bitlocker zum Einsatz? Werden die Keys nicht mehr benötigt oders existieren Recovery Keys?
- zum Entfernen ungenutzter Geräte aus einem Tenant eignet sich ein zweistufiger Prozess:

  * Step1: Deaktivieren des Gerätes.
  * Step2: endgültiges Löschen des Gerätes nach einem definierten Zeitraum.


### Ermitteln unbenutzter Benutzkonten
Noch kritischer als bei den Geräteobjekten sind verwaiste Benutzerkonten. Diese sind einfacher zu missbrauchen und sie sind mit Berechtigungen und Rollen ausgestattet.

#### Azure AD Dashboard
Das Azure AD User Dashboard eignet sich nur bedingt für eine Untersuchung von nicht mehr genutzten Identitäten. Wenn es doch eingesetzt wird, muss die Spalte mit dem letzten Anmeldezeitpunkt Teil der Ansicht sein. Diese ist standardmäßig nicht sichtbar. Sie lässt sich einfach über "Manage view" hinzufügen. Danach kann nach Kräften gefiltert und die Ansicht angepasst werden.

<figure>
  <img src="/MyPics/2023-03-22-FindStaledObjectsInAAD_3.png" style="width:100%">
  <figcaption>Figure 3: Azure AD Dashboard showing device objects</figcaption>
</figure>

### Powershell vs. Graph API
Um Informationen über den letzten Anmeldezeitpunkt per Script zu erhalten, müssen wir einen kleinen Umweg wählen. Das geht bei Benutzern nicht ganz so einfach wie oben bei den Geräten gezeigt, oder wie es eventuell auch vom ADDS On-Premises über die Eigenschaft "LastLogonTimestamp" bekannt ist. Zumindest geht es im Azure AD nicht direkt mit dem Cmdlet "Get-AzureADUser", da die Eigenschaft "lastSignInDateTime" nicht beim Benutzerobjekt gespeichert wird. Diese Informationen erhalten wir über den signInActivity recource type der Graph API. Damit dies gut funktioniert sind Graph Berechtigungen notwendig. Welche das sind, und welche weiteren Abhängigkeiten es gibt, beschreibt Microsoft in diesem  [Beitrag](https://learn.microsoft.com/en-us/azure/active-directory/reports-monitoring/howto-manage-inactive-user-accounts)

Folgende Cmdlets zeigen den Zugriff auf Benutzerinformationen:

```posh
# Login/Connect
Connect-MgGraph -Scopes 'User.Read.All', "Auditlog.Read.All"

# Get list with all users
Get-MgUser -All | Format-Table  ID, DisplayName, Mail, UserPrincipalName
```

Das Ergebnis sieht dann beispielsweise so aus:

<figure>
  <img src="/MyPics/2023-03-22-FindStaledObjectsInAAD_4.bmp" style="width:100%">
  <figcaption>Figure 4: List device objects with Powershell</figcaption>
</figure>

Umfassendere Möglichkeiten wie Filterung, oder den Export von Benutzern, die sich vor einem bestimmten Datum zuletzt angemeldet haben, habe ich in meinem [Powershell Repository ](https://github.com/KlaBier/Powershell/FindUnusedObjects)

#### Access Reviews zum Ermitteln unbenutzter Konten

Access Reviews sind ebenfalls eine Möglichkeit inaktive Benutzer aufzusprüren und diese automatisch nach einer gewissen Zeitspanne zu löschen. Mit Zutun eines Administrators (Approval) oder ohne. Nur für Gastbenutzer oder für sämtliche Benutzerkonten. Also auch diejenigen, die nicht extern sind. All das und mehr lässt sich mit Access Reviews bewerkstelligen. Sie bringen also das ganze Drumherum bereits mit, dass bei Vorgehensweise über die Powershell erst zu bauen ist. Sie kommen komplett ohne Code aus. Das klingt sehr verlockend und in der Tat empfehle ich sich mit dieser Möglichkeit auseinanderzusetzen, wenn noch nicht geschehen.
Access Reviews gibt es schon länger. Sie wurden seinerzeit eingeführt um Mitgliedschaften in Gruppen oder um den Zugriff auf Anwendungen zu prüfen und gegebenenfalls zu verlängern oder zu entziehen. Neu hinzugekommen ist im Mai 2022 ( [Link](https://techcommunity.microsoft.com/t5/microsoft-entra-azure-ad-blog/review-and-remove-aad-inactive-users-in-public-preview/ba-p/3290632)) die beschriebene Möglickeit ungenutzte Benutezeraktivitäten zu ermitteln.

Nachfolgende Abbildung zeigt den elementaren Teil eines Access Reviews der ein Ziel definiert, in dem Fall Gastkonten.

<figure>
  <img src="/MyPics/2023-03-22-FindStaledObjectsInAAD_5.png" style="width:100%">
  <figcaption>Figure 5: Access Reviews to find unused User Objects</figcaption>
</figure>

Übrigens lässt sich neben dem direkten Zuweisen von "Approvern" angeben, dass automatisch der jeweilige Manager des Benutzerkontos oder der Owner der jeweiligen Gruppe, die im Review ist, zu Approver werden. Es besteht auch die Möglichkeit mehrere Genehmiger zu hinterlegen, die zuvor geprüfte Entscheidungen neu treffen können usw. Und da es auch sein kann, das Mitarbeiter die Ergebnisse eines Access Reviews bearbeiten müssen, die aber aus Gründen der Sicherheit keinen Zugriff auf das Azure AD Portal haben dürfen(Projektofficemitarbeiter z.B.), besteht die Möglichkeit über myaccess.microsoft.com die Access Reviews anzuzeigen und die Ergebnisse zu administrieren. Das ist absolut zeitgemäß in Zeiten von Zero Trust.

<figure>
  <img src="/MyPics/2023-03-22-FindStaledObjectsInAAD_6.png" style="width:100%">
  <figcaption>Figure 6: "mayaccess" website for non admins to administer Access Reviews</figcaption>
</figure>

Wir haben hier nur an der Oberfläche gekratzt, bei den Möglichkeiten, die Access Reviews bieten. Microsoft hat in Microsoft Docs gute Artikel die Access Reviews im Detail beschreiben, auch Videos. Mit entsprechender Schlagwortsuche wirst du dort schnell fündig. Viel Spaß beim Ausprobieren 😀

Cover image Mohamed Hassan from Pixabay

{% include  share.html %}


