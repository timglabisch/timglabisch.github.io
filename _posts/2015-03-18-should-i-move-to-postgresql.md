---
layout: post
location: Düsseldorf
tags: [ php architektur postgres ]
title: "[de] Soll ich zu PostgreSQL wechseln?"
---

Wenn ich als Berater unterwegs bin werde ich häufig mal gefragt, ob es sich lohnt die Datenbankengine zu wechseln.
Solche Fragen kommen oft auf wenn neue Software evaluiert wird, welche alternative Technoligien nutzen.

Gründe reichen von persönlichen vorlieben neuer Entwickler bis hin zu Performanceversprechen oder
geringere Wartungskosten.

Meine erste Reaktion ist grade bei bestehenden Projekten erstmal ein abgeschrecktes "wozu?".

## Es ist Gesund.
Es spricht für das Team solche Fragen zu stellen. Es ist sehr gesund vorhandenes
optimieren und hinterfragen zu wollen. In gesunden Teams kommen solche Fragen auf und sollten entsprechend besprochen werden.

# Cases
Gehen wir davon aus, wir haben ein bestehendes System.
Im Normalfall werde ich in E-Commerce Projekte gerufen.

Wechselt man beispielsweise bei einer einfachen Anwendung welche einen ORM verwendet von MySQL
zu PostgreSQL hat man erstmal wenig gewonnen.

Die Datenbanken sind in dem was sie tun bei einfachen Anfragen so stark optimiert,
dass Performanceunterschiede sich hier nicht auswirken.

Interessant wird es jedoch, wenn es spezielle Gründe gibt eine alternative Datenbank oder
Technologie einzusetzen.

Beispielsweise kann es sein, dass das Team sich mit den alternativen Technologien besser auskennen,
dass diese besser ins Entwicklungssystem passen oder einfach weil man es schaffen möchte,
mal was neues zutun. Es ist wichtig, dass man die Gründe für solche Entscheidungen immer hinterfragt.

Es ist wichtig, hier zu verstehen, dass bei etablierten Datenbanken wie MySQL oder PostgreSQL
kein Performancegewinn zu erzeilen ist, wenn man diese einfach nur austauscht.
Performance ist eine Frage von den verwendeten Algorithmen sowie die Frage, in welcher Softwareschicht
die Anforderungen aufgelöst werden können. Die Aufgabe von klassischen SQL Datenbanken liegt so nah
beieinander, dass diese sich insbesondere bei einfachen Anforderungen nichts weiter tun.

Klassische Probleme bei Produktlisten sind beispielsweise, dass für jedes Produkt eine einzelne Abfragen
getätigt werden müssen. Noch dramatischer wird es, wenn beispielsweise die Sortierung, Gruppierung oder Paginierung
nicht direkt in der entsprechenden Datenschicht getätigt werden kann.


## Algorithmen
Die Frage ob Technologie A oder B verwendet werden soll, kann effektiver beantwortet werden, wenn man sich fragt,
mit welchem Algorithmus möchte man die konkrete Aufgabe lösen. Welche Datenstrukturen sind sinnvoll für das aktuelle Problem?

Es ist wichtig dabei zu verstehen, dass man performante Software die entsprechenden Probleme performant löst.
Im Klartext bedeutet das, wähle die passende Technologie für das passende Problem.

Nutzt man einen ORM und ist man mit einer klassichen SQL Datenbank gut beraten.
Hier sollte man darauf achten, dass man möglichst zu der Lösung greift, welche von der Community intensiv genutzt und unterstützt wird.
Wichtig ist auch, dass das Team sich mit der Lösung auskennt, oder danach strebt. Aus Motivationsgründen kann es durchaus
sinnvoll sein, sich auch das Experiment einzulassen, mal was neues auszuprobieren.

Ist man darauf angewiesen, komplexere SQL Statements zu schreiben, so ist man vermutlich mit PostgreSQL besser dran als mit MySQL.

Architektonisch hat sich hier der Microservice Gedanke durchgesetzt. Entwickel eine Lösung für ein Problem und wähle die besten Mittel.
Es ist nichts falsch daran, auch unterschiedliche Technologien zu verwenden, schließlich löst man auch unterschiedliche Probleme.

Eine komplexe Software lässt sich idr. kaum entwickeln. Viel schauer ist es die Probleme in unterschiedliche kleine Softwarestückchen runterzubrechen.


## Case Performance
Klassische Probleme sind hier die Produktlisten.
Produktlisten, Facettierungen und andere Filtermöglichkeiten kosten oft viel Performance.
Es ist schon fast normal, dass im Laufe eines E-Commerce Projektes hier irgendwann eine
gewisse Frustrationsrate überschritten wird. Sei es bedingt durch ein enttäuschtes Projektmanagement
oder dadurch, dass die Entwickler einfach nicht mehr effektiv arbeiten können weil die Seite zu lange zum Laden braucht.

Es ist sehr wichtig solche Anzeichen erst zu nehmen.
Kann eine alternative Technologie hier helfen? Oft Hilft einfach ein anderer Blick auf das Problem.
Um diesen zu Blick zu gewinnen, hilft es oft sich mit alternativen Technologien zu beschäftigen.

Dies liegt nicht daran, dass man eine Software nutzt, dies liegt daran, dass man das Problem mit anderen Lösungen begegnet.

Wer jemals versucht hat eine klassische Facettierung mittels einer standard SQL Datenbank zu implementieren wird
mit einer hohen wahrscheinlichkeit eine komplexe und wenig effektive Lösung erarbeiten.

Ein Teil von CQRS beschreibt die Nutzung von Views auf gewisse Daten. Die Idee ist einfach.
Bei einem klassischen Projekt werden alle Daten normalisiert in eine Datenbank geschrieben.
Die Daten liegen so vor, dass diese effektiv bearbeitet werden können. Die Lehre besagt, die Daten "sinnvoll" abzulegen.
Die Welt wäre sehr eintönig, wenn sich "sinnvoll" so einfach definieren lassen könnte. Oft wird hier einfach beschrieben wie man die Daten
in einer der Normalformen ablegt.

Die Realität zeigt jedoch, dass man mit den Normalformen hier oft nicht weit kommt. In vielen Anwendungen geht es darum, Daten effektiv zu lesen,
nicht diese möglichst kompakt in einer Normalform abzulegen.
Es ist kein Zufall, dass es Wochen dauert, bis beispielsweise Facebook ein Profil gänze gelöscht hat.


## Take Care
Nutzt man unterschiedliche Datenbanktechnologien entstehen gerne ungewollte nebenwirkungen.
Beispielsweise darf nicht vernachlässigt werden, dass viele Datenbanken keine Transaktionen unterstützen. Insbesondere in verteilten
Systemen kann dies bei mangelnder architektonischer Erfahrung schnell zum Problem werden.
Auch sind die Datenbanken dadurch zwangsläufig nicht immer synchon.
Im Zweifel geht es darum, selbstheilende Systeme zu schaffen, welche fehlertolerant sind.
Entsprechende Architekturen bringen entsprechende Lösungen mit entsprechenden Einschränkungen hier mit.
Im Zweifel bleibt nur, sich genau damit zu beschäftigen.

## Speichern von was?
Hin und wieder sehe ich, dass beispielsweise fertig gerendertes HTML in einen Redis (memcache(d) ist ja out) gelegt wird um dieses dann zu nutzen.
Im Normalfall ist dies eine sehr schlechte Idee.

Software ist idr. in unterschiedliche Schichten aufgeteilt. Legt (pusht) man statisches HTML als Sicht ab, so werden diese schichten nicht beachtet.
Viel besser ist es, die Daten in serialisierter Form abzulegen. Wichtig ist dabei, dass wirkliche Daten in einer Rohform abgelegt werden.
Google und Facebook machen mit Protobuffer bzw. Thrift erfolgreich vor, wie ein solches versionierbares Serialisierungsformat aussehen kann.
Der Fokus liegt hier auf Versionierbar.


