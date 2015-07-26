---
layout: post
location: Düsseldorf
tags: [ php architektur DPS ]
title: "[en] Deleteable Package Structure"
---

There are several kinds of ways to structure projects at larger scale.
This blogpost describes one architectural style I like to use in medium / larger applications.
To make it easier to reference this style, i'll start calling it `DPS` (`Deleteable Package Structure`).
The examples are written using Symfony, but `DPS` is framework agnostic.


# Who Should Take A Look At DPS?
- `DPS` is one of many styles you can apply to teams that are building one or many applications.
- `DPS` is designed for agile Teams, that are building in house projects with a duration of more than 4 months.
- `DPS` focus on building a lightweight project with the possibility to delete as much complexity as possible.
- `DPS` suits well, if the core domain is well known but the project requirements change frequently.
- `DPS` ensures that rewrites of core functionality can be made within sprints.
- `DPS` highly tries to prevent Big Ball of muds, even with short sprints and indecisive product owners.
- `DPS` scales well in terms of, what kind of tasks should be parallelized and when the team should come together and design interfaces for new services.
- `DPS` ensures fast and efficient code reviews.

# Basic Structure
`DPS` encores 3 different kinds of Packages.


    | Package Type          | Responsibility                               |
    | --------------------- |--------------------------------------------  |
    | Project Package       | Project specific Files / Sourcecode.         |
    | --------------------- |--------------------------------------------  |
    | Domain Package        | Interfaces and Abstract Classes all          |
    |                       | Packages can refer. Defines the Public API   |
    |                       | Of all Data Structures and Services          |
    | --------------------- |--------------------------------------------  |
    | Project Package       | Implementations of Private and               |
    |                       |  Public Services. Public Services must       |
    |                       |  implement and return an Interface provided  |
    |                       |  by the Domain Package                       |
    | --------------------- | -------------------------------------------- |


# Project Package
DPS enforces one or many `Project Packages`.
The Project Package obtains all of your endpoints (Routing Informations, Controllers, Webservices, Templates, ...).
In a typical Symfony Application this could be the `AppBundle`.
Most projects will start with one `Project Package`.

The Project Package is allowed to contain:

## Framework Specific Functionality
It's totally fine to couple the framework with the Project Package.
One of the goals of `DPS` is to avoid coupling whenever it makes sense.

## API Endpoints and Controller
Every endpoint of the Application is provided by the `Project Package`.
The related controllers must only use classes / resources from it's own package namespace or the domain namespace.

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

(1*) A Typically Controller using `DPS` just use interfaces that are provided by the
`Domain Package`.

(2*) The `Project Bundle` can provide public services, it's also fine if
the `Project Bundle` use it's own services if the service is highly
project specific.

## Templates
Every template is provided by the Project Package.
For Example, HTML Templates and also the structure any kind of Format provided by an API.

# Wire Public Services and provide a Domain Specific Service Name.
Packages are providing services with knowledge of the technical implementation
and the use case they are used for.

For Example these are Valid Package Names:

- ProductMysqlBundle
- ProductDoctrineBundle


For example the MysqlProductService would register a service called:
`[project].product_mysql_bundle.product_service`

The `Project Package` is responsible for providing a `product_service`, for this
responsible it `can` use `Public Services` provided by other packages.
In case of an Symfony Application, this would totally fine:

    <service id="product_service" alias="[project].product_mysql_bundle.product_service"/>

The `mysql` keyword becomes an implementation detail, other packages can now rely
on the `product_service` because it's wired by the `Project Package`, and the `product_service`
implements an interface provided by the `Domain Package`.
Changing the implementation of the product service, is up to the `Project Package`.
Different `Project Packages` could use the same packages in other situations,
or replace `Packages` with other `Packages`. It's also totally fine to A / B Test
different `Packages`.


## Public / Private / Tagged Services
It's totally fine if the `Project Package` defines `Public Services` and
`Private Services`. This is useful if the logic behind the service is quite easy and
specific to the project. For further reading, study the Public / Private Service Part of the packages,
the restrictions are the same.

## Tips
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

## Public Service Interfaces
Interfaces that are designed for stateless, framework-agnostic services.

## Domain DTO Interfaces
Every data structure that is used to share data between packages must be provided by
as in interface in the `Domain Package`.
For example, if the core domain is a shop, the `Domain Package` provides an interface for a product,
if the product contains variants, or other subclasses, they must be provided as interfaces, too.

## Domain DTO Abstract Classes
Abstract Classes with boilerplate data structures. Use abstract classes wisely and rare, prefere interfaces over
Abstract Classes whenever possible.

## Tags
The `Domain Package` can define some `Service Tags` Tags can be used by the `Project Package` and by
any other `Package` to wire things together.

## Tips
Focus on your Domain Services, think about the best interface for your domain.
Every team member should know (at least) most of your Domain Interfaces, so design them with care.
It's a good idea to work on the `Domain Package` with other team members and discussing it.
Your domain should be defined wisely, with care and should be correct (now and in years).
Focus on facts of your domain, not on technical decisions.
Design the interface as small as possible. Design nothing you don't need.
By reading the service definition a new Team Member knows every important service, this ensures
that new team members don't get overstrained.


# Packages
Different functionality should live in different Packages.

Every package must only use classes / resources from it's own Package Namespace or the Domain Namespace.
A Package `should not` contain Interfaces or any abstractions. The Package is responsible for providing one, efficient solution.
`DPS` prefers rewriting the `Package` over building a Big Ball of Mud.

Packages must only contain:

## Public Services
Every `Public Service` `must` implement an Interface from the Domain Package.
A `Public Service` `must` return native PHP Types (DateTime, Array, String, Int, ...) or objects that implement
an interface in the `Domain Package`.

## Public Tagged Services
Services can be wired together using Tags. Global Tags are provided by the `Domain Package` by definition.
Every `Public Tagged Service` `must` implement a tag related interface, provided by the `Domain Package`.

## Private Services
Every package is allowed to contain services / classes that are used internally to structure the package.
An `Private Service` `must not` implement an interface from the `Domain Package`.
An `Private Service` is allowed to use and return classes provided by the `Domain Package` and be the package itself.

## Instance Handling
Every package has to provide solution to access the `Public Services`.
For example in a typical Symfony Application, a `Package` could be represented as a Symfony bundle.

## Tips
Packages do the hard work, don't overengineer them. No more abstractions, keep performance in mind and do the job.
This doesnt mean that every package should be a mess, it means, packages can be rewritten if a requirement is changing.
Packages should focus on one requirement, this ensures that a package is easy enough to be rewritten.

# Deleteability
`DPS` enforce to write deleteable code. Small interfaces are representing all of
the functionality that has to be shared between `Packages` and the `Project Package`.
Packages `must` be able to rewrite within 2 Weeks by one developer.
`DPS` ensures that parts of the application can be rewritten.
Especially in agile teams this ensures, that core can be rewritten.

# Faq
## Where should repositories live in a `DPS` structure?
`DPS` don't focus on the differences between repositories and services.
For example if you use doctrine, the repository is part of the implementation detail
of a doctrine specific package. The Project Package never deals with any kind of repository.
In case of managing products, the package `may` has an doctrine repository.
It's up the the developer of the package, if the repository implements some kind of product service
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
scalar types as Input. For example it's a bad practice to share a generic findBy(array $args).
Such methods should be replaced by special methods. This makes refactoring much easier and
your Public API stays as small, as possible (even with more methods).
Using findBy(array $args) as implementation detail of the package, is totally fine.

Even if the second example isn't that smart, it don't violate `DPS`.
Everything is hidden between the ProductInterface and can be rewritten without changing
any `Project Package` or any other `Package`.

## Where should Entities / Dto's live?
Entities and Data Transfer Objects (Dto's) lives in the related `Package`.
Consider there are two different implementations of a product `Package`.

one implementations takes care of loading products from a MySQL Database.
Now the Team wants to change the way how products are stored, the idea is to
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
A typical Scrum Team with 5 backend developers theoretically should be able to build
5 different implementations of one package peer sprint.

All these Dto's must implement the ProductInterface provided by the `Domain Package`.
Deleteing / Replacing `Packages` `must` be easy.


