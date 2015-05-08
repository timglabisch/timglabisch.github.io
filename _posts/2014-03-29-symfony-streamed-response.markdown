---
layout: post
tags: [ http flush stream symfony ]
title: "Streaming Http Content using Symfony2"
---

The HTTP standard allows you to flush data as often as you can.
This allows you for example to stream a huge file to the user.

But this isn't limited to files, videos or binary data.
You can also stream HTML.
The browser tries to parse and paint this HTML as fast as possible.

So it's possible to flush content to the user as fast as you can provide it.
For example if you have a static header you could send it to the client before doing heavy tasks like running your controller actions, parsing templates or other stuff.

The idea isn't a new idea. Companies like Google and Facebook are using such optimizations for a long time.

## Framework Support

I never met a framework in the PHP world that's developed with the idea of streaming content as fast as possible.
I can't really understand why, but most frameworks want you to prepare a response and if everything is done the framework
finally flushes the response.

In the wild, this idea just sucks. Why should a client wait for the footer to display the header or the content?

## It's not just about painting
Think about external resources like CSS styles. Without loading the styles it's not possible to render a website.
Loading styles depends on network, network is io, io is slow.
Wouldn't it be awesome if you could flush the head as fast as possible to allow the browser fetching external resources?

I hope you got the idea, now lets get a bit into the technical implementation using Symfony2.

## Example using Symfony2

### Demo
I pushed a fully working demo to Github:
http://github.com/timglabisch/SymfonyDemoStreamBundle

This is just a proof of concept, it's not built to win a design award :).

Symfony by default forces you to return a "Response" instance,
but it's also possible to return an instance of the class StreamedResponse.

StreamedResponse takes a lambda that can produce any kind of output, like streaming a file, a video or HTML :).

```php

    <?php

    /**
     * @Route("/streamDemo")
     */
    public function indexAction()
    {
        $helper = new FlushHelper();

        return new StreamedResponse(function() use ($helper) {

            $top = $this->renderView('TgDemoStreamBundle:StreamDemo:top.html.twig');
            $helper->out($top);

            sleep(10);
            $helper->out('still here ?');

            sleep(10);
            echo $this->renderView('TgDemoStreamBundle:StreamDemo:bottom.html.twig');

        });
    }

```

This is a quite complex example, you can play with this demo later on.
I wrote a small helper, for now just think about `echo` if you see `$helper->out`.

The markup is flushed to the client as fast as possible. So the client gets all of the content, even if the server
is blocked by the heavy sleep task.

Let's take a look at the `$helpers->out` method.

```php

    <?php

    class FlushHelper {

        static $BUFFER_SIZE_APACHE = 65536;

        protected $bufferSize;

        function __construct($bufferSize = null) {
            $this->bufferSize = $bufferSize ? $bufferSize : max(static::$BUFFER_SIZE_APACHE, 0);
        }

        function out($output) {

            if(!is_scalar($output))
                throw new InvalidArgumentException();

            echo $output;
            echo str_repeat(' ', min($this->bufferSize,  $this->bufferSize - strlen($output)));
            ob_flush();
            flush();
        }
    }

```

Here i am fighting against the webserver. Yeah, it sucks.
The problem is about the output buffer in Apache. Even if you flush the content in PHP using flush and ob_flush, Apache will buffer all the content - for performance reasons -- kind of paradox, isn't it? :)
If you want to use such a technique make sure that the webserver buffer is small enough and you don't need to add useless characters to force the webserver to flush the markup.

## Order Matters
One problem is that the order of the markup matters.
Using CSS you can do funny things to reorder the markup.
Also keep in mind that it is possible to flush CSS that overwrites other CSS.
So it's possible to add CSS if it's need and you can change existing styles if something is loaded.

### May a Solution?
For example Facebook is using an alternative way.
They just push a grid of the website and insert the markup using Javascript.

The cool thing about this is that the order of markup doesn't matter anymore.
You just can stream the content in any order if it's ready.
Another great thing is that you can prioritize the content.
Think about HHVM's async features. Wouldn't it be cool to load all widgets in parallel and flush them as they are ready? :)

So lets create a grid we want to push:

```php

    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8" />
        <link rel="icon" type="image/x-icon" href="{{ asset('favicon.ico') }}" />
        <!-- Latest compiled and minified CSS -->
        <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.1.1/css/bootstrap.min.css">

        <!-- Optional theme -->
        <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.1.1/css/bootstrap-theme.min.css">

        <!-- Latest compiled and minified JavaScript -->
        <script src="//netdna.bootstrapcdn.com/bootstrap/3.1.1/js/bootstrap.min.js"></script>
    </head>
    <body>

    <style type="text/css">
        div {
            border:1px solid black;
            padding:1px;
        }

        #info {
            bottom:0;
            right:0;
            position:absolute;
            padding: 20px;
            font-size: 20px;
        }
    </style>

    <div id="info">
        just to notice reloads: {{ random(9999) }}
    </div>

    <div class="row show-grid">
        <div class="col-md-1"><span id="_1">...</span></div>
        <div class="col-md-1"><span id="_2">...</span></div>
        <div class="col-md-1"><span id="_3">...</span></div>
        <div class="col-md-1"><span id="_4">...</span></div>
        <div class="col-md-1"><span id="_5">...</span></div>
        <div class="col-md-1"><span id="_6">...</span></div>
        <div class="col-md-1"><span id="_7">...</span></div>
        <div class="col-md-1"><span id="_8">...</span></div>
        <div class="col-md-1"><span id="_9">...</span></div>
        <div class="col-md-1"><span id="_10">...</span></div>
        <div class="col-md-1"><span id="_11">...</span></div>
        <div class="col-md-1"><span id="_12">...</span></div>
    </div>
    <div class="row show-grid">
        <div class="col-md-8"><span id="_13">...</span></div>
        <div class="col-md-4"><span id="_14">...</span></div>
    </div>
    <div class="row show-grid">
        <div class="col-md-4"><span id="_15">...</span></div>
        <div class="col-md-4"><span id="_16">...</span></div>
        <div class="col-md-4"><span id="_17">...</span></div>
    </div>
    <div class="row">
        <div class="col-md-6"><span id="_18">...</span></div>
        <div class="col-md-6"><span id="_19">...</span></div>
    </div>

```

Now we have a bunch of ids which we can provide content for.
Lets modify the helper class.

```php

    <?php

    namespace Tg\DemoStreamBundle\Helper;

    class FlushHelper {

        // see above

        function outPlaceholder($output, $id) {
            $out = '<script type="text/javascript">';
            $out .= 'document.getElementById("'.$id.'").innerHTML = "'.htmlentities($output, ENT_QUOTES, 'UTF-8').'" ';
            $out .= '</script>';
            return $this->out($out);
        }
    }

```

Now we can start to use the placeholder in our controller.

```php

    <?php

    namespace Tg\DemoStreamBundle\Controller;

    use Doctrine\Common\Proxy\Exception\InvalidArgumentException;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;
    use Symfony\Component\HttpFoundation\StreamedResponse;
    use Tg\DemoStreamBundle\Helper\FlushHelper;


    class StreamDemoController extends Controller
    {

        /**
         * @Route("/streamDemo")
         */
        public function indexAction()
        {
            $helper = new FlushHelper();

            return new StreamedResponse(function() use ($helper) {

                $top = $this->renderView('TgDemoStreamBundle:StreamDemo:top.html.twig');
                $helper->out($top);

                sleep(2);
                $helper->outPlaceholder('start loading other stuff...'.mt_rand(0, 999999), 'info');

                sleep(3);

                $idRange = range(1, 19);
                shuffle($idRange);

                foreach($idRange as $id) {
                    $helper->outPlaceholder('LOADED_'.$id, '_'.$id);
                    $helper->outPlaceholder('Loaded '.$id, 'info');
                    sleep(1);
                }

                $helper->outPlaceholder('change styles...'.mt_rand(0, 999999), 'info');
                # lets modify some styles...

                foreach(array('yellow', 'yellowgreen', 'red', 'transparent') as $color) {
                    $helper->out('
                        <style>
                            div {
                                background-color:'.$color.';
                            }
                        </style>
                    ');
                    sleep(1);
                }

                $helper->outPlaceholder('all done.'.mt_rand(0, 999999), 'info');

                echo $this->renderView('TgDemoStreamBundle:StreamDemo:bottom.html.twig');

            });
        }

    }

```

If you want to play with this code. On my Github page is a demo bundle
http://github.com/timglabisch/SymfonyDemoStreamBundle
