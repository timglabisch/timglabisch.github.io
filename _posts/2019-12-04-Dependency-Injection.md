---
layout: post
location: Düsseldorf
tags: [ php architektur ]
title: "Dependency Injection"
---

i am slightly surprised myself to write a blogpost about dependency injection in 2019. 
But the blogpost is more about what you can actually achieve with dependency injection and is probably more suitable for people who work with it every day.

```php
class ServiceA {
    private $serviceB;
    
    public function __construct(ServiceB $serviceB) {
        $this->serviceB = $serviceB;
    }
    
    private function doX() {
        doSomething();
        return $this->serviceB->doXYZ();
    }

}
```

I'll leave the example here abstract. In the end, a ServiceA delegates work to ServiceB. 
With DependencyInjection the whole thing can be wired.

But we always have to ask ourselves when it makes sense and when it doesn't.

Let's compare the whole thing with this:

```php
class ServiceA {
    private $channel;
    
    public function __construct(Channel $channel) {
        $this->channel = $channel;
    }
    
    private function doX() {
        doSomething();
        return $this->channel->send(new DoXYZTask());
    }

}
```

Ultimately, we have similar advantages here, instead of a method we call up on some channel and expect a response.

If you look at the whole thing in a tool like deptrac you would see that you have one more dependency, namely on the "channel". You also lose any typehinting concerning the send method.

I changed my view of things for the first time when I tried to reimplement a DI container in Rust. 

the question that you quickly ask yourself is what is the lifetime of such a service.
ServiceA should be available longer than ServiceB? A complex graph then becomes more difficult.

if you ask yourself how long a class is available in the 2nd example which can answer the DoXYZTask, you can't answer this so easily.
the second example has many disadvantages, but is more flexible regarding the lifetime of the class that can respond to DoXYZTask.

dependency injection has the major advantage that dependencies between classes can be cleanly and transparently interlinked with the possibility of indirection.

it is important to understand that a service does not necessarily have to be a class. A service can also be a function.
Here both examples rewritten as Functions:

```php

function ServiceA(ServicB $serviceB) { 
    doSomething();
    return $serviceB->doXYZ();
}

function ServiceA(Channel $channel) { 
    doSomething();
    return $channel->send(new DoXYZTask());
}
```

in the next step we try to optimize the channel example to get rid of the dependency to the channel class.

```php
function ServiceA() { 
    return yield new DoXYZTask();
}
```

we can also solve this theoretically with the yield statement.

Now it gets interesting because we get some advantages to it. The big disadvantage is that we now need a runtime in which our service runs.

It should be noted, however, that we are on a par with the first example in terms of dependency injection.

```php
function ServiceA() { 
    $a = yield new DoTaskA();
    $bOrC = yield new Race(new DoTaskB(), new DoTaskC());
    yield [new SleepTask(1), new SleepTask(2)];
    return $bOrC;
}
```

the execution is interrupted thanks to the yield statement.
However, there is only a direct dependency to the "method call" which is represented by the classes `DoTaskA`, `Race`, `SleepTask`, ... in this example.

we are still fulfilling all the advantages dependency injection brings us from the first example.

i wrote about something similar 2 years ago, you can do exciting things like collect all yields and pass them to a resolver in an orderly way. with such constructs you can dynamically load data with high performance. Also great for answering graphQL queries. we use this very successfully in production.

we are used to working in languages where we write the program after the execution path.
In the course of time I became aware that in many places it makes sense to write a program based on the structure of the data in the memory.

this is a completely different approach, but it is being used more and more.

```php
spawn(function($chan) {
    $counter = 0;
    while($req = yield $chan->poll(DoTaskA::class)) {
        $req->resolve($counter++);
    }
});

```

in a language where threading is not very common, the example seems strange. But it shows a little where the journey can go.

Now it almost looks like an actor. the interesting thing about this approach is that we have a routine that only takes care of the state (counter).

the counter is just an example, it could also be a huge hashmap. the whole thing would be very performant, because you don't need any locking or similar.

we have created an actor here, which only takes care of the corresponding data structure. Theoretically we could spawn several of them. It's the runtime how it distributes the tasks.

the exciting thing about it all is that we could now make many small actors on our huge service graph (which inevitably emerges). The lifetime of the services corresponds to a tree, the lifetime of the actors is undefined and therefore much more flexible.

i could talk about it for days, i just want to give you a superficial impression of what i think about the topic. 

please don't start removing your DI container now. Especially in the PHP world, this one is to be preferred in the sense that it is used much more and is probably enough. 
 But it's good to keep this in mind, especially if you work with languages where you can think more about memory management.
