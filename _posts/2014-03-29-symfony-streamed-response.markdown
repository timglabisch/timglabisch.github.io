---
layout: post
tags: [ http flush stream symfony ]
title: "Streaming Http Content using Symfony2"
---

the http standard allows you to flush data as often as you can.
this allows you for example to stream a huge file to the user.

but this isn't limited to files, videos or binary data.
You can also stream html.
the browser tries to parse and paint this html as fast as possible.

so its possible to flush content to the user as fast as you can provide it.
for example if you have a static header you could send to the client before doing heavy tasks like running your controller actions, parsing templates or other stuff.

the idea isn't a new idea, companies like google and facebook are using such optimizations for a long time.

## Framework Support

i never met a framework in the php world that's developed with the idea of streaming content as fast as possible.
i can't really understand why, but most frameworks want you to prepare a response and if everything is done the framework
finally flush the response.

in the wild, this idea just sucks. why should a client wait for the footer to display the header or the content?

## its not just about paining
think about external resources like css styles. without loading the styles it's not possible to render a website.
loading styles depends on network, network is io, io is slow.
wouldn't it be awesome if you could flush the head as fast as possible to allow the browser fetching external resources?

i hope you got the idea, now lets get a bit into the technical implementation using Symfony2.

## Example using Symfony2

### Demo
i pushed a fully working demo at github:
http://github.com/timglabisch/SymfonyDemoStreamBundle

this is just a prove of concept, it's not build to win a design award :).

Symfony by default force you ro return a "Response" instance,
but it's also possible to return an instance of the class StreamedResponse.

StreamedResponse takes a lambda that can produce any kind of output. like streaming a file, a video or html :).

```php
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

this is a quite complex example, you can play with this demo later on.
i wrote a small helper, for now just think about echo if you see $helper->out.

the markup is flushed to the client as fast as possible. so the client gets all of the content, even if the server
is blocked by the heavy sleep task.

lets take a look at the helpers out method.

```php

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

here i am fighting against the webserver. yeah, it sucks.
the problem is about the output buffer in the apache. even if you flush the content in php using flush and ob_flush the
the apache will buffer all the content - for performance reason, some kind of paradox? :)
if you want to use such a technic make sure that the webserver buffer is small enough and you don't need to add useless characters
to force the webserver to flush the markup.

## Order Matters.
one problem is that the order of the markup matters.
using css you can do funny things to reorder the markup.
also keep in mind that it is possible to flush css that overwrites other css.
so it's possible to add css if it's need and you can change existing styles if something is loaded.

### May a Solution?
for example facebook is using an alternative way.
they just push a grid of the website and insert the markup using javascript.

the cool thing about this is that the order of markup doesnt matter anymore.
you just can stream the content in any order if it's ready.
another great thing is that you can prioritize the content.
think about hhvm's async features, would be cool to load all widgets in parallel and flush them as they are ready? :)

So lets create a grid we want to push:

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

now we have a bunch of id's which we can provide content for.
Lets modify the helper class.

```php
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

now we can start to use the placeholder in our controller.

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

````

if you want to play with this code, on my github page is a demo bundle
http://github.com/timglabisch/SymfonyDemoStreamBundle