# Setup faile2ban for Apache

### Warning
> **This How-to is outdated, it may be useful... or not...**


## Intro

_Fail2ban_ is a tool who use iptable to ban suspicious people for a while. It is generally used to add a security layer to apache, ssh, ftp, etc.


## Setup Fail2ban

For the install and setup just follow the [Ubuntu How-to](http://help.ubuntu.com/community/Fail2ban). 

For the jails, I configure them all in ``jail.local`` configuration file instead of ``jail.conf`` to avoid merges during updates. The important options to take care are ``ignoreip`` and ``destemail``. 

For the jails, it is highly recommended to activate at last the ``ssh`` jail. For the other jails I have mainly add Apache related jails.


## Default Apache jails

Fail2ban come up by default with a few Apache jails. I activate the following ones :
* apache-auth
* apache-noscript
* apache-badbots

I am a bit aggressive as I consider one attack to be enough to ban except for ``apache-auth`` where I allow an user to make a few retry to type their password. Generally my jail config file is like this :
```ini
[apache-noscript]
# ban script kiddies looking for setup.php and so on
enabled  = true
port     = http,https
filter   = apache-noscript
logpath  = /var/log/apache*/*error.log
maxretry = 1
```

I am using ``maxretry=1`` to ban as soon as possible because there is a reaction time before fail2ban catch something suspicious. Even with this setup, there can have many attempts by a suspicious user before the ban. See the [Fail2ban manual](http://www.fail2ban.org/wiki/index.php/MANUAL_0_8#Reaction_time) for more infos.


## Personal Apache jails

I have write or adapt a few personal jails to complete the default ones:

* apache-clientdenied
* apache-nokiddies
* apache-nokiddies2

This jails are saved under ``/etc/fail2ban/filter.d`` and must be activated in the ``jail.local`` file.

### apache-clientdenied jail

This jail check for attempt to access forbidden places. For example, if you have reserved directories on you web server with a ``.htaccess`` file.

```text
# Fail2Ban configuration file
# Author: Miniwark

[Definition]
failregex = [[]client <HOST>[]] client denied by server configuration
ignoreregex = 
```

The corresponding ``jail.local`` setup:
```ini
[apache-clientdenied]
# ban clients who try to access to forbidden places
enabled  = true 
port     = http,https
filter   = apache-clientdenied
logpath  = /var/log/apache*/*error.log
maxretry = 1
```
This jail is aggressive, but I am still waiting for legitimate users to complain about it.


### apache-nokiddies jail

This jail try to catch vulnerability scanners tools. This scanners tests for know vulnerabilities. They generally are legitimate if your are using them yourself for checking the security of your own server, but they are certainly not if done by someone else. You have a list of some of this tools on [sectools.org](http://sectools.org/tag/web-scanners/) but there are many more.
```text
# Fail2Ban configuration file
# Author: Miniwark

[Definition]
failregex = ^<HOST> .*"GET .*w00tw00t
# try to access to an admin directory
            ^<HOST> .*"GET .*admin.* 403
            ^<HOST> .*"GET .*admin.* 404
# try to access to non-existent install directory
            ^<HOST> .*"GET .*install.* 404
# try to access to phpmyadmin (not installed)
            ^<HOST> .*"GET .*dbadmin.* 404
            ^<HOST> .*"GET .*myadmin.* 404
            ^<HOST> .*"GET .*MyAdmin.* 404
            ^<HOST> .*"GET .*mysql.* 404
            ^<HOST> .*"GET .*websql.* 404
            ^<HOST> .*"GET \/pma\/.* 404
# try to access to wordpress (I use another CMS)
            ^<HOST> .*"GET .*wp-content.* 404
            ^<HOST> .*"GET .*wp-login.* 404
# try to access to typo3 (I use another CMS)
            ^<HOST> .*"GET .*typo3.* 404
# try to access to tomcat (I do not use it)      
            ^<HOST> .*"HEAD .*manager.* 404
# try to access various strange scripts and malwares
            ^<HOST> .*"HEAD .*blackcat.* 404
            ^<HOST> .*"HEAD .*sprawdza.php.* 404

ignoreregex = 
```

This is my personal setup, mainly done by looking for strange 404 errors in the Apache access log. You can see for example, than I ban everyone who try to access to the Wordpress login page, witch is certainly suspicious because I am using another CMS. Of course, if you use Wordpress, this you must adapt this jail accordingly.

The corresponding ``jail.local`` setup:
```ini
[apache-nokiddies]
# ban script kiddies
enabled  = true
port     = http,https
filter   = apache-nokiddies
logpath  = /var/log/apache*/*access.log
maxretry = 1
```

### apache-nokiddies2 jail

This is a variation of the above jail but for the Apache error log.
```text
# Fail2Ban configuration file
# Author: Miniwark

[Definition]
# try to access to admin directory
failregex = [[]client <HOST>[]] File does not exist: .*admin.*

# vtiger suspect access
            [[]client <HOST>[]] File does not exist: .*vtigercrm.*
ignoreregex =
```

I do not use _Vtiger_ and do not ave an ``admin`` directory so nobody is expected to go there.

The corresponding ``jail.local`` setup:
```ini
[apache-nokiddies-2]
# ban some more script kiddies
enabled  = true
port     = http,https
filter   = apache-nokiddies-2
logpath  = /var/log/apache*/*error.log
maxretry = 1
```

## Roundcube webmail jail
[Roundcube webmail](http://roundcube.net/) is a popular web mail client. It is possible to add a plugin to log failed login attempts.

See [here](http://mattrude.com/projects/roundcube-fail2ban-plugin/) for more infos.
