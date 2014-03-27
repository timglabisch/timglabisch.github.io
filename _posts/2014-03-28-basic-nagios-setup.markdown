---
layout: post
location: Düsseldorf
tags: [ nagios tutorial ]
title: "Basic Nagios Setup + checking using a Http Request"
---

# Basic Nagios Setup + checking using a Http Request

if you don't monitor your webserver they couldn't be that important.
If you want to build great Software you have important webservers, so monitoring is a MUST.

Nagios is about monitoring your infrastructure. Nagios is a bit like Wordpress, it sucks but works.
If you are looking for a rock solid and easy way to monitor a few Server with less effort nagios works for you.

this tutorial shows how to setup nagios on a precise64 ubuntu.

## Install Nagios

    sudo apt-get install nagios3-common nagios-plugins-extra nagios3 curl

yeah, that was easy? verify that it works by running curl against your system.

    curl http://127.0.0.1/nagios3

you should get an error that you're not authorized. Great.

you can login to your new system using "nagiosadmin" as user and the password you choosed during the installation.


## monitor a simpel http request

navigate to

    cd /etc/nagios3

now its time to let nagios know about your webserver.

by default nagios is scanning the conf.d directory, so we can define out host in an arbitrarily file that suffixs .cnf

/etc/nagios3/conf.d/[YOUR_HOSTNAME].cnf


    define host{
            use             generic-host            ; Inherit default values from a template
            host_name       [YOUR HOSTNAME]               ; The name we're giving to this host
            alias           [YOUR_HOSTNAME]               ; A longer name associated with the host
            address         [YOUR_HOSTNAMES_IP]          ; IP address of the host
    ;       hostgroups      allhosts                ; Host groups this host is associated with
    }

i just uncomment the hostgroup, we don't dig into using hostgroups right now.
thats all you need to let nagios know about your host.

## Define a Service
services are tasks (or checks).
to define a new service we will add a new configuration file called services.cnf.

    vim /etc/nagios3/nagios.cnf

just add another cnf_file

    cfg_file=/etc/nagios3/services.cfg

now lets create the file

    define service {
            use     generic-service         ; Inherit default values from a template
            host_name       [HOSTNAME]
            service_description     HTTP
            check_command   check_http
            check_interval  1
    }

this runs the default command (check_http) in the check_interval (1 = 60 Seconds, default to 10).

## Restart Nagios

after restarting nagios you can see the new host the task and all the stats.