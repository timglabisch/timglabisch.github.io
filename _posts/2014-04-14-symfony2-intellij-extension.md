---
layout: post
location: Düsseldorf
tags: [ symfony bundle httpkernel ]
title: "Symfony2 PHPStorm Extension Extension."
---

yesterday i thought about writing a PHPStorm plugin for symfony.
most time i start to develop new stuff i spend a few minutes to look for existing implementations.

a few minutes later i looked at the code of [this awesome Symfony2 Plugin](https://github.com/Haehnchen/idea-php-symfony2-plugin).

today i'll write a bit about extending this extension to do more.
it was hard to find something useful to extend because this plugin provides so much great stuff around symfony.

this tutorial is just about learning how this kind of plugins works. it's not a best practice advice.

## challenge
this example is as simple as possible, we will provide an autocompletion for the function foo inside of a controller.

## pre
1. download intelliJ ultimate and a jdk
2. create a Plugin.
3. add the phpstorm, twig and yaml jars from phpstorm to your project and select "provided" as build type.
4. clone the plugins sourcecode.


## create the foo extension

### reference contributor
at first we register a reference contributor.
a reference contributor is about analyzing the ast (PSI) using event hooks like "call this on every String".
a refernce contributor can register unlimit of these hooks.

```java
package fr.adrienbrault.idea.symfony2plugin.foo;

import com.intellij.patterns.PlatformPatterns;
import com.intellij.psi.*;
import com.intellij.util.ProcessingContext;
import com.jetbrains.php.lang.psi.elements.StringLiteralExpression;
import fr.adrienbrault.idea.symfony2plugin.Symfony2ProjectComponent;
import fr.adrienbrault.idea.symfony2plugin.util.MethodMatcher;
import org.jetbrains.annotations.NotNull;

public class FooTypeReferenceContributor extends PsiReferenceContributor {

    @Override
    public void registerReferenceProviders(PsiReferenceRegistrar psiReferenceRegistrar) {
        psiReferenceRegistrar.registerReferenceProvider(  // register one callback for resolving psi stuff
                PlatformPatterns.psiElement(StringLiteralExpression.class), // match on every String
                new PsiReferenceProvider() {
                    @NotNull
                    @Override
                    public PsiReference[] getReferencesByElement(@NotNull PsiElement psiElement, @NotNull ProcessingContext processingContext) {
                        if (!Symfony2ProjectComponent.isEnabled(psiElement)) {
                            return new PsiReference[0];
                        }

                        // check if the method signature matches.
                        MethodMatcher.MethodMatchParameter methodMatchParameter = new MethodMatcher.StringParameterMatcher(psiElement, 0)
                                .withSignature("Tg\\BlogBundle\\DependencyInjection\\TgBlogExtension", "foo") // replace this to your class!
                                .match();

                        if(methodMatchParameter == null) {
                            return new PsiReference[0];
                        }

                        // return an instance of an abstract PsiReferenceBase<PsiElement>
                        return new PsiReference[]{ new FooTypeReference((StringLiteralExpression) psiElement) };
                    }
                }
        );
    }

}
```

look at the package name if you dont know where to place the code.

intelliJ merge autocompletions for all PsiReferences that all PsiReferenceProdiders return.
Every PsiRefernce has a Method to provide autocompletion for this PsiReference.

### lets look at the FooTypeReference.

```java
package fr.adrienbrault.idea.symfony2plugin.foo;

import com.intellij.codeInsight.lookup.LookupElement;
import com.intellij.psi.PsiElement;
import com.intellij.psi.PsiReferenceBase;
import com.jetbrains.php.lang.psi.elements.StringLiteralExpression;
import org.jetbrains.annotations.NotNull;
import org.jetbrains.annotations.Nullable;

import java.util.ArrayList;
import java.util.List;

public class FooTypeReference extends PsiReferenceBase<PsiElement> {

    private StringLiteralExpression element;

    public FooTypeReference(@NotNull StringLiteralExpression element) {
        super(element);
        this.element = element;
    }

    @Nullable
    @Override
    public PsiElement resolve() {
        return null;
    }

    @NotNull
    @Override
    public Object[] getVariants() {

        List<LookupElement> lookupElements = new ArrayList<LookupElement>();

        lookupElements.add(new FooTypeLookup("hello world 123", "hello world 123"));

        return lookupElements.toArray();
    }

}
```

the getVariants is called for the autocompletion stuff. resolve is used for things like "go to".
we will focus on autocompletion right know.
the example returns one row with the static text "hello world 123".

### TypeLoookup
the typeLookup represents one row in the autocompletion, for example you can change how the tet is rendered,
change icons or somethign else.

```java
packge fr.adrienbrault.idea.symfony2plugin.foo;

import com.intellij.codeInsight.lookup.LookupElement;
import com.intellij.codeInsight.lookup.LookupElementPresentation;
import fr.adrienbrault.idea.symfony2plugin.Symfony2Icons;
import org.jetbrains.annotations.NotNull;


public class FooTypeLookup extends LookupElement {

    private String key;
    private String name;

    public FooTypeLookup(String key, String name) {
        this.key = key;
        this.name = name;
    }

    @NotNull
    @Override
    public String getLookupString() {
        return name;
    }

    public void renderElement(LookupElementPresentation presentation) {
        presentation.setItemText(getLookupString());
        presentation.setTypeText(key);
        presentation.setTypeGrayed(true);
        presentation.setIcon(Symfony2Icons.FORM_TYPE);
    }

}
```

## register the extension
now look at the /META-INF/plugin.xml
just add another psi.referenceContributor

```
<psi.referenceContributor implementation="fr.adrienbrault.idea.symfony2plugin.foo.FooTypeReferenceContributor"/>
```

## Compile :)
now you're ready, compile the extension and test the extension :)


