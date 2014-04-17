---
layout: post
location: Düsseldorf
tags: [ symfony entity form disble ]
title: "Disable An Option From An EntityFormType"
---

today a coworker of mine tryed to disable one option field based on an attribute for the entity form type.
the entity form type provides a great solution for displaying a selectboxes (with optgroups) or multiselects based on a doctrine entity.

```php
<?php
/**
 * @Template("TgDemoFormBundle:Demo/Form:base.html.twig")
 * @Route("/demo/form/data")
 */
function dataAction() {
    $builder = $this->formFactory->createBuilder()
        ->add('foo', 'entity', array(
            'class' => ChoiceDemo::CLASS,
            'data' => 'TgDemoFromBundle:ChoiceDemo'
        ))
    ;

    return array(
        'form' => $builder->getForm()->createView()
    );
}

```

at least you've to define the entity (data) and the entity class (class).
there a a lot of optional stuff you can provide, just read the awesome manual :).

the entity looks like this:

```php
<?php

namespace Tg\Demo\FormBundle\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * Class ChoiceDemo
 * @package Tg\Demo\FormBundle\Entity
 * @ORM\Entity
 */
class ChoiceDemo {

    /**
     * @ORM\Id
     * @ORM\Column(type="integer")
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    public $id;

    /**
     * @ORM\Column(type="string", length=255, nullable=true)
     */
    public $fieldA;

    /**
     * @ORM\Column(type="string", length=255, nullable=true)
     */
    public $fieldB;

    /**
     * @ORM\Column(type="string", length=255, nullable=true)
     */
    public $fieldC;

    /**
     * @ORM\Column(type="boolean")
     */
    public $isActive = false;

    function __toString() {
        return $this->fieldA;
    }

}

```

go on and define some Fixtures (never skip this :)).

```php
<?php

namespace Tg\Demo\FormBundle\DataFixtures;

use Doctrine\Common\DataFixtures\Doctrine;
use Doctrine\Common\DataFixtures\FixtureInterface;
use Doctrine\Common\Persistence\ObjectManager;
use Symfony\Component\DependencyInjection\ContainerInterface;
use Tg\Demo\FormBundle\Entity\ChoiceDemo;

class LoadDemoChoices implements FixtureInterface {

    function load(ObjectManager $manager)
    {
        $choiceDemo = new ChoiceDemo();
        $choiceDemo->fieldA = 'fieldA1';
        $choiceDemo->fieldB = 'fieldB1';
        $choiceDemo->fieldC = 'fieldC1';
        $choiceDemo->isActive = true;
        $manager->persist($choiceDemo);

        $choiceDemo = new ChoiceDemo();
        $choiceDemo->fieldA = 'fieldA2';
        $choiceDemo->fieldB = 'fieldB2';
        $choiceDemo->fieldC = 'fieldC2';
        $choiceDemo->isActive = false;
        $manager->persist($choiceDemo);

        $choiceDemo = new ChoiceDemo();
        $choiceDemo->fieldA = 'fieldA3';
        $choiceDemo->fieldB = 'fieldB3';
        $choiceDemo->fieldC = 'fieldC3';
        $choiceDemo->isActive = true;
        $manager->persist($choiceDemo);

        $manager->flush();
    }

}
```

the challenge is that we want to display a disabled="disabled" attribute if the isActive flag is false.
the solution is quite simple. just overwrite the correct part of your template.
{% raw %}
```twig

{% extends '::base.html.twig' %}
{% form_theme form.foo _self %}

{% block choice_widget_options %}
    {% spaceless %}
        {% for group_label, choice in options %}
            {% if choice is iterable %}
                <optgroup label="{{ group_label|trans({}, translation_domain) }}">
                    {% set options = choice %}
                    {{ block('choice_widget_options') }}
                </optgroup>
            {% else %}
                <option {% if choice.data.isActive == true %}" disabled="disabled" {% endif %} value="{{ choice.value }}"{% if choice is selectedchoice(value) %} selected="selected"{% endif %}>{{ choice.label|trans({}, translation_domain) }}</option>
            {% endif %}
        {% endfor %}
    {% endspaceless %}
{% endblock %}

{% block body %}
    {{ form(form) }}
{% endblock %}

```
{% endraw %}
look at the options tag.
instead of using _this as form theme i would suggest to use a different file.
i just overwrote the template for form element, so it's possible to just overwrite this block.

you can add a list of form themes to the form_theme tag :)


