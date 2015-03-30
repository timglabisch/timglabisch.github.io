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
Es spricht für das Team solche Fragen zu stellen. Es ist sehr gesund, vorhandenes
optimieren und hinterfragen zu wollen. In gesunden Teams kommen solche Fragen auf und sollten entsprechend besprochen werden.


## Cases
Gehen wir davon aus, wir haben ein bestehendes System.
Im Normalfall werde ich in E-Commerce Projekte gerufen.

Wechselt man beispielsweise bei einer einfachen Anwendung, welche einen ORM verwendet, von MySQL
zu PostgreSQL, so bringt dies erst mal wenig Vorteile.

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

Schafft man es, bedingt durch ein breiteres Featureset die eigene Anwendung signifikant zu vereinfachen, kann sich
auch ein wechsel einer Datenbank positiv auf die stabilität des Systems auswirken. Hier sind I.d.R.
Detailfragen interessant.


## Algorithmen
Die Frage, ob Technologie A oder B verwendet werden soll, kann effektiver beantwortet werden, wenn man sich fragt,
mit welchem Algorithmus man die konkrete Aufgabe lösen möchte. Welche Datenstrukturen sind sinnvoll für das aktuelle Problem?

Es ist wichtig dabei zu verstehen, dass Performante Software geschrieben wird, indem Probleme effektiv gelöst werden.
Im Klartext bedeutet das: "Wähle die passende Technologie für das passende Problem."

Nutzt man einen ORM und ist man mit einer klassichen SQL Datenbank gut beraten.
Hier sollte man darauf achten, dass man möglichst zu der Lösung greift, welche von der Community intensiv genutzt und unterstützt wird.
Wichtig ist auch, dass das Team sich mit der Lösung auskennt, oder danach strebt. Aus Motivationsgründen kann es durchaus
sinnvoll sein, sich auf das Experiment einzulassen, mal etwas neues auszuprobieren. Ist dies klar kommuniziert und der Rückweg offen ist dies oft zu begrüßen.

Ist man darauf angewiesen, komplexere SQL Statements zu schreiben, so ist man vermutlich mit PostgreSQL besser dran als mit MySQL.
Dies sollte jedoch von Fall zu Fall gründlich überlegt sein.

Architektonisch hat sich hier der Microservice Gedanke durchgesetzt. Entwickle eine Lösung für ein Problem und wähle die besten Mittel.
Es ist nichts falsch daran, auch unterschiedliche Technologien zu verwenden, schließlich löst man auch unterschiedliche Probleme.

Eine komplexe Software lässt sich i.d.R. kaum entwickeln. Viel schlauer ist es die Probleme in unterschiedliche kleine Softwarestückchen runterzubrechen.
Die Software wird damit deutlich weniger komplex, weil sie nur ein Problem löst. Identisches ist auf viele andere Bereiche der Informatik übertragbar.
Clean Code beschreibt beispielsweise, dass eine Funktion / Klasse /Package eine Aufgabe haben sollte, nicht mehrere.


## Case Performance
Klassische Probleme bei E-Commerce Projekten sind die Produktlisten.
Produktlisten, Facettierungen und andere Filtermöglichkeiten kosten oft viel Performance.

Es ist schon fast normal, dass im Laufe eines E-Commerce Projektes irgendwann eine
gewisse Frustrationsrate überschritten wird. Sei es bedingt durch ein enttäuschtes Projektmanagement
oder dadurch, dass die Entwickler einfach nicht mehr effektiv arbeiten können, weil die Seite zu lange zum Laden braucht.

Es ist sehr wichtig, solche Anzeichen ernstzunehmen.
Kann eine alternative Technologie hier helfen? Oft Hilft einfach ein anderer Blick auf das Problem.
Um diesen zu Blick zu gewinnen, hilft es oft, sich mit alternativen Technologien zu beschäftigen.

Dies liegt nicht daran, dass man eine andere Software nutzt. Es geht darum, dass man dem Problem mit anderen Lösungen begegnet.

Wer jemals versucht hat eine klassische Facettierung mittels einer Standard SQL Datenbank zu implementieren, wird
mit einer hohen Wahrscheinlichkeit eine komplexe und wenig effektive Lösung erarbeitet haben.
Es funktioniert, keine Frage. Aber zu welchem Preis?

Ein Teil von CQRS beschreibt die Nutzung von Views auf gewisse Daten. Die Idee ist einfach.
Bei einem klassischen Projekt werden alle Daten normalisiert in eine Datenbank geschrieben.
Die Daten liegen so vor, dass diese effektiv bearbeitet werden können. Die klassische Lehre besagt, die Daten "sinnvoll" abzulegen.
Die Welt wäre sehr eintönig, wenn sich "sinnvoll" so einfach definieren lassen könnte. Gemeint ist hier i.d.R., dass die Daten
in einer der Normalformen abzulegen sind.

Die Realität zeigt jedoch, dass man mit den Normalformen hier oft nicht weit kommt. In vielen Anwendungen geht es darum, Daten effektiv zu lesen,
nicht diese möglichst kompakt in einer Normalform abzulegen.
Es ist kein Zufall, dass es Wochen dauert, bis beispielsweise Facebook ein Profil in Gänze gelöscht hat.
Sehr empfehlenswert in diesem Kontext sind auch Talks über die Datenbank/Redis Struktur von Twitter.


## Take Care
Nutzt man unterschiedliche Datenbanktechnologien entstehen gerne ungewollte Nebenwirkungen.
Beispielsweise darf nicht vernachlässigt werden, dass viele Datenbanken keine Transaktionen unterstützen. Insbesondere in verteilten
Systemen kann dies bei mangelnder architektonischer Erfahrung schnell zum Problem werden.
Auch sind die Datenbanken dadurch zwangsläufig nicht immer synchron.
Im Zweifel geht es darum, selbstheilende Systeme zu schaffen, welche fehlertollerant sind.


## Speichern von was?
Hin und wieder sehe ich, dass beispielsweise fertig gerendertes HTML in einen Redis (memcache(d) ist ja out) gelegt wird, um dieses dann zu nutzen.
Im Normalfall ist dies eine sehr schlechte Idee.

Software ist i.d.R. in unterschiedliche Schichten aufgeteilt. Legt (pusht) man statisches HTML als in die Datensenke, so werden diese Schichten nicht beachtet.
Viel besser ist es, die Daten in serialisierter Form abzulegen. Wichtig ist dabei, dass wirkliche Daten in einer Rohform abgelegt werden.
Google und Facebook machen mit Protobuffer bzw. Thrift erfolgreich vor, wie ein solches versionierbares Serialisierungsformat aussehen kann.
Der Fokus liegt hier auf versionierbar.


## Tools, Community und Featureset
Neben klassischen Anforderungen bringen einige Datenbanken oft ein weiteres sehr interessantes Feature Set mit.
PostgreSQL beispielsweise kann mittels "Foreign Data" und Materialized Views auch bei einfachen Abfragen eine Vereinfachung der Architektur ermöglichen.


## Aber die Performance?
Möchte man wirklich an der Performanceschraube drehen, so kommt man nicht darum sehr spezielle und sehr optimierte Lösungen zu entwickeln.
Hier ist es besonders wichtig, dass die Lösungen sauber gekapselt und austauschbar implementiert sind.
Neue Anforderungen können schnell zur Notwendigkeit für neue Lösungen führen.
Wer jemals versucht hat, einem bestehenden Algorithmus was völlig neues beizubringen, wird sicher schnell aufgeben und ist gut damit beraten, sich nach einer
neuen Lösung umzusehen.

Oft ist es so, dass versucht wird an den falschen stellen zu optimieren. Braucht beispielsweise die Startseite 2 Sekunden zum ausliefern, brauche ich mich
nicht mit der Rendering-Performance zu beschäftigen um 4ms zu gewinnen.

## N+1 Problem
Sehr interessant ist das sogenannte N+1 Problem. Dieses beschreibt ein Problem welches viele Anwendungen haben.
Es wird eine Query ausgeführt (1). Auf Basis dieser Informationen werden nun für jeden(!) Eintrag N Daten nachgeladen.

Übertragen auf eine Produktliste bedeutet dies, dass beispielsweise eine Anfrage für das laden der ID's stattfindet,
sowie N abfragen für jeden(!) Eintrag. Skriptsprachen wie PHP blocken bei solchen Abfragen (i.d.R.). Netzwerk IO ist teuer.

Beispielsweise sehe ich oft, dass aus einer Suchengeine ID's geladen werden, anhand dieser ID's werden dann die entsprechenden Dokumente geladen.
Dies ist nicht zwingend ein schlechter Ansatz, es gibt jedoch ein paar Dinge zu berücksichtigen. Ich arbeite sehr viel mit ElasticSearch,
kann man die Produktinformationen direkt aus der Suche-Engine laden, so hat man auch den Vorteil, dass es unmöglich ist, dass die
unterschiedlichen Datensenken auseinander gelaufen sind.

Ich sehe zudem sehr oft, dass Beispielsweise ein Suchserver genutzt wird, welcher ID's zu Dokumenten liefert, welche es im restlichen System nicht gibt.
Das vermeintliche fehlen von Dokumenten kann unterschiedliche Gründe haben.
Vermutlich wurden diese aufgrund eines Fehlers nicht in alle Systeme importiert oder warten noch auf den Import. Eine effektive Fehlerbehandlung ist hier
nicht zu unterschätzen. Insbesondere wenn Datenseken genutzt werden, welche keine Garantieren dafür geben, wann, welche Daten verfügbar sind,
ist zwangsläufig die Stabilität des Systems gefährdet.

Oft ist es das einfachste die Daten entsprechend nur aus einer Datensenke zu siehen.
Besitzt man eine klassiche Produktliste und ist von dem N+1 Problem betroffen, so kann es sich beispielsweise lohnen, alle Daten mit einer
ElasticSearch Abfrage zu holen und zu verarbeiten.

## Der Webserver und vorbei.
Scharfe zungen behaupten, sofern man einen Webserver trifft hat man performancetechnisch verloren.
Webserver sind sehr performant, wenn es um das ausliefern von Daten handelt.
Hier ist ausschließlich der Fall gemeint, dass ein Request z.B. PHP trifft.

Schafft man es Anfragen aus dem RAM auszuleifern in dem man beispielsweise Proxies wie Varnish verwendet, ist die Auslieferung zwangsläufig drastisch schneller.
Insgesamt entlastet dies die komplette Infrastruktur und führt zwangsläufig zu einer Verbesserung der Stabilität.
Dies liegt schlicht daran, dass die Infrastruktur deutlich weniger zutun hat.

Solche Einschnitte in die Infrastruktur sollten immer optional sein. Baut man ein System, welches auf einen warmen Cache angewiesen ist, so fängt es an sehr kompliziert zu werden.
Dies sollte man sich sehr gut überlegen und im besten Falle vermeiden.

Stellt man bei einer eh schon performanten Anwendung einen Varnish davor, gewinnt man schlicht mehr Nutzerkompfort.
Versteckt man Performanceprobleme hinter einem Reverse Proxy, so wird nachhaltig die Anwendung darunter leiden.