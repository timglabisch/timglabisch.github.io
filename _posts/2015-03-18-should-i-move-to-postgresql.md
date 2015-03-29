---
layout: post
location: Düsseldorf
tags: [ php architektur postgres ]
title: "[de] Soll ich zu PostgreSQL wechseln?"
---

Wenn ich als Berater unterwegs bin, werde ich häufig mal gefragt, ob es sich lohnt die Datenbankengine zu wechseln.
Solche Fragen kommen oft auf, wenn neue Software evaluiert wird, welche alternative Technologien nutzen.

Gründe reichen von persönlichen Vorlieben neuer Entwickler bis hin zu Performanceversprechen oder
geringeren Wartungskosten.

Meine erste Reaktion ist gerade bei bestehenden Projekten erst einmal ein abgeschrecktes "Wozu?".


## Es ist Gesund.
Es spricht für das Team solche Fragen zu stellen. Es ist sehr gesund, Vorhandenes
optimieren und hinterfragen zu wollen. In gesunden Teams kommen solche Fragen auf und sollten entsprechend besprochen werden.


## Cases
Gehen wir davon aus, wir haben ein bestehendes System.
Im Normalfall werde ich in E-Commerce Projekte gerufen.

Wechselt man beispielsweise bei einer einfachen Anwendung, welche einen ORM verwendet, von MySQL
zu PostgreSQL, so bringt dies wenig Vorteile.

Die Datenbanken sind in dem, was sie tun, bei einfachen Anfragen so stark optimiert,
dass Performanceunterschiede sich hier nicht auswirken.

Interessant wird es jedoch, wenn es spezielle Gründe gibt eine alternative Datenbank oder
Technologie einzusetzen.

Beispielsweise kann es sein, dass das Team sich mit den alternativen Technologien besser auskennt,
dass diese besser ins Entwicklungssystem passen oder einfach weil man es schaffen möchte,
mal etwas Neues zu tun. Es ist wichtig, dass man die Gründe für solche Entscheidungen immer hinterfragt.

Es ist wichtig, hier zu verstehen, dass bei etablierten Datenbanken wie MySQL oder PostgreSQL
kein Performancegewinn zu erzielen ist, wenn man diese einfach nur austauscht.
Performance ist eine Frage von den verwendeten Algorithmen sowie die Frage, in welcher Softwareschicht
die Anforderungen aufgelöst werden können. Die Aufgabe von klassischen SQL Datenbanken liegt so nah
beieinander, dass die gägigen SQL Datenbanken bei einfachen Anforderungen kaum Unterschiede aufweisen.

Klassische Probleme bei Produktlisten sind beispielsweise, dass für jedes Produkt eine einzelne Abfrage
getätigt werden muss. Noch dramatischer wird es, wenn beispielsweise die Sortierung, Gruppierung oder Paginierung
nicht direkt in der entsprechenden Datenschicht getätigt werden kann.

Ist dies gegeben, skaliert die Anwendung nicht. Dieses Problem ist sehr ernstzunehmen.
Eine solche Basis lässt keine sinnvolle Weiterentwicklung zu.


## Algorithmen
Die Frage, ob Technologie A oder B verwendet werden soll, kann effektiver beantwortet werden, wenn man sich fragt,
mit welchem Algorithmus man die konkrete Aufgabe lösen möchte. Welche Datenstrukturen sind sinnvoll für das aktuelle Problem?

!Es ist wichtig dabei zu verstehen, dass man performante Software die entsprechenden Probleme performant löst.
Im Klartext bedeutet das: "Wähle die passende Technologie für das passende Problem."

Nutzt man einen ORM und ist man mit einer klassichen SQL Datenbank gut beraten.
Hier sollte man darauf achten, dass man möglichst zu der Lösung greift, welche von der Community intensiv genutzt und unterstützt wird.
Wichtig ist auch, dass das Team sich mit der Lösung auskennt, oder danach strebt. Aus Motivationsgründen kann es durchaus
sinnvoll sein, sich auf das Experiment einzulassen, mal etwas neues auszuprobieren.

Ist man darauf angewiesen, komplexere SQL Statements zu schreiben, so ist man vermutlich mit PostgreSQL besser dran als mit MySQL.
Dies sollte jedoch von Fall zu Fall gründlich überlegt sein.

Architektonisch hat sich hier der Microservice Gedanke durchgesetzt. Entwickle eine Lösung für ein Problem und wähle die besten Mittel.
Es ist nichts falsch daran, auch unterschiedliche Technologien zu verwenden, schließlich löst man auch unterschiedliche Probleme.

Eine komplexe Software lässt sich i.d.R. kaum entwickeln. Viel schlauer ist es die Probleme in unterschiedliche kleine Softwarestückchen runterzubrechen.
Die Software wird damit deutlich weniger komplex, weil sie nur ein Problem löst.


## Case Performance
Klassische Probleme bei E-Commerce Projekten sind die Produktlisten.
Produktlisten, Facettierungen und andere Filtermöglichkeiten kosten oft viel Performance.
Es ist schon fast normal, dass im Laufe eines E-Commerce Projektes irgendwann eine
gewisse Frustrationsrate überschritten wird. Sei es bedingt durch ein enttäuschtes Projektmanagement
oder dadurch, dass die Entwickler einfach nicht mehr effektiv arbeiten können, weil die Seite zu lange zum Laden braucht.

Es ist sehr wichtig, solche Anzeichen ernstzunehmen.
Kann eine alternative Technologie hier helfen? Oft Hilft einfach ein anderer Blick auf das Problem.
Um diesen zu Blick zu gewinnen, hilft es oft, sich mit alternativen Technologien zu beschäftigen.

Dies liegt nicht daran, dass man eine Software nutzt. Es geht darum, dass man dem Problem mit anderen Lösungen begegnet.

Wer jemals versucht hat eine klassische Facettierung mittels einer Standard SQL Datenbank zu implementieren, wird
mit einer hohen Wahrscheinlichkeit eine komplexe und wenig effektive Lösung erarbeiten.

Ein Teil von CQRS beschreibt die Nutzung von Views auf gewisse Daten. Die Idee ist einfach.
Bei einem klassischen Projekt werden alle Daten normalisiert in eine Datenbank geschrieben.
Die Daten liegen so vor, dass diese effektiv bearbeitet werden können. Die Lehre besagt, die Daten "sinnvoll" abzulegen.
Die Welt wäre sehr eintönig, wenn sich "sinnvoll" so einfach definieren lassen könnte. Oft ist hier einfach definiert, dass man die Daten
in einer der Normalformen ablegt.

Die Realität zeigt jedoch, dass man mit den Normalformen hier oft nicht weit kommt. In vielen Anwendungen geht es darum, Daten effektiv zu lesen,
nicht diese möglichst kompakt in einer Normalform abzulegen.
Es ist kein Zufall, dass es Wochen dauert, bis beispielsweise Facebook ein Profil in Gänze gelöscht hat.


## Take Care
Nutzt man unterschiedliche Datenbanktechnologien entstehen gerne ungewollte Nebenwirkungen.
Beispielsweise darf nicht vernachlässigt werden, dass viele Datenbanken keine Transaktionen unterstützen. Insbesondere in verteilten
Systemen kann dies bei mangelnder architektonischer Erfahrung schnell zum Problem werden.
Auch sind die Datenbanken dadurch zwangsläufig nicht immer synchron.
Im Zweifel geht es darum, selbstheilende Systeme zu schaffen, welche fehlertollerant sind.


## Speichern von was?
Hin und wieder sehe ich, dass beispielsweise fertig gerendertes HTML in einen Redis (memcache(d) ist ja out) gelegt wird, um dieses dann zu nutzen.
Im Normalfall ist dies eine sehr schlechte Idee.

Software ist i.d.R. in unterschiedliche Schichten aufgeteilt. Legt (pusht) man statisches HTML als Sicht ab, so werden diese Schichten nicht beachtet.
Viel besser ist es, die Daten in serialisierter Form abzulegen. Wichtig ist dabei, dass wirkliche Daten in einer Rohform abgelegt werden.
Google und Facebook machen mit Protobuffer bzw. Thrift erfolgreich vor, wie ein solches versionierbares Serialisierungsformat aussehen kann.
Der Fokus liegt hier auf versionierbar.


## Tools,Community und Featureset
Neben klassischen Anforderungen bringen einige Datenbanken oft ein weiteres sehr interessantes Feature Set mit.
PostgreSQL beispielsweise kann mittels "Foreign Data" und Materialized Views auch bei einfachen Abfragen eine Vereinfachung der Architektur ermöglichen.
