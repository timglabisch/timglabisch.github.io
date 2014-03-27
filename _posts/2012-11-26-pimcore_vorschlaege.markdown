---
layout: post
location: Düsseldorf
tags: [ ]
title: "[de] Pimcore 2 Verbesserungsvorschläge"
---

# Vorwort zu Pimcore 2.x

Pimcore 1 hat uns in den letzten Jahren viel Freude gemacht. Der Grund dafür war hauptsächlich, dass Pimcore nicht versucht das Rad neu zu erfinden, sondern möglichst intensiv auf das (u.a. Zend) Framework setzt. Pimcore 1.x hat jedoch auch seine Schwächen, insbesondere in der Testabdeckung kann es überhaupt nicht punkten.

## Neuentwicklung
Es ist zwar ein wenig traurig und mit viel Arbeit verbunden, jedoch wird man - wenn man es richtig macht - nicht um eine komplette Neuimplementierung herumkommen. Der Hauptgrund ist nicht die Migration auf das Zend Framework 2 oder Extjs, sondern die derzeit fehlende Testabdeckung. Diese nachträglich zu implementieren wäre im Aufwand / Leistungsverhältnis vermutlich sinnfrei.

## Testabdeckung
Pimcore unterscheidet sich von vielen Systemen dadurch, dass die Community sehr professionell ist. Um die qualitativen Anforderungen an dem System gerecht zu werden, ist eine Testabdeckung von 100% (realisitisch ~ 99%) anzupeilen.
Besonders in der späteren Wartung und Erweiterung des Systems wird sich dies auszahlen.
Die Testabdeckung ist ein Hauptgrund dafür, dass Pimcore 2 eine Neuimplementierung sein muss.

## Zend Framework 2
Das Zend Framework 2 ist ein wirklicher Meilenstein und bringt einige neue sinnvolle Konzepte mit. Besonders zu beachten wären hier das Management von Abhängigkeiten zwischen Services (Service Locator / Dependency Injection) und das Modulmanagement. Auf die Implementierung sollte man hier besonders viel Wert legen.

## Die Wolke kommt.
Pimcore hat uns immer wieder geärgert, wenn es darum ging, dass System halbwegs performant zu verteilen. Ich spreche mich dagegen aus, Pimcore explizit “Cloud Support” beizubringen. Viel mehr sollte die Architektur so gut durchdacht sein, dass Pimcore sauber skaliert. Wie ich mir das im Detail Vorstelle folgt.
Wünsche zur Entwicklung
Coding Styles
Das Team sollte vorab einige Regeln zur Codeformatierung von PHP / JS / CSS und HTML definieren und schriflich festhalten.


## Transparente Entwicklung
Die Entwicklung von Pimcore 2 sollte völlig transparent und im Idealfall über Github stattfinden.
Ich würde es bevorzugen, wenn Designfragen vom Core-Team definiert und von mind. 2 Entwicklern in Zusammenarbeit umgesetzt werden. Nach fertiger Umsetzung prüft ein weiterer unabhängiger Entwickler den Code. Das Projekt muss dafür in Teilbereiche unterteilt werden, welche von unterschiedlichen Teams gleichzeitig bearbeitet werden können.
Ich könnte mir gut vorstellen, das Projekt hauptsächlich in Services und dazugehörige Entities zu unterteilen (s.u.). Code der übergreifend zwischen diesen Services agiert gehört zur “öffentlichen” API der Services und muss besonders gut getestet sein. Als Regel wäre hier nahliegend, dass die API besprochen und vollständig getestet sein muss, bevor diese geprüft und aufgenommen wird. Daraufhin könnte die Testabdeckung vervollständigt werden und sofern sich die API nicht verändert hat, der Code aufgenommen werden.

## Nutzung der Community
Die Pimcore Community grenzt sich durch professionalität von vielen anderen Open Source Projekten ab. Die Entwicklergemeinde sollte fähig sein, einen größeren Beitrag zu Pimcore beizutragen. Im Interesse des Systems sollte versucht werden, diese Community bei der Entwicklung für sich und für das Projekt zu gewinnen. Gute Kommunikation ist hier zwingend erforderlich.


## Lizenz
Die Gerüchteküche brodelt bezüglich der Enterprisevariante von Pimcore. Aus Entwicklersicht würde ich es nicht begrüßen wenn hier ein Closed Source Produkt entstehen würde. Verständlich wäre die Nutzung einer Dual Lizenz wie Unternehmen wie Sencha und Redhat erfolgreich vormachen. Damit sinnvoll Code beigesteuert werden kann, muss die Lizenzierung jedoch im Vorfeld geklärt sein. Bitte betrachtet auch die Möglichkeit einer Trennung zwischen privater und kommerzieller Nutzung.

# Ein wenig Technik.

## Integration in das ZF
Pimcore als Abhängig
Pimcore sollte über Composer inkl. aller Abhängigkeiten nachgeladen werden können.
Dies bedeutet, dass sich Pimcore nicht mehr als komplett betrachten darf, sondern als Modulsammlung für das Zend Framework.
Analog zum Zend Framework Skeleton sollte es demnach auch ein Skeleton für Pimcore geben.


## Serviceorientiert, keine Gott-Controller
Schaut man sich den derzeitigen Code der Controller an, findet man viel zu viel Datenlogik in den Controllern. Controller sollten nur zwischen Services delegieren und möglicht wenig direkt auf zu persistierende Strukturen zugreifen. Jegliche Kommunikation, beispielsweise zu der Datenbank, sollte vom Controller durch die Serviceschicht delegiert werden. Der Controller sollte nur den minimalen Code für die Delegation der Anfrage auf den Servicelayer beinhalten.
Der Vorteil liegt hauptsächlich darin, das eine sinnvolle und ausgeprägte Serviceschicht ensteht, welche leicht tauschbar und vorallem von überall aus verwendet werden kann.


## Alle Daten
Bei der Modellierung der Datenstrukturen sollte darauf geachtet werden, dass keinerlei, via autoincrement generierte id's verwendet werden. Diese sind zwar äußerst performant, lassen sich jedoch nur sehr schwer in verteilten Systemen einsetzen, auch das Zusammenfügen von Daten auf Datenbankebene ist unnötig kompliziert. Als Lösung würde ich UUID's empfehlen.
Persistieren via Service
Bei Pimcore 1 waren die Models/Entities und die Logik zum Speichern von Dokumenten allesamt vermischt, der Versuch dies über _Resource Dateien zu entkoppeln war unperformant, undurchsichtig, Fehleranfällig, und extrem schlecht erweiterbar.

## Data Mapper
Die Lösung dafür ist relativ simpel. Nehmen wir als Beispiel das klassische Dokument. In Pimcore 1 konnten wir über den Aufruf

```php
Document::getById(1);
```

eine Instanz eines Dokuments holen. Der Code scheint auf dem ersten Blick womöglich einfach und sinnvoll. Die Probleme bei diesem Ansatz sind folgende:

1. Wir verletzen das erste Prinzip von SOLID > Single responsibility principle. Es ist vollkommen unklar was die Klasse Document tut. Auf der einen Seite ist es ein Service der Daten liefert und speichert, es beinhaltet aber auch Daten und womöglich kümmert es sich auch um dessen Versionierung, Übersetzung, ….
2. Ganz dramatisch ist, dass das Dokument nicht wirklich mehr erweiterbar ist, da der Aufruf “Document::” statisch ist. Nutzern des Systems ist es nicht möglich dies zu überschreiben. Generell müssen statische Aufrufe komplett vermieden werden.
3. Das Dokument kümmert sich um seine Persisierung. Dies hat gewaltige Nachteile und schränkt sehr ein. Stellen wir uns vor wir wollen das Dokumente aus verschiedenen Quellen geladen sowie gespeichert werden können. Auf Basis von Pimcore 1 wäre es beispielsweise nicht möglich ein Dokument über eine andere Datenbankverbindung zu laden, oder anhand einer XML-Datei. Würde man versuchen dies einzubauen, sähe das API vermutlich so aus:

Beispiel:

```php
Document::getByXML(‘...’);
Document::getById(‘...’, $dbConnection);
Document::getBySOAP(‘...’);
```

Das Beispiel verdeutlicht, dass die komplette Logik in dem Dokument abgebildet werden müsste. Alles andere wären fiese Workarounds und Ausnahmen. Vermutlich ist dies der Grund für die grenzwertige SOAP Schnittstelle von Pimcore, mehr dazu später.

Die Lösung des Problems ist einfach, das Dokument muss von allen Abhängigkeiten bzw. dessen Speicherung getrennt werden. Das Dokument selbst darf nur noch über den Aufbau eines Dokuments wissen.
Schauen wir uns dieses Beispiel an:

```php
$document = $this->locator->get(‘Pimcore\Service\Document’)->loadByUuid(‘....’)
$document->setTitle(‘...’);
$this->locator->get(‘Pimcore\Service\Document’)->persistAndFlush($document);
``

In diesem Beipiel wüsste das Dokument nicht wo es her kommt und wo es hin gespeichert wird - dem Dokument ist dies völlig gleichgültig. Dokumente können von beliebigen Services geladen werden. Das Dokument selber würde über keinerlei Logik wissen müssen, es kennt nur seine Datenstrukturen. Dank dem Locator wäre es jedem möglich die Implementierung von Pimcore\Service\Document zu tauschen. Selbst Pimcore könnte sich diesen Mechanismus für unterschiedliche Datenbanktypen, zum Vorschalten vom Caching, ... zu Nutze machen.

Beispiel:

```php
$document = $this->locator->get(‘Pimcore\Service\Document\Doctrine’)->loadByUuid(‘....’)
$document = $this->locator->get(‘Pimcore\Service\Document\Mongo)->loadByUuid(‘....’)
$document = $this->locator->get(‘Pimcore\Service\Document\Soap’)->loadByUuid(‘....’)
$document = $this->locator->get(‘Pimcore\Service\Document\XmlRpc’)->loadByUuid(‘....’)
$document = $this->locator->get(‘Pimcore\Service\Document\Rest’)->loadByUuid(‘....’)
$document = $this->locator->get(‘AnyMotion\Service\Document\GITHUB’)->loadByUuid(‘....’)
$document = $this->locator->get(‘AnyMotion\Service\Document\SOLR)->loadByUuid(‘....’)


// Standard könnte getauscht werden, vll SOAP?,
$this->locator->get(‘Pimcore\Service\Document’)
```

Diese paar Zeilen verdeutlichen schon einen enormen Gewinn an Flexibilität, welche sich beliebig ausbauen lässt. Anhand der unterschiedlich unterstützten API’s und der Tatsache, dass das Dokument nichts darüber wissen muss, zeigt sich eigentlich schon, dass dieser Weg optimal wäre. Die Api’s sollten dem Interface Pimcore\Service\Document entsprechen. Die Standardimplementierung für dieses Interface könnte Pimcore\Service\Document\Doctrine sein, welches die Dokumente aus einer Standard MySql Datenbank beschafft. Die Standardimplementierung könnte frei verändert werden, beispielsweise könnten von der Community Speicherengines hinzugefügt werden, Firmen könnten eigene schreiben, …
Weiteres zu den speziellen Speicherengines folgt im entsprechenden Bereich des Dokumentes, als Beispiel soll uns dies vorerst genügen.


## ORM
Doctrine
Doctrine würde sich hervorragend dafür eignen die Daten unabhägig dessen Ursprungs zu persistieren. Sollte man sich im Nachhinein aufgrund Performanceengpässe an einigen Stellen dagegen entscheiden, so könnte Dank dem serviceorientierten Ansatz, den Standard Service für diesen Bereich ohne Probleme gegen eine performantere Lösung, getauscht werden.
Doctrine als Abhängigkeit zu Pimcore auszuliefern, würde die Entwicklergemeinde nur freuen, nie wäre es einfacher gewesen in Pimcore eigene (Datenbank?) Tabellen hinzuzufügen, zu deployen und zu nutzen.

Dies muss keine Entscheidung für Doctrine in Pimcore sein, viel mehr könnte es die Entscheidung sein, in den Standardservices vorerst Support für Doctrine zu ermöglichen :).


## SQL-lite
Pimcore sollte SQL-Lite unterstützen. Besser ausgedrückt, die Implrmentierung der Services mit Doctrine Implementierung sollte mit SQL-lite laufen. Nicht damit diese jemand produktiv nutzt, sondern damit Unittests mit einer in-memory SQL-Lite Datenbank ultraperformant und parallel beim Entwickler laufen.




## Dokumente
Vorwort
Das Erste, was ich gemacht habe um die Implementierung von Pimcore auf Basis des Zend Frameworks auszuprobieren war, wie persistiere ich die Pimcore Objektstruktur. Der Grundaufbau der Dokumente ist bis auf ein paar Ausnahmen sinnvoll.
Problematisch ist jedoch, wie die Daten abgelegt werden.


## Objekte
Probleme derzeitiger Objekte
Wir haben uns häufiger mit den Objekten in Pimcore geärgert, Gründe dafür:

### 1. ID’s
1.1 Pimcore speichert die Tabellen anhand der Objekt_Id’s, dies führt zwingend zu Chaos, wenn man unterschiedliche Umgebungen miteinander abgleichen möchte. Die Lösung wäre Einfach > Namen des Objektes nutzen.
1.2 Pimcore nutzt für die Speicherung von Objekten Id’s mit einem autoincrement Wert. Diese sind nur schwierig zu merken. Lösung: Pimcore sollte (überall!) auf UUID’s setzen.

### 2. Store, Query, …
Die Datenbankstruktur ist für den Entwickler nicht wirklich transparent. Hier wurde aufgrund von Performanceoptimierung die Datenbank aufgebläht. Ich hätte dies lieber in einem schlaufen Caching gesehen.

### 3. Objektbaum
nicht alle Objekte sollten zwingend in dem Objektbaum hängen müssen.

### 4. Oberfläche
Die Oberfläche ist sehr schlecht erweiterbar.

### 4. nicht jedes Objekt muss pflegbar sein.


Desweiteren treffen alle Punkte welche unter “Alle Daten” (s.o.) aufgeführt wurden auf Dokumente zu.


## Generieren von Objekten im Backend
Aus meiner Erfahrung mit dem Umgang und beim Schulen von Pimcore habe ich die Erfahrung gemacht, dass das Anlegen von Objektstrukturen vom Entwickler durchgeführt wird. Ein fähiger Entwickler benötigt die Oberfläche zum Anlegen von Objektstrukturen nicht. ich würde diese aus diesem Grund auslagern und als “nice-to-have” ansehen.


## Was wird generiert / benötigt?
Generiert werden sollten nur noch Entities welche von dem Standard Objektservice verarbeitet und gespeichert werden können. Dies würde nach meiner Empfehlung Doctrine Entities entsprechen. Diese Entities sollten jedoch auch problemlos vom Entwickler händisch angelegt werden können. Durch den klaren Aufbau und der Tatsache, dass die zu persistierenden Objekte nichts mehr über die Datenbank wissen müssten, wäre dies jedoch sowieso möglich.


## Generierung einer Bearbeitungsoberfläche
Genau hier sehe ich Pimcore. Die Entitiers müssen (bei Bedarf) bearbeitet werden können.
Schauen wir uns mal ein Standard Doctrine Entity an:



Ist euch aufgefallen, dass ich die Entity um Pimcore Spezifische Informationen erweitert habe? Wenn nicht schaut nochmal genau hin.
Was ermöglicht uns dieser Ansatz?
1. Jede standard Doctrine Entity wäre editierbar über Pimcore.
2. Pimcore kümmert sich nur um das, was es wirklich tun soll -> Oberfläche zum Speichern von Daten.
3. Das ganze ist völlig ausbaufähig, betrachten wir z.B. die Problematik, dass wir Objektvarianten speichern wollen würden. Man könnte die Klasse mit @Pimcore\Variants annotieren und ggf. einen anderen Service zum Speichern und Laden dieser Objekte nutzen und die Informationen Extjs zur Verfügung stellen.

Hier geht es primär um die Idee die Annotationen (bzw. Konfigurationen) der Entities dazu nutzt, die Oberfläche zum Editieren zu gestalten. Pimcore würde sich so elegant mit Doctrine vermischen, für den Entwickler sieht alles sauber aus und auch die gruselig wartbaren Layout Definitionen könnte man elegant umgehen.

## Erweiterbarkeit der Bearbeitungsoberfläche
Dank der Annotationen würden sich viele Eigenschaften für die Bearbeitungsoberfläche setzen lassen können. Jedoch ermöglicht dies gleichzeitig auch das Modifizieren der kompletten Oberfläche, schauen wir uns dazu dieses Beispiel an:


Dieses einfache Beispiel würde schon die komplette Renderlogik ersetzen können.
Die Entity würde Dank der Klassenannotation von einem anderen Modul und Controller gerendert werden und die Datentypen haben wir auch noch gegen völlig eigene ersetzt.
Vermutlich wird es in 99% der Fälle völlig ausreichen, eigene Datentypen hinzuzufügen und über Annotationen zu nutzen. Die Möglichkeit zusätzlich den kompletten renderer zu tauschen ist jedoch ein Wahnsinnsgewinn und ermöglicht völlige Freiheit. Sicher gibt es noch weitere interessante Annotationen und Konfigurationen, wie beispielsweise Hooks. Ich möchte es jedoch vorerst bei der Bemerkung “total ausbaufähig und erweiterbar“ belassen.

## Klassen / Mapping
Wird beispielsweise ein Textfeld mit @anyMotion\Doctrine\Mapping\Text annotiert, so würde sich seitens PHP dies Klasse anyMotion\Doctrine\Mapping\Text um die Speicherung, das Laden, die Konfiguration … kümmern. In Javaskript könnte die Klasse anyMotion.doctrine.mapping.text ermöglichen. Das man wie derzeit Klassen in den Namensraum von Pimcore legen müsste, würde der Vergangenheit angehören.





Admin-Bereich
Integration in das Zend Framework
Der komplette Admin Bereich sollte als völlig losgelöstes Modul im Zend Framework registriert werden können. Dies hat den Vorteil, dass man es nicht zwingend im Produktivsystem ausliefern muss. Zudem hilft es das System sauber und modular zu entwickeln.
Statische Dateien sollten via Assetic (s.o.) eingebettet werden.

Assets / Assetic
Die Zend Module erlauben wunderbar die Aufteilung in beliebig viele Submodule. Hier entsteht jedoch das Problem, dass diese alle jenseits des Webroots liegen. Es macht jedoch keinen Sinn die teilweise statischen Dateien aus den Modulen zu entfernen. Dies würde auch der möglichen Nutzung einer Skeleton Applikation nach dem Vorbild von Zend Skeleton widersprechen. Vielmehr ist es notwendig jedem Modul zu ermöglichen den Webroot zu erweitern. Die schlauste Möglichkeit bietet hier Assetic. Pimcore sollte zum Verwalten statischer Dateien  die Möglichkeiten von Assetic nutzen. Diese sind erprobt und von der Symfony gemeinde bereits auf Herz und Nieren getestet. Ein Fertiges Modul für das Zend Framework 2 welches als Abhängigkeit definiert werden könnte existiert bereits. Hier wäre zu prüfen ob dies ausreicht und den Qualitätsanforderungen entspricht.
ExtJs 4
Extjs 4 bringt viele neue Möglichkeiten, besonders das MVC Konzept sollte hier umgesetzt werden.

Event Basierend
// ...
Kommunikation mit dem Backend
Die derzeitige Kommunikation mit dem Backend ist weniger optimal. Die Controller sind aufgebläht und wenig wiederverwendbar. Die Kommunikation mit dem Backend sollte vollständig über eine REST Api stattfinden. Dies u.a. hat den Vorteil, dass keiner weitere API Supportet werden muss, da das Backend schon auf ihr basiert.

Authentifizierung
Die Authentifizierung zum PHP-Backend sollte Standardisiert stattfinden. Derzeit würde ich OAuth 2 bevorzugen. Aufgeführt habe ich diesen Punkt hauptsächlich, damit er bei der Planung berücksichtigt wird.
Coffee
Ich bin ein großer Fan von Coffeescript. Man könnte die Wartbarkeit und Entwicklungsgeschwindigkeit drastisch steigern, wenn man anstelle von klassischem Javaskript auf Coffeescript setzt. Warum man dies tun sollte zeigen die Entwickler von Dropbox auf:
https://tech.dropbox.com/2012/09/dropbox-dives-into-coffeescript/
Ich bin mir sicher, dass jeder Entwickler der gut mit javascript umgehen kann, innerhalb von 30 Minuten Coffeescript nie mehr zurück möchte. Probiert es aus :).
PHPStorm Support ist auch da, … .



Dateisysteme
Virtuelles Dateisystem
Pimcore 1 hat direkt mit dem Dateisystem kommuniziert. Dies ist weder flexibel noch skaliert dieser Ansatz nicht. Insbesondere um Pimcore in die Wolke zu bekommen, muss ein Abstrakter und austauschbarer Ansatz die derzeitige Implementation ersetzen.
Erwähnenswert wäre hier das Projekt https://github.com/KnpLabs/Gaufrette.


Mounten von Services/Treibern
Ähnlich wie Dateisysteme in Betriebssystemen könnte man ermöglichen unterschiedliche “Treiber” bzw. Services für die Persistierung von Daten in einen gewissen Bereich des virtuellen Dateisystems zu mounten. Dies könnte beispielsweise so aussehen, dass unter dem Pfad / ein Service für die Speicherung auf der Festplatte eingehangen wird. Für einen anderen Ordner könnte ein Treiber für die Amazon S3 eingehangen werden, oder beispielsweise ein inMemory Service zum ausführen von Tests. Eine Beispielimplementation für eine Basisimplementation wurde von mir vor einiger Zeit geschaffen, Beispiel: https://github.com/timglabisch/gaufretteVirtualFilesystem

Mounten von Services/Treibern in Dokumenten, Objekten, ...
Ich habe darüber nachgedacht und könnte mir vorstellen, dass dieses Konzept nicht nur für Assets interssant sein könnte. Betrachten wir das zum Laden eines Dokumentes:

$this->locator->get(‘Pimcore\Service\Document)->getByPath(‘/s3’)

Dieses Beispiel würde öhne weiteres erlauben, dass anhand des Pfades /s3 geprüft werden würde, welcher Service für das Laden bzw. das Speichern von Dokumenten zuständig wäre.
Dies würde interessante Konstrukte erlauben, so könnte ein Teil des Dokumentbaumes aus anderen Systemen über einen Rest Service geladen werden, Dokumente elegant von System A nach System B kopiert werden, …

Dadurch das sich mehrere Pimcore Installationen einen Dokumentbaum teilen könnten, wären solche Kontrukte völlig unproblematisch:

```php
// Kopiert ein Dokument in einen anderen Ordner (über REST in eine andere Installation)
$this->locator->get(‘Pimcore\Service\Document)->copy(‘/installation1’, ‘/installation2’);
Der Ansatz ist extrem erweiterbar und würde sogar weitere interessante Spielereien erlauben.
Problematischer wird es, wenn wir ein Dokument anhand dessen UUID abfragen möchten:

$this->locator->get(‘Pimcore\Service\Document)->getByUuid(‘....’)
``

nun ist völlig unklar welcher Service angefragt werden soll. Die Lösung ist einfach, jedoch etwas schmerzhaft. Es müssen alle gemounteten Services nach der UUID gefragt werden.
Da die UUID suggeriert, dass diese wirklich eindeutig ist, entsteht hier kein Problem.

Bei langsamen Services könnte man überdenken wie man dies schlau löst. jedoch ist zu beachten, dass Pimcore standardmäßig mit nur einem Mountpunkt auskommt. Wenn man die Vorteile betrachtet, Dokumente, Assets, Objekte, … völlig frei überall verteilen zu können, so würde ich dies als gewaltigen Sprung sehen. Mir ist kein System (bis auf Betriebssysteme) bekannt, welche dies schlau lösen bekannt. Ob Assets im Dateisystem, in der Amazon S3, in Hadoop, in Mysql, auf Github, auf einem anderen System via Webservice oder gemischt verteilt liegen würden, wäre völlig egal. Unterschiedliche Systeme könnten (sofern supportet) identische Storage Engines mounten. Das schönste daran ist, dass diese Services nicht vom Core Team selbst entwickelt werden müssten, man gibt jedoch den Unternehmen  die freie Wahl einen wo welche Dateien wie gespeichert werden würden.

Wenn man noch weiter über das Konzept nachdenkt, fallen einem interessante weitere Möglichkeiten ein:

```php
// Dokumente liegen nun in der MongoDb mit identischen Id’s
// Das Lookup über MongoDb ist deutlich flotter, wir haben implizit einen Cache gebaut.
$this->locator->get(‘Pimcore\Service\Document)->copy(‘/mysql, ‘/mongo);

// Same, ...
$this->locator->get(‘Pimcore\Service\Document)->copy(‘/mysql, ‘/memcache);

// so könnte man beispielsweise Solr Dokumente in Solr ablegen
// und natürlich auslesen und löschen!
$this->locator->get(‘Pimcore\Service\Document)->copy(‘/mysql, ‘/solr);
``

Ob und in wie weit man dieses erlauben / nutzen wollen würde sei dahingestellt, jedoch wären hier kaum Grenzen gesetzt.


Caching
Ich Könnte mir gut vorstellen, Caching nicht explizit in die Services zu intigrieren, sondern Adapter für unterschiedliche Services bereitzustellen. Über die Konfiguration in dem Servicelocator / di könnte das Caching elegant und flexibel genutzt werden.
Vorteile:
1. Beliebig viele Cache Implementationen können aneinander gereit werden, z.b. APC, Mongo, Memcache.
2. Beliebig erweiterbar.
Beispiel:
Decorator....



Sicherheit
Mehrere Datenbankverbindungen
Pimcore sollte zulassen, dass für unterschiedliche Aufgaben unterschiedliche Datenbankverbindungen geordert werden können. Diese sollten sich unterteilen in lesende, schreibende und Administrationsverbindungen.
Vorteile
1. Sicherheit wird drastisch erhöht. Beispielsweise muss das Frontend nicht automatisch die Verbindung nutzen,  welche ermöglicht Backend-User auszulesen.
2. Durch die Trennung von lesenden und schreibenden Verbindungen, würde sich Pimcore deutlich besser skalieren lassen.
3. Dazu kommt, dass lesende Verbindungen keine Schreibrechte benötigen würden, dies würde der Sicherheit weiter zugute kommen.



Tests
Code Coverage
Testabdeckung ist das A und O für Qualitativ hohe Software. Pimcore sollte eine Testabdeckung von nahezu 100%  anpeilen. Dabei ist zu beachten, dass Tests möglich Performant und richtig entwickelt werden, Abhängigkeiten sollten aus Performancegründen sauber gemockt werden.

Integrationstests
Neben Unittests sollten Integrationstests das Zusammenspiel von unterschiedlichen Services sicherstellen.
Continuous Integration
Für jede offizielle unterstütze Konfiguration von Pimcore sollte es einen automatisierten Prozess geben der diesen Prüft.

Travis - CI
die Überschrift sagt alles, Pimcore sollte - wie Symfoy, das ZF2, …  https://travis-ci.org/ nutzen.


Puppet oder Chef Skripte - Deployment
Die Installation, verwaltung und einrichtung von Pimcore sollte mittels beigelefter Chef bzw Puppet Skripte erfolgen können. Dies erleichtert das automatisierte Testen enorm und hilft Administratoren bei der späteren Einrichtung von Pimcore stark.
Unklarheiten bei der Einrichtung von Servern würden der Vergangenheit angehören und die Community würde dazu beitragen die Server auf denen Pimcore läuft sicher, aktuell und performant zu halten.


Logging
Das Logging sollte deutlich intensiver ausgebaut werden. Als Standardkomponente sollte Monolog verwendet werden können. Das Logging sollte aus Performancegründen jedoch standardmäßig deaktiviert werden.
Statusmonitore in Echtzeit sind toll. Sie motivieren die Belegschaft, zeigen direkt Engpässe oder Gewinneinbrüche auf. Pimcore sollte hier Support für Echtzeitstatistiken / Echtzeitmonitoring mitbringen. Pimcore sollte Systeme wie graphite unterstützen.

