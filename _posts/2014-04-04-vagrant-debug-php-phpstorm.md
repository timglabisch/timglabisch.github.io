---
layout: post
tags: [ vagrant cli xdebug breakpoint debug ]
title: "Debugging Cli Scripts in Vagrant Boxes using PHPStorm"
---

Debugging CLI Scripts in Vagrant Boxes is a bit pain by default.

normal times i use this script:

```
     #!/bin/bash
     XDEBUG_CONFIG="idekey=xdebug" PHP_IDE_CONFIG="serverName=wayne" php -dxdebug.remote_host=`echo $SSH_CLIENT | cut -d "=" -f 2 | awk '{print $1}'` "$@"
```

save this to your /usr/bin/xdebug or /usr/bin/phpd.
now you can activate the xdebug listener on your host system and just run xdebug or phpd and the file as argument.


