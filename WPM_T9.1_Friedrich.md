# MALIS WPM_T9.1 : Beschreibung von datenintensiven und datenfokussierten Aktivitäten im eigenen Arbeitsalltag
Vorgelegt von Tim Markus Friedrich

Sommersemester 2020 / 10.05.2020
Dozent: Prof. Dr. Konrad Förstner  
Technische Hochschule Köln  
Fakultät für Informations- und Kommunikationswissenschaften  
Master in Library and Information Science  

## Inhaltsverzeichnis
[TOC]
* Einleitung
* Die DigiBib
    * Datenverarbeitung in der Metasuche
    * Weitere Datenverarbeitung
    * Logfiles
* Statistiken
* Fazit
* Verweise

## Einleitung
In dieser Hausarbeit sollen verschiedene datenintensive und datenfokussierte Aktivitäten um die DigiBib herum betrachtet werden. Zudem werden Überlegungen angestellt wie sich diese optimieren ließen.

## Die DigiBib
Die [Digitale Bibliothek (DigiBib)](https://www.digibib.net/) ist ein Metasuchportal des Hochschulbibliothekszentrum NRW (hbz). Dort können simultan hunderte verschiedener Bibliothekskataloge oder Datenbanken recherchiert werden. Die Ergebnisse werden zudem mit verschiedenen Mehrwerten angereichert. Neben der Metasuche sind in der __DigiBib__ auch eine Linkliste sowie die Online-Fernleihe integriert. Die DigiBib ist bei über 280 Bibliotheken in Nordrhein-Westfalen, Rheinland-Pfalz und darüber hinaus im Einsatz, wird an das Layout der jeweiligen Bibliothek angepasst und an die jeweiligen Bedürfnisse konfiguriert (z.B. die Datenbankauswahl der Metasuche). Ende 2019 sind die ersten Kunden auf die neuste Version (Release DigiBib 7) umgestiegen. Einige DigiBib-Kunden haben zudem das Produkt [IntrOX]( https://www.hbz-nrw.de/produkte/digibib-loesungen/digibib-introx) im Einsatz, welches die Recherche um einen Suchindex erweitert und Lokalsystemfunktionen integriert.
### Datenverarbeitung in der Metasuche
Die __Metasuche__ ist eine der zentralen Funktionen der DigiBib. Es lassen sich auf einer Oberfläche Bibliothekskataloge und (lizenzierte oder freie) Datenbanken auswählen, in denen dann recherchiert werden kann. Dafür stehen eine Reihe verschiedener Suchfelder zur Verfügung.  
An der Benutzeroberfläche sieht eine Recherche simpel aus, doch wie ist der technische Hintergrund? Um Datenbanken abzufragen wird allgemein eine technische Schnittstelle benötigt (API). An diese werden Suchanfragen gerichtet und es wird eine Antwort zurückgeschickt. Diese muss dann verarbeitet werden.  
Damit die Kommunikation standardisiert ablaufen kann, läuft diese mit Protokollen wie [Z39.50]( http://www.loc.gov/z3950/) oder [SRU]( http://www.loc.gov/standards/sru/) ab. In dieser Art kann zum Beispiel eine SRU-Anfrage nach dem Term „library“ aussehen:
> https://www.example.gov:210/api?version=1.1&operation=searchRetrieve&query=library

Die Anfrage des Benutzers muss also erst einmal in die o.g. Form übersetzt werden. Dazu wird diese mit speziellen Programmen (Filtern) zerlegt und neu zusammengesetzt. Hier kommen unterschiedliche Anforderungen zusammen – z.B. werden unerwünschte Sonderzeichen entfernt und die Suchanfrage in eine Form übersetzt, welche die Schnittstelle auf der anderen Seite verarbeiten kann. Das wäre der erste Schritt der Datenverarbeitung.
Nachdem die Schnittstelle kontaktiert wurde, liefert sie eine Antwort in Form in der Form entweder eines in der Anfrage spezifizierten Formates (z.B. MARC21 oder MAB) oder in einem datenbankeigenen Format. Dieses wiederum wird wieder in der DigiBib zum Software-eigenen Format verarbeitet (geparst). Dies geschieht wiederum in Programmen bzw. Scripten, den Filtern. In der Regel besteht die Schnittstellenantwort aus einem Dokument, welches mehrere Treffer-Datensätze enthält. In diesen sind die Informationen in Feldern strukturiert. Aus diesen werden dann die Inhalte extrahiert und den Gegebenheiten entsprechend für das DigiBib-Feldformat aufbereitet. Beispielsweise könnte eine kumulierte Quellenangabe so aussehen:
> In: Zeitschrift für Bibliothekswesen und Bibliographie 58, 2011, H. 1, S. 10-18.

Durch entsprechende Regeln könnte man hier also durch reguläre Ausdrücke beispielsweise das Jahr extrahieren. Vereinfacht könnte das Programm das Feld nach folgenden Inhalten scannen:
> „In:“ + [Quellenname] + [Jahrgang] + [Jahr] + [Heftnummer] + [Seitenzahl]

(_Genau genommen müsste der reguläre Ausdruck sogar nach Leer- und Satzzeichen suchen. Außerdem muss mit einer gewissen Fehlerbehaftung der Daten gerechnet werden was die Verarbeitung erschwert._)  
Nachdem alle Inhalte der Datensätze der Schnittstellenantwort verarbeitet und in das interne Format überführt wurden, werden die so erstellten Datensätze mit verschiedenen Mehrwerten angereichert. Dies geschieht so, dass mit einem im Datensatz enthaltenen Identifikator (ISBN, ISSN, DOI o.ä.) verschiedene weitere Schnittstellen automatisiert angefragt werden. Hierbei handelt es sich um ausgesuchte Dienste, welche beispielsweise gescannte Inhaltsverzeichnisse oder eine Schlagwortanreicherung anbieten (die DNB bietet beispielsweise das set „dnb:toc“ über eine offene [OAI-Schnittstelle]( https://www.dnb.de/DE/Professionell/Metadatendienste/Datenbezug/OAI/oai_node.html;jsessionid=4AAF020F922243150D6DFEA6D1A9B337.internet532)).  
Zuletzt werden die gesammelten Inhalte zu einem Datensatz zusammengesetzt, welcher dann in der DigiBib angezeigt wird. Beim Auslösen einer Suche geschehen die beschriebenen Schritte für alle ausgewählten Datenbanken simultan. Es werden also eine ganze Reihe von Schnittstellen angesprochen und die zurückkommenden Daten verarbeitet.  
Wie man sich vorstellen kann, ist die Geschwindigkeit der Rechercheergebnisse in einer Metasuche von den Antworten der jeweiligen angesprochenen Datenbanken abhängig. Gerade breit gefächerte Rechercheanfragen (_z.B. das Schlagwort „Geschichte“_) oder die Verwendung älterer Datenprotokolle wie Z39.50 verlangsamen die Recherche für den Benutzer stark.  
Hier wäre ein Verbesserungsansatz zu sehen. Teilweise werden neben älteren Schnittstellen wie Z39.50 alternativ SRU-Schnittstellen angeboten. Wo dies gegeben ist könnte man diese also in den DigiBib-Einstellungen durch neuere Formen ersetzen und die Geschwindigkeit des Datenaustauschs erhöhen.  
Teilweise werden die Katalogschnittstellen in der DigiBib durch eigens gehostete Indizes ersetzt. Bei dem Produkten DigiBib IntrOX und hbzFix (Fernleihindex von Öffentlichen und Spezialbibliotheken) ist dies der Fall. Bei diesen liegt die Antwortgeschwindigkeit deutlich höher und es bestehen bessere Möglichkeiten der Datenverarbeitung. Durch eine Erhöhung der Anzahl der Indizes könnte man die Problematik ebenfalls verbessern.  
### Weitere Datenverarbeitung
Die DigiBib ist ein Portal mit mehreren integrierten Angeboten. Auch in diesen werden (automatisiert) Daten verarbeitet.  
Zu nennen wäre die __Online-Fernleihe__. Benutzer können nach einer Recherche Treffer direkt über die Fernleihe bestellen. Dazu werden Standortinformationen von Medien im hbz-Verbund, Öffentlichen Bibliotheken sowie den anderen Verbünden abgefragt. Falls ein Medium ermittelt werden konnte, wird eine Benutzeranmeldung verlangt. Hierfür wird eine Benutzerdatenschnittstelle der jeweiligen Bibliothek angesprochen und die Anmeldeinformationen übermittelt. Die zurückkommende Antwort enthält Informationen über die Gültigkeit des Benutzerausweises und teilweise auch die Benutzergruppe im Lokalsystem. Falls der Benutzer fernleihberechtigt ist, kann die Bestellung abgeschlossen werden. Im Weiteren werden die Fernleihbestellungen im Fernleihsystem verarbeitet.  
Bei __Anmeldeproblemen__ wird Benutzern angeboten eine Fehlermeldung zu senden. Diese landet dann, angereichert mit Kontextinformationen wie der Bibliothek, bei der der Anmeldeversuch stattfand, beim DigiBib-Kundendienst. Hierfür wird das Ticketingsystem [OTRS]( https://otrs.com/de/home/) eingesetzt. Dort wurden bereits zur Verbesserung des Workflows Antwortvorlagen hinterlegt, welche Hinweise zum Login und Kontaktinformationen der Bibliothek enthalten. Eine Automatisierung der Antworten würde sich hier anbieten – Antwortbausteine liegen bereits vor und eine Identifikation der „Heimatbibliothek“ des Benutzers, sodass die passende Antwort zugeordnet werden kann. Durch eine Anpassung des OTRS ließe sich dies ggf. erreichen. Ein Vorteil der manuellen Beantwortung liegt jedoch darin, dass so größere Ausfälle bei den Bibliothekssystemfunktionen wie der Anmeldung festgestellt und die technischen Ansprechpartner informiert werden können. Eine technische Lösung wäre hier ein Monitoring des skizzierten Szenarios einer automatischen Antwort – bei einer bestimmten Schwelle von gemeldeten Problemen könnte der DigiBib-Kundendienst benachrichtigt werden.  
In der DigiBib befindet sich ebenfalls eine __Linksammlung (DigiLink)__ und eine __DBIS- / EZB-Integration__. Für die Anzeige der DBIS und EZB der Bibliotheken werden deren jeweilige Schnittstellen angesprochen und die Ergebnisse in der DigiBib gespiegelt. Für DigiLink gibt es eine Administrationsoberfläche, mit der Bibliothekare die Datensätze in der Linkverwaltung verwalten können.
### Logfiles
Um die Funktionsweise eines Systems gewährleisten und nachvollziehen zu können, werden eine ganze Reihe verschiedener __Logfiles__ betrieben. Diese zeichnen wichtige Teile von Funktionen des Systems auf. Zu nennen wäre hier beispielweise verschiedene Logfiles über die Datenverarbeitung in der Metasuche, über Benutzeranmeldungen und interne Programme.
>_Platzhalter für Beispiel_
### Statistiken
Über DigiBib-Funktionen werden verschiedene __Statistiken__ erhoben. Zum einen gibt es eine Metasuchstatistik, in der erfolgreiche und nicht erfolgreiche Datenbanksuchen aufgezeichnet werden. Zudem gibt es eine Statistik über die Zahl von Benutzeranmeldungen in der DigiBib. Außerdem gibt es Statistiken rund um DigiLink.  
Man könnte in der Ausweitung der Statistiken eine Verbesserung sehen. Die vorhandenen System-Daten in den Logfiles könnten mehr und spezifischere Statistiken liefern. Interessant wären zum Beispiel Daten über Session-Dauern oder mehr Informationen über die Nutzung elektronischer Medien.
## Fazit
In der DigiBib finden viele automatisierte datenintensive Prozesse ab, da diese die Grundlage für die Funktionalitäten der Software bilden.  
Eine Verbesserung der datenfokussierten Aktivitäten wäre zum einen die Steigerung der Such-Geschwindigkeit durch __Indizes oder modernere Schnittstellen__. Für die Auswertung der Prozesse wären auch weitere Statistiken hilfreich.  
Benutzer mit Anmeldeproblemen könnten mit __automatisierten Antworten__ mit hilfreichen Informationen versorgt werden.  
Im neuen Release der DigiBib wurden viele Bereiche der Anwendung bereits verbessert. Da hier neue Programme und Techniken eingebettet sind, bieten sich bei dieser in Zukunft vermutlich ein weiteres Verbesserungspotenzial.  
## Verweise
* [Portal DigiBib](https://www.digibib.net)
* [Informationen zur DigiBib](https://www.hbz-nrw.de/produkte/digibib-loesungen)
* [hbz Homepage](https://www.hbz-nrw.de)

