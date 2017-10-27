# Setup bayessian filtering with Spammassin and Roundcube webmail

### Warning
> **This How-to is outdated, it may be useful... or not...**
>
> It is also at the Draft level forever...

## setup Spammassasin

By default spamassassin have user based rules and bayesian databases. We change this to a site wide setup in `/etc/spamassassin/local.cf`:
```
# Site wide bayesian database
bayes_path /var/lib/amavis/.spamassassin/bayes
```

We need then to restart spamassassin and amavis:
```
> service spamassassin restart  
> service amavis restart  
```

Note: It's useful for me as I have a very little user base with similar usage profiles. It's probably better to keep individual spamassassin rules if you have a medium to big user base.

Create two `spam` and `ham` learning mailboxes:
```
> mkdir /var/mail/sa-learn  
> mkdir /var/mail/sa-learn/spam  
> mkdir /var/mail/sa-learn/ham  
```

This directories will be used by the `markasjunk2` plugin withc is executed by the web server. So we need to give writing right to the `www-data` user:
```
> chown -R www-data:www-data /var/mail/sa-learn
```

Install roundcube webmail and the markasjunk2 plugin.

Edit the plugin configuration `roundcube/plugins/markasjunk/config.inc.php`
```php
// Learning driver
$rcmail_config['markasjunk2_learning_driver'] = 'dir_learn';

// The full path of the directory used to store spam (must be writable by webserver)
$rcmail_config['markasjunk2_spam_dir'] = '/var/mail/sa-learn/spam/';

// The full path of the directory used to store ham (must be writable by webserver)
$rcmail_config['markasjunk2_ham_dir'] = '/var/mail/sa-learn/ham/';
```

At this point the `spam` and `ham` folder will be filled with copy of the mail little by little.


### Start a fresh database

Save the actual database in case of:
```
> sa-learn --backup > /root/bayes-backup
```

In case of a unclean database you can start with a fresh empty one:
```
> sa-learn --clear
> service spamassassin restart
```

You can check the spam database statistics with:
```
> sa-learn --dump magic | grep am
```

Witch will give you something like:
```
0.000          0          **0**          0  non-token data: **nspam**
0.000          0          **1**          0  non-token data: **nham**
```

The third column of the first line is the number of spam mail records. The second line contain the number of ham records (non-spam mails).

The filter will only be effective when both of them have at last 200 records.

Take the time to sort spam/ham in one of your mailbox with Roundcube. Public mail addresses available on internet are a good starting point (things like contact@mydomain.com).

When it's done you will probably have a bunch of files in `/var/mail/sa-learn/spam` so we can begin the learning:
```
> sa-learn --spam /var/mail/sa-learn/spam  
> Learned tokens from 100 message(s) (100 message(s) examined)
```

You can check again the learning process:
```
> sa-learn --dump magic | grep am
```
```
0.000          0        100          0  non-token data: nspam  
0.000          0          1          0  non-token data: nham
```
Continue like this until you have a last 200 spams.

To be effective spamassassin also need to learn ham. The 'Mark as non spam' button in Roundcube will add a few little by little in `/var/mail/sa-learn/spam` but it may be long before you get enough of ham.

In this case open your regular inbox with Roundcube and check than there is no remaining spams. It's important to catch them all before as they me be learned as 'non-spam'. When your done the make spamassassin learn ham from your mailbox:
```
> sa-learn --ham /home/myseltf/.Maildir/cur 
```

You may do the same with other mailboxes (if you have the right to do so!).


# Add cron scripts for regular learning

After a wile, you can ask to cron to do the learning automaticaly for you.
In /etc/cron.weekly (or daily) create a `learn-spam` script:
```sh
#!/bin/sh
# learn spam and ham
sa-learn --spam /var/mail/sa-learn/spam > /dev/null
sa-learn --ham /var/mail/sa-learn/ham > /dev/null
```

```
> chown +x learn-script
```

Note: as an alternative to Spamassassin look for th `dovecote-spam` module.


