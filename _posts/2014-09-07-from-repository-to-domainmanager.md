---
layout: post
location: Düsseldorf
tags: [ uml tutorial basic oop ]
title: "History of persistence in PHP"
---

for a few years the php community started to think about software design.
a lot of (todays anti-) patterns were introduced like huge registries, serviceLocators, singletons and so on.

this is a small overview about technics the php community use to store data.
i tryed to sort the patterns in the order i met them in open source projects.

## active record
by default the active record pattern breaks the SRP.

example:

```php
    <?php
    $p = new product;
    $p->setName('foo');
    $p->save();

```

the problem with this is that a product is responsible for storing product information and persisting the data.
holding data and knowing about how to persist it is too much for a single class :)

the active record pattern has some more problems.
normal times you try to build your application using different layers.
for example one service layer for application services, one layer for domain services and one layer for infrastructure services.
the problem with the product class is that it knows about the infrastructure and your domain.
so it's not just about doing to much, it's also about breaking software layers at the heart of any application.


## table gateway
i used the table gateway pattern the first time when the Zend Framework 2 RC was released.
the Zend Framework isn't shipped with any kind of ORM or model-layer.
the table gateway is an oop representation of an table. Consider you have a product table:

```php
    <?php
    $p = new poduct;
    $p->setName('foo')
    $productTable->insert($p);
``

the product represents one row in the products table.
the table gateway provide low level access to the database.
the important thing is that it doesn't care about your domain, it's just an service in the infrastructure layer
to provide access to the database.
there is nothing wrong with a table gateway, for exampe a repository could use an table gateway for low level access to the database.

## respository pattern
thanks to tools like doctrine the php community stared to use the repository pattern.

example:

```php
    <?php
    $p = new product;
    $p->setName('foo');
    $productRepository->save($product);
```

this approach is quite good. make sure that you save your product instance using the productRepository and not a entityManager.
in the wild i often see code that is using the entityManager directly.
the entityManager could be an implementation detail of the productRepository but nothing more.

the goal of the reporitoy pattern is that the object is decoupled from any storage engine.
for example it's possible to implement different repositories for the same product or using patterns like dependency
injection to replace a repository. make sure that you have an easy interface for your repository and just provide functions
you really need. for example inherit from doctrines base repository makes it hard to replace it because you have to implement
so much (may useless) functions. if you really need inherit from the doctrine base repository you could cheat by implementing
an application specific interface for your repository and use the interface in your application.
later on you could refactor and just provide an repository with just the needed api.

using repositories is the best practice if you write an application based on a crud model.
building models the crud way shouldn't be a technical decision.
there are applications that are very data driven.
some people call this approach "anemic model". most of them share the opinion that this is an antipattern.
from my point of view this isn't correct, it just depends on your application.
in the wild you'll find application that are just dealing with raw data.
just make sure that your application's domain is editing raw data - than the repository pattern and the
anemic (anti-)pattern suits well for your application.


## bit more domain driven design
most of us try to build great software. writing great software is just about 2 things.
referring decisions and build a a software that suits it's domain.
it would be naughty to think that you can use software that fit's someones domain based on crud.
there a a bunch of books about domain driven design, i am not going to talk about details.
ddd is about thinking your domain and model the software for your domain.
writing great software requires that the developers are knowing about the problems they are solving,
talking to domain experts and building the software that really fits your domain.
if you don't want to spend time about your software by building a useful models just avoid thinking about ddd.
but isn't it in vain if you don't care of your domain? - most time it is.

```php
    <?php
    $p = new Product(new ProductId(3), new ProductName('some product'));
    $p->changePrice(new Money(4.99, Money::EUR));
    $p->changeSizes(Size::XXL(), Size:S());
    $productRepository->add($p);
```

writing models that fits you business can be very complex.
using valueObjects and building small aggreation roots can highly improve your domain structure.

## domain manager
domain managers are simple classes that sits in front of repositories to provide domain logic.
think about what functionality really should be provided by a repository.
there are a bunch of ideas to build smaller repository classes. one if them is using a domain manager that provides
domain specific functions as an facade for your repositories. these managers could provide high level access to the repositories.

```php
    <?php

    class OrderReminderManager {
        function __construct(OrderRepository $orderRepository, DateProvider $dateProvider) {
            // ...
        }

        function getOrders() {
            $criteria = new OrderCriteria();
            $cirteria->isUnpaid();
            $criteria->submittedBefore($dateProvider->now('3 weeks ago'));
            return $this->orderRepository->getMatched($criteria);
        }
    }

```

## event sourcing
event sourcing is more about how to persist your entities.
instead of just persisting the state of an entity using event sourcing you store every event that changes the entity.
to restore a state of an entity you can compute all state changes against the entity.
providing a logbook about all changes will allows you to reinterprete data later on.
performance can be accomplish using patterns like CQRS or taking snapshots an Eventstore is an append only datastore.
think kind of datastores can be infinitely cached.
quite intersting about event sourcing is that the only suitable solution for persisting such a logbook
is using serializing mechanisms. the database structure will be simple, but i am not a big fan of serialized data in
a database. Also it isn't possible to query serialized data by default. so you need to provide alternative views
for your data. Providing specialized views using patterns like CQRS could improvide your architecture and
the applications performance.

```php
    <?php

    class Product {

        protected $recordedEvents = array();

        static function reconstituteFrom(AggregationHistory $aggregateionHistory) {
            // ...
        }

        function changePrice(Money $price) {
            $event = new ChangePriceEvent($this->productId, $price);

            $this->recordEvent($event);
            $this->processChangePriceEvent($event);
        }

        protected function processChangePriceEvent(ChangePriceEvent $changePriceEvent) {
            $this->changePrice($changePriceEvent->getPrice());
        }

        protected function recordEvent($event) {
            $this->recordedEvents[] = $event;
        }

        function getRecordedEvents() {
            return $this->recordedEvents;
        }

    }
```

the repository can extract and persist the events. every event is processed in memory to reflect the changes and stored
for persistence. sometimes espacially in books you will see examples where an global event manager is used to dispatch
domain events in entities. i prefer using the repository to extract these events.
Events happened in the past, so name them in the past tense.
execute these events if the repository saves the changes to your entity.

## Commands
instead of using a repository to just just changes for an entity you can also procide a Handler for specific Commands.
for example the ui could create a deleteProduct command, the ProductCommandHelper can process this command by using the repository.

commands are imperative and can be send within bounded context. events are in the past tense and just notified that something had happen.

```php
    <?php

    class ProductCommandHandler {

        function __construct(ProdctRepository $productRepository) {
            // ...
        }

        function handle($command) {
            // ...
        }
    }
```

## CQRS
CQRS is an advanced pattern for segregating models to read and write models.
i suggest ready this docs to understand the basics of CQRS http://msdn.microsoft.com/en-us/library/jj591577.aspx .
there is too much to say about CQRS in this small overview.
