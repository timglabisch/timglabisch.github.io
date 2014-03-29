---
layout: post
location: Düsseldorf
tags: [ uml tutorial basic oop ]
title: "[de] Uml Basics"
---

## Notes

Einfache Notizen können an jeder Stelle verwendet werden.
Diese werden in rechteckiger Form dargestellt und mit einer gestrichelten Linie verbunden


![](http://yuml.me/diagram/scruffy;/class/[Customer]-[note: einer von ihnen könnte Superman sein{bg:cornsilk}])

Oft werden solche Notizen farbig anders dargestellt, ist aber nicht zwingend notwendig.

## Grundaufbau eines Klassendiagrams

![](http://yuml.me/diagram/scruffy;/class/[Name_der_Klasse|+property1;-property2|+funktion1\(\);-funktion2\(\)])

Der Name der Klasse kommt ganz oben, gefolgt von beliebig vielen Properties. Danach folgen die Funktionen.

### Properties
Die Properties können deutlich mehr Informationen aufnehmen und werden nach diesen Aufbau:
visibility name: type multiplicity = default {property-string}

## Basic OOP Stuff

### Vererbung

![](http://yuml.me/diagram/scruffy;/class/[Car]^-[MonsterTruck])

Dieses UML Diagramm zeigt, dass die Klasse MonsterTruck von der Klasse Car erbt.
Jeder, der schon mal etwas geerbt hat, dem ist bewusst, dass dies eine aktive Handlung ist.
Demnach geht der Pfeil vom Erben aus. Hierbei handelt es sich um einen "geöffneten" Pfeil.


### Implementiert / Interface

![](http://yuml.me/diagram/scruffy;/class/[Car]^-[MonsterTruck], [<<driveAble>>]^-.-[Car])

Dies zeigt die Klasse Car die das Interface driveAble implementiert. Die Klasse MonsterTruck erbt von Car.

Bei Klassen kann das entsprechende Interface auch direkt mit angegeben werden.

![](http://yuml.me/diagram/scruffy;/class/[<<driveAble>>;Car])

## Associations

![](http://yuml.me/diagram/scruffy;/class/[ClassA]-[ClassB])

Dieses Diagramm sagt nchts anderes aus, als das ClassA Beziehung mit ClassB steht.
Oft bedeutet dies, dass die eine Klasse eine Instanz der anderen Klasse hält, dies ist aber nicht zwingend notwendig ist.


![](http://yuml.me/diagram/scruffy;/class/[Customer]->[Address])

Customer beinhaltet eine Instanz einer Adresse. Vom Customer lässt sich auf die Adresse zugreifen.
Dies wäre z.b. für folgenden Code zulässig:

```php
class Customer {
    function getAddress() {
        return new Address();
    }
}
```

![](http://yuml.me/diagram/scruffy;/class/[Customer]-getAddress>[Address])

Optional lässt sich angeben, über welche Property auf die Klasse zugegriffen werden kann.

Anstelle des Pfeils könnte man auch ein "X" setzen, dieses würde bedeuten, dass es nicht navigierbar ist.
Dies wird jedoch so selten verwendet, dass yuml.me das Tool, welches ich zum Zeichnen verwende, dies nicht unterstützt.

### Bidirectional

Es ist durchaus möglich, dass ein bidirektionalen Graphen modellieren möchte,
In diesem Fall kann man von der Adresse auf den entsprechenden Customer und von dem Customer auf die Adresse schließen.
Idr. sollte man dies jedoch versuchuchen zu vermeiden.

![](http://yuml.me/diagram/scruffy;/class/[Customer]<->[Address])



### Cardinality

Bei Associationen lässt sich optional angeben wieviele Objekte zurückgegeben werden.
Beispiel:


![](http://yuml.me/diagram/scruffy;/class/[Customer]1-1>[Address])

Dies bedeutet schlicht ein Customer gibt immer eine Addresse zurück.

![](http://yuml.me/diagram/scruffy;/class/[Customer]1-1>[Address],[Customer]1-0..1>[DeathDate])

Alternativ würde sich auch signalisieren, dass die Addresse optional ist.
Dies macht beispielsweise Sinn, würde es sich um ein Todesdatum halten.
Dieses ist schlicht nicht bei jedem Kunden gegeben :).

Dies würde beispielsweise auf folgenden Code zutreffen:

```php
class Customer {
    function getDeathData() {
        if(/*...*/)
            return new DeathData();

        return null;
    }

    function getAddress() {
        return new Address();
    }
}
```

### Association Example


![](http://yuml.me/diagram/scruffy;/class/[Customer]->[Address],[Customer]1-0..1>[DeathDate],[Name|firstname;lastname],[Address]-name>[Name],[Customer]->[EMail], [Customer]-[note: Aggregate Root{bg:cornsilk}])

Das Beispiel zeigt ein typisches Aggregate Root, welches eine E-Mail, eine Adresse und optional ein Todesdatum des Kunden hält.
Die Adresse beinhaltet die Klasse "Name", welches mind. die Attribute firstname und lastname besitzt.


### Aggregationen und Kompositionen


Uml bietet unterschiedliche Möglichkeiten die Beziehungen zwischen Objekten aufzuzeigen.

![](http://yuml.me/diagram/scruffy;/class/[Customer]1-n[Address])

Dies sagt schlicht aus, dass mehrere Adressen in Beziehung zu einem Kunden stehen.

n kann für folgendes stehen:

- eine beliebige Zahl
- ein "*", dies bedeutet eine beliebige Zahl inkl. 0
- eine Range
    - "1..2" bedeutet zwischen 1 und 2
    - "2..*" bedeutet 2 oder mehr.


Einfache Assoziationen lassen sich auch so darstellen, alternativ können "Assoziationen" hervorgehoben werden.

![](http://yuml.me/diagram/scruffy;/class/[Customer]<>-[Address])

#### Kompositionen vs. Assoziationen

UML definiert Assoziationen und Kompositionen, beide sind sehr ähnlich, jedoch geben Kompositionen explizit an,
dass eine Klasse von der anderen völlig abhängig ist und quasi nur unter ihr existiert.
Kompositionen werden durch einen gefüllte Raute angeben.


##### Beispiel Association

![](http://yuml.me/diagram/scruffy;/class/[Vereien]<>-100..*[Mitgield])

Dies bedeutet, dass ein Verein aus mind 100 Mitgliedern besteht.
Dies schließt jedoch explizit nicht aus, dass ein Mitglied in mehreren Vereinen ist.


![](http://yuml.me/diagram/scruffy;/class/[Rugby_Verein]<>-200..*[Mitgield],[Fußball_Verein]<>-100..*[Mitgield])

Es ist möglich, dass ein Mitglied in beiden Bereichen ist.
Den Rugby-Verein zeichnet aus, dass er mind. aus 200 Mitgliedern besteht, den Fußball-Verein, dass dieser mind. 100 Mitglieder aufnimmt.
Um beide Vereine zu implementieren, müssten also wenigstens 200 Mitglieder vorhanden sein,
jedes Mitglied im Fußball-Verein müsste bei 200 Mitgliedern auch im Fußball-Verein sein.

##### Qualified Associations

bei Qualified Associations kann mit angegeben werden, über welchen Wert (z.b. benötigter Parameter oder HashKey) eine Assoziation aufgebaut wird.
Dies wird durch einen kleineren eckigen Kasten neben der der Klasse dargestellt. Yuml unterstützt leider keine Qualified Associations, aus diesem Grund gibt es hier kein Beispiel.

##### Beispiel Komposition

Gibt man explizit über die gefüllte Raute an, dass es sich um eine Komposition handelt,
so besteht nicht die Möglichkeit das z.B. ein Mitglied in beiden Vereinen ist.

![](http://yuml.me/diagram/scruffy;/class/[Rugby_Verein]++-200..*[Mitgield],[Fußball_Verein]++-100..*[Mitgield])

Dies würde explizit nicht zulassen, dass ein Mitglied in mehren Vereinen ist.
Entsprechend bräuchte man also mindestens 300 Mitglieder.

Komposition wird idr. dafür verwendet eine explizte Zuordnung zu kennzeichen.

![](http://yuml.me/diagram/scruffy;/class/[Customer]++->[Address],[Customer]++1-0..1>[DeathDate],[Name|firstname;lastname],[Address]++1-1>[Name],[Customer]++1-1>[EMail], [Customer]-[note: Aggregate Root{bg:cornsilk}])

Möchte man einen Aggratation Root definieren, kann man über die Komposition explizit mit angeben,
dass ein Entity nur für ein Aggrate Root bzw. ein Value Objekt nur für ein Entity existiert.
Die Objekte werden nicht von anderen Instanzen verwendet.

Ein Customer hat in diesem Beispiel immer eine E-Mail. Diese E-Mail wird von keinem anderen Customer verwendet.


## Dependencies

Abhängigkeiten lassen sich in UML durch eine gestrichelte linie aufzeigen.
Durch Abhängigkeiten signalisiert man, dass wenn sich in einer etwas ändert, dies womöglich zu Änderungen in einer anderen Klasse führt.


![](http://yuml.me/diagram/scruffy;/class/[Some_Service]-[Legacy_Api_Adapter],[Legacy_Api_Adapter]-.->[Legacy_Api])

Wird etwas im Legacy_Api_Adapter verändert, so verändert dieser die Legacy_Api.
Diese wird von "some_service" verwendet.





