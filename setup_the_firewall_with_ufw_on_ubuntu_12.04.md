# Setup the Firewall with ufw on Ubuntu 12.04

### Warning
> **This How-to is outdated, it may be useful... or not...**


## Intro

In this setup we have:
* services than we want to be available everywhere (e.g. apache)
* services restricted to the local network (e.g. ssh)
* services available only for localhost (e.g. mysql)


## First steps

We will use `ufw` to manage the firewall (`iptables`) so we need first to remove `apparmor` than may interfere with our config.
```
> apt-get remove apparmor
```
(this will need a reboot to take full effect)

Now we add a default deny rule for everything
```
> ufw default deny
```
This will disable everything, including service like mysql and ldap

## Apache setup
We allow apache stuff:
```
> ufw allow http  
> ufw allow https
```
or
```
> ufw allow "Apache Full"
```

## Mail services setup

We allow a few mail services
```
> ufw allow smtp  
> ufw allow submission  
> ufw allow imaps  
```
This open ports : 25 / 587 / 993

You can be more restrictive for incoming mails if your are behind a mail relay. Instead of ``ufw allow smtp`` who open port 25 to the rest of the world you can use rules like:

```
> ufw allow from XXX.XXX.XXX.XXX to any port 25 proto tcp
```

where ``XXX.XXX.XXX.XXX`` is the mail server who relay mail for you (your ISP).
Be careful, this can be dangerous, so double tests than you are correctly receiving all your emails.


## SSH setup

I want ssh only from the local network
```
> ufw allow from 192.168.1.0/24 to any port 22
```

or even more psychotic:
```
> ufw allow in on eth0 from 192.168.1.0/24 to any port 22
```

Eventually I accept one IP from a friend admin from outside (my home with fixed IP adress)
```
> ufw allow from 123.123.123.123 to any port 22  
```

I also ass the `limit` feature with is supposed to avoid brute force attacks:
```
> ufw limit from 192.168.1.0/24 to any port 22
```

I recommend you to test the ssh rules before anything else if you are working exclusively with ssh
as a mistake may forbid you to connect to your own server.


## Samba firewall setup

In the `/etc/samba/smb.conf` Samba was setup to respond only to the local interface:
```conf
hosts allow = 127.0.0.1 192.168.1.
hosts deny = ALL
```

So, even if the server have another public IP address it will respond only to the local one, even without a firewall setup. But, as we have closed everything by default with the firewall, we need to open the samba ports for the local network:
```
> ufw allow from 192.168.1.0/24 to any port 139,445 proto tcp  
> ufw allow from 192.168.1.0/24 to any port 137,138 proto udp
```

Like this we have two barriers:
 * the firewall one who open the ports only from the local network
 * the samba one, who refuse connections except from localhost and local network

Without the firewall, samba would refuse access, but the ports are still 'open' to the outside world, and can be seen by port scanning tools.


## Last steep
```
> ufw enable
```

We can list the active rules with:
```
> ufw status verbose
```
```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing)
New profiles: skip

To                         Action      From
--                         ------      ----
80                         ALLOW IN    Anywhere
443                        ALLOW IN    Anywhere
25/tcp                     ALLOW IN    Anywhere
993                        ALLOW IN    Anywhere
995                        ALLOW IN    Anywhere
22                         LIMIT IN    192.168.1.0/24
137,138/udp (Samba)        ALLOW IN    Anywhere
139,445/tcp (Samba)        ALLOW IN    Anywhere
80                         ALLOW IN    Anywhere (v6)
443                        ALLOW IN    Anywhere (v6)
25/tcp                     ALLOW IN    Anywhere (v6)
993                        ALLOW IN    Anywhere (v6)
995                        ALLOW IN    Anywhere (v6)
137,138/udp (Samba (v6))   ALLOW IN    Anywhere (v6)
139,445/tcp (Samba (v6))   ALLOW IN    Anywhere (v6)
```

I recommend you to check after the ports who are still with `nmap` from inside and outside, to see if you need to add more rules.


## If something go wrong
Disable ufw:
```
> ufw disable
```

And eventually completely reset the rules:
```
> ufw reset
```

## Tips

### to remove a service
```
> ufw delete allow service 
```
### testing a rule
```
> ufw --dry-run allow rule
```

# References
https://help.ubuntu.com/community/UFW
