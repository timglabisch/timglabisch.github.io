---
layout: post
location: Düsseldorf
tags: [ uml tutorial basic oop ]
title: "PHP Dynamic Class Generation Problems"
---

## Thoughts about Symfony2 Validation

the basic validation is quite simple just add just add a php, xml or yml file called validation.[FILE_SUFFIX]
to the Resources/config directory.

the yml file looks like

    Tg\BlogBundle\Entity\SomeEntity:
      properties:
        foo:
          - NotBlank: ~
      getters:
        getFoo:
            - NotBlank: ~

you can use any class as validator that extends the Validator\ContraintValidator class.
you can set framework.validation.enable_annotation to true.
this allows you add the validation to a class using annotations.

you can register a validator using the ServiceContainer.
just register the tag with an alias.