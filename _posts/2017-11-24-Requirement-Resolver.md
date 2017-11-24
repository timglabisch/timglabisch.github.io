---
layout: post
location: Düsseldorf
tags: [ php architektur ]
title: "Requirement Resolver"
---

i worked on something i call Requirement Resolver Pattern.

consider you've a simple productlist. you may want to load product names from tableA,
ratings from tableB and prices from tableC.

in the wild you often see two differnt solutions to solve this:

## Query in Loop

consider you've a productlist with 20 items.

Item1
Item2
Item3

if you would iterate over the products and load for each entry every product name, rating and price you would end up
with sequential 30 queries.

## Bulk Preload

in the past i often saw on such an implementation:

```php

$products->loadAllProducts();

foreach ($products as $product) {
    $productIDs[] = $product->getID();
}

$ratings = $ratingService->loadByProductIds($productIDs);
$categories = $categoryService->loadByProductIds($productIDs);

```

The second solution is much smarter, but can we even improve this?

# Requirement Resolver Simple Example

when we transform the bulk preload example to the requirement resolver pattern it would look like this

```php

$this->requirementResolver->resolveAll([
    $allProductsRequirement = ProductsRequirement::All()
]);

$products = $allProductsRequirement->getProducts();

```

at this level this doesn't look much smarter, but we'll see later why this can be very cool.

so, what we're doing here?

instead of querying data, we describe that we've a requirement, we want all products.

the requirement resolver is responsible for resolving such requirements.

its important to notice that the requirement resolver can resolve multiple requirements at the same time.

consider you would have more than one requirement, for example you would like to show the top ratings, top products and the latest 10 products:

```php

$this->requirementResolver->resolveAll([
    $productWithID10 = ProductRequirement::WithID(10),
    $topProducts = ProductsRequirement::Top(),
    $latest10Products = ProductsRequirement::Latest(10),
    $topRatings = Ratings::Top()
]);

```

may the topProducts and the latest10Products will overlap? or may one of the topRating will be also required in one of the latestProducts or in the top Products?

the requirement resolver can take all these requirements and try to solve them more efficient.

## Requirements

a requirement is any kind of dto that describes something you would like to have.
for every type of requirement you've to implement a dedicated resolver (we'll see later).

a requirement can be decomposed in subrequirements.

for example if we've such a requirement, 

```php
$this->requirementResolver->resolveAll([
    $topProduct1 = ProductsRequirement::WithID(1),
    $topProduct2 = ProductsRequirement::WithID(2),
]);
```

it would be possible to solve both requirements with a simple `IN` SQL Query.
when our ProductRequirement would rely on rating informations, we also could load these ratings with a simple `IN` SQL Query.

just to clarify, the requirement resolver pattern is not related to sql, you could query the data from any kind of sink.

if you would ask the resolver for the ratings for product2, you could get these for free, if they are part of the ProductRequirement.

```php
$this->requirementResolver->resolveAll([
    $topProduct1 = ProductsRequirement::WithID(1),
    $topProduct2 = ProductsRequirement::WithID(2),
    $ratingsForProduct2 = ProductRatings::forProduct(2)
]);

static::assertSame($topProduct2->getRatings(), $ratingsForProduct2);
```

take a closer look at this example. $topProduct2->getRatings() and $ratingsForProduct2 could even point to the same instance.

this would allow you to collect a lot of requirements from different widgets / parts of your application (or may graphQL API?).
the requirement resolver tries to help you to resolve all these requirements as fast as possible.

# Nested Requirements

![](https://yuml.me/diagram/scruffy;/class/[Product]-%3E[Rating],[Product]-%3E[Category],[Rating]-%3E[Author])

what do we need to load such a graph?

To solve this using the Requirement Resolver Pattern we would need to have

a requirement for all types:

- ProductRequirement
- RatingRequirement
- CategoryRequirement
- AuthorRequirement

and a resolver for each type:

- ProductRequirementResolver
- RatingRequirementResolver
- CompanyRequirementResolver
- AuthorRequirementResolver


when we than want to resolve such a product Requirement:

```php

$this->requirementResolver->resolveAll([
    $product = ProductRequirement::WithID(1)
]);

 ```
 
the `ProductRequirementResolver` would emit a `RatingRequirement` and a `CategoryRequirement`

all these requirements would be collected and send  to the related resolver (`RatingRequirementResolver` and `CategoryRequirementResolver`)

the category resolver would resolve the category and the `RatingRequirementResolver` would emit a `AuthorRequirement`.

than the `AuthorRequirementResolver` would get all `AuthorRequirement`'s, resolve it.

the `ProductRequirementResolver` would now finish to resolve the `ProductRequirement`.
 
 
now consider we would have 

```php

$this->requirementResolver->resolveAll([
    $product1 = ProductRequirement::WithID(1)
    $product11 = ProductRequirement::WithID(1),
    $product2 = ProductRequirement::WithID(2),
]);

 ```
 
the `ProductRequirementResolver` would emit *3* `RatingRequirement`s and *3* `CategoryRequirement`

all these requirements would be collected and send  to the related resolver (`RatingRequirementResolver` and `CategoryRequirementResolver`)

the category resolver would resolve *2* categories and the `RatingRequirementResolver` would emit *2* `AuthorRequirement`'s.

than the `AuthorRequirementResolver` would get all `AuthorRequirement`'s, resolve it.

the `ProductRequirementResolver` would now finish to resolve all `ProductRequirement`'s.


when you would compare the number of queries, they shoud stay the same. 

#  Juice Example


Consider we want to make awesome juice.

to get the juice we simply could create a juiceRequirement and ask the resolver for it.

```php
$resolver->resolveAll([
    $juice = new JuiceRequirement()
], $this->getResolverContext());

```

so lets first take a look at the `JuiceRequirementResolver`

```php
class JuiceRequirementResolver implements RequirementResolverInterface
{
    public function supports(ResolveableInterface $requirement, ResolverContext $resolverContext)
    {
        return $requirement instanceof JuiceRequirement;
    }

    /**
     * @param ResolveableCollection|JuiceRequirement[] $resolveables
     * @param ResolverContext $resolverContext
     */
    public function resolveAll(ResolveableCollection $resolveables, ResolverContext $resolverContext)
    {
        $apples = [];
        $bananas = [];

        // for every juice we need 2 apples and one banana
        $appleAndBananas = yield $resolveables->flatmap(function (JuiceRequirement $juiceRequirement) use (&$map) {
            return [
                $apples[] = new AppleRequirement(),
                $bananas[] = new BananaRequiement(),
            ];
        });

        $i = 0;
        foreach ($resolveables as $resolveable) {

            $resolveable->resolve(new ResolvedJuice(
                $apples[$i]->getResolvedValue(),
                $bananas[$i]->getResolvedValue()
            ));
            
            $i++;
        }
    }
};
```

to produce great tasting juice, we need an apple and a banana.
the yield statement *stops the requirement resolver and gives the control back to the requirement resolver*.
the requirement resolver is now responsible to resolve these banana / apple requirements before the JuiceRequirementResolver will continue.

it's important to understand that the variable `$appleAndBananas` must be resolved. if the requirement resolver couldnt resolve these requirements
the JuiceRequirementResolver would fail because it has a requirement that could not be resolved.

even multiple calls to the resolveAll method to one resolver are possible. this could happen when the resolver yields a requirement, and than, for example another requirement resolver requires a Juice (in this example).

it's a bit hard to understand, but consider:

```php
    /**
     * @param ResolveableCollection|JuiceRequirement[] $resolveables
     * @param ResolverContext $resolverContext
     */
    public function resolveAll(ResolveableCollection $resolveables, ResolverContext $resolverContext)
    {
        $foo = yield [new FooRequirement()];
        $juice = yield [new JuiceRequirement()];
    }
```

if the juice requirement resolver would require a juice requirement, 2 differnt generators of the JuiceRequirementResolver::resolveAll would run at the same time.
normal times this isn't a good idea. but it's good to understand that a resolver could run more than once at the same time.

But lets get back to our example, to make juice, we need an apple and a banana.
Both Requirement Resolvers are looking more or less the same, they just require a Farmer.


```php

BananaRequirementResolver implements RequirementResolverInterface
{
    private $bananaFarmer;

    public function supports(ResolveableInterface $requirement, ResolverContext $resolverContext)
    {
        return $requirement instanceof BananaRequiement;
    }

    /**
     * @param ResolveableCollection|FarmerRequirement[] $resolveables
     * @param ResolverContext $resolverContext
     */
    public function resolveAll(ResolveableCollection $resolveables, ResolverContext $resolverContext)
    {
        /** @var $appleFarmer ResolvedFarmer */
        if (!$this->bananaFarmer) {
            $this->bananaFarmer =  (yield FarmerRequirement::newBananaFarmerRequirement())->getResolvedValue();
        }

        foreach ($resolveables as $resolveable) {
            $resolveable->resolve($appleFarmer->grow());
        }
    }
};

```

and the Farmer Requirement Resolver



```php 

    FarmerRequirementResolver implements RequirementResolverInterface
    {


         /** @var ResolvedFarmer */
         private $appleFarmer;

         /** @var ResolvedFarmer */
         private $bananaFarmer;

         public function __construct()
         {
             $this->appleFarmer = ResolvedFarmer::newAppleFarmer();
             $this->bananaFarmer = ResolvedFarmer::newBananaFarmer();
         }

         public function supports(ResolveableInterface $requirement, ResolverContext $resolverContext)
         {
             return $requirement instanceof FarmerRequirement;
         }

         /**
          * @param ResolveableCollection|FarmerRequirement[] $resolveables
          * @param ResolverContext $resolverContext
          */
         public function resolveAll(ResolveableCollection $resolveables, ResolverContext $resolverContext)
         {
             foreach ($resolveables as $resolveable) {
                 if ($resolveable->requiresAppleFarmer()) {
                     $resolveable->resolve($this->appleFarmer);
                     continue;
                 }

                 if ($resolveable->requiresBananaFarmer()) {
                     $resolveable->resolve($this->bananaFarmer);
                     continue;
                 }

                 throw new \LogicException();
             }
         }

     };
```

its important to understand how `yield` works.
the resolver is more or less a scheduler that is spawning generators that are generating and resolving requirements.

# async resolvers

implementing async. resolvers is quite easy. 
at the moment we dont use it, but you could simple do something like this:

```php
public function resolveAll(ResolveableCollection $resolveables, ResolverContext $resolverContext)
 {
    while ($iAmNotReadyJet) {
     yield [RequireMoreTime::Seconds(0.1)]
    }
    
    // ...
 }
```

when `!$iAmNotReadyJet` the resolver would give control back to the scheduler and other requirement resolvers could work on.

# current state

we're starting to use this pattern, for large item lists. Seems to work great.
i extracted these examples from out unittests.

at the moment we didn't opensouced this implementation, but if you're interested in, feel free to let me know.
