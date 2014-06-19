---
layout: post
location: Düsseldorf
tags: [ composer toran ]
title: "Composer requires Toran"
---

# Today is a bad day, Composer requires Toran.

Today [@seldaek](https://twitter.com/seldaek) announced [Toran](https://toranproxy.com).
At first i was very happy about the announcement, but then i realized that this is quite bad for the php ecosystem.

Composer becomes the standard package manager for php.
A lot of todays php code depends on composer and it's infrastructure.
There was some discussion about the ["The Only Way To Install"-Issue](https://github.com/guzzle/guzzle/issues/707#issuecomment-46255543) in the Guzzle Issue Tracker.

Toran is a proxy for composer, it allows you to make your deployment not dependending on Github and Packagist.
From my point of view this is a must have to build critical applications - but in the wild most people don't really care about.

Toran is licenced under a proprietary licence.
[@seldaek](https://twitter.com/seldaek) explains the reason in
[torans announcement](http://seld.be/notes/toran-proxy-and-the-future-of-composer).

Toran was developed under this kind of licence because @seldaek had to find a way to fund composer.
This is some kind of annoying, keep in mind, most php projects use composer but nobody cares about funding.

Composer is all about sharing code and supports the idea of opensource in a such great way.
It's so annoying that a so important tool with this deep opensource background is forced to depend on such licences.
This isn't the fault of composer, it's just disgrace for the php community.

## why this is risky
depending on something is risky. Composer is too important to depend on another project.
There are a bunch of more problems, like you have to restrict the functionality of Composer to find people to pay for the
better solution. Normal times this isn't good for a healthy community. It's not about any kind of price, it's about
that you have to restrict a so widely used tool.

i tweeted
> [@seldaek](https://twitter.com/seldaek) i know it's a problem, but from my point of view Composer is too important to force someone to pay for stability.

and good some interesting feedback like

> ... and you can add satis and medusa from the Composer ecosystem for free to enhance stability even more ...

i don't think this is true. If you start to build better alternatives to Toran you just hurt Composer. This is some kind of interesting,
by improving Composers ecosystem you hurt the business model and jeopard Composer.

[@seldaek](https://twitter.com/seldaek) mentioned
 > think of it like donating except you get something for it.

this is also not true. it's more like a vicious circle.
everyone who is paying for Toran makes Composer more depending on it and ensures that composer will never get the
ecosystem it requires. [@seldaek](https://twitter.com/seldaek) would never know if someone would like to support
Composer or just pay because he needs to use Toran.

## a better solution
of course it would be awesome if the community would be able to fund Composer and the Packagist platform to ensure
that [@seldaek](https://twitter.com/seldaek) is able to build the infrastructure we all are depending on.

if this isn't possible there should be a easy solution to support Composer without paying for Toran.
The important idea is that there should be a support plan that is independent to Toran, a funny solution would be to make
this plan as expensive as the Toran plan and give Toran as a free present. The cool thing about this idea is that sometimes,
if there are enough supporters you would't depend on people that are paying for Toran and may would be able to give it back
to Composer.

it's all about the community.











