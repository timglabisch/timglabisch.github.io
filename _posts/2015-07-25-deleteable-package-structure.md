---
layout: post
location: Düsseldorf
tags: [ php architektur DPS ]
title: "[en] Deleteable Package Structure"
---

There are several ways to structure projects at larger scale.
This blogpost describes one architectural style I like to use in medium / larger applications.
To make it easier to reference this style, i'll start calling it `DPS` (`Deleteable Package Structure`).
The examples are written using Symfony, but `DPS` is framework agnostic.


# Goals Of DPS?
- `DPS` is designed for agile teams, that are building in house projects with a duration of more than 4 months.
- `DPS` focus on building a lightweight project with the possibility to delete as much complexity as possible.
- `DPS` suits well, if the core domain is well known but the project requirements change frequently.
- `DPS` ensures that rewrites of core functionality can be made within sprints.
- `DPS` highly tries to prevent big ball of muds, even with short sprints and indecisive product owners.
- `DPS` scales well in terms of, what kind of tasks should be parallelized and when the team should come together.
- `DPS` ensures fast and efficient code reviews.

# Basic Structure
`DPS` encores 3 different kinds of `Packages`.


    | Package Type          | Responsibility                               |
    | --------------------- |--------------------------------------------  |
    | Project Package       | Project specific Files / Sourcecode.         |
    | --------------------- |--------------------------------------------  |
    | Domain Package        | Interfaces and Abstract Classes all          |
    |                       | Packages can refer. Defines the Public API   |
    |                       | Of all Data Structures and Services          |
    | --------------------- |--------------------------------------------  |
    | Package               | Implementations of Private and               |
    |                       | Public Services. Public Services must        |
    |                       | implement and return an Interface provided   |
    |                       | by the Domain Package                        |
    | --------------------- | -------------------------------------------- |


# Project Package
DPS enforces one or many `Project Packages`.
The Project Package obtains all code for all endpoints (Routing Informations, Controllers, Webservices, Templates, ...).
In a typical Symfony Application this could be the `AppBundle`.
Most projects will start with one `Project Package`.

The `Project Package` is allowed to contain:

### Framework Specific Functionality
It's totally fine to couple the framework with the Project Package.
One of the goals of `DPS` is to avoid coupling whenever it makes sense.

### API Endpoints and Controller
Every endpoint of the Application is provided by the `Project Package`.
The related controllers must only use classes / resources from it's own `Project Package` namespace or the `Domain Package` namespace.
The `Project Package` is not allowed to use any class provided by a regular `Package`.

Example

```php
    <?php

    //....
    use [Project]\Domain\Service\ProductServiceInterface;        // 1*
    use [Project]\Domain\Service\CategoryServiceInterface;
    use [Project]\Domain\Domain\ProductInterface;                // 2*
    use [Project]\AppBundle\API\ProductTransformer;              // 3*

    class ProductController {
        // ....

        function __constructor(
            ProductServiceInterface $productService,
            CategoryServiceInterface $categoryService,
            ProductTransformer $productTransformer
        ) {
            // ...
        }

        public function indexAction() {

            /** @var $products Products[] */                     // 2*
            $products = $this->productService->findAll();

            return [
                'products' => $productTransformer->tranform(
                    $products
                )
            ];
        }

    }
```

(1*) it's a good practice if controllers are just using `interfaces that
provided by the `Domain Package` and framework related code.

(2*) The `Project Bundle` can provide `public services`, it's also fine if
the `Project Bundle` makes use of it's own services.
This seems to be a good idea if the service is highly project specific.

### Templates
Every template is provided by the Project Package.

# Wire Public Services And Provide a Domain Specific Service Name.
`Packages` are providing services with knowledge of the technical implementation
and the use case they are used for.

For Example these are valid `Package` names:

- ProductMysqlBundle
- ProductDoctrineBundle

Consider the domain of the project is a classic shop.
In this case the `Project Package` is responsible for providing a `product_service`.

Instead of implementing some kind of `product_service` the `Project Package` can
`can` use `Public Services` provided by other `Packages`.
In case of a symfony application, the `Project Package` can use the alias functionallity,
to wire any service to the required `product_service`.

    <service id="product_service" alias="[project].product_mysql_bundle.product_service"/>

The `mysql` keyword becomes an implementation detail, other packages can now rely
on the `product_service` because it's wired by the `Project Package`, and the `product_service`
implements an interface provided by the `Domain Package`.
Changing the implementation of the product service, is up to the `Project Package`.
Different `Project Packages` could use the same packages in other situations,
or replace `Packages` with other `Packages`. It's also totally fine to A / B Test
different `Packages`.


### Public / Private / Tagged Services
It's totally fine if the `Project Package` defines `Public Services` and
`Private Services`. This is useful if the logic behind the service is quite easy and
specific to the project. For further reading, study the Public / Private Service Part of the `Packages`,
the restrictions are the same.

### Tips
The `Project Package` should be as small as possible and can become
that huge that rewriting isn't possible anymore. Keeping the `Project Package`
as simple as possible is a good idea.



# Domain Package
`DPS` enforces a special `Domain Package` in a distinct domain namespace.

    tree .
    |-- Domain
        |-- ProductInterface.php
        `-- CategoryInterface.php
    `-- Service
        |-- ProductServiceInterface.php
        `-- CategoryServiceInterface.php

The `Domain Package` is `must` be framework-agnostic.

The `Domain Package` must only contain:

### Public Service Interfaces
Interfaces that are designed for stateless, framework-agnostic services.

### Domain DTO Interfaces
Every data structure that is used to share data between `Packages` must be provided by
as in interface by the `Domain Package`.
For example, if the core domain is a shop, the `Domain Package` provides an interface for a product,
if the product contains variants, or other subclasses, they must be provided as interfaces, too.

### Domain DTO Abstract Classes
Abstract classes with boilerplate data structures. Use abstract classes wisely and rare, prefere interfaces over
abstract classes whenever possible.

### Tags
The `Domain Package` can define some `Service Tags` Tags can be used by the `Project Package` and by
any other `Packages` to wire services together.

### Tips
Every team member should know (at least) most of the domain interfaces.
Interfaces in the `Domain Package` should be defined wisely and with care.
Focus on facts of your domain, not on technical decisions.
Design the interface as small as possible. Design nothing you don't need.
By reading the service definition a new team member knows every important service, this ensures
that new team members don't get overstrained.


# Packages
Different functionality should live in different `Packages`.

Every `Package` must only use classes / resources from it's own `Package` namespace or the `Domain Package` namespace.
`Packages` `should not` contain interfaces or any abstractions.
`Package` are responsible for providing one, efficient solution.
`DPS` prefers rewriting `Packagey` over building a big ball of mud.

`Packages`are designed to be small, efficient, easy to understand and maintainable by a small team.

Packages must only contain:

### Private Services
Every `Package` is allowed to contain services / classes that are used internally to structure the `Package`.
A `Private Service` must not be used outside of the `Package` namespace.
By default every service is a `Private Service`.

### Public Services
A `Private Service` becomes a `Public Service` if it implements an interface from the `Domain Package.
A `Public Service` `must` return native PHP Types (DateTime, Array, String, Int, ...) or instances that implement
an interface in the `Domain Package`.

`Public Services` can be wired by the `Project Package`, so the `Project Package` and any other `Package` can
make use of any `Public Service` by using the interface the `Public Service` implements from the
`Domain Package`.

### Public Tagged Services
Services can be wired together using tags. Global tags are provided by the `Domain Package` by definition.
Every `Public Tagged Service` `must` implement a tag related interface, provided by the `Domain Package`.


### Instance Handling
Every `Package` has to provide a solution to access the `Public Services`.
For example in a typical Symfony Application, a `Package` could be represented as a Bundle.

### Tips
`Packages` do the hard work, don't overengineer them. No more abstractions, keep performance in mind and do the job.
This doesn't mean that every `Package` should be a mess, it means, `Packages` can be rewritten.
`Packages` should focus on one requirement, this ensures that a `Package` is easy enough to be rewritten.

# Deleteability
`DPS` enforce to write deleteable code. Small interfaces are representing all of
the functionality that has to be shared between `Packages` and the `Project Package`.
Packages `must` be able to rewrite within 2 weeks by one developer.
`DPS` ensures that parts of the application can be rewritten.
In agile teams this ensures, that one `Package` can be rewritten within one sprint.

# Faq
### Where Should Repositories Live In A `DPS` Structure?
`DPS` don't focus on the differences between repositories and services.
For example if you use doctrine, the repository is part of the implementation detail
of a doctrine specific package. The `Project Package` never deals with any kind of repository.
In case of managing products, the `Package` `may` make use of an doctrine repository.
It's up the the developer of the `Package`, if the repository implements some kind of product service
interface provided by the `Domain Package` or if the `Package` uses one or many repositories under the hood.

```php
    <?php

    //....
    use [Project]\Domain\Service\ProductServiceInterface;        // 1*
    use Doctrine\ORM\EntityRepository;

    class ProductRepository extends EntityRepository implements ProductServiceInterface {

        /**
        * @return ProductInterface|null
        */
        public function getById($productId) {                    // 2*

            return [
                'products' => $this->find($productId)
            ];
        }

    }
```

OR

```php
    <?php

    //....
    use [Project]\Domain\Service\ProductServiceInterface;
    use [Project]\DoctrineProductService\Dto\ProductDto;
    use Doctrine\ORM\EntityRepository;

    class ProductService implements ProductServiceInterface {

        // ...
        public function __constructor(EntityManager $em) {
            $this->em = $em;
        }

        /**
        * @return ProductInterface|null
        */
        public function getById($productId) {

            $products = $em->getRepository(ProductDto::class)
                           ->find($productId)
                        ;

            return [
                'products' => $products
            ];
        }

    }
```

(1*) It doesn't matter what kind of class the ProductServiceInterface implements.
Other `Packages` must not use the repository, just the ProductServiceInterface.

(2*) It's totally fine to declare typically repository functions provided by EntityRepository
in the ProductServiceInterface as long as they take some kind of structured data / object or
scalar types as input. For example it's a bad practice to share a generic findBy(array $args).
Such methods should be replaced by special methods. This makes refactoring much easier and
your public API stays as small, as possible (even with more methods).

Even if the second example isn't that smart, it don't violate `DPS`.
Everything is hidden between the ProductInterface and can be rewritten without changing
any `Project Package` or any other `Package`.

### Where should Entities / Dto's live?
Entities and Data Transfer Objects (Dto's) lives in the related `Package`.
Consider there are two different implementations of a product `Package`.

One implementations takes care of loading products from a MySQL database.
Now the team wants to change the way how products are stored, the idea is to
challenge PostgreSQL against MongoDB as product storage.

The team starts developing 2 new `Packages`.

The structure now looks:

    tree .
    |-- MySqlProductBundle
        |-- Dto
            |-- ProductDto
            |-- ProductVariantDto
        `-- // ...
    `-- PostgresProductBundle
        |-- Dto
            |-- ProductDto
            |-- ProductVariantDto
        `-- // ...
    `-- MongoDBProductBundle
        |-- Dto
            |-- ProductDto
            |-- ProductVariantDto
            `-- // ...

Now the team can play around with all these solutions and pick the best one.
`Packages` should be as smart as possible, developing such a `Package` should
never takes more than one sprint (max. 14 days) for one developer.
A typical scrum team with 5 backend developers theoretically should be able to build
5 different implementations of the same `Package` peer sprint.

All these Dto's must implement the ProductInterface provided by the `Domain Package`.
Deleteing / Replacing `Packages` `must` be easy.


### In Symfony, Is A Package A Bundle?
well, it seems to be a good idea, to model the `Project Package` as an AppBundle,
`Packages` as Bundles and the `Domain Package` as Library.

#### Why Is A Package Not A Library?
`Packages` have to expose some kind of service configurations.
Libraries can't integrate services without becoming a Bundle.
`DPS` makes it easy to refactor `Packages` to libraries, if (and only if) your requirements are changing.

### In PHP, Should Every Package Live In A Composer Package?
In theory this works great, in practice, it's much easier to develop the `Packages` and the
`Domain Package` next to your main `Project Package`.
If and only if the Project has more than one `Project Packages` it's a good practice to use a
subtree splitter, to provide `Packages` and the `Domain Package` as composer package.

### How To Model Deep Bidirectional Object Structures, Like Huge ORM Object Graphs?
`DPS` enforce multiple small `Packages`. Huge object graphs violates the idea of `DPS`.
Most time it makes sense to define one `Package` for one aggregate root (DDD).
An aggregate root `must not` contain other aggregate roots.

#### How To Model References Between Object Structures?
References `must` be modeled using an identifier. An identifier `could` be any scalar value or
a dedicateded Id Object. The Id Object `must` is part of the `Domain Package` knowledge,
therefore it `must` implement a dedicated Interface provided by the `Domain Package`.

```php
    <?php

    //....
    use [Project]\Domain\Domain\ProductInterface;

    class Product implements ProductInterface {

        /** @return string */
        public function getId() {
            // ...
        }

    }
```

OR

```php
    <?php

    //....
    use [Project]\Domain\Domain\ProductInterface;
    use [Project]\Domain\Domain\Product\ProductIdInterface;
    use [Project]\ProductBundle\Dto\ProductId;

    class Product implements ProductInterface {

        /** @return ProductIdInterface */
        public function getId() {
            return new ProductId(/* ... */);
        }

    }
```

### Why Enforce `DPS` Small and Deleteable `Packages`?
`DPS` is about writing as simple and as clean code as possible.
To enrich this `DPS` ensures that with changed requirements or new
technical solutions `Packages` `can` be rewritten.

`DPS` is written with Facebooks `The Hacker Way` in mind.
"Be Open" for other implementations.
Every `Package` "can be replaced by "a better one".
Make sure, that everyone can play / hack on new and hopefully better implementations.
"Focus On The Impact" of every `Package`, "Move Fast" and "Don't Be Afraid" by changing,
deleting code.

Every `Package` should make strict decisions, this ensures that the `Package`
is simple and the implementation highly specific to the problem it solves.

The `Domain Packages` Interfaces are responsible to make sure, that everything
is replaceable by a better implementation.

`Packages` must not contain `Abstractions`, `Packages` `must` implement
decisions. Decisions the `Project Package` and the `Domain Package` `must not`
take.

###  Why `DPS` Makes Developers Happy?
`DPS` focus on a clean and easy structure, that is really hack and replaceable.
As developer you can change and rewrite parts of the application fast, change
parts of the technology stack and it's hard to encounter situations, where
the design of the software slows you down.

As developer you're able to play with the best implementation for a Package.

### Why `DPS` Makes Product Owner Happy?
Packages are highly specific to a problem. Most time this means, the application
performs very well. During a longer project the complexity stays low, this
ensures that the development team isn't slowed down. `DPS` focus on a
clean structure, this ensures that it's very hard to end up with a big ball of mud.

### Why `DPS` Makes Your Company Happy?
`DPS` Focus on small, but nearly perfect `Packages`. This means the development
team isn't building one huge software.
For new team members it's very easy to learn enough about the project,
to work on it. This ensures that new team members can join and unjoin the team efficiently.

Projects written using `DPS` `should` never need be rewritten.
`DPS` preferes rewriting a `Package` whenever there is a better solution,
over huge and risky rewrites of the complete software.