# Setup Mandriva Directory Server on Ubuntu 12.04

### Warning
> **This How-to is outdated, it may be useful... or not...**
>
> By the way the project is not maintenaid anymore...
> 
> but is available on [Github](https://github.com/mandriva-management-console)


## Objective

Install Mandriva Directory Server with: OpenLDAP, Postfix, Dovecot, Samba, Amavis, ClamAV, Spamassassin  .
We do not install: Bind (DNS), Squid-Squidgard (Proxy), ICS (DHCP), OpenXchange, Print server.


## Install prerequistes

### LDAP
```
> apt-get install slapd ldap-utils libltdl7 libodbc1 libperl5.14 libslp1
```

### NSS-LDAP
```
> apt-get install  libnss-ldap auth-client-config ldap-auth-client ldap-auth-config libpam-ldap
```

### Mail
```
> apt-get install postfix posfix-ldap dovecot-core dovecot-imapd
```

### Samba
```
> apt-get install samba-common samba-common-bin acl libtalloc2 libtdb1 libwbclient0
```

### Python prerequisites for Mandriva management console
```
> apt-get install python-ldap python-support python-twisted-web python-configobj python-pylibacl python-smbpasswd
```

### MMC console prerequisites (mmc-web-base)
```
> apt-get install apache2 libapache2-mod-php5 php5-gd php5-xmlrpc fontconfig-config libfontconfig1 libgd2-xpm libt1-5 libxpm4 ttf-dejavu-core wwwconfig-common
```

## Add the MMC repository
Edit the `/etc/apt/sources.list` file to add
```
deb http://mds.mandriva.org/pub/mds/debian squeeze main
```

## Install MMC components
```
> apt-get install mmc-agent python-mmc-base python-mmc-core python-mmc-mail mmc-web-base mmc-web-mail
```
(in our case we do not need DNS, proxy, squid, dhcp, openXchange and print servers)


## Activate the MMC web frontend
```
> ln -s /etc/mmc/apache/mmc.conf /etc/apache2/conf.d/mmc.conf 
> /etc/init.d/apache2 restart
```

Eventually edit `/etc/mmc/apache/mmc.conf` to restrict allowed connections.  
For example to allow only localhost and localnet:
```apacheconf
<Directory /usr/share/mmc>
    AllowOverride None
    Order deny,allow     
    deny from all
    allow from 127.0.0.1
    allow from 192.168.1.0/24                      
    php_flag short_open_tag on
    php_flag magic_quotes_gpc on
</Directory>
```

## Activate xml-rpc for the agent
xml-rpc is used by the mmc-agent to communicate with the web frontend.
Edit the `/etc/default/mmc-agent` file and set :
```ini
ENABLE=yes
```

## Configure LDAP
```
> dkpg-reconfigure slapd
```
| Option                   | Value        |
|:-------------------------|:-------------|
| Omit configuration       | No           |
| Domain name              | mydomain.com |
| Organisation             | mydomain     |
| Database module          | HDB          |
| Delete database on purge | No           |
| Save previous database   | Yes          |
| LDAPv2 protocol          | No           |

You can check if it's OK with
```
> slapcat
```

Copy the needed mmc schemas:
```
> mmc-add-schema /usr/share/doc/python-mmc-base/contrib/ldap/mmc.schema /etc/ldap/schema/  
> mmc-add-schema /usr/share/doc/python-mmc-base/contrib/ldap/mail.schema /etc/ldap/schema/
```

## Configure NNS-LDAP
```
> dkpg-reconfigure ldap-auth-config
```
| Option                                     | Value                       |
|:-------------------------------------------|:----------------------------|
| debconf manage LDAP configuration          | Yes                         |
| LDAP server Uniform Resource Identifier    | ldapi:///127.0.0.1/         |
| Distinguished name of the search base      | dc=mydomain,dc=com          |
| LDAP Version                               | 3                           |
| local root Database admin                  | Yes                         |
| LDAP database require login                | No                          |
| LDAP account for root                      | cn=admin,dc=mydomain,dc=com |
| Local crypt to use when changing passwords | md5                         |

Edit `/etc/nsswitch.conf` :
```yaml
passwd:     files ldap
shadow:     files ldap
group:      files ldap
```

## Generate the base64 password
This is not secure, it's just a bit of obfuscation
```
> python -c 'print("secret").encode("base64")'  
```
Witch will give you:
```
c2VjcmV0
```

## Configure the base plugin
The agent need to know the LDAP credentials.  
Edit `/etc/mmc/plugins/base.ini`
```ini
baseDN = dc=mydomain, dc=com  
password = {base64}c2VjcmV0
```

## Create the user backup directory
```
> mkdir /home/archives
```

## Test the install

Test the agent:  
```
> /etc/init.d/mmc-agent stop
> /etc/init.d/mmc-agent start
```

Test the web frontend:
http://myhostname.mydomain.com/mmc  
with "root" for user and the admin LDAP password 

### In case of errors
* check the logs in `/var/log/mmc/mmc-agent.log`


## Configure Postfix

### Add the LDAP mail aliases to postfix
Edit `/etc/postfix/main.cf`
```ini
alias_maps = hash:/etc/aliases, ldap:/etc/postfix/ldap-aliases.cf
alias_database = hash:/etc/aliases
```

Create a `/etc/postfix/ldap-aliases.cf` file
```ini
search_base = ou=People,dc=lazaret,dc=unice,dc=fr
query_filter = (&(objectClass=mailAccount)(mailalias=%u)(mailenable=OK))
version = 3
```

You can test the postfix-ldap configuration with :
```
> postmap -v -q test@mydomain.com ldap:/etc/postfix/ldap-aliases.cf
```
where test@mydomain.com is a virtual mail created with MMC.

Finally you can check your postfix configuration with
```
> postfix check
```

Check also `/var/log/mail.err` and `/var/log/mail.log` for errors.

### Spamassassin and ClamAV support

This steep is optional, and will not interact with MMC itself. It's just recommended to:
* Add spam filtering
* Add virus filtering
to coming-in and coming-out emails.

The spam and mail filtering will be delegated by postfix to amavis. And amavis will use spamassassin and clamav.

There is a very nice how-to for this steep at [Ubuntu wiki](https://help.ubuntu.com/community/PostfixAmavisNew). Just go to follow it and come back here.

When your done you can tests antivirus and spam filtering:  
* Send to yourself a message with one of the EICAR test file from [here](http://www.eicar.org/85-0-Download.html)
* send you a message with the spam test line in the body of the mail:
```
XJS*C4JDBQADN1.NSBN3*2IDNEN*GTUBE-STANDARD-ANTI-UBE-TEST-EMAIL*C.34X
```
Then check the mail logs `/var/log/mail.log` to verify than both message have been deleted witch is the default Ubuntu behaviour.

### Keep the spam !

By default, the spam is deleted on Ubuntu as configured in `/etc/amavis/conf.d/21-ubuntu_defaults`. I prefer to keep the spam in the Junk folder of the mailbox in case of false positives. We then edit the `/ect/amavis/conf.d/50-user` file witch will override the default one:
```perl
use strict;

# Place your configuration directives here.  They will override those in
# earlier files.
# See /usr/share/doc/amavisd-new/ for documentation and examples of
# the directives you can use in this file

# Spam is delivered
$final_spam_destiny       = D_PASS;                     

# No need for subject tag, header are enough 
$sa_spam_subject_tag = '';

# we want to alert postmaster about viruses
$virus_admin = "postmaster\@$mydomain";

#------------ Do not modify anything below this line -------------
1;  # ensure a defined return
```

As you can see I have also added a postmaster warning about viruses and removed the ***SPAM*** flag in the mail subject as, I sort them based on header flagd added by spamassasin.

You can read `/etc/amavis/conf.d/21-ubuntu_defaults` to see other default filtering like black and white lists. If you need to add things, it's better to add them only in `/ect/amavis/conf.d/50-user` as other files may me changed during Ubuntu upgrades.

**Warning:** In any case do no bounce virus and/or try to send a virus alert to the sender as they are most probably forged.

Normally at this point:
* virus are quarantined, and postmaster receive a notification about them
* Spam have flags in them but are relayed to the user

For the Spam, we will use Dovecot Pigeonhole Sieve interpreter to sort them in the Junk folder (see above).


## Configure Dovecot

Dovecot is used for
* mail delivery with dovecot-lta
* Imap server
* Sieve filtering

### Mail delivery and mailbox configuration

For the mail delivery edit postfix `/etc/postfix/main.cf` and replace procmail delivery by dovecot-lda:
```ini
mailbox_command = /usr/lib/dovecot/dovecot-lda -f "$SENDER" -a "$RECIPIENT"
```
We now setup the mailbox as the Maildir format by editing `/etc/dovecot/conf.d/10-mail.conf`:
```ini
mail_location = maildir:~/.Maildir
```

Note than I have decided to hide the maildir directory, so the users will not see it if they connect to their home folder with Samba. This will avoid than they delete the folder by mistake.

Finally we tell to dovecot to create the mailbox if he did not exist yet by editing `/etc/dovecot/conf.d/15-lda.conf`:
```ini
lda_mailbox_autocreate = yes
lda_mailbox_autosubscribe = yes
```

### MMC mail plugin configuration
Edit `/etc/mmc/plugins/mail.ini` accordingly to the dovecot mail delivery configuration
```ini
[userdefault]
# For Postfix delivery
#mailbox = %homeDirectory%/Maildir/
# For Dovecot delivery
mailbox = maildir:%homeDirectory%/.Maildir/
```

### Protocols configuration

We will only use `imap` and `imaps` support, so we first make sure to remove `dovecot-pop3d`:
```
> apt-get remove dovecot-pop3d
```

For more security, in case of a later unwanted install of dovecot-pop3d, we edit `/etc/dovecot/dovecot.conf`
```ini
# Enable installed protocols
#!include_try /usr/share/dovecot/protocols.d/*.protocol
protocols = imap
```

Note: Ubuntu 12.04 use dovecot2, witch have depreciated the `imaps` protocol in this line,
so we do not need to add it here. `imap` in this line now activate imap+imaps.

Now we want `imap` available only on localhost and `imaps` for the rest of the world.  
For this we edit `/etc/dovecot/conf.d/10-master.conf to tell to dovecot to only allow
`imap` on localhost. Like this a webmail setup on localhost can talk directly to dovecot
without encryption will users are forced to use `imaps`.
```
service imap-login {
  inet_listener imap {
    address = localhost
    #port = 143
  }
...
```

If you prefer of for more security you can also add firewall rules:
```
> ufw allow imaps  
> ufw deny imap  
> ufw deny pop3  
> ufw deny pop3s
```

### Sieve filtering

Sieve is a language used to filter mail. User can write their own script as some mail clients (e.g. roudcube webmail with one of the sieve plugins). We will add the feature and create a default filtering rule to sort spams in the Junk folder of the users.

First install packages pour Dovecot Pigeonhole sieve interpreter:
```
> apt-get install dovecot-sieve dovecot-managesieved
```

If you have edited the protocol line `/etc/dovecot/dovecot.conf` the we need to add the managesieve server plugin:
```ini
protocols = imap sieve
```

If not, then Ubuntu normally have already activated it.

We also nee to tell to dovecot-lda to filter mail with sieve in `/etc/dovecot/conf.d/15-lda.conf`:
```
protocol lda {
  # Space separated list of plugins to load (default is global mail_plugins).
  mail_plugins = $mail_plugins sieve
}
```

Note than the first plugin in `/etc/dovecot/dovecot.conf` is for the managesieve server and the secondary is to add sieve language capabity to the dovecot delivery agent.

As for the the Maildir folder i want to hide the sieve folder to the users. Edit `/etc/dovecot/conf.d/90-sieve.conf`
```ini
sieve_dir = ~/.sieve
```

Like this the basic users will not delete the folder by mistake. I give the job to create/edit sieve script to mail reader plugins.

In the same file we will edit the `sieve_before` option to add a default Spam filtering feature (see above).
This is the place where you may add other global sieve scripts.
```ini
sieve_before = /etc/dovecot/sieve_before
```

Create the directory and add inside the spam sieve filters as `01-spam.sieve`
```
require "fileinto";
# Move spam into Junk folder
if header :contains "X-Spam-Flag" ["YES"] {
  fileinto "Junk";
  stop;
}
```

This one is very basic and use the `X-Spam-Flag` header added by spamassassin. You may want to add more fine grained filters based on `X-Spam-Score` or `X-Spam-Level`. For example to delete any spam above a certain level.

Finally compile the script:
```
> sievec /etc/dovecot/sieve_before/01-spam.sieve
```
(this will avoid lda permission errors in your logs)


# Configure samba
```
> apt-get install samba samba-common acl smbldap-tools
```

Edit `/etc/samba/smb.conf' according to your needs. 
You can use the example provided by MMC for [here](https://raw.github.com/mandriva-management-console/mmc/master/mds/agent/contrib/samba/smb.conf)

Here is an example:  
The important part are LDAP settings, the `workgroup` and the `add machine` script. For the rest take the time to check the SAMBA manual.
```ini
[global]
    workgroup = MYDOMAIN
    netbios name = MYSERVER
    server string = My Samba Server
    # log options
    syslog = 0
    log file = /var/log/samba/log.%m
    max log size = 1000
    # domain setup
    domain master = Yes
    preferred master = Yes
    domain logons = Yes
    wins support = Yes
    time server = Yes
    os level = 65
    # ldap and password options
    ldap admin dn = cn=admin,dc=mydomain,dc=com
    ldap suffix = dc=mydomain,dc=com
    ldap ssl = no
    ldap group suffix = ou=Group
    ldap user suffix = ou=People
    ldap machine suffix = ou=Computers
    ldap passwd sync = yes
    passdb backend = ldapsam:ldap://127.0.0.1/
    idmap config * : backend = tdb
    admin users = administrator
    map acl inherit = Yes
    # samba scripts
    add machine script = /usr/lib/mmc/add_machine_script '%u'
    add share command = /usr/lib/mmc/add_change_share_script
    delete share command = /usr/lib/mmc/delete_share_script 
    addprinter command = /usr/lib/mmc/add_printer_script
    interdependent command = /usr/lib/mmc/delete_printer_script
    # respond only for the local ip address (192.168.1.12 here)
    interfaces = 192.168.1.12
    bind interfaces only = yes
    # forbid and hide common files
    veto files = /.*/
    hide files = /IPC$/Thumbs.db/
    # user logon options
    logon script = logon.bat
    logon path = 
    logon home =   
[netlogon]
    comment = Network Logon Service
    path = /etc/samba/scripts
    guest ok = Yes
    browseable = No
[home]
    path = %H
    read only = No
    create mask = 0755
[share]
    path = /srv/samba/share
    read only = No
    create mask = 0755
[archives]
    path = /home/archives
    valid users = administrator
    browseable = No 
```
(paths for your shares must be created)


## stop samba
```
> service smbd stop  
> service nmbd stop
```

Tell the LDAP admin password to samba
```
> smbpasswd -w secret  
```
The result will be something like:
```
Setting stored password for "cn=admin,dc=mydomain,dc=com" in secrets.tdb
```

Add the Samba SID in the ldap database
```
> net getlocalsid MYDOMAIN  
```
The result will be something like:
```
SID for domain MYDOMAIN is: S-1-5-12-123456789-123456789-1234567890
```

If your are migrating use instead:
```
> net setlocalsid S-1-5-21-9876543210-9876543210-987654321
```

This will normaly set you machine and domain SID to the previous one  
To check:
```
> net getdomainsid
```

To check the SID in the LDAP database
```
> slapcat | grep sambaDomainName 
```
The result will be something like:
```yaml
dn: sambaDomainName=MYDOMAIN,dc=mydomain,dc=com  
sambaDomainName: MYDOMAIN
```

Edit the smbldap-tool config files  
Edit `/etc/smbldap-tools/smbldap_bind.conf`
```ini
slaveDN="cn=admin,dc=mydomain,dc=comm"
slavePw="secret"
masterDN="cn=admin,dc=mydomain,dc=com"
masterPw="secret"
```

Edit `/etc/smbldap-tools/smbldap.conf`
```ini
SID="S-1-5-12-123456789-123456789-1234567890"
sambaDomain="MYDOMAIN"
ldapTLS="0"
suffix="dc=mydomain,dc=com"
sambaUnixIdPooldn="sambaDomainName=${sambaDomain},${suffix}"

usersdn="ou=People,${suffix}"
groupsdn="ou=Group,${suffix}"

userSmbHome=""
userProfile=""
userHomeDrive=""

scope="sub"
hash_encrypt="SSHA"
crypt_salt_format="%s"
with_smbpasswd="0"
smbpasswd="/usr/bin/smbpasswd"
with_slappasswd="0"
slappasswd="/usr/sbin/slappasswd"
```

Then populate the LDAP database with `smbldap-populate`
```
> smbldap-populate -m 512 -a administrator
```

This will add samba related stuff in he LDAP database and set the `administrator` user. You may have a few perl warnings then you can forget.

Start samba:
```
> service smbd start
> service nmbd start
```

Finaly grant to the administrator user the righ to join computers to the domain :
```
> net -U administrator rpc rights grant 'MYDOMAIN\Domain Admins' SeMachineAccountPrivilege
```

In case of error have a look to the logs in `var\log\samba\`.


## Samba password policy

In my case a want a never expire policy :
```
> pdbedit -P "maximum password age" -C -1
```

You can check your actual policy with :
```
> dbedit -P "maximum password age"
```

## Install the MMC samba plugin
```
> apt-get install python-mmc-samba mmc-web-samba
```

Add the samba schema to LDAP schemas
```
> gunzip /usr/share/doc/python-mmc-base/contrib/ldap/samba.schema.gz
> mmc-add-schema /usr/share/doc/python-mmc-base/contrib/ldap/samba.schema /etc/ldap/schema/
```

Finally edit `/etc/mmc/plugins/samba.ini`
```ini
[main]
disable = 0
# Computers Locations
baseComputersDN = ou=Computers, %(baseDN)s
sambaConfFile = /etc/samba/smb.conf
sambaInitScript = /etc/init.d/smbd 
sambaAvSo = /usr/lib/samba/vfs/vscan-clamav.so
# Default SAMBA shares location
defaultSharesPath = /srv/samba/share
# You can specify authorized paths for share creation
# Default value is the defaultSharesPath value
# authorizedSharePaths = /shares, /opt, /srv
    
# Default value when adding samba attributes to an user
# DELETE means the attibute is removed from the user LDAP entry
[userdefault]
sambaPwdMustChange = DELETE
```

# ClamAV intergration

ClamAV libraries and manual scanner are already installed on Ubuntu but we need the daemon:
```
> apt-get install clamav-daemon
```

## integrate with samba

Not done.
The recommended module is to use vscan-clamav in conjunction with Samba but:
* there is no Debian/Ubuntu package actually
* the project seems more or less staled
* this is an on-demand scanner witch mean than it can have performance issue if everybody is asking for the same file

It's probably more easy to create a cron job to scan regularly the shares


# Various tricks

## get rid of the slapd `bdb_equality_candidates` warnings in the logs

If you read your logs you may have messages like :
```
Oct 15 08:48:49 myhostname slapd[1309]: <= bdb_equality_candidates: (uid) not indexed
Oct 15 08:48:49 myhostname slapd[1309]: <= bdb_equality_candidates: (memberUid) not indexed
Oct 15 08:48:49 myhostname slapd[1309]: <= bdb_equality_candidates: (uniqueMember) not indexed
```

It's mean than there are missing indexes in the LDAP database

You can check the actual index with:
```
> ldapsearch -Y EXTERNAL -H ldapi:/// -b "cn=config" | grep ^olcDbIndex
```

This will probably give you something lile:
```
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
olcDbIndex: objectClass eq
```

In my case the missing index member where : `uid`, `memberUid` and `uniqueMember`  
I have also added `cn`, `ou` and `dc`

Edit `index.ldf` file to add them:
```yaml
dn: cn=config
changetype: modify
add: olcDbIndex
olcDbIndex: uid pres,eq
-
add: olcDbIndex
olcDbIndex: memberUid pres,eq
-
add: olcDbIndex
olcDbIndex: uniqueMember pres,eq   
-
add: olcDbIndex
olcDbIndex: cn pres,eq
-
add: olcDbIndex
olcDbIndex: ou eq
-
add: olcDbIndex
olcDbIndex: dc eq
```

`eq` is used by equality queries and `pres` by presence queries.

## backup the LDAP database
```
> slapcat -v -l /home/backup/ldap.ldif
```


# References

* [Mandriva Directory Server (french)](http://wiki.mandriva.com/fr/Mandriva_Directory_Server)
* [Mandriva Management Console @ ReadTheDocs](http://mandriva-management-console.readthedocs.org)
* [Mandriva Directory Server On Debian Etch](http://www.howtoforge.com/mandriva-directory-server-on-debian-etch)
* [Mandriva Directory Server On Debian Squeeze](http://www.isalo.org/wiki.debian-fr/index.php?title=Debian_Squeeze:_LDAP_-_Mandriva_Directory_Server_%28MDS%29)
* [Mandriva Directory Server on Ubuntu 10.4 (russian)](http://gordeev.pro/2010/07/mds-na-ubuntu-10-4/)
* [MMC generic install](http://mds.mandriva.org/content/MMC/install/en/mmc-generic-installation.html)
