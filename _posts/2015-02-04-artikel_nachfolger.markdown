---
layout: post
location: Hamburg
tags: [ php architektur de ]
title: "[de] Linked List zwischen Produkten"
---

Es gibt Domänen in denen Produkte in sich eine Linked List bilden.
Beispielsweise kann an einem Produkt ein nachfolgendes Produkt gepflegt sein.

Technisch gibt es unterschiedliche Möglichkeiten dies abzubilden, dieser Post beleuchtet entsprechende
Vor- und Nachteile.


### Linked List
Zum einen besteht die Möglichkeit diese Information direkt am Produkt zu hinterlegen.
Sprich Produkt A kennt Produkt B, Produkt B kennt Produkt C.


### Import einer Linked List zwischen Produkten
Ich habe schon einige Ansätze gesehen, wie eine solche Linked List durch Produktimporte erstellt werden kann.



eine nahliegende und effizente Möglichkeit eine frische Linked List zu erstellen, ist diese direkt in korrekter Reihenfolge anzulegen.
Besteht Beispielsweise wie Abhängigkeit

Produkt A -(kennt)-> Produkt B -(kennt)-> Produkt C

So könnte ohne Problem:
Produkt C wird angelegt
Produkt B wird angelegt und Referenz zwischen Produkt C und B gesetzt werden
Produkt A wird angelegt und Referenz zwischen Produkt A und B gesetzt werden.

Dies setzt voraus, dass alle Referenzen tatsächlich auch in korrekter Reihenfolge importiert werden können.
Zudem ist es zwigend notwendig, dass es keine Referenzen auf Produkte gibt, welche nicht importiert werden.
In diesem Fall würde eine nicht schließbare Lücke entstehen.


### Import einer sortierten Linked List mit "unbekanntenen"
erhöhen wir die Komplexität ein wenig und gehen davon aus, dass die Daten lücken enthalten,
sprich wir eine Linked List erstellen möchten, wir aber nicht alle Produkte tatsächlich importieren.

In diesem Fall ist es notwendig, dass alle Nachfolger *oder* alle Vorgänger (je nach Richtung der Linked List) bekannt sind.

Beinhaltet beispielsweise:
Produkt A die Informationen A <-
und Produkt D die Information  D <- C <- B <- A

uns es werden nur die Produkt A und D importiert, so besteht die Möglichkeit:
Produkt A wird angelegt
Produkt D wird angelegt
Es wird geprüft ob Produkt C existiert, sollte dies der Fall sein wird eine Referenz zwischen D und C erstellt und abgebrochen
Es wird geprüft ob Produkt B existiert, sollte dies der Fall sein wird eine Referenz zwischen D und B erstellt und abgebrochen
Es wird geprüft ob Produkt D existiert, sollte dies der Fall sein wird eine Referenz zwischen D und A erstellt und abgebrochen

Dieser Ansatz ist etwas aufwendiger, es besteht jedoch die Möglichkeit lücken in einer Linked List zu identifizieren und zu schließen.

Problematisch hier ist, dass die zu importierenden Produkte innerhalb einer Linked List sortiert vorliegen müssen.
Dies ist oft aufwendiger zu optimieren bzw zu parallelisieren.
Sollten mehrere Produkte eine Linked List bilden, kann sich ggf. damit beholfen werden, dass die sortierung nur innerhalb einer Linked List zutreffen muss.

### Import einer unsortieren Liste mit "unbekannten"
Im Idealfall spielt die Sortierung der Produkte im keine Rolle.
Wenn die Sortung Start- oder Endpunkt der Linked List nicht bekannt ist, muss jeder Datensatz Informationen über alle Vor- und Nachfolger besitzen:

Beispiel:
Produkt C die Informationen A <- B <- C <- D
Produkt A die Informationen A <- B <- C <- D

Im Export lassen sich solche Informationen sehr gut cachen, können somit sogar günstiger sein, als Vorgänger oder Nachfolger zu ermitteln.
In diesem Fall wird
Produkt C angelegt
Geprüft ob es Produkt B gibt, wenn ja eine Referenz zwischen Produkt C und Produkt B angelegt, wenn weine Referenz vorhanden ist, wird mit der anderen Seite begonnen
Geprüft ob es Produkt A gibt, wenn ja eine Referenz zwischen Produkt C und Produkt A angelegt, wenn weine Referenz vorhanden ist, wird mit der anderen Seite begonnen
Geprüft ob es Produkt D gibt, wenn ja eine Referenz zwischen Produkt C und Produkt D angelegt und abgebrochen.

bei kleineren Linked Lists lassen sich solche Abfragen idr. sehr effizient beantworten, beispielsweise kann eine exists query formuliert werden, welche gleich alle existierenden
Einträge ermittelt.


### Warum aber nicht einfach alle Einträge im RAM merken und dann eine Linked List erzeugen?
Oft stellt sich die Frage, warum nicht einfach die Daten im RAM oder in einem anderen Datenspeicher (z.b. Datenbank) vorhalten und die Linked List daraufhin erstellen.

Importieren wir Beispielsweise wieder
Produkt C die Informationen A <- B <- C <- D
Produkt A die Informationen A <- B <- C <- D

So müssten wir und alle Produkte merken (A, B, C, D) inkl. dessen Sortierung.
In der Datenbank sind jedoch nur Produkt A und C hinterlegt, eine sinnvolle Datenstruktur um die komplette Liste abzulegen existiert nicht.

Eine einfache Lösung wäre diese Liste im RAM zu belassen.
Faktisch ist dies möglich, praktisch gibt es einige Gründe welche dagegen sprechen:
- Dies funktioniert nur bei komplettimporten.
- Eine weitere Lösung muss implementiert werden für Delta Updates
- Mehrere Lösungen sind immer sehr viel fehleranfälliger und aufwendiger
- Der Ansatz skaliert nicht linear
- Dieser Ansatz läuft nicht auf einem verteilten System
- Daten sind kurzzeitig in einem nicht definierten Zustand
- Delta Updates benötigen identisches globales Lock wie der Vollimport

Noch extremer fällt dies aus, wenn ein zweiter Job im nachhinein die Linked List auflöst.


### Die Sache mit dem Locking
In größeren Setups ist es faktisch nicht möglich die Tabelle (Linked List) zu sperren oder davon auszugehen, dass
kein anderer Job in die Datenbank schreibt.
Schreiben mehrere Prozesse wie z.b. ein Vollimport und ein Deltaupdate gleichzeitig in der Linked List,
so kann die Linked List durch parallele Veränderungen einen nicht definierbaren Zustand annehmen.
Aus diesem Grund sollten nur kleine und vorallem atomare Veränderungen getätigt werden.
Um dies zu erfüllen, kann eine Variation eines Optimisitc Locking's genutzt werden.

Beispiel:
In der Datenbank ist hinterlegt: A <- C <- D
Importiert wird das Produkt B mit den Informationen A <- B <- C <- D

Als erstes findet eine Abfrage statt, welche die Verfügbarkeit von A, C und D prüft.
SELECT id from products where id IN (A, C, D)

Nun kann ein Update inkl. optimistic locking stattfinden:
update product where id = "A" AND nachfolger = "C" SET nachfolger = "B"

Bei Erfolg gibt das Update Statement eine Row Count von 1 an, sollte dies nicht zutreffen,
muss das Material neu importiert werden.

Wird eine Double Linked List genutzt, so ist es notwendig in einer Transaktion 2 Updates durchzuführen.


### Linked List vs Double Linked List
Angenommen wir möchten die Linked List ausgeben, so gibt es mehrere Möglichkeiten.
Im einfachsten Fall wird anstelle einer klassischen Linked List eine Double Linked List verwendet.
Durch diese kann schlicht in beide richtungen iteriert werden.

Wird eine Linked List in einer Datenbank persistiert, so lässt sich daraus eine Double Linked List generieren,
die Datenbank wird schlicht dazu gewungen, die Knoten in der kompletten Ergebnismenge zu suchen.
Was algorithmisch erstmal nicht optimal erscheint, kann aufgrund der einfachereren und atomareren Datenhaltung in der Praxis eine kluge Entscheidung sein.
Zudem sind Datenbanken genau auf solche Tasks optimiert.

Angenommen wir befinden uns auf der Detailseite von Produkt B.
Gegeben ist die Linked List A <- B <- C <- D
In der Datenbank ist folgendes Hinterleft:

id    Nachfolger
A  -> B
B  -> C
C  -> D
D  -> NULL

SELECT id, nachfolger FROM products where id = C or nachfolger = B // A, C
SELECT id, nachfolger FROM products where id = A or nachfolger = A // A  Erster Eintrag weil nur ein Treffer.
SELECT id, nachfolger FROM products where id = C or nachfolger = C // C, D
SELECT id, nachfolger FROM products where id = D or nachfolger = D // D, NULL // Letzter Eintrag weil NULL

Nutzt man ein ORM ist es oft einfacher, schlicht den ersten EIntrag zu ermitteln und daraufhin linear durch
die Einträge durchzuiterieren:

SELECT id, nachfolger FROM products where id = C or nachfolger = B // A, C
SELECT id, nachfolger FROM products where id = A or nachfolger = A // A  Erster Eintrag weil nur ein Treffer.

Faktisch sind zum auflösen einer Linked List in einer Datenbank einige Queries notwendig, Diese beziehen sich jedoch auf
unique ID's und sind somit hochperformant. Die Liste lässt sich natürlich auch Cachen.

#### Performance Tricks
Ganz davon ab, sind bei jedem Produktimport immer die Informationen zu allen Vorgängern und Nachfolgern vorhanden,
faktisch kann dies dazu genutzt werden in einem Cache für jedes Produkt eine denormalisierte Ansicht der Linked List zu persistieren.
Diese müsste nicht einmal errechnet, sondern lediglich geschrieben werden. Durch ein klassisches Sharding lässt sich dies beliebig skalieren und ist
hochperformant.

Natürlich können stattdessen auch Sichten auf die Daten erzeugt werden, Ergebnisse gecacht werden o. etc.





