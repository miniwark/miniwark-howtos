# Setup the Hardware Watchdog Timer on Ubuntu 12.04

### Warning
> **This How-to is outdated, it may be useful... or not...**

## Intro

The Hardware Watchdog Timer is a tool provided by IPMI on servers. It is generally used to automatically reboot the computer if this one crash or hang for too long. It can also be used to force stop a server in case of heat problem.

## Activate the Watchdog

The Hardware Watchdog Timer must first be enable from the BIOS (or UEFI) to be effective. You may also be able to activate it remotely if your server have a specific administration tool for this (like Dell DRAC).

When done, the kernel modules for IPMI will be automatically added at the next boot. You can check them with:
```
> lsmod | grep ipmi  
```

Witch will list something like:
```
ipmi_devintf
ipmi_si
ipmi_msghandler
```

## Install the IPM tools and the Watchdog daemon

Install the tools to check and manage IPMI:
```
> apt-get install openipmi  
> apt-get install ipmitool
```

And install the Watchdog daemon :
```
> apt-get install watchdog  
```

## Configure the watchdog

Before configuration you can check the hardware watchdog timer status:
```
> ipmitool mc watchdog get
```
You will get a result like:
```
Watchdog Timer Use:     Reserved (0x00)
Watchdog Timer Is:      Stopped
Watchdog Timer Actions: No action (0x00)
Pre-timeout interval:   1 seconds
Timer Expiration Flags: 0x00
Initial Countdown:      15 sec
Present Countdown:      15 sec
```

In this case, the timeout is only 15 seconds, the Watchdog is inactive and it will perform no actions.

Personality I have decided for this options: 
* a 5 minutes countdown (300 secs)
* a ping from the daemon every 10 seconds
* a reboot in case of freeze

For this we need to edit the watchdog daemon config file `/etc/watchdog.conf`:
```
watchdog-device   = /dev/watchdog
watchdog-timeout  = 300
interval          = 10
```

Make sure also to keeps this options in the file to give a hight priority to the watchdog daemon:
```
realtime         = yes
priority         = 1
```

By default the `impi_watchdog` module is not automatically loaded by the kernel. You need to explicitly add it at the end of  `/etc/modules`:
```
loop
lp
rtc
ipmi_watchdog
```

The reboot (reset) in case of freeze is the default action. Other actions are 'none', 'power_cycle' and 'power_off'. To change the action it may be necessary to use `modprobe` or edit the `/etc/modules` but I have not tested this.


## The machinery

The job of the daemon is to reset the Hardware Watchdog Timer every 10 seconds (the `interval` parameter). If the daemon can't speak to the timer (kernel panic, memory freeze, etc.) then the countdown will continue until 0. When the countdown is 0 then the action is done (a reboot by default). 

The daemon can also force an action if it is configured to do so (for example halt the computer if the temperature is too high or launch a custom script). See the Watchdog man page for the other options.


## Warning 

There is two obvious cases where the daemon can't speak any-more to the hardware timer:
 * The system have crashed or is overloaded
 * The daemon itself have die

The first case, is the expected behaviour (reboot if crashed). For the second case you can also setup a service monitor utility (like _monit_) to avoid an unexpected reboot.


## Testing

After reboot test first than all the necessary modules are loaded:
```
> lsmod | grep ipmi
```
You result must be like:
```
ipmi_devintf
ipmi_si
ipmi_watchdog
ipmi_msghandler
```

Check the daemon
```
> service watchdog status
```
With must says:
```
* watchdog is running
```

Check the hardware timer status:
```
> ipmitool mc watchdog get
```
Witch display something like:
```
Watchdog Timer Use:     SMS/OS (0x44)
Watchdog Timer Is:      Started/Running
Watchdog Timer Actions: Hard Reset (0x01)
Pre-timeout interval:   0 seconds
Timer Expiration Flags: 0x00
Initial Countdown:      300 sec
Present Countdown:      292 sec
```

If you do it a few times, you will see the countdown being reset regularly by the daemon.


## Deactivate the watchdog

You may need to deactivate the watchdog for maintenance stuff.
```
> ipmitool mc watchdog off  
> service watchdog stop
```

## References
http://manpages.ubuntu.com/manpages/precise/man5/watchdog.conf.5.html
http://publib.boulder.ibm.com/infocenter/lnxinfo/v3r0m0/index.jsp?topic=%2Fliaai%2Fipmi%2Fliaaiipmiwatchdog.htm
http://buttersideup.com/docs/howto/IPMI_on_Debian.html
http://ipmitool.sourceforge.net/manpage.html
http://www.admin-linux.fr/?p=5267
http://blog.crifo.org/post/2010/04/02/Configurer-IPMI-simplement
