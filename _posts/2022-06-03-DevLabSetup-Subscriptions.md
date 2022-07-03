---
title: "Hybrid identity Testlab: Move Subscriptions between Tenants (Part #2)"
date: 2023-06-03T22:01:32
#search: true
#classes:
#  - wide
layout: post
#permalink: /splash-page/
# find more icons here
# https://fontawesome.com/icons?d=gallery&s=solid&m=free

#header:
#   overlay_image: /MyPics/CP1_1.png
#   overlay_color: "#5e616c"
#   overlay_filter: "0.5"
#   #overlay_filter: rgba(255, 0, 0, 0.5)
#   teaser: /MyPics/CP1_1.png
   
#feature_row:
#  - image_path: /MyPics/CP_Filtergruppen.bmp
#    url: "/MyPics/CP_Filtergruppen.bmp"
#    title: "Placeholder 1"
#    excerpt: "Sample text 1 with **markdown** formatting."
#
#
#  - image_path: "/MyPics/CP_GET_Prinzipal_ID.bmp"
#    title: "Placeholder 2"
#    excerpt: "This is some sample content that goes here with **Markdown** formatting."
#    url: "#test-link"
#    btn_label: "Read More"
#    btn_class: "btn--secondar"
---

In diesem Beitrag möchte ich mit dir die wichtigsten Facetten einer Testumgebung basierend auf Azure und mit Fokus auf ein hybride Setup richten. Das sind einige Aspekte, die ich als wichtig erachte und ich habe zuerst überlegt den Beitrag auf mehrere Artikel zu splitten. nach einigen Überlegungen habe ich mich dazu entschieden alles zentral zu dokumentieren mit einer Navigation der Themen im Gliederungsmenü rechts. Zugegeben, die Themen sind verschieden und vielfältig, vom Transfer einzelner Subscription von einem Test tenant zu einem anderen bis hin zum erneuten Verwenden des gewünschten Anker Attributes ms-DSConsistencyGUID On-Premises, aber für den schnellen Zugriff eignet sich das Gliederungsmenü rechts.

## Free Azure Subscription
Möchtest Du schnell mal etwas mit Azure ausprobieren, mit Fokus auf Infrastruktur Diensten, so bist Du mit einer Free Subscription gut bedient.

https://azure.microsoft.com/en-us/free/

In diesem Beitrag legen Da wir hier aber den Fokus auf ein Identity Lab legen, das im Grunde genommen davon lebt, dauerhaft betrieben zu werden, wird eigentlich eine andere Lösung benötigt, besonders wenn Kostenersparnis auch ein thema ist

Die fre Subscrition ist hier nur der Vollständigkeit halber erwähnt. Sehr schnell kommst Du hier in den bereich von Bezahldiensten und auch elementare Identity Dienste, die Teil einer Premium P2 Lizenz sind, sind hier nicht enthalten. Das bringt uns sehr schnell zu dem Testlab, dass Du eigentlich benötigst, wenn dauerhaft ein Testlab betreiben möchtest und dabei den vollen Funktionsumfang nutzen möchtest

## E5 Developer Tenenat aspects

Seit geraumer Zeit arbeite ich mit dem Microsoft 365 Dev Tenant. 

## Lizense vs. subscription

## transfer subcsriptions between tenants

## Domain name registration

## AAD Connect Setup

## AAD Connect Transfering Rules

## ms-DSConsistencyGUID behavior




<figure class="medium">
  <a href="/MyPics/2021-01-22-ZeroTrust Monitoring_I.png"><img src="/MyPics/2021-01-22-ZeroTrust Monitoring_I.png"></a>
  <figcaption>Exclude BGAs from CAs. A "must" in CA deployment</figcaption>
</figure>


```posh
SigninLogs
| where OperationName == "Sign-in activity" | where UserPrincipalName startswith "BG". 
```
