---
title: "Ask me anything - Synchronization"
date: 2022-07-10T12:34:32
layout: list

image:
  path: /MyPics/2022-07-12-AMA_AADConnect_I.png
---

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_I.png" style="width:75%">
  <figcaption>Figure 1: AMA AAD Connect Teaser</figcaption>
</figure>


* this unordered seed list will be replaced by the toc
{:toc}

This was a German Session. But I have seen a couple of english speaking attandies in the audience, please find the english content below.

Azure AMA zum Thema Synchronisation, zusammen mit Gregor Reimling vom Azure Metup Bonn. Das waren spannende Fragestellungen rund um den endenden Support von AAD Connect v1 und dem Update auf die Version 2 des Sync Boliden. Wir hatten aber noch weitere Themen, die ich ebenfalls sehr spannend fand. Eine Liste der Fragen und Antworten findet ihr in diesem Beitrag

Die Session wurde aufgezeichnet und ist auf [YouTube](https://www.youtube.com/watch?v=W2YGSMEvaKA&t=1s){:target="_blank"}

**Hinweis**

Wenn in diesem Beitrag von AADC Server die Rede ist, ist immer die aktuelle Variante der Synchronisation, der On-Premises Bolide, gemeint. Wann immer Cloud Sync (Azure AD Cloud Sync) angesprochen wird, ist die neue Technologie der Synchronisation gemeint. Die macht mit Einschränkungen das Gleiche, arbeitet aber unter der Haube komplett anders. In den ersten Versionen (vor GA) wurde Cloud Sync mit Azure AD Cloud Provisioning betitelt.

# Deutsche Inhalte
## Configuration, Setup
### Wo finde ich die Version von AADConnect Server?
Die Frage ist berechtigt, denn lange Zeit war die Version nur an wenigen Stellen und nicht wirklich intuitiv zu finden.

Ein Ort war (und ist) in "Apps & features", wie folgende Abbildung zeigt:

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_II.png" style="width:90%">
  <figcaption>Figure 2: Die Version von AADC ist unter anderem in "Apps & features" zu finden</figcaption>
</figure>

Zwischenzeitlich hat Microsoft die Verion auch im "Synchronization Service Manager" integriert, wo sie meiner Meinung nach auch hingehört. Dort ist sie im Menü "Help" und dann "About ..." zu finden

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_III.png" style="width:100%">
  <figcaption>Figure 3: Version findet sich seit Neuestem auch im Sync Manager ...</figcaption>
</figure>

... oder mittlerweile auch im AAD Portal

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_IV.png" style="width:100%">
  <figcaption>Figure 4: ... oder im Azure AD Portal</figcaption>
</figure>
 	
**Hinweis**

Sollten die erforderlichen Berechtigungen fehlen, um den "Synchronization Manager" auszuführen, lässt sich die Version auch im Azure AD ermitteln. Und zwar im Bereich Connect Health, wie oben abgebildet. Sollte das aus irgendeinem Grund nicht gehen, weil zum Beispiel der Health Agent nicht registriert ist oder keine Infos liefert, bleibt immer noch "Apps & features".

Die Version ist bei AADC wichtig, da es häufiger Updates gibt und es von Bedeutung sein kann, zu wissen, was aktuell installiert ist. Umso verwirrender war es bislang, die Version nicht dort zu finden, wo sie intuitiv vermutet wird. Aber umso besser jetzt, dass MS dies nun an den richtigen Stellen integriert hat.

Solltest Du eine ältere Version von AADC einsetzen, und im AAD Connect Health Dashboard und auch im Synchronization Manager" nicht fündig werden, ist "Apps & features" ebenfalls die beste Wahl für Dich, um die version herauszufinden

### Welche Möglichkeiten von Hochverfügbarkeit und Lastenausgleich gibt es?
Cloud Sync bietet die Möglichkeit mehrere Agenten On-Premises auf Memberserver zu installieren. Diese Server bieten Hochverfügbarkeit, aber keinen Lastenausgleich. Mit anderen Worten: fällt ein Server aus, übernimmt ein anderer. Müssen x tausend Identitäten synchronisiert werden, geschieht dies durch einen Agenten (Server), ohne dass sich mehrere die Last teilen werden.

Diesen Lastenausgleich gibt es bei AADC nicht. Genauso gibt es keine "richtige" Möglichkeit der Hochverfügbarkeit. Einzig die Implementierueng eines oder mehrere sogenannter "Staging" Server ist eine Option, für ein Ausfallszenario vorzusorgen. Bei AADC ist immer ein Server der aktive Server. Das heisst, er schreibt (exportiert) Änderungen. Staging Server machen das Gleiche, jedoch exportieren sie nicht, auch machen sie keinen Password Hash Sync. In ihrer Datenbank ist aber der komplette Bestand an Identitäten enthalten. Fällt der aktive Servers aus, wird manuell aus dem Staging Server der aktive Server und ein kompletter Import aller zu synchronisierenden Objekte ist nicht notwendig, da diese Infos schon in der Datenbank liegen. Somit bedeutet dies eine Zeitersparnis, da quasi nur der Export aktiviert wird.
Das ist aber nicht wirklich Hochverfügbarkeit.

Das Thema ist spannend und herausfordernd. Da es einige Aspekte gibt, die hier unbedingt beachtet werden müssen. Zum Beispiel hat jeder AADC Server seine eigene Konfiguration, die mit den anderen AADC Servern nicht synchronisiert oder abgeglichen wird, wie das von einem Farm Setup bekannt ist, bei ADFS beispielsweise. Das birgt ein Risiko im Falle eines Wechsels von Staging zu aktivem Server, wenn der Staging Server ein anderes Setup als sein Pendant besitzt. Es gibt hier Maßnahmen und Tools wie dem zu begegnen ist. zum Teil ist dies hier beschrieben [Local components in AADC](https://nothingbutcloud.net/2020-03-21-CP/#local-components){:target="_blank"}

Die ganze Story beschreibt Microsoft in diesem [Docs Artikel](https://docs.microsoft.com/de-de/azure/active-directory/hybrid/how-to-connect-sync-staging-server){:target="_blank"}


### Wobei kann der Azure AD Connect Documenter helfen?
Der Azure AD Connect Documenter ist ein Tool, das nicht Teil des Setups ist und separat installiert werden muss. Die Installation auf einem AADC Server ist einfach und besteht nur aus einem simplen Copy der Dateien aus Github.
Danch wird für den Vergleich ein Dump der Konfiguration auf jedem der AADC Server erstellt, die es zu vergleichen gilt.

```posh
Get-ADSyncServerConfiguration -Path "C:\ExportFolder"
```

Dieser Export jedes Servers, mit allen Unterverzeichnissen, muss sich auf dem Computer befinden, auf dem der Documenter ausgeführt wird. Der wertet diesen aus und erstellt einen HTML Report der farblich die Unterschiede sichtbar macht.

Die Dateien und die Dokumentation sind hier zu finden: [Azure AD Connect Documenter](https://github.com/Microsoft/AADConnectConfigDocumenter){:target="_blank"}

Erwähnenswert ist an dieser Stelle noch, das der "Documenter" dabei helfen kann "Sync Rules" von einem Server zu einem anderen zu transferieren. Der gesamte Vorgang ist zwar manuell, aber der Export mittels generiertem PS1 Script kann auf dem Zielserver ausgeführt werden, ohne das die Connector-ID anzupassen ist, wie das beispielsweise bei einem Transfer mittels Regeleditor der Fall ist. Hierfür ist in dem HTML-Report lediglich die Option "Only show changes" zu aktvieren. Danach erscheint die Möglichkeit "Download Sync Rule Changes Script"

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_V.png" style="width:100%">
  <figcaption>Figure 5: AADC Documenter vergleicht den Export von zwei AADC Servern</figcaption>
</figure>

### Welche Möglichkeiten gibt es für den Export der Config?
Ab Version 1.5.42.0 (July 2020) ist der Export der Konfiguration über den Setup Assistenten von AADC möglich

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_VI.png" style="width:90%">
  <figcaption>Figure 6: AADC Config manager erlaubt Export der Konfiguration für den Import auf einem weiteren Server</figcaption>
</figure>

Beim Einsatz einer älteren Version, die den Export in der GUI des Config Managers nicht unterstützt, wie oben abgebildet, gibt es einen alternativen Weg für den Export. Details sind hier zu finden ["Migrate settings from an existing server"](https://docs.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-import-export-config#migrate-settings-from-an-existing-server){:target="_blank"}

Etwas umständlicher, aber der Weg führt hier über die Powershell genauso zum Ziel

Übrigens erstellt der AADC Config Wizard bei Änderungen ein JSON File mit der Konfiguration, die für den Import auf einem weiteren Server (z.B. als Staging) genutzt werden kann.

(Ordner: C:\ProgramData\AADConnect)

### Was bedeutet es wenn AADC im Azure Health portal unmonitored ist?
Ist ein AADC Server länger als 30 Tage inaktiv, wechselt sein Status im Dashboard von Azure AD Connect Health zu "unmonitored", auch wenn der Server danach wieder gestartet wird, bleibt er "unmonitored"
Das lässt sich beheben, in dem der Health Agent neu installiert wird oder eine Neuregistrierung des Agent mit dem folgendem Cmdlet erfolgt:

```posh
Register-AzureADConnectHealthSyncAgent -AttributeFiltering $true -StagingMode $true
```

### Was sind denn die Unterschiede von Cloud Sync zu AAD Connect Server (Vorteile/Nachteile!)

Azure AD Connect Cloud Sync ist die neuere (Public Preview Okt. 2019) Technology für die Synchronisation und vollständig in das Azure AD Portal integriert. Der Teil, der On-Premises installiert wird, ist gering und beschränkt sich auf einen schlanken Agent. Konfiguration, und alles was damit zusammenhängt, liegt im Azure AD. Der lokale Agent kommuniziert lediglich mit dem Forest und transferiert die Identitäten in Richtung Azure AD.

Im Gegenzug dazu ist AADC Server die seit langem etablierte, On-Premises Server basierte Technologie der Synchronisation. Sie entstammt ursprünglich dem Sync Client von MIIS (miisclient.exe) aus dem Jahre 2007 und ist eine für das Azure AD angepasstes Variante des Sync Client

Ich habe zwei Artikel um Thema Cloud Sync hier in meinem Blog, die auch auf die Unterschiede zwischen den beiden Technologien eingehen:

["First experience with Azure AD Connect Cloud Provisioning"](https://nothingbutcloud.net/2020-03-21-CP/){:target="_blank"}

["More Infos on Azure AD Connect Cloud Provisioning"](https://nothingbutcloud.net/2020-09-24-CP-Advanced/){:target="_blank"}

Ein entsprechender Microsoft Artikel, der auf die Unterschiede eingeht, ist hier zu finden:

["Unterschiede zwischen Cloud Sync und AADC Server"](https://docs.microsoft.com/de-de/azure/active-directory/cloud-sync/what-is-cloud-sync#comparison-between-azure-ad-connect-and-cloud-sync
){:target="_blank"}

### Ist Migration nach Cloud Sync empfohlen, sinnvoll oder möglich?
Beide Technlogien sind von der Architektur her verschieden und bislang ist mir kein Migrationspfad bekannt. Warum auch? Eine intakte AADC Server Implementierung durch eine Cloud Sync Implementierung zu ersetzen macht aktuell keinen Sinn, da AADC Server derzeit noch einen größeren Funktionsumfang besitzt. Die Frage stellt sich somit aktuell nicht. Aber ich bin ziemlich sicher, dass irgendwann, wenn die Funktionen in Cloud Sync zunehmen und ein Einsatz dieser Technologie attraktiver wird, eine Migration von nutzen sein kann und Microsoft einen Migrationspfad beschreiben wird.

## Operations:
### Wie sieht ein Health Check aus?

### ... bei Cloud Sync

Bei Cloud Sync gibt es mehrere Möglichkeiten, dem Service auf dem Zahn zu fühlen. Ein guter und erster Start für einen Check ist das Cloud Sync Dashboard sowohl beim Service aber auch bei den jeweiligen Agenten, die hier aufgelistet sind:


<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_VII.png" style="width:100%">
  <figcaption>Figure 7: "Healthy" als Service Status sieht gut aus</figcaption>
</figure>

Die Option "Logs" in der Auswahl oben bezieht sich übrigens auf die Protokolle der Synchronisation. Hier ist nicht der Status zu den Agenten oder Ähnliches zu finden

Auch wenn Cloud Sync den Schwerpunkt der Administration in Azure hat, so gibt es doch auf den Servern mit installiertem Agent die Möglichkeit den Status zu prüfen und auch gezielt nach Fehlern zu suchen. Meiner Meinung nach aber als eine Option, wenn die Aussagekraft der Fehler in Azure AD nicht schlüssig ist, quasi als nächsten Schritt in der Fehlersuche. 

Das Eventlog ist immer eine gute Quelle um auf einem Server nach dem Rechten zu sehen, so auch bei Cloud Sync. Mit dem Agentsetup werden zwei Protokolle mit installiert, **AgentUpdater** und **ProvisioningAgent** Ersteres dient dem Update, wie der Name auch andeutet, interesseanter ist da schon das Log des Provioning Agenten, das zeigt ob alles rund läuft oder eben nicht.

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_VIII.png" style="width:100%">
  <figcaption>Figure 8: "Das Log des Provioning Agenten gibt Aufschluss ob alles richtig tickt</figcaption>
</figure>

Im Übrigen sind das "Application Log" und auch das "System Log" gleichfalls Orte, die der Agent für die Ablage von Informationen verwendet. Es bietet sich also an sämtliche Logs zu betrachten.

Dann wäre da natürlich noch die Powershell. Mit dem Setup des Agenten wird alles erforderlich bereitgestellt. Mit einigen wenigen Cmdlets ist das Module am Start und bietet somit die Möglichkeit von der Kommandozeile aus aktiv zu werden, wie die nächste Abbildung zeigt:
<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_IX.png" style="width:100%">
  <figcaption>Figure 9: "Das mit installierte Powershell Module für Cloud Sync</figcaption>
</figure>

Für eine noch detailliertere Analyse bieten sich die Trace Logs an. Hier sind übrigens auch die Eventlogs (EVTX files) zu finden, die mit folgendem Cmdlet aus dem Cloud Sync Module generiert werden:

```posh
Export-AADCloudSyncToolsLogs
```
<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_X.png" style="width:100%">
  <figcaption>Figure 10: Die Tracelogs bieten einen noch tieferen Einblick in die Arbeitsweise des Cloud Sync Agenten</figcaption>
</figure>

### ... und bei AADC Server ###

Bei Azure AD Connect Server gibt es mehr Möglichkeiten für einen Health Check, verglichen zu Cloud Sync. Für einen ersten Blick, und um zu prüfen, ob alles passt, macht es Sinn im Azure AD Portal zu starten. Hier wird ein umfassender Status angezeigt und das Health Dashboard ist zu finden im Portal des Azure AD in der Navigation unter **Azure AD Connect** und dann im Bereich von Azure AD Connect über die Auswahl **HEALTH AND ANALYTICS**

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_XI.png" style="width:100%">
  <figcaption>Figure 11: Health Dashboard zeigt den Gesamtstatus von Azure AD Connect Server</figcaption>
</figure>

Da es auch einen Health Agenten für ADFS und einen für die AD Domain Services gibt, lässt sich hier alles gesammelt betrachten, natürlich nur wenn die lokalen Systeme mit dem jeweiligen Agenten ausgestattet sind. Alle Infos zu AADC befinden sich hinter den entsprechenden Quick start links (siehe roter Rahmen in Abbildung 11)

Das Dashboard hier besteht aus einer Vielzahl aktiver Elemente, die per Quicklink zwischen den Bereichen wechselt, andere Einsichten liefert und eine detaillierte Analyse der Health Infos gestattet. Wie am Beispiel der beiden Sever unten bei Auswahl eines der jeweiligen Serverobjekte.

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_XII.png" style="width:100%">
  <figcaption>Figure 12: So soll es aussehen: alles gesund meldet der health Agent von den On-prem AADC Servern</figcaption>
</figure>

Sollte das Azure AD und die Health Dashboards keinen Hinweis zu einer Fehlersituation liefern, besteht auch bei den Azure AD Connect Servern die Möglichkeit das Eventlog zu untersuchen. Das Augenmerk sollte hier auf dem "Application Log" liegen, hier listet der Sync Service akkribisch was Sache ist.

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_XIII.png" style="width:100%">
  <figcaption>Figure 13: Application Event Log liefert detaillierte Infos zum Zustand der Synchronisation</figcaption>
</figure>

Die Powershell bietet perfekte Möglichkeiten um auf einem AADC Server nach dem Rechten zu sehen. Das mächtige PS-Module "adsync" (117 cmdlets) liefert viele Möglichkeiten hierzu. Genauso hilfreich ist das Modul "ADSyncDagnostics" mit dem Cmdlet

```posh
Invoke-ADSyncDiagnostics
```

wie Abbildung 14 zeigt.

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_XIV.png" style="width:100%">
  <figcaption>Figure 14: Powershell eignet sich perfekt für Health Check und Analyse. Hier das Cmdlet "Invoke-ADSyncDiagnostics"</figcaption>
</figure>

### Was ist zu tun wenn localDb zu klein oder nicht mehr ausreichend ist (10GB)?
Beim Setup von Azure AD Connect wird, wenn im Wizard nicht anders angegeben, ein Local DB installiert. Dies ist praktisch und einfach. Local DB ist aber auf eine maximale DB Größe von 10GB beschränkt, was sich viel anhört, aber unter Umständen schnell erreicht ist. Es lässt sich pauschal nicht beziffern, das bei x Usern im Sync die 10GB bei dieser oder jener Anzahl erreicht ist. Das hängt von mehreren Faktoren ab. Aber der wichtigste Faktor ist **wieviele Connectoren (AD Forests) sind Teil der Synchronisation?**

In der Gesamt DB ist schließlich alles enthalten: Metaverse (sämtliche Objekte). Genauso die Objekte der jeweiligen Connectoren, quasi als "Rohdaten Import" für den Abgleich mit der Metaverse.

Um es für die Frage hier auf den Punkt zu bringen, habe ich zuletzt ein AADC Setup mit einer AD Domain gesehen. Hier wurden ca. 500 Gruppen synchronisiert. Keine Computerobjekte. Und es waren um die 30.000 Benutzer vorhanden als die DB Größe die Grenze von 10GB erreichte.

Microsoft beschreibt in [diesem Artikel](https://docs.microsoft.com/de-de/azure/active-directory/hybrid/how-to-connect-install-move-db){:target="_blank"} und auch [hier](https://docs.microsoft.com/de-de/azure/active-directory/hybrid/how-to-connect-install-existing-database){:target="_blank"} die einzelnen Schritte, die beim Verschieben der Datenbank, bzw. beim Wiederverwenden einer vorhandenenen DB, zu tun sind.

Um es hier zusammenfassend auf den Ounkt zu bringen:

- DB (MDF- und LDF- Dateien) auf dem aktuellen Sync Server dem Dateisystem an einen sicheren Ort kopieren bzw. direkt auf den dedizierten, entfernten SQL Server.

- DB dort einbinden

- AADC neu installieren, und wie in den Beitrag aus den Links oben beschrieben, das Setup mit dem Switch **/useexistingdatabase** aufrufen. Im Setup-Wizard besteht dadurch die Option einen SQL Server anzugeben. Im Weiteren wird die dort befindliche DB erkannt und verwendet.

- Bitte unbedingt die Post-Tasks aus den MS Beiträgen beachten!

## Was gibt es beim Update auf V2 zu beachten?
Aus meiner Sicht ist die Aktualisierung auf die v2 unkritisch und kein Problem! Ich habe schon in-place Updates gemacht, würde aber strategisch gesehen einen anderen Weg gehen, wie weiter unten beschrieben. Zunächst einmal einige Eckpunkte rund um das Thema AADC Update auf v2

- v2 enthält keinerlei gravierende Änderungen an den Funktionalitäten und Features. Lediglich die Serverkomponenten wurden an die aktuellen Server Versionen angepasst: Windows Server 2016 und SQL 2019. Eine Gesamtliste der Anforderungen befindet sich hier [Functional changes in 2.0.3.0](https://docs.microsoft.com/en-us/azure/active-directory/hybrid/reference-connect-version-history#2030){:target="_blank"}

- Auch wenn automatisches Update "Set-ADSyncAutoUpgrade -AutoUpgradeState enabled"  eingeschaltet ist, greift das Setting hier nicht. Die Aktualisierung auf v2 ist ein manueller Schritt, der nicht automatisiert ausgeführt wird

- Eine von einem v1 AADC Server erstellter Export der Config kann auf einem neuen v2 Server importiert werden. Die JSON Files sind kompatibel

- Wie oben erwähnt ist ein In-Place Update unterstützt. Da hier aber Altlasten auf dem Server zurückbleiben (SQL Server Komponenten) würd ich zu einer neuen Installation raten. Alte SQL Server Überbleibsel lassen sich zwar manuell entfernen, aber eine neue Version macht hier aus meiner Sicht Sinn.

Wie bereits erwähnt unterstützt das Setup ein In-Place Update. ich würde aber folgende Schritte ausführen um einen Server auf v2 zu aktualisieren:

1. Von aktuellem AADC Server die Config exportieren (AADC Config Wizard)
2. Aktiven AADC Server zum Staging machen
3. Neuen Server (neuer Computername) bereitstellen.
4. TLS 1.2 aktivieren, wenn noch nicht geschehen. Sonst droht eine Fehlermeldung beim Setup von AADC
   [Erzwingen von TLS 1.2 für AADC](https://docs.microsoft.com/de-de/azure/active-directory/hybrid/reference-connect-tls-enforcement#enable-tls-12){:target="_blank"}
5. Installieren von AADC mit v2 und Import der JSON Config von anderem AADC Server (Punkt #1)
6. Server ist Staging. Health Check, ob alles passt. AD Health Dashboard, Eventlog, Synchronization Service Manager Console. Im Regeleditor prüfen ob die Regeln vorhanden sind
7. Ändern von Testdaten On-Premises (Benutzer Telefonnummer, neuen Benutzer anlegen etc.) und manuelle Synchronisation starten. Cmdlet: "Start-ADSyncSyncCycle"
   Werden Änderungen verarbeitet? Prüfen von Pending Exports für den Azure AD Connector Space. Der Server ist Staging. Er wird nichts Richtung Azure AD schreiben
8. Wenn alles ok ist, den neuen Server aktivieren (AADC Config Wizard)
9. Geht bei den Punkten oben irgendetwas schief, kann der zuvor zum Staging Server ausgemusterte Sync Server wieder in Betrieb genommen werden und es besteht Zeit in Ruhe das Problem zu untersuchen.
10. Ist bis hierhin alles in Ordnung (wovon auszugehen ist), sind jetzt der Reihe nach die vorhandenen Staging Server zu aktualisieren. Auch hier gilt wieder, entweder In-Place oder mit einer komplett neuen Maschine

# English content from the AMA

Azure AMA (Ask me anything) on the topic of synchronization, together with Gregor Reimling from Azure Metup Bonn. These were exciting questions around the ending support of AAD Connect v1 and the update to version 2 of the Sync Bolide. However, we had other topics that I also found very exciting. You can find a list of the questions and answers in this post

The session was recorded and is available on [YouTube](https://www.youtube.com/watch?v=W2YGSMEvaKA&t=1s){:target="_blank"}


## Configuration, Setup
### Where can I find the version of AADConnect Server?
The question is justified, because for a long time the version could only be found in a few places and not really intuitively.

One place was (and is) in "Apps & features", as the following figure shows:

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_II.png" style="width:90%">
  <figcaption>Figure 2: The version of AADC can be found in "Apps & features" among others</figcaption>
</figure>

In the meantime, Microsoft has also integrated the Verion in the "Synchronization Service Manager", where I think this is the perfect place. There it can be found in the menu "Help" and then "About ..."

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_III.png" style="width:100%">
  <figcaption>Figure 3: version can now also be found in the Sync Manager ...</figcaption>
</figure>

... or meanwhile also in the AAD Portal

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_IV.png" style="width:100%">
  <figcaption>Figure 4: ... or in the Azure AD Portal</figcaption>
</figure>
 	
**Note**

If the required permissions to run the "Synchronization Manager" are missing, the version can also be determined in Azure AD. This is done in the Connect Health area, as shown above. If for some reason this does not work, for example because the Health Agent is not registered or does not provide any info, there is still "Apps & features" the location to check the version

The version is important with AADC because updates are more frequent and it can be significant to know what is currently installed. So it was all the more confusing not to find the version where it is intuitively assumed. But all the better now that MS has integrated this in the right places.

If you are using an older version of AADC and can't find it in the AAD Connect Health Dashboard and also in the Synchronization Manager, Apps & features is also the best choice for you to find out the version.

### What are the options for high availability and load balancing?
Cloud Sync provides the ability to install multiple agents on-premises on member servers. These servers provide high availability, but no load balancing. In other words, if one server fails, another one takes over. If x thousand identities need to be synchronized, this is done by one agent (server) without to share the load between them.

This load balancing does not exist in AADC. In the same way there is no "real" possibility of high availability. Only the implementation of one or more so-called "staging" servers is an option to provide for a failure scenario. With AADC one server is always the active server. This box writes changes (exports). Staging servers do the same, but they do not run an export, nor do they do password hash sync. However, their database contains the complete set of identities. If the active server fails, the staging server manually becomes the active server and a complete import of all objects to be synchronized is not necessary, because this info is already in the database. Thus this means a time saving, since quasi only the export is activated.
But this is not really high availability.

The topic is exciting and challenging. As there are some aspects that have to be taken into account. For example, each AADC server has its own configuration, which is not synchronized or aligned with the other AADC servers, as is known from a farm setup, with ADFS for example. This poses a risk in case of a change from staging to active server, if the staging server has a different setup than its counterpart. There are measures and tools to counter this, some of which are described here. [Local components in AADC](https://nothingbutcloud.net/2020-03-21-CP/#local-components){:target="_blank"}

Microsoft describes the whole story in this [Docs Artikel](https://docs.microsoft.com/de-de/azure/active-directory/hybrid/how-to-connect-sync-staging-server){:target="_blank"}


### What the Azure AD Connect Documenter can help with?
Azure AD Connect Documenter is a tool that is not part of the built-in tools and it must be installed separately. The installation on an AADC server is simple and only consists of a simple copy of the files from Github.
Then a dump of the configuration on each of the AADC servers is created for comparison by the Documenter.

```posh
Get-ADSyncServerConfiguration -Path "C:\ExportFolder"
```

This dump from each server, with all subdirectories, must be on the computer on which the Documenter is executed. The Documenter evaluates this and creates a HTML report that shows the differences in color.

The files and the documentation can be found here: [Azure AD Connect Documenter](https://github.com/Microsoft/AADConnectConfigDocumenter){:target="_blank"}

It is worth mentioning here that the "Documenter" can help to transfer "Sync Rules" from one server to another. The whole process is manual, but the export using the generated PS1 script can be executed on the target server without having to adjust the connector ID, as is the case with a transfer using the rule editor, for example. To do this, simply activate the option "Only show changes" in the HTML report. After that the option "Download Sync Rule Changes Script" appears.

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_V.png" style="width:100%">
  <figcaption>Figure 5: AADC Documenter compares the export from two AADC servers</figcaption>
</figure>

### hat are the options for exporting the Config?
As of version 1.5.42.0 (July 2020) the export of the configuration is possible via the setup wizard of AADC

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_VI.png" style="width:90%">
  <figcaption>Figure 6: AADC Config manager allows export of configuration for later import on another server</figcaption>
</figure>

When using an older version that does not support export in the Config Manager GUI, as shown above, there is an alternative way to export. Details can be found here ["Migrate settings from an existing server"](https://docs.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-import-export-config#migrate-settings-from-an-existing-server){:target="_blank"}

A bit more cumbersome, but the way leads here via the Powershell just as to the goal

By the way, the AADC Config Wizard creates a JSON file with the configuration when changes are made, which can be used for import on another server (e.g. as staging).
(Ordner: C:\ProgramData\AADConnect)

### What does it mean when AADC is unmonitored in Azure Health portal?
If an AADC server is inactive for more than 30 days, its status in the Azure AD Connect Health dashboard changes to "unmonitored", even if the server is restarted afterwards, it remains "unmonitored".
This can be fixed by reinstalling the Health Agent or re-registering the agent with the following cmdlet:

```posh
Register-AzureADConnectHealthSyncAgent -AttributeFiltering $true -StagingMode $true
```

### What are the differences between Cloud Sync and AAD Connect Server (advantages/disadvantages!)?

Azure AD Connect Cloud Sync is the newer (Public Preview Oct. 2019) technology for synchronization and fully integrated with Azure AD Portal. The part that is installed on-premises is minor and limited to a lean agent. Configuration, and everything related to it, resides in Azure AD. The on-premises agent only communicates with the forest and transfers identities towards Azure AD.

In contrast, AADC Server is the long-established on-premises server-based technology of synchronization. It originates from the Sync Client of MIIS (miisclient.exe) from 2007 and is a variant of the Sync Client adapted for Azure AD.

I have two articles around Cloud Sync here on my blog that also discuss the differences between the two technologies:

["First experience with Azure AD Connect Cloud Provisioning"](https://nothingbutcloud.net/2020-03-21-CP/){:target="_blank"}

["More Infos on Azure AD Connect Cloud Provisioning"](https://nothingbutcloud.net/2020-09-24-CP-Advanced/){:target="_blank"}

A corresponding Microsoft article that goes into detail about the differences can be found here:

["Differences between Cloud Sync and AADC Server"](https://docs.microsoft.com/de-de/azure/active-directory/cloud-sync/what-is-cloud-sync#comparison-between-azure-ad-connect-and-cloud-sync
){:target="_blank"}

### Is migration to Cloud Sync recommended, useful or possible?
Both technologies are different in terms of architecture and so far I am not aware of a migration path. And why? Replacing an intact AADC Server implementation with a Cloud Sync implementation does not currently make sense, since AADC Server currently still has a larger range of functions. So the question does not arise at the moment. But I am quite sure that at some point, when the features in Cloud Sync increase and the use of this technology becomes more attractive, a migration may be useful and Microsoft will describe a migration path.

## Operations:
### What does a Health Check look like?

### ... for Cloud Sync

With Cloud Sync there are several ways to check the service. A good and first start for a check is the Cloud Sync Dashboard, both for the service but also for the respective agents, which are listed here:

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_VII.png" style="width:100%">
  <figcaption>Figure 7: "Healthy" as service status looks good</figcaption>
</figure>

By the way, the option "Logs" in the selection above refers to the logs of the synchronization. Here is not the status to the agents to be found

Even if Cloud Sync has the main focus of administration in Azure, there is still the possibility to check the status on the servers with an installed agent and also to search specifically for errors. In my opinion, however, as an option if the meaningfulness of the errors in Azure AD is not conclusive, virtually as the next step in troubleshooting. 

The event log is always a good source to check on a server, and Cloud Sync is no exception. With the agent setup, two logs are installed, **AgentUpdater** and **ProvisioningAgent**. The former is used for updates, as the name suggests, but the log of the provisioning agent is more interesting, as it shows whether everything is running smoothly or not.

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_VIII.png" style="width:100%">
  <figcaption>Figure 8: The log of the Provioning Agent provides information if everything is ticking correctly</figcaption>
</figure>

By the way, the "Application Log" and the "System Log" are also places that the agent uses to store information. It is therefore a good idea to look at all logs.

Then, of course, there is the Powershell. With the setup of the agent everything necessary is provided. With a few cmdlets the module is up and running and offers the possibility to be active from the command line, as the next figure shows:

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_IX.png" style="width:100%">
  <figcaption>Figure 9: The installed Powershell module for Cloud Sync</figcaption>
</figure>

The trace logs can be used for an even more detailed analysis. Here you can also find the event logs (EVTX files), which are generated with the following cmdlet from the Cloud Sync Module:

```posh
Export-AADCloudSyncToolsLogs
```
<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_X.png" style="width:100%">
  <figcaption>Figure 10: The tracelogs provide an even deeper insight into how the Cloud Sync agent works</figcaption>
</figure>

### ... and for AADC Server ###

With Azure AD Connect Server there are more options for a health check compared to Cloud Sync. For a first look, and to check if everything fits, it makes sense to start in the Azure AD portal. Here a comprehensive status is displayed and the health dashboard can be found in the Azure AD portal in the navigation under **Azure AD Connect** and then in the area of Azure AD Connect via the selection **HEALTH AND ANALYTICS**.

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_XI.png" style="width:100%">
  <figcaption>Figure 11: Health Dashboard shows the overall status of Azure AD Connect Server</figcaption>
</figure>

Since there is also a Health Agent for ADFS and one for the AD Domain Services, everything can be viewed collectively here, of course only if the local systems are equipped with the respective agent. All information about AADC can be found behind the corresponding Quick start on the left (see red frame in Figure 11).

The dashboard here consists of a multitude of active elements, which switch between the areas via quicklink, provide other insights and allow a detailed analysis of the health info. As in the example of the two servers below, by selecting one of the respective server objects.

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_XII.png" style="width:100%">
  <figcaption>Figure 12: This is how it should look: everything healthy is reported by the health agent from the on-prem AADC servers</figcaption>
</figure>

If Azure AD and the health dashboards do not provide any information about an error situation, it is also possible to examine the event log of the Azure AD Connect servers. The focus here should be on the "Application Log", where the Sync Service meticulously lists what is going on.

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_XIII.png" style="width:100%">
  <figcaption>Figure 13: Application Event Log provides detailed info about the state of synchronization</figcaption>
</figure>

The Powershell offers perfect possibilities to check on an AADC server. The powerful PS module "adsync" (117 cmdlets) provides many possibilities for this. Equally helpful is the module "ADSyncDagnostics" with the cmdlet

```posh
Invoke-ADSyncDiagnostics
```

as Figure 14 shows.

<figure>
  <img src="/MyPics/2022-07-12-AMA_AADConnect_XIV.png" style="width:100%">
  <figcaption>Figure 14: Powershell is perfect for health check and analysis. Here is the "Invoke-ADSyncDiagnostics cmdlet"</figcaption>
</figure>

### What to do if localDb is too small or insufficient (10GB)?
During Azure AD Connect setup, a Local DB is installed unless otherwise specified in the wizard. This is convenient and simple. However, Local DB is limited to a maximum DB size of 10GB, which sounds like a lot, but may be reached quickly. It is not possible to say that with x users in sync the 10GB is reached with this or that number of GB. That depends on several factors. But the most important factor is **how many Connectors (AD Forests) are part of the synchronization?**

Everything is contained in the total DB: Metaverse (all objects). As well as the objects of the respective connectors, quasi as "raw data import" for the comparison with the Metaverse.

To get to the point for the question here, I recently saw an AADC setup with an AD domain. Here, about 500 groups were synchronized. No computer objects. And there were around 30,000 users when the DB size reached the 10GB limit.

Microsoft describes in [this docs article](https://docs.microsoft.com/de-de/azure/active-directory/hybrid/how-to-connect-install-move-db){:target="_blank"} and also [here](https://docs.microsoft.com/de-de/azure/active-directory/hybrid/how-to-connect-install-existing-database){:target="_blank"} the individual steps to be taken when moving the database or reusing an existing DB.

To summarize it here to the point:

- Copy DB (MDF and LDF files) on the current sync server to the file system in a safe place or directly to the dedicated remote SQL server.

- Mount DB there

- Reinstall AADC, and as described in the post from the links above, call the setup with the switch **/useexistingdatabase**. In the setup wizard you have the option to specify a SQL server. Afterwards the DB located there will be recognized and used.

- Please pay attention to the post-tasks from the MS articles!

## What do I have to consider when updating to V2?
From my point of view the update to v2 is uncritical and no problem! I have done in-place updates, but would strategically go a different route as described below. First of all, some key points around the topic of AADC update to v2

- v2 does not contain any serious changes to functionality and features. Only the server components have been adapted to the current server versions: Windows Server 2016 and SQL 2019. An overall list of requirements can be found here. [Functional changes in 2.0.3.0](https://docs.microsoft.com/en-us/azure/active-directory/hybrid/reference-connect-version-history#2030){:target="_blank"}

- Even if automatic update "Set-ADSyncAutoUpgrade -AutoUpgradeState enabled" is enabled, the setting does not take effect here. The upgrade to v2 is a manual step that is not automated

- A config export created by a v1 AADC server can be imported on a new v2 server. The JSON files are compatible

- As mentioned above, an in-place update is supported. However, since there is legacy on the server (SQL Server components) I would advise a new installation. Old SQL Server leftovers can be removed manually, but a new version makes sense from my point of view.

As mentioned before the setup supports an in-place update. but I would recommend the following steps to update a server to v2:

1. export config from current AADC server (AADC Config Wizard).
2. make active AADC server staging
3. deploy new server (new computer name).
4. activate TLS 1.2, if not already done. Otherwise there is a risk of an error message during the setup of AADC.
   [Enforce TLS 1.2 for AADC](https://docs.microsoft.com/de-de/azure/active-directory/hybrid/reference-connect-tls-enforcement#enable-tls-12){:target="_blank"}
5. install AADC with v2 and import JSON config from other AADC server (point #1).
6. server is staging. Health Check if everything fits. AD Health Dashboard, Eventlog, Synchronization Service Manager Console. In the rule editor check if the rules are present.
7. change test data on-premises (user phone number, create new user etc.) and start manual synchronization. Cmdlet: "Start-ADSyncSyncCycle".
   Are changes processed? Checking pending exports for Azure AD Connector space. The server is staging. It will not write anything towards Azure AD
8. if everything is ok, enable the new server (AADC Config Wizard).
9. if something goes wrong with the points above, the sync server that was previously retired to staging can be brought back online and there is time to investigate the issue at your leisure.
If everything is in order up to this point (which is to be assumed), the existing staging servers must now be updated one after the other. Again, either in-place or with a completely new machine.

{% include  share.html %}