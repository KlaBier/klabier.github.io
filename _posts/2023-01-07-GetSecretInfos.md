---
title: "The Challenge with Client Secrets in Azure AD"
author: Klaus Bierschenk
date: 2023-01-07T09:00:32

layout: list

image:
  path: /MyPics/2023-01-07-GetSecretInfos_Cover.jpg
---

<figure>
  <img src="/MyPics/2023-01-07-GetSecretInfos_Cover.jpg" style="width:75%">
<!--  <figcaption>Figure 1: This can be any text here</figcaption> -->
</figure>


* this unordered seed list will be replaced by the toc
{:toc}

# English version
To enable app registrations to authenticate themselves to other services, they have the option of adding credentials to them. Either certificates or so-called client secrets are used. Federated credentials are also possible, but the first two are probably the most common in my experience. 

Microsoft recommends certificates. These offer higher security compared to Client Secrets. A Secret is basically nothing more than an "application password", with all the uncertainties that come with passwords. They are hard-coded into applications, posted in MS Teams chats, or sent by mail. In times of zero trust, you really don't want that anymore.
## In general

<figure>
  <img src="/MyPics/2023-01-07-GetSecretInfos_I.png" style="width:100%">
  <figcaption>App Registration Dashboard - showing a Secret</figcaption>
</figure>

So, Microsoft's recommendation to prefer certificates is understandable. In my practice, however, I often see the use of client secrets. Not surprisingly, after all, they are quick and easy to set up and available in no time.
Microsoft offers a wealth of Docs articles that address the authentication of App Registrations. To describe the topic here again in my own words makes little sense. Rather, I feel it is important to point out a particular circumstance that Microsoft does not address in the docs in this way, but which I encounter frequently in practice and which we will now look at. 

## Challenge with Client Secrets
Besides the mentioned disadvantages of passwords, there is a serious disadvantage of Client Secrets. The maximum validity period can individually specified and it has a maximum of 24 months. This is configured when the secret is created. Unfortunately, there is no possibility to inform the owner of the app registration or other mail addressees when the validity period of a client secret is about to end. This would leave time to generate a new one and integrate the new password (the secret) for example on a web page or in the code.

In practice, this means that a secret is configured for an app registration and is logically forgotten. At some point, the validity of the secret ends and an application that used to work suddenly stops working. The search continues until the evil is finally found. And on top of that, it's urgent, because the application is definitely no longer working as it should.

Let's hope that Microsoft will soon integrate the possibility of a notification, similar to the SAML certificate for an enterprise application.

If the threat is known, it can be dealt with in advance in various ways, for example, a SIEM can keep an eye on the expiration dates of the Secrets.

In case this is not available, it might make sense to create an old-fashioned CSV file that generates a list of all app registrations in the tenant. Act instead of react is the motto here. For my activities in the field, I have a small Powershell script that creates two CSV text files:

(a) list with all App Registrations, all Owners and then again per Owner all Secrets.
This is redundant in terms of information, but if the CSV file is imported into Excel, there is a good possibility of filtering in many ways.

<figure>
  <img src="/MyPics/2023-01-07-GetSecretInfos_II.png" style="width:100%">
  <figcaption>Imported CSV with all App Regs, their Secrets and "one" Owner</figcaption>
</figure>

b) A list of all app registrations and all secrets. In this case, however, only one owner.

<figure>
  <img src="/MyPics/2023-01-07-GetSecretInfos_III.png" style="width:100%">
  <figcaption>Imported CSV with all App Regs, their Secrets and "all" Owner</figcaption>
</figure>

It may make sense to regularly create these lists as part of an operational manual, when no other monitoring is available, to see what is happening here with expiring Secrets.

The script is primarily intended as an illustrative material. It is possible to extend it in many ways and to adapt it to your own needs. The integration into an automation runbook, into a logic app with resulting mail dispatch to the mail addresses of the listed owners are only a few examples.

Please find the Powershell Script here: [Script ](https://github.com/KlaBier/Powershell)


# Beitrag in Deutsch
## Grundlegendes
Damit App Registrations sich gegenüber anderen Diensten authentifizieren können, verfügen sie über die Möglichkeit ihnen Credentials hinzuzufügen. Dabei kommen entweder Zertifikate zum Einsatz oder sogenannte Client Secrets. Federated Credentials sind ebenfalls möglich, aber die beiden erstgenannten sind nach meiner Erfahrung wohl die am gebräuchlichsten. 

Microsoft empfiehlt Zertifikate. Diese bieten höhere Sicherheit gegenüber den Client Secrets. Ein Secret ist im Grunde genommen nichts anderes als ein "Applicationspassword", mit allen Unsicherheiten, die es im Zusammenhang mit Passwörtern gibt. Sie werden in Anwendungen hart codiert eingefügt, in MS Teams Chats geposted oder per Mail versendet. Das möchte man in Zeiten von Zero Trust eigentlich nicht mehr haben.

<figure>
  <img src="/MyPics/2023-01-07-GetSecretInfos_I.png" style="width:100%">
  <figcaption>App Registration Dashboard - einzelnes Secret</figcaption>
</figure>

Also, Microsofts Empfehlung, Zertifikate vorzuziehen, ist nachvollziehbar. In meiner Praxis erlebe ich aber häufig die Verwendung von Client Secrets. Wen wundert es, schließlich sind diese schnell und unkompliziert angelegt und im Nullkommanix verfügbar.
Microsoft bietet eine Fülle an Docs Artikeln, die die Authentifizierung von App Registrations thematisieren. Das Thema hier mit meinen Worten erneut zu beschreiben, macht wohl wenig Sinn. Mir ist es vielmehr wichtig auf einen besonderen Umstand hinzuweisen, den Microsoft so nicht in den Beiträgen thematisiert, der mir aber in der Praxis häufig begegnet und den wie wir uns jetzt anschauen. 

## Herausforderung bei Client Secrets
Neben den genannten Nachteilen von Passwörtern gibt es einen gravierenden Nachteil von Client Secrets. Die maximale Gültigkeitsdauer eines manuell hinzugefügten Secrets liegt bei 24 Monaten. Diese wird angegeben, wenn das Secret erstellt wird. Leider besteht nicht die Möglichkeit den Owner der App Registration oder andere Mailadressaten darüber zu informieren, wenn der Zeitraum für die Gültigkeit eines Client Secret alsbald endet. Somit bliebe Zeit einen neuen zu generieren und das neue Passwort (den Secret) beispielsweise auf einer Webseite oder im Code zu integrieren.

In der Praxis sieht das dann oft so aus, dass ein Secret für eine App Registration konfiguriert ist und logischerweise in Vergessenheit gerät. Irgendwann endet dann die Gültigkeit und eine Anwendung, die bislang funktionierte, funktioniert plötzlich nicht mehr. Die Sucherei geht los, bis das Übel endlich gefunden ist. Und obendrein ist Eile ist geboten, da die Anwendung definitiv nicht mehr funktioniert, wie sie sollte.

Bleibt zu hoffen, dass Microsoft hier bald die Möglichkeit einer Benachrichtigung integriert, ähnlich wie dies beispielsweise bei einem SAML Zertifikat für eine Enterprise Application der Fall ist

Wenn man die drohende Gefahr kennt, lässt sich im Vorfeld verschiedenartig damit umgehen, beispielsweise kann ein SIEM ein Auge auf die Ablaufdaten der Secrets werfen.

Für den Fall, das dies nicht vorhanden ist, kann aber eventuell das Erstellen einer altmodische CSV-Datei Sinn machen, die eine Liste aller App Registrations im Tenant generiert. Agieren statt regieren ist hier die Devise. Aus meiner täglichen Arbeit habe ich ein kleines Powershell Script, das zwei CSV-Textdateien erstellt:

a)  Liste mit allen App Registrations, allen Ownern und dann wiederum pro Owner alle Secrets
Das ist von den Infos her gesehen zwar redundant, aber wird die CSV-Datei in Excel importiert, besteht so eine gute Möglichkeit der Filterung in vielerlei Ausprägung


<figure>
  <img src="/MyPics/2023-01-07-GetSecretInfos_II.png" style="width:100%">
  <figcaption>Importierte CSV-Datei mit allen Secrets und "einem" Owner</figcaption>
</figure>

b) Eine Liste aller App Registrations und allen Secrets. In diesem Fall aber nur ein Owner.

<figure>
  <img src="/MyPics/2023-01-07-GetSecretInfos_III.png" style="width:100%">
  <figcaption>Importierte CSV-Datei mit allen Secrets und "allen" Ownern</figcaption>
</figure>

Es kann Sinn machen regelmäßig diese Listen dokumentiert als Teil eines Betriebshandbuches für das Azure AD zu erstellen, wenn keine weitere Überwachung verfügbar ist, um zu sehen was der Status in Bezug auf ablaufende Secrets im Tenant ist.

Das Script ist primär als Anschauungsmaterial gedacht. Es besteht die Möglichkeiten es in vielerlei Hinsicht auszubauen und es an eigene Bedürfnisse anzupassen. Die Integration in ein Automation Runbook, in eine Logic App mit daraus resultierender Mailversand, an die Maildressen der gelisteten Owner, sind nur einige Beispiele

[Link zu dem Powershell Script ](https://github.com/KlaBier/Powershell/tree/main/AAD_Secret_Infos)

Cover image Stefan Coders from Pixabay

{% include  share.html %}
