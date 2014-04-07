---
layout: post
location: Düsseldorf
tags: [ uml tutorial basic oop ]
title: "History of persistence in PHP"
---

for a few years the php community stated to think about software design.
a lot of (todays anti-) patterns were introduced like huge registries, serviceLocators, singletons and so on.
this time the community started to use the active record pattern to store data.

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

### ddd

## event sourcing


## CQRS