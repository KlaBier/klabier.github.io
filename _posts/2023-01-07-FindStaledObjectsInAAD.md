---
title: "Find staled and unused objects in Azure AD"
author: Klaus Bierschenk
date: 2023-09-07T09:00:32

layout: list

image:
  path: /MyPics/2022-02-10-Azure-MeetupStuttgart_2.png
---

<figure>
  <img src="/MyPics/2022-02-10-Azure-MeetupStuttgart_2.png" style="width:75%">
  <figcaption>Figure 1: Azure Meetup Teaser</figcaption>
</figure>

Deutscher Text ist weiter unten

## Worum geht es?
Anders als im Active Directory On-Premises bietet das Azure Active Directory Möglichkeiten um Computer oder Benutzerobjekte in das Verzeichnis zu integrieren. Beispielsweise Einladen von Gastbenutzer in den Tenant oder BYOD (Bring your own device) Szenarien. Das ist im Sinne der Benutzererfahrung sehr komfortabel und zeitgemäß, bringt aber den Nachteil mit sich, das im Laufe der Zeit mehr und mehr Identitäten zurückbleiben, die evtl. nicht mehr benutzt werden. Diese Überbleibsel sind überflüssig und stellen obendrein ein Sicherheitsrisiko dar. Aus diesem Grund sind regelmäßige Maßnahmen erforderlich, dies auf ein Minimum zu reduzieren.
Wir schauen uns hier an welche Möglichkeiten es gibt und worauf dabei zu achten ist.

## Computerkonten
Im besten Fall sollten Geräte in einem Lifecycle Prozess wieder deregestriert werden, sobald sie nicht mehr benötigt werden, zum Beispiel bei einem Gerätetausch. Im Alltag sieht das aber anders aus. Geräte werden verloren, gestohlen oder aus einem anderen Grund im bislang benutzten Kontext nicht mehr verwendet.

Unter einem sogenannten "Staled Device" versteht man ein im Azure AD registriertes Gerät, dass aber über eine gewissen Zeitraum nicht mehr in dem Tenant benutzt wurde.
Dies kann zu Irritationen im Helpdesk führen und auch den Sync Prozess von Azure AD Connect Server aufblähen, weil ungenutzte Objkte Teil der Synchronisation sind. Und aus Gründen der Sicherheit haben diese Objekte im Azure AD schon gar nichts verloren.

## Detect staled devices






All my demos were running perfect and stable during the session.

Murphy, thanks for that 😀

I have enjoyed the evening and being a speaker at the [Meetup in Stuttgart](https://www.meetup.com/de-DE/Azure-Stuttgart/events/282805130/)

Please find my slides here: [Session slides](/MySlides/Meetup_Stuttgart_Zero_Trust_with_Identity.pdf)

{% include  share.html %}
