---
layout: post
location: Düsseldorf
tags: [ phpstorm qucikfix extension intellij ]
title: "PHPStorm Quickfix Extension"
---

today i had a bit time to play around with building a small QuickFix-Extension for PHPStorm.

the demo extension i build is useless but i think the code is very straight forward and easy to understand.
the code is written by try and error, if you've feedback and ideas how to improve it, please let me now.

the extension will promote itself as "MAKE EVERYTHING FINE!" Quickfix.
it will occur if there is any string that starts with "some:".

if you use the quickfix it will add an constructor to your class.

## The Annotator

at first we need an annotator, it is responsible to find code smells and add a hint.
for example if you use a codesniffer, it will promote some type of Annotation.

```java
package de.timglabisch.idea.demo.quickfix;

import com.intellij.lang.annotation.AnnotationHolder;
import com.intellij.lang.annotation.Annotator;
import com.intellij.openapi.util.TextRange;
import com.intellij.psi.PsiElement;
import com.jetbrains.php.lang.psi.elements.StringLiteralExpression;
import fr.adrienbrault.idea.symfony2plugin.util.PsiElementUtils;
import org.jetbrains.annotations.NotNull;

public class ContructorInjectionAnnotator implements Annotator {

    @Override
    public void annotate(@NotNull final PsiElement element, @NotNull AnnotationHolder holder) {

        if (!(element instanceof StringLiteralExpression))
            return;

        String t = PsiElementUtils.getText(element);

        if(!t.startsWith("simple:"))
            return;

        TextRange range = new TextRange(element.getTextRange().getStartOffset() + 8, element.getTextRange().getEndOffset());
        holder.createErrorAnnotation(range, "this just sucks!").registerFix(
                new UseContructorInjectionQuickFix()
        );
    }
}
```

here we create an errorAnntation with the infotext "this just sucks".
intellij will promote the textrange as "error".

you can register an Fix for this issue, in our case it's called "UseContructorInjectionQuickFix"


## Fixing The Code

now lets look at the Fix, it's a bit more complicated.

```java
package de.timglabisch.idea.demo.quickfix;

import com.intellij.codeInsight.intention.impl.BaseIntentionAction;
import com.intellij.openapi.application.ApplicationManager;
import com.intellij.openapi.command.CommandProcessor;
import com.intellij.openapi.editor.Editor;
import com.intellij.openapi.project.Project;
import com.intellij.psi.PsiElement;
import com.intellij.psi.PsiFile;
import com.intellij.psi.util.PsiTreeUtil;
import com.intellij.util.IncorrectOperationException;
import com.jetbrains.php.lang.psi.PhpPsiUtil;
import com.jetbrains.php.lang.psi.elements.Method;
import com.jetbrains.php.lang.psi.PhpPsiElementFactory;
import com.jetbrains.php.lang.psi.elements.PhpClass;
import com.jetbrains.php.lang.psi.elements.impl.PhpClassImpl;
import org.jetbrains.annotations.NotNull;

public class UseContructorInjectionQuickFix extends BaseIntentionAction {

    UseContructorInjectionQuickFix() { }

    @NotNull
    @Override
    public String getText() {
        return "MAKE EVERYTHING FINE!";
    }

    @NotNull
    @Override
    public String getFamilyName() {
        return "Simple properties";
    }

    @Override
    public boolean isAvailable(@NotNull Project project, Editor editor, PsiFile file) {
        return true;
    }

    @Override
    public void invoke(@NotNull final Project project, final Editor editor, final PsiFile file) throws IncorrectOperationException {
        ApplicationManager.getApplication().invokeLater(new Runnable() {
            @Override
            public void run() {

                ApplicationManager.getApplication().runWriteAction(new Runnable() {
                    @Override
                    public void run() {

                        final PsiElement klass = PsiTreeUtil.findChildOfType(file, PhpClass.class);

                        if (klass == null)
                            return;

                        if (!(klass instanceof PhpClassImpl))
                            return;

                        Method constructor = ((PhpClassImpl) klass).getConstructor();

                        if (constructor == null) {
                            CommandProcessor.getInstance().runUndoTransparentAction(new Runnable() {
                                @Override
                                public void run() {
                                    Method newConstructor = PhpPsiElementFactory.createClassEmptyConstructor(project);
                                    newConstructor.setName("some_name");
                                    PsiElement topenclass = PhpPsiUtil.getFirstChildMatchedText(klass, "{", false);
                                    klass.addAfter(newConstructor, topenclass);
                                }
                            });
                        }

                    }
                });
            }

        });
    }

}
```

if you chose the Fix in IntelliJ the invoke method is called.
in the invoke method you have to make sure that you're in a thread that is allowed to change the ast.

in the run command we are able to create a new constructor.
the PhpPsiUtil will help you to find the first child element of the class with the content "{".
you should use the psiViewer plugin to find elements in the ast.

i didn't any high level api to modify the classes, if there is one - let me know :)

than you have to add the annotator to your plugin.xml

```
<annotator language="PHP" implementationClass="de.timglabisch.idea.demo.quickfix.ContructorInjectionAnnotator"/>
```

