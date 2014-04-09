---
layout: post
location: Düsseldorf
tags: [ symfony bundle httpkernel ]
title: "The Http-Kernel knows."
---

most time building a new bundle using symfony i just use generate:bundle and drop all the useless stuff.
to learn more about symfony i decided to create bundles from scratch or build my own generator.
the first thing i stumbled around was that the bundle i create inherit from the Symfony\Component\HttpKernel\Bundle\Bundle .
why the hell does the HttpKernel provide an interface for Bundles?
it a bundle about the Symfony/FrameworkBundle or about the HttpKernel?
the HttpKernel should be just about converting responses to requests?
so i opened the docs for the HttpKernel pressed strg+f and searched for Bundle.
i didn't found anything related to the concept of bundles in the HttpKernel documentation.

lets look at the description of the HttpKernel

```
The HttpKernel component provides a structured process for converting a Request into a Response by making use of the EventDispatcher. It's flexible enough to create a full-stack framework (Symfony), a micro-framework (Silex) or an advanced CMS system (Drupal).
```

this task shouldn't be related to MVC, to Controllers or Bundles.
It should be just about converting?

in reality the HttpKernel is about resolving a Controller. I really would prefer something like CallableResolver.
but a controller could be everything, may it's not about a MVC-Controller :)

in the documentation you will find this:

```
...at this point the kernel has a PHP callable (the controller)...
```

take a look at the code you'll find the function getController

```php
<?php
public function getController(Request $request)
{
    if (!$controller = $request->attributes->get('_controller')) {
        if (null !== $this->logger) {
            $this->logger->warning('Unable to look for the controller as the "_controller" parameter is missing');
        }

        return false;
    }

    if (is_array($controller) || (is_object($controller) && method_exists($controller, '__invoke'))) {
        return $controller;
    }

    if (false === strpos($controller, ':')) {
        if (method_exists($controller, '__invoke')) {
            return new $controller();
        } elseif (function_exists($controller)) {
            return $controller;
        }
    }

    $callable = $this->createController($controller);

    if (!is_callable($callable)) {
        throw new \InvalidArgumentException(sprintf('The controller for URI "%s" is not callable.', $request->getPathInfo()));
    }

    return $callable;
}
```

you can see that a controller could be everything that is callable and event better the variable returned from the getController
function is called callable.


let's agree on that the HttpKernel needs something that is callable and calls this controller.
its not about mvc or related patterns. its just something called controller :)

if you'll look at the Kernel class you'll notice that the kernel is explicit responsible for booting bundles, terminating them and so on.
the concept of bundles isn't really about the SymfonyFramework. It's about the HttpKernel.

think about the default AppKernel, it inherits directly from the the HttpKernel.
the function registerBundle is used to register all kind of bundles.
if you take a look at the list, the first bundles you will notice takes care aboutbooting the SymfonyFramework.

the FrameworkBundle is nothing more than a bundle like yours. and you have the control to enable and disable it.

just keep in mind that bundles and controllers (callables :)) are about the HttpKernel and not the FrameworkBundle.
the FrameworkBundle is really just a bundle for the HttpKernel.

