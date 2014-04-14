---
layout: post
location: Düsseldorf
tags: [ symfony bundle httpkernel ]
title: "Symfony2 PHPStorm Extension Extension."
---

a few days ago i thought about writing a PHPStorm plugin for symfony.
most time i start to develop new stuff i spend a few minutes to look for existing implementations.

a few minutes later i looked at the code of [this awesome Symfony2 Plugin](https://github.com/Haehnchen/idea-php-symfony2-plugin).

i spend a few minutes with this code, installed IntelliJ and played around with modifying the code.

this tutorial is just about learning how this kind of plugins works. it's not a best practice advice.
i am quite new to intelliJ Plugins, so keep on reading the documentation :)

i got some feedback from Alexey Gopachenko, so i started to
refactor the code, droped useless stuff, wrote a simpler and hopefully not deprecated code.

i also split the code from the symfony extension.

## challenge
this example is as simple as possible, we will provide an autocompletion for something like a service locator.
i'll focus on completion and resolving the parameter of the "get" function.
If i have some spare time i'll try to add the autocompletion to the return Type of the "get"-Function.

## pre
1. download intelliJ ultimate and a jdk
2. create a Plugin.
3. add the phpstorm, twig and yaml jars from phpstorm to your project and select "provided" as build type.
4. clone the plugins sourcecode.


## create the foo extension

### reference contributor
at first we register a reference contributor.
a reference contributor is about analyzing the ast (PSI) using event hooks like "call this on every String".
a refernce contributor can register unlimit of these hooks using the registerReferenceProvider method.
intellJ colled this method everytime it requires a reference for autocompletion or to resolve a resource.

```java
package de.timglabisch.idea.demo;

import com.intellij.patterns.PlatformPatterns;
import com.intellij.psi.*;
import com.intellij.util.ProcessingContext;
import com.jetbrains.php.lang.psi.elements.*;
import org.jetbrains.annotations.NotNull;


public class FooTypeReferenceContributor extends PsiReferenceContributor {

    @Override
    public void registerReferenceProviders(PsiReferenceRegistrar psiReferenceRegistrar) {
        psiReferenceRegistrar.registerReferenceProvider(
            PlatformPatterns.psiElement(StringLiteralExpression.class),
            new PsiReferenceProvider() {
                @NotNull
                @Override
                public PsiReference[] getReferencesByElement(@NotNull PsiElement psiElement, @NotNull ProcessingContext processingContext) {

                    PsiElement variableContext = psiElement.getContext();
                    if(!(variableContext instanceof ParameterList)) {
                        return new PsiReference[0];
                    }

                    ParameterList parameterList = (ParameterList) variableContext;
                    if (!(parameterList.getContext() instanceof MethodReference)) {
                        return new PsiReference[0];
                    }

                    MethodReference methodReference = (MethodReference) parameterList.getContext();
                    PsiElement method = methodReference.resolve();
                    if(!(method instanceof Method)) {
                        return new PsiReference[0];
                    }

                    // not we have the method and it's context
                    // you could for example check some constraints...

                    /*
                    if(!((Method) method).getContainingClass().getNamespaceName().equals("\\Tg\\BlogBundle\\DependencyInjection\\")) {
                        return new PsiReference[0];
                    }

                    if(!((Method) method).getContainingClass().getName().equals("TgBlogExtension")) {
                        return new PsiReference[0];
                    }
                    */

                    if(!((Method) method).getName().equals("get")) {
                        return new PsiReference[0];
                    }

                    return new PsiReference[] {
                            new FooTypeReference((StringLiteralExpression) psiElement)
                    };

                }
            }
        );
    }

}

```

look at the package name if you dont know where to place the code.

intelliJ merges autocompletions for all PsiReferences that all PsiReferenceProviders are returning.
i had some trouble with doing heavy work like getting stuff from the index in the ReferenceContributor.

the example returns a reference to a TypeReference.
such types are responsible for resolving a list of autocompletions and resolving references.

### FooTypeReference

```java
package de.timglabisch.idea.demo;

import com.intellij.codeInsight.completion.PlainPrefixMatcher;
import com.intellij.codeInsight.lookup.LookupElement;
import com.intellij.codeInsight.lookup.LookupElementBuilder;
import com.intellij.psi.*;
import com.jetbrains.php.lang.psi.elements.*;
import com.jetbrains.php.PhpIndex;
import com.jetbrains.php.lang.psi.elements.StringLiteralExpression;
import org.jetbrains.annotations.NotNull;
import org.jetbrains.annotations.Nullable;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;

public class FooTypeReference extends PsiReferenceBase<PsiElement> implements PsiPolyVariantReference {

    private StringLiteralExpression element;

    public FooTypeReference(@NotNull StringLiteralExpression element) {
        super(element);
        this.element = element;
    }

    public Collection<String> getPossibleClassnames(String prefix) {
        return PhpIndex.getInstance(this.getElement().getProject()).getAllClassNames(new PlainPrefixMatcher(prefix));
    }

    public String getClassName() {
        return this.element.getContents().replace("IntellijIdeaRulezzz ", "");
    }

    @NotNull
    @Override
    public ResolveResult[] multiResolve(boolean incompleteCode) {
        List<ResolveResult> results = new ArrayList<ResolveResult>();

        for(String s : this.getPossibleClassnames(this.getClassName())) {

            PhpClass klass = PhpIndex.getInstance(this.element.getProject()).getClassByName(s);

            if (klass != null) {
                results.add(new PsiElementResolveResult(klass));
            }
        }

        return results.toArray(new ResolveResult[results.size()]);
    }

    @Nullable
    @Override
    public PsiElement resolve() {
        ResolveResult[] resolveResults = multiResolve(false);
        return resolveResults.length == 1 ? resolveResults[0].getElement() : null;
    }

    @NotNull
    @Override
    public Object[] getVariants() {

        List<LookupElement> lookupElements = new ArrayList<LookupElement>();

        for(String s : this.getPossibleClassnames(this.getClassName())) {
            lookupElements.add(LookupElementBuilder.create(s));
        }

        return lookupElements.toArray();
    }

}
```



## register the extension
now look at the /META-INF/plugin.xml
just add another psi.referenceContributor

```
<psi.referenceContributor implementation="de.timglabisch.idea.demo.FooTypeReferenceContributor"/>
```

## Compile :)
now you're ready, compile the extension and test the extension :)


