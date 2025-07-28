---
title: "Conditional Access Meets AI – My First Experience with the CA Optimization Agent"
author: Klaus Bierschenk
date: 2025-07-28T08:00:32

layout: list

image:
  path: /MyPics/2025-07-28-CA-PolicyAgent1-Cover.png
---

<figure>
  <img src="/MyPics/2025-07-28-CA-PolicyAgent1-Cover.png" style="width:75%">
</figure>


* this unordered seed list will be replaced by the toc
{:toc}

Warm welcome to my new blog post!

**For English**, please jump directly to the [English version further down](https://nothingbutcloud.net/2025-07-28-CA-PolicyAgent1/#english-version)

Ich habe mich heute ganz bewusst entschieden, diesen Beitrag zuerst auf Deutsch zu schreiben; einfach weil es meine Muttersprache ist, ich sie sehr schätze, und ich meine Gedanken darin abseits von KI und generiertem Content direkter und natürlicher ausdrücken kann.
Da ich aber auch viele internationale Kontakte und Leser habe, und die Technologien, über die ich schreibe, ohnehin aus dem englischsprachigen Raum stammen, stelle ich den Inhalt weiter unten zusätzlich auch auf Englisch zur Verfügung.
So kann jeder den Beitrag in der Sprache lesen, in der er oder sie sich am wohlsten fühlt – ganz ohne Übersetzungstool

# Deutsche Version

## Einstieg
Jetzt wird es smart in Sachen CA-Policies und Administration: Microsoft hat den Conditional Access Optimization Agent zuerst in der Public Preview und kurz darauf als GA veröffentlicht. In diesem Beitrag teile ich meinen ersten Eindruck

Der **CA-Policiy Optimization Agent** macht genau das, was der Name sagt, er analysiert die CA-Policies und liefert dann Infos zum Setup, macht Vorschläge mit Verbesserungen und ist, wie ich finde, ein wertvoller Baustein in Richtung automatisierter Zero-Trust-Maßnahmen. Hierdurch erhält ein Adminteam einen neuen digitalen Kollegen zur Verstärkung

Um zu testen wie sich der Agent "anfühlt" habe ich meine CA-Policy Landschaft angepasst und den Agent mit folgendem konfrontiert:

  - Ich habe in meinem Tenant die Legacy Authentifizierungspolicy ausgeschaltet
  - Eine Gruppe erstellt und diese in der All Users MFA Policy excluded
  - Einen neuen Benutzer erstellt, der Mitglied dieser Gruppe ist
  - Eine CA-Policy dupliziert, die das gleiche macht, wie eine bereits vorhandene Policy

Und dann schauen wir mal, was passiert …

An dieser Stelle, nochmal der Hinweis zu einem [früheren Beitrag von mir](https://nothingbutcloud.net/2025-01-16-SCU_ON_OFF/), in dem beschreibe ich den dosierten Umgang mit Security Copilot und der Bereitstellung von SCUs.
Läuft in deinem Tenant nämlich Security Copilot nicht, erhältst Du beim Aufruf des CA Policy Optimization Agenten folgende Fehlermeldung:



[![CA-Agent_1](/MyPics/2025-07-28-CA-PolicyAgent1-1.png)](/MyPics/2025-07-28-CA-PolicyAgent1-1.png){:target="_blank"}
Figure 1: Security Copilot required for CA-Policy Optimization Agent
{:.figcaption}

Also, ohne Security Copilot geht gar nichts, das ist auch verständlich, weil der Agent darauf basiert

## Setup
Der Agent ist mit einigen wenigen Klicks installiert, vorausgesetzt du bist Security Administrator oder Global Admin. Es gibt noch eine Liste mit Voraussetzungen und Limitierungen. Schau hierfür gerne bei Microsoft nach, sie haben das wunderbar und umfassend hier dokumentiert

[Microsoft Entra Conditional Access optimization agent with Microsoft Security Copilot
](https://learn.microsoft.com/en-us/entra/identity/conditional-access/agent-optimization)

## An die Arbeit
Der Agent läuft einmal am Tag oder auf Knopfdruck. Hast Du umfangreiche Arbeiten an deiner CA-Infrastruktur vorgenommen, lass deinen digitalen Kollegen gerne auf Kommando seinen Job machen und schicke ihn per Knopfdruck an die Arbeit.

In meinem Lab-Tenant mit 14 CA-Policies benötigt der Agent für eine Analyse knapp zwei Minuten

Danach präsentiert er mir folgendes Ergebnis:

[![CA-Agent_2](/MyPics/2025-07-28-CA-PolicyAgent1-2.png)](/MyPics/2025-07-28-CA-PolicyAgent1-2.png){:target="_blank"}
Figure 2: CA-Policy Optimization Agent - Result and overview
{:.figcaption}

Anders aufbereitet werden die Infos in der Activity Map, eine weitere Darstellung, die Ergebnisse zu dem letzten Run liefert. Die Darstellung hier finde ich eigentlich sehr gelungen. Besonders der Blick auf die Drifts zeigt wie der Agent systematisch arbeitet

[![CA-Agent_3](/MyPics/2025-07-28-CA-PolicyAgent1-3.png)](/MyPics/2025-07-28-CA-PolicyAgent1-3.png){:target="_blank"}
Figure 3: CA-Policy Optimization Agent - Summary and Agent activity
{:.figcaption}

Richtig spannend wird es bei den Suggestions. Eine Liste an Maßnahmen zeigt etwaige GAPs im CA-Policy Setup mit Hinweis darauf wie diese zu schließen sind.


[![CA-Agent_4](/MyPics/2025-07-28-CA-PolicyAgent1-5.png)](/MyPics/2025-07-28-CA-PolicyAgent1-5.png){:target="_blank"}
Figure 4: CA-Policy Optimization Agent - Suggestions
{:.figcaption}


Hier wird aufgelistet welche Maßnahmen vorgeschlagen werden. In meinem Fall hat der Agent einen guten Job gemacht und die fehlende Legacy Policy angemahnt und auch der neu angelegten Benutzer, der nicht im Bereich wichtiger Einstellungen wie MFA oder SignInRisk Policies liegt, ist hier ein Thema.

Die doppelten Konfigurationen hat unser intelligenter, digitaler Freund allerdings übersehen. Hier will ich aber nicht zu kritisch sein. Meine Tests waren eher ad hoc. Manche Dinge benötigen einen Moment und ich möchte nicht ausschließen, dass diese Dopplungen bei einem späteren Run angeprangert würden, eventuell wenn die Policy Signale verarbeitet hat.

Es zeigt auch die Arbeitsweise: Der Agent läuft wie erwähnt 1x am Tag und dabei liegt das Augenmerk immer auf neue Policies, Benutzer und Applikationen die ein GAP haben. Es wird also nicht die ganze Infrastruktur betrachtet, was auch genauso sinnvoll ist. Meist sind es Neuerungen oder Änderungen die ein Problem entstehen lassen

In den Optionen lässt sich einiges konfigurieren. Standardmäßig ist es so, dass fehlende Policies im Report Only Mode erstellt werden.

[![CA-Agent_5](/MyPics/2025-07-28-CA-PolicyAgent1-4.png)](/MyPics/2025-07-28-CA-PolicyAgent1-4.png){:target="_blank"}
Figure 5: CA-Policy Optimization Agent - New CAs created in Read-Only-Mode
{:.figcaption}


Und für mehr Klarheit sorgen Infos zu der neuen CA-Policy

[![CA-Agent_6](/MyPics/2025-07-28-CA-PolicyAgent1-6.png)](/MyPics/2025-07-28-CA-PolicyAgent1-6.png){:target="_blank"}
Figure 6: CA-Policy Optimization Agent - Insights and commentary on specific policies
{:.figcaption}

Die ganze Power von AI wird in den Optionen bei den Custom Instructions deutlich. Hier kann die Arbeitsweise des Agenten in natürlicher Sprache angepasst werden. Quasi Instruktionen, die der Agent bei seiner Analyse beherzigen soll

[![CA-Agent_7](/MyPics/2025-07-28-CA-PolicyAgent1-7.png)](/MyPics/2025-07-28-CA-PolicyAgent1-7.png){:target="_blank"}
Figure 7: CA-Policy Optimization Agent - Use custom instruction to set agent behavior
{:.figcaption}

## Fazit
Abschließend sei noch erwähnt, das der Agent nichts an existierenden Policies ändert. Er erstellt auch keine Policies die enabled sind. Die maximalen Änderungen sind neue Policies im Read-Only Mode

Der neue Optimization Agent ist eine willkommene Verstärkung für jedes Admin-Team, mit Potenzial, sich zum festen Bestandteil einer Zero-Trust-Strategie zu entwickeln. Aber auch hier gilt: Vertrauen ist gut, Test ist besser

Viel Spaß beim Ausprobieren 😀

# English Version

## Getting Started

Things are getting smart when it comes to Conditional Access policies and administration: Microsoft has released the Conditional Access Optimization Agent, first in Public Preview and shortly after as General Availability (GA). In this post, I’d like to share my first impressions.

The CA Policy Optimization Agent does exactly what the name suggests: it analyzes your CA policies, provides insights into your setup, suggests improvements, and—if you ask me—is a valuable step toward more automated Zero Trust practices. For any admin team, this means gaining a digital assistant to help with daily operations.

To test how the agent “feels” in action, I made a few changes to my CA policy landscape and challenged the agent with the following:
  - I disabled the legacy authentication policy in my tenant
  - I created a group and excluded it from the All Users MFA policy
  - I created a new user and added them to that group
  - I duplicated an existing CA policy that does exactly the same as another one

Let’s see what happens next …

At this point, just a quick reminder of a [previous post of mine](https://nothingbutcloud.net/2025-01-16-SCU_ON_OFF/). Here I explain the importance of using Security Copilot and provisioning SCUs in a measured way.

If Security Copilot is not running in your tenant, you’ll see the following error message when trying to launch the CA Policy Optimization Agent:

[![CA-Agent_1](/MyPics/2025-07-28-CA-PolicyAgent1-1.png)](/MyPics/2025-07-28-CA-PolicyAgent1-1.png){:target="_blank"}
Figure 1: Security Copilot required for CA-Policy Optimization Agent
{:.figcaption}

So, without Security Copilot, nothing works, which makes sense, since the agent is built on top of it.

## Setup
The agent can be installed with just a few clicks — assuming you’re a Security Administrator or Global Admin. There’s a list of prerequisites and limitations, so be sure to check Microsoft’s documentation, which covers it all clearly and thoroughly.

[Microsoft Entra Conditional Access optimization agent with Microsoft Security Copilot
](https://learn.microsoft.com/en-us/entra/identity/conditional-access/agent-optimization)

## Time to Get to Work

The agent runs once a day or on demand. If you’ve made significant changes to your Conditional Access infrastructure, feel free to send your digital colleague to work with the press of a button.

In my lab tenant with 14 CA policies, the agent took just under two minutes to complete an analysis.

After that, it presented the following results:



[![CA-Agent_2](/MyPics/2025-07-28-CA-PolicyAgent1-2.png)](/MyPics/2025-07-28-CA-PolicyAgent1-2.png){:target="_blank"}
Figure 2: CA-Policy Optimization Agent - Result and overview
{:.figcaption}

The information is also presented differently in the Activity Map, an additional view that shows the results of the most recent run. I find this representation quite well done. In particular, the view of drifts demonstrates how systematically the agent works.

[![CA-Agent_3](/MyPics/2025-07-28-CA-PolicyAgent1-3.png)](/MyPics/2025-07-28-CA-PolicyAgent1-3.png){:target="_blank"}
Figure 3: CA-Policy Optimization Agent - Summary and Agent activity
{:.figcaption}

Things get really interesting with the Suggestions. A list of recommended actions highlights potential gaps in the CA policy setup and provides guidance on how to close them.

[![CA-Agent_4](/MyPics/2025-07-28-CA-PolicyAgent1-5.png)](/MyPics/2025-07-28-CA-PolicyAgent1-5.png){:target="_blank"}
Figure 4: CA-Policy Optimization Agent - Suggestions
{:.figcaption}

The suggested actions are listed here. In my case, the agent did a good job: it flagged the missing legacy policy and pointed out the newly created user who isn’t covered by key settings like MFA or sign-in risk policies.

However, our intelligent digital assistant missed the duplicate configurations. I don’t want to be too critical here. My tests were rather ad hoc. Some things take time, and I wouldn’t rule out the possibility that these duplicates might be flagged in a later run, possibly once the policy signals have been processed.

This also shows how the agent works: as mentioned, it runs once a day and focuses on newly introduced policies, users, and applications with potential gaps. It doesn’t scan the entire infrastructure—which makes sense. It’s usually new or changed elements that introduce problems.

There are several configuration options. By default, missing policies are created in Report-only mode.

[![CA-Agent_5](/MyPics/2025-07-28-CA-PolicyAgent1-4.png)](/MyPics/2025-07-28-CA-PolicyAgent1-4.png){:target="_blank"}
Figure 5: CA-Policy Optimization Agent - New CAs created in Read-Only-Mode
{:.figcaption}

And for more clarity, additional information about the new CA policy is provided.


[![CA-Agent_6](/MyPics/2025-07-28-CA-PolicyAgent1-6.png)](/MyPics/2025-07-28-CA-PolicyAgent1-6.png){:target="_blank"}
Figure 6: CA-Policy Optimization Agent - Insights and commentary on specific policies
{:.figcaption}

The full power of AI becomes apparent in the options under Custom Instructions. Here, you can tailor the agent’s behavior using natural language, essentially giving it instructions to consider during its analysis.

[![CA-Agent_7](/MyPics/2025-07-28-CA-PolicyAgent1-7.png)](/MyPics/2025-07-28-CA-PolicyAgent1-7.png){:target="_blank"}
Figure 7: CA-Policy Optimization Agent - Use custom instruction to set agent behavior
{:.figcaption}

## Conclusion

Finally, it should be noted that the agent does not change any existing policies. It also does not create any enabled policies. The maximum changes it makes are new policies in read-only mode.

The new Optimization Agent is a welcome addition to any admin team, with the potential to become a core component of a Zero Trust strategy. But as always: trust is good, testing is better.

Enjoy trying it out 😀

... and let me know, when you have hints, problems, questions, using the e-mail option below

{% include  share.html %}


