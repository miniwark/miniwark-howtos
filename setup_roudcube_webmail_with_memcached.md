# Setup Roundcube webmail with Memcached

### Warning
> **This How-to is outdated, it may be useful... or not...**

## Setup

Install Memcached:
```
> apt-get install memcached php5-memcache
```

Edit roundcube config file `roudcube/config/main.inc.php`
```
$rcmail_config['imap_cache'] = 'memcache';
$rcmail_config['session_storage'] = 'memcache';
$rcmail_config['memcache_hosts'] = array('localhost:11211');
```

Eventually edit the cache size in `/etc/memcached.conf`  
```
# memory
-m 255
```

If you want to check the memcached memory usage do:
```
> telnet localhost 11211  
> stats  
> quit
```

Restart `memcached` and `apache`
```
> service memcached restart  
> service apache2 restart
```

If you want to track Memcached, you can use various tools like:
* `memcstat` from libmemcached-tools package
* [memcache-top](http://code.google.com/p/memcache-top/) 
* [Cacti](http://dealnews.com/developers/cacti/memcached.html)
* [phpMemcachedAdmin](http://code.google.com/p/phpmemcacheadmin/)
