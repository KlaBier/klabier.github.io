---
title: "Can I restore deleted Entra objects? Yes? No? Maybe?"
author: Klaus Bierschenk
date: 2025-08-30T08:00:32

layout: list

image:
  path: /MyPics/2025-08-30-DeletedEntraObject_Cover.jpg
---

<figure>
  <img src="/MyPics/2025-08-30-DeletedEntraObject_Cover.jpg" style="width:50%">
</figure>


* this unordered seed list will be replaced by the toc
{:toc}

Warm welcome to my new blog post

**For English**, please jump directly to the [English version further down](https://nothingbutcloud.net/2025-07-28-CA-PolicyAgent1/#english-version)

Den folgenden Beitrag habe ich wieder zweisprachig verfasst. Dafür habe ich mich ganz bewusst entschieden, denn Deutsch ist meine Muttersprache, die ich sehr schätze und in der ich meine Gedanken, jenseits von KI, oder anderweitig automatisch erzeugten Texten, am unmittelbarsten und authentischsten ausdrücken kann.

Gleichzeitig weiß ich, dass viele meiner Kontakte und Leser international sind und die Technologien, über die ich schreibe, ohnehin aus dem englischsprachigen Umfeld stammen. Deshalb findet sich der Text weiter unten zusätzlich in englischer Sprache.

So hat jede und jeder die Wahl den Beitrag in der Sprache lesen, in der man sich am wohlsten fühlt.

# Deutsche Version

## Einstieg
In Entra ID gibt es verschiedene Typen von Objekten: klassische Benutzer und Gruppen, aber auch Konfigurationsobjekte wie Conditional Access Policies oder Administrative Units.
Diese Objekte sind oft schneller gelöscht, als man denkt. Während sich manche davon problemlos wiederherstellen lassen, sind andere unwiederbringlich verloren.

Das wirft zentrale Fragen auf:

  - Was unterscheidet wiederherstellbare von endgültig gelöschten Objekten?
  - Worauf ist beim Löschen und besonders beim Wiederherstellen spezieller Objekte wie Anwendungen besonders zu achten?
  - Und vor allem: Welche Maßnahmen lassen sich im Vorfeld treffen, um auch „hard deleted“ Objekte zu "restaurieren"?

  
Kurz vorweg: Da CA Policies ein besonders kritisches Thema sind, habe ich dafür bereits einen eigenen Artikel geschrieben [Link](https://nothingbutcloud.net/2025-06-22-Backup-CA-Policies/). Hier geht es um die anderen essenziellen Objekte in Entra ID.

Die folgende Tabelle bildet den roten Faden dieses Artikels: Wir stellen die wichtigsten Objekttypen vor, beleuchten ihre Besonderheiten und zeigen, worauf es beim Löschen und Wiederherstellen ankommt.

[![EntraBackup_0](/MyPics/2025-08-30-DeletedEntraObjects_0.png)](/MyPics/2025-08-30-DeletedEntraObjects_0.png){:target="_blank"}
Figure 1: Objecttypes and deletion states
{:.figcaption}

Wenn Objekte in Entra ID gelöscht werden, befinden sie sich entweder im Zustand Soft Deleted oder Hard Deleted. Diese Unterscheidung ist entscheidend, wenn es um Wiederherstellung oder endgültiges Entfernen geht.

## Soft deleted - aber kein Papierkorb?
Die meisten Objekte landen nach dem Löschen zunächst für 30 Tage im Papierkorb (Soft Deleted) und werden erst nach Ablauf dieser Frist endgültig entfernt (Hard Deleted). Ein Hard Delete lässt sich auch manuell erzwingen, indem das Objekt aus dem Papierkorb direkt gelöscht wird.

Wird ein Objekt innerhalb dieser 30 Tage wiederhergestellt, stehen alle Eigenschaften exakt so zur Verfügung, wie sie vor dem Löschen waren. Das ist vergleichsweise unkompliziert und betrifft alle Objekte aus der obigen Tabelle, die in der Spalte Soft Delete gekennzeichnet sind.

Es gibt jedoch auch Objekte, die zwar Soft Deleted werden, für die es aber keinen Entra-Papierkorb gibt. Dazu zählen beispielsweise Administrative Units und Enterprise Applications. Ihre Wiederherstellung erfolgt ausschließlich über die Deleted Items API, entweder mit PowerShell oder noch direkter über den Graph Explorer.

Das folgende Beispiel zeigt, wie sich alle gelöschten Enterprise Applications auflisten lassen.

[![EntraBackup_1](/MyPics/2025-08-30-DeletedEntraObjects_1.png)](/MyPics/2025-08-30-DeletedEntraObjects_1.png){:target="_blank"}
Figure 2: List deleted EAs with Graph Explorer
{:.figcaption}

In der Abbildung oben ist zu sehen, dass die Liste der gelöschten Enterprise Applications über eine Abfrage des Endpunkts servicePrincipal in der Deleted Items API erstellt wird.
Soll eine Administrative Unit wiederhergestellt werden, verwendet man stattdessen den Endpunkt administrativeUnit, das Vorgehen bleibt ansonsten identisch.

Entscheidend ist dabei die id des Objekts: Diese kopieren wir und nutzen sie im nächsten Schritt in einem POST Graph-Befehl, wie in der folgenden Abbildung dargestellt.

[![EntraBackup_2](/MyPics/2025-08-30-DeletedEntraObjects_2.png)](/MyPics/2025-08-30-DeletedEntraObjects_2.png){:target="_blank"}
Figure 3: Recover specific EA with Graph Explorer
{:.figcaption}

## Enterprise Application ist nicht gleich Enterprise Application

Bei Enterprise Applications gibt es eine wichtige Besonderheit: Es wird unterschieden zwischen Single-Tenant- und Multi-Tenant-Apps.

  - Single-Tenant-App: Wird eine Anwendung selbst erstellt, liegen sowohl die App Registration als auch die Enterprise Application (Service Principal) im eigenen Tenant.
  - Multi-Tenant-App: Handelt es sich um eine Anwendung eines Drittanbieters, etwa aus der Galerie, wird diese von vielen Tenants genutzt. In diesem Fall verbleibt die App Registration im ursprünglichen Quell-Tenant des Anbieters, und jeder konsumierende Tenant erhält lediglich seinen eigenen Service Principal (Enterprise Application).

Warum ist das relevant für das Wiederherstellen von Anwendungsobjekten?
Beim Restore über die Deleted Items API unterscheidet Microsoft wie folgt:

  - Handelt es sich um eine Multi-Tenant-App, lässt sich der zugehörige Service Principal problemlos wiederherstellen, wie in den Abbildungen oben gezeigt, mit einem POST und der entsprechenden ID gegen die API.
  - Bei Single-Tenant-Apps sieht die Sache anders aus, hier muss man genauer hinschauen, da sowohl App Registration als auch Service Principal betroffen sein können.

Bei einer Single-Tenant-Anwendung funktioniert dieses Vorgehen nicht. In diesem Fall wird beim Löschen auch immer die zugehörige App Registration im eigenen Tenant entfernt. Eine Wiederherstellung über die Graph-API, wie zuvor bei Multi-Tenant-Enterprise-Applications gezeigt, ist hier nicht möglich.

Stattdessen muss die App Registration direkt aus dem Papierkorb wiederhergestellt werden. Mit ihr erscheint automatisch auch die verknüpfte Enterprise Application wieder.

## Hard Deleted – und nun?
Unabhängig vom Objekttyp gilt: Ein Objekt, das einen Hard Delete erfahren hat, ist endgültig aus dem Tenant entfernt und damit auch nicht wiederherstellbar. Wichtig zu wissen: **Auch der Microsoft Support kann hier nicht helfen.**

Die konkreten Auswirkungen hängen jedoch stark vom betroffenen Objekttyp ab. Beginnen wir mit den Benutzern und Gruppen.

### Benutzer und Gruppen
Natürlich kann man im Vorfeld Exporte anlegen, um Benutzer oder Gruppen nach einem Verlust anhand dieser Informationen wiederherzustellen. Allerdings erhalten die Objekte dabei immer eine neue Objekt-ID, selbst dann, wenn UPN oder andere Eigenschaften identisch zum ursprünglichen Objekt sind.

Genau hier liegt das Problem: In Entra ID erfolgen alle Referenzen über die Objekt-ID. Wird also eine Gruppe in einer Conditional-Access-Policy als Exclude genutzt oder ein Benutzer explizit in den Scope aufgenommen, helfen neu angelegte Objekte nicht weiter, ihre neue ID ist den Richtlinien schlicht unbekannt.

### Was ist mit synchronisierten Objekten?
Das Gleiche gilt auch für synchronisierte Objekte.

**➜ Beispiel:** Wird ein synchronisierter Benutzer in Entra gelöscht, landet er zunächst im Papierkorb. Bei der nächsten Synchronisation wird er von dort automatisch wiederhergestellt.

Wird der Benutzer jedoch gelöscht und zusätzlich aus dem Papierkorb entfernt (Hard Delete), erzeugt die nächste Synchronisation ein komplett neues Objekt. In Entra zugewiesene Elemente, etwa Lizenzen, sind dann nicht mehr mit diesem neuen Objekt verknüpft.

Um den Schaden im Ernstfall möglichst gering zu halten, sind proaktive Maßnahmen notwendig, etwa saubere Dokumentation, regelmäßige Exporte oder andere Sicherungsschritte.
Gerade für besonders wichtige Benutzer oder Gruppen empfiehlt es sich, rechtzeitig Schutzmechanismen zu etablieren, um einem Verlust vorzubeugen.

Da dieses Thema sehr weitreichend ist, werde ich in einem separaten Beitrag noch ausführlicher darauf eingehen.
An dieser Stelle möchte ich den Entra Exporter vorstellen, mit dem ich selbst gute Erfahrungen gesammelt habe. Er ist auch bereits in meinem Artikel zu CA-Policies und Backup-Strategien [Link](https://nothingbutcloud.net/2025-06-22-Backup-CA-Policies/) Thema.

**➜ Wichtig:**
Der Entra Exporter ist kein klassisches Backuptool, sondern ein Werkzeug zum Exportieren von Objekten. Die Ergebnisse landen in JSON-Dateien, die wir im Notfall nutzen können, um verlorene Objekte oder Konfigurationselemente in Entra gezielt wiederherzustellen.
Gerade bei bestimmten Objekten, wie zum Beispiel Administrative Units, kann das äußerst hilfreich sein.

### Beispiel: Rekonstruieren einer hard deleted AU mit EntraExport JSON
Bei Administrative Units oder Enterprise Applications spielt die Objekt-ID in der Regel keine so große Rolle, zumindest solange sie nicht explizit referenziert werden. Bei Sicherheitsgruppen ist das deutlich häufiger der Fall, bei AUs dagegen eher selten.

Schauen wir uns nun die manuellen Schritte zur Wiederherstellung einer Administrative Unit an, die hard deleted wurde und daher komplett aus einem EntraExporter-Dump neu erstellt werden muss.

Die JSON-Dateien, die für die manuelle Rekonstruktion einer Administrative Unit benötigt werden, legt der EntraExporter in folgender Ordnerstruktur ab.

Grundsätzlich erstellt der EntraExporter für alle Objekttypen vergleichbare Ordner und Unterordnerstrukturen.
Die nachfolgende Abbildung zeigt hier exemplarisch nur die Inhalte für die Administrative Units.“

[![EntraBackup_3](/MyPics/2025-08-30-DeletedEntraObjects_3.png)](/MyPics/2025-08-30-DeletedEntraObjects_3.png){:target="_blank"}
Figure 4: Example Folderstructure generated by EntraExporter
{:.figcaption}

Im Folgenden schauen wir uns an, was nötig ist, um eine hard-deletete Administrative Unit (AU) mit den Informationen aus einem EntraExporter-Dump neu aufzubauen.

  1. AU neu anlegen. Der Name ist zunächst egal. Wichtig: In den Eigenschaften die Objekt-ID kopieren und für die nächsten Schritte im Graph Explorer bereithalten.
  
  2. Graph Explorer verwenden. Den Request-Bodys entnehmen wir dem JSON im Stammordner des AU-Exports (siehe Abbildung oben) und spielen damit die Eigenschaften wieder ein.

```html
PATCH
https://graph.microsoft.com/v1.0/administrativeUnits/{NEW_AU_ID}

Request Body
{
  "displayName": "New Displayname",
  "description": "All this content is from the JSON which was created with the EntraExporter"
"
}
```

In Aktion schaut es dann so aus:

[![EntraBackup_4](/MyPics/2025-08-30-DeletedEntraObjects_4.png)](/MyPics/2025-08-30-DeletedEntraObjects_4.png){:target="_blank"}
Figure 5: Customizing AU General settings
{:.figcaption}

Im JSON-Export befinden sich auch Attribute, die read-only sind und daher nicht zurückgeschrieben werden können. Diese müssen vor dem Ausführen des PATCH-Befehls aus dem Body entfernt werden, um Fehler zu vermeiden, beispielsweise die Attribute **"id"** oder **"deletedDateTime"**. Es macht schlicht keinen Sinn, diese Werte in das neue Objekt zu übernehmen.

Nach dem Bereinigen der JSON-Datei können wir die AU wiederherstellen. Sie besitzt dann erneut den ursprünglichen Namen und die Description.

In der vom EntraExporter erzeugten Dateistruktur finden sich außerdem die Mitglieder der AU: Für jeden Benutzer wird ein eigenes Unterverzeichnis mit den entsprechenden JSON-Dateien angelegt (siehe Abbildung oben). Aus diesen Dateien entnehmen wir die jeweilige Objekt-ID und ordnen die Benutzerkonten anschließend manuell wieder der AU zu.

```html
POST
https://graph.microsoft.com/beta/$batch

Request Body:
{
  "requests": [
    {
      "id": "1",
      "method": "POST",
      "url": "/administrativeUnits/{NEW_AU_ID}/members/$ref",
      "headers": { "Content-Type": "application/json" },
      "body": { "@odata.id": "<USER_DEVICE_OR_GROUP_OBJECT_ID>" }
    },
    {
      "id": "2",
      "method": "POST",
      "url": "/administrativeUnits/{NEW_AU_ID}/members/$ref",
      "headers": { "Content-Type": "application/json" },
      "body": { "@odata.id": "<ANOTHER_USER_<USER_DEVICE_OR_GROUP_OBJECT_ID>" }
    }
  ]
}
```

[![EntraBackup_5](/MyPics/2025-08-30-DeletedEntraObjects_5.png)](/MyPics/2025-08-30-DeletedEntraObjects_5.png){:target="_blank"}
Figure 6: Add member to AU
{:.figcaption}

Die Behandlung von Members funktioniert bei allen drei Objekttypen identisch: Es genügt, die jeweilige Objekt-ID anzugeben, die im Unterordner Members hinterlegt ist.

Aber was, wenn die Mitgliedschaft dynamic ist…
Ist die Mitgliedschaft dynamisch, wird es sogar noch einfacher: Bereits im ersten Schritt, beim Anlegen bzw. Anpassen der AU, kann die entsprechende Query direkt mitgegeben werden.

```html
PATCH
https://graph.microsoft.com/v1.0/administrativeUnits/{NEW_AU_ID}

Request Body
{
  "displayName": "New Displayname",
  "description": "All this content is from the JSON which was created with the EntraExporter",
  "membershipRule": "(user.displayName -startsWith \"adm\") and (user.dirSyncEnabled -eq true)",
  "membershipType": "Dynamic",
  "membershipRuleProcessingState": "On"
}
```

[![EntraBackup_6](/MyPics/2025-08-30-DeletedEntraObjects_6.png)](/MyPics/2025-08-30-DeletedEntraObjects_6.png){:target="_blank"}
Figure 7: Modify general AU settings with dynamic query
{:.figcaption}

## Fazit
Ein Hard Delete kann gravierende Folgen haben, denn endgültig gelöschte Objekte lassen sich nicht mehr zurückholen.

Das Beispiel mit der Administrative Unit verdeutlicht, wie die einzelnen Elemente zusammenspielen und welche Schritte nötig sind, um ein dauerhaft gelöschtes Objekt wieder aufzubauen. Die gezeigte Vorgehensweise ist exemplarisch, ob mit Graph Explorer, PowerShell oder anderen Methoden: Die Grundprinzipien bleiben gleich. Entscheidend ist, sich rechtzeitig zu fragen: Welche Objekte sind kritisch und wie sichere ich ihre Informationen so, dass ich sie im Ernstfall reproduzieren kann?

Am besten werden allerdings Vorbereitungen getroffen, damit es gar nicht erst zum Verlust wichtiger Konfigurationen kommt.

Restore ist gut – doch noch besser ist es, den Verlust zu verhindern. Wie sich Objekte gezielt schützen lassen, werde ich in einem kommenden Beitrag betrachten

Viel Spaß beim Ausprobieren 😀

Weiterführende Infos von [Microsoft Learn](https://learn.microsoft.com/de-de/entra/architecture/recoverability-overview)

# English Version

## Introduction
In Entra ID, there are different types of objects: classic users and groups, but also configuration objects such as Conditional Access policies or Administrative Units.

These objects can often be deleted faster than you might expect. While some can be easily restored, others are permanently lost.

This raises some key questions:

- What distinguishes recoverable objects from those that are permanently deleted?
- What should you pay particular attention to when deleting, and especially when restoring, specific objects such as applications?
- And most importantly: What measures can be taken in advance to make it possible to “re-create” even hard deleted objects?

Quick note upfront: Since Conditional Access policies are a particularly critical topic, I have already written a dedicated article about them [Link](https://nothingbutcloud.net/2025-06-22-Backup-CA-Policies/). In this current post, we will focus on the other essential objects in Entra ID..

The table below provides the roadmap for this article: we’ll cover the key object types, their specifics, and what to watch out for when deleting or restoring them.

[![EntraBackup_0](/MyPics/2025-08-30-DeletedEntraObjects_0.png)](/MyPics/2025-08-30-DeletedEntraObjects_0.png){:target="_blank"}
Figure 1: Objecttypes and deletion states
{:.figcaption}

When objects in Entra ID are deleted, they end up either in a soft deleted or hard deleted state. This distinction is crucial when it comes to recovery or permanent removal

## Soft deleted – but where’s my recycle bin?
Most objects, once deleted, first end up in the recycle bin (soft deleted) for 30 days and are only permanently removed (hard deleted) after that period expires. A hard delete can also be enforced manually by deleting the object directly from the recycle bin.

If an object is restored within those 30 days, all its properties are available exactly as they were before deletion. This process is relatively straightforward and applies to all objects in the table above that are marked in the Soft Delete column.

However, there are also objects that are technically soft deleted but do not appear in any Entra recycle bin. These include, for example, Administrative Units and Enterprise Applications. Their recovery is only possible via the Deleted Items API, either with PowerShell or directly through Graph Explorer.

The following example shows how to list all deleted Enterprise Applications.

[![EntraBackup_1](/MyPics/2025-08-30-DeletedEntraObjects_1.png)](/MyPics/2025-08-30-DeletedEntraObjects_1.png){:target="_blank"}
Figure 2: List deleted EAs with Graph Explorer
{:.figcaption}

In the figure above, you can see that the list of deleted Enterprise Applications is generated by querying the servicePrincipal endpoint in the Deleted Items API.
If you want to restore an Administrative Unit, you instead use the administrativeUnit endpoint, the process otherwise remains the same.

The crucial part here is the **"id"**: copy this value and use it in the next step in a POST Graph command, as shown in the following figure.

[![EntraBackup_2](/MyPics/2025-08-30-DeletedEntraObjects_2.png)](/MyPics/2025-08-30-DeletedEntraObjects_2.png){:target="_blank"}
Figure 3: Recover specific EA with Graph Explorer
{:.figcaption}

## Enterprise Application is not always Enterprise Application

With Enterprise Applications, there is one important distinction: the difference between single-tenant and multi-tenant apps.

  - Single-tenant app: If you create an application yourself, both the App Registration and the Enterprise Application (Service Principal) reside in your own tenant.
  - Multi-tenant app: If the application comes from a third party, for example, from the gallery, it is used across many tenants. In this case, the App Registration remains in the original source tenant of the provider, while each consuming tenant only receives its own Service Principal (Enterprise Application).

Why is this relevant for restoring application objects?
When using the Deleted Items API for restore, Microsoft makes a distinction here:

  - Single-tenant app: Things look different here and require closer attention, since both the App Registration and the Service Principal may be affected.
  - Multi-tenant app: The associated Service Principal can be easily restored, just as shown in the figures above, by issuing a POST request with the corresponding ID against the API.

For a single-tenant application, this approach does not work. In this case, deleting the application also always removes the associated App Registration in your own tenant. A restoration via the Graph API, as shown previously with multi-tenant Enterprise Applications, is not possible here.

Instead, the App Registration must be restored directly from the recycle bin. Once restored, the linked Enterprise Application will automatically reappear as well.

## Hard Deleted – now what?
Regardless of the object type, once an object has been hard deleted, it is permanently removed from the tenant and cannot be restored. Important to note: **even Microsoft Support cannot help in this case.**

The specific impact, however, depends heavily on the type of object affected. Let’s start with users and groups.


### Users and Groups
Of course, you can create exports in advance to restore users or groups after a loss based on that information. However, the restored objects will always receive a new object ID, even if the UPN or other properties are identical to the original object.

And that’s where the real problem lies: in Entra ID, all references are based on the object ID. So, if a group is used as an exclude in a Conditional Access policy, or if a user is explicitly included in the scope, newly created objects won’t help, because their new ID is simply unknown to those policies.


### What about synchronized objects?
The same applies to synchronized objects.

**➜ Example:** if a synchronized user is deleted in Entra, the account first ends up in the recycle bin. At the next synchronization, it is automatically restored from there.

However, if the user is deleted and then also removed from the recycle bin (hard delete), the next synchronization will create a completely new object. Items assigned in Entra, such as licenses, will no longer be linked to this new object.

To keep the damage as low as possible in such cases, proactive measures are essential, such as proper documentation, regular exports, or other backup steps. Especially for critical users or groups, it’s recommended to establish protective mechanisms early on to prevent loss.

Since this is a very broad topic, I will cover it in more detail in a separate article.
At this point, however, I’d like to introduce the Entra Exporter, which I’ve had very good experience with. It’s also already featured in my article on CA policies and backup strategies [Link](https://nothingbutcloud.net/2025-06-22-Backup-CA-Policies/).


**➜ Important:**
The Entra Exporter is **not** a traditional backup tool, but rather a utility for exporting objects. The results are stored in JSON files, which can be used in an emergency to selectively restore lost objects or configuration elements in Entra.

This can be particularly helpful for certain object types, such as Administrative Units.


### Example: Rebuilding a hard-deleted Administrative Unit using EntraExporter JSON
For Administrative Units or Enterprise Applications, the object ID usually doesn’t play a major role, at least as long as they are not explicitly referenced. For security groups, however, this is much more common, while for AUs it’s rather rare.

Let’s now look at the manual steps for restoring an Administrative Unit that has been hard deleted and therefore needs to be completely recreated from an EntraExporter dump.

The JSON files required for manually reconstructing an Administrative Unit are stored by EntraExporter in the following folder structure.

In general, EntraExporter creates comparable folder and subfolder structures for all object types. The figure below shows this exemplarily for Administrative Units only.

[![EntraBackup_3](/MyPics/2025-08-30-DeletedEntraObjects_3.png)](/MyPics/2025-08-30-DeletedEntraObjects_3.png){:target="_blank"}
Figure 4: Example Folderstructure generated by EntraExporter
{:.figcaption}

In the following, I’ll show what’s required to rebuild a hard-deleted Administrative Unit (AU) using the information from an EntraExporter dump.

  1. Create a new AU. The display name doesn’t matter at this stage. What’s important is to copy the object ID from the properties and keep it ready for the next steps in Graph Explorer.
  2. Use Graph Explorer. The request body are taken from the JSON file in the root folder of the AU export (see figure above), which we then use to restore the AU’s properties.

```html
PATCH
https://graph.microsoft.com/v1.0/administrativeUnits/{NEW_AU_ID}

Request Body
{
  "displayName": "New Displayname",
  "description": "All this content is from the JSON which was created with the EntraExporter"
"
}
```

Here’s what it looks like in action:

[![EntraBackup_4](/MyPics/2025-08-30-DeletedEntraObjects_4.png)](/MyPics/2025-08-30-DeletedEntraObjects_4.png){:target="_blank"}
Figure 5: Customizing AU General settings
{:.figcaption}

The JSON export also contains attributes that are read-only and therefore cannot be written back. These must be removed from the body before running the PATCH command in order to avoid errors, for example, the attributes **"id"** or **"deletedDateTime"**. It simply doesn’t make sense to transfer these values into the new object.

After cleaning up the JSON file, we can restore the AU. It will then once again have its original name and description.

In the directory structure generated by EntraExporter, you will also find the AU’s members: for each user, a dedicated subfolder with the corresponding JSON files is created (see figure above). From these files, we take the respective object ID and then manually reassign the user accounts to the AU.

```html
POST
https://graph.microsoft.com/beta/$batch

Request Body:
{
  "requests": [
    {
      "id": "1",
      "method": "POST",
      "url": "/administrativeUnits/{NEW_AU_ID}/members/$ref",
      "headers": { "Content-Type": "application/json" },
      "body": { "@odata.id": "<USER_DEVICE_OR_GROUP_OBJECT_ID>" }
    },
    {
      "id": "2",
      "method": "POST",
      "url": "/administrativeUnits/{NEW_AU_ID}/members/$ref",
      "headers": { "Content-Type": "application/json" },
      "body": { "@odata.id": "<ANOTHER_USER_<USER_DEVICE_OR_GROUP_OBJECT_ID>" }
    }
  ]
}
```

[![EntraBackup_5](/MyPics/2025-08-30-DeletedEntraObjects_5.png)](/MyPics/2025-08-30-DeletedEntraObjects_5.png){:target="_blank"}
Figure 6: Add member to AU
{:.figcaption}

Handling members works the same way for all three object types: it’s sufficient to provide the respective object ID, which is stored in the Members subfolder.

But what if the membership is dynamic…
If the membership is dynamic, things get even easier: already in the first step, when creating or updating the AU, you can directly include the corresponding query.

```html
PATCH
https://graph.microsoft.com/v1.0/administrativeUnits/{NEW_AU_ID}

Request Body
{
  "displayName": "New Displayname",
  "description": "All this content is from the JSON which was created with the EntraExporter",
  "membershipRule": "(user.displayName -startsWith \"adm\") and (user.dirSyncEnabled -eq true)",
  "membershipType": "Dynamic",
  "membershipRuleProcessingState": "On"
}
```

[![EntraBackup_6](/MyPics/2025-08-30-DeletedEntraObjects_6.png)](/MyPics/2025-08-30-DeletedEntraObjects_6.png){:target="_blank"}
Figure 7: Modify general AU settings with dynamic query
{:.figcaption}

## Summary
A hard delete can have serious consequences, as permanently deleted objects cannot be recovered.

The example with the Administrative Unit illustrates how the individual elements interact and which steps are required to reconstruct a permanently deleted object. The approach shown here is exemplary, whether you use Graph Explorer, PowerShell, or other methods: the core principles remain the same. What really matters is asking yourself in advance: Which objects are critical, and how do I back up their information so I can reproduce them if necessary?

Of course, the best option is to prepare in such a way that you never lose important configurations in the first place.

Restore is good, but preventing loss is even better. In an upcoming post, I will cover how to specifically protect objects to avoid such scenarios.

Enjoy trying it out 😀

... and let me know, when you have hints, problems, questions, using the e-mail option below

Additional reading from [Microsoft Learn](https://learn.microsoft.com/en-us/entra/architecture/recoverability-overview)

{% include  share.html %}


