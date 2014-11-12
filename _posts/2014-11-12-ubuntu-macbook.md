# ubuntu on MacBookPro11,1

i installed ubuntu 14.04.01 LTS on my macbook.
i cheated a bit, i used an external ssd instead of my internal hard drive.
this allows me to boot from my external ssd (usb) by pressing "alt" during the boot.

make sure you don't format your internal drive and install the boot loader on the main drive of the external ssd,
not the new partition.

## wifi
wifi doesnt work out of the box, you've to install [bcmwl-kernel-source](http://packages.ubuntu.com/de/trusty/bcmwl-kernel-source) .
the easierst solution is to plug in an external usb rj45 network connector and install the package, of course you can
also download the package (and its dependecies) and install them using dpkg.

## kworker cpu usage
i realized that the kworker process used around 75% cpu usage of on of my core.

run 

    grep . -r /sys/firmware/acpi/interrupts/
    
and find the entry with a high number (more than 200k), now disable it (in my case gpe66):

    echo "disable" > /sys/firmware/acpi/interrupts/gpe66
    
    
## doesnt work
i have some trouble with suspending my macbook.