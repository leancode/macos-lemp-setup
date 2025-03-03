## Install local LEMP for macOS

For *Front End development*, a full Vagrant box is not always needed. If you have a new Macbook Pro, you can install local LEMP (Linux, nginx, MariaDB and PHP) with this single liner below. 

Don't just run the oneliner in blind, please see [installation steps](#installation) first.

```` bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/digitoimistodude/macos-lemp-setup/master/install.sh)"
````

Oneliner may not go through in macOS Big Sur and macOS Monterey, in that caes you need to copy and paste commands manually from [install.sh](https://raw.githubusercontent.com/digitoimistodude/macos-lemp-setup/master/install.sh).

**Please note:** Don't trust blindly to the script, use only if you know what you are doing. You can view the file [here](https://github.com/digitoimistodude/osx-lemp-setup/blob/master/install.sh) if having doubts what commands are being run. However, script is tested working many times and should be safe to run even if you have some or all of the components already installed.

## Table of contents

1. [Background](#background)
2. [Features](#features)
3. [Requirements](#requirements)
4. [Installation](#installation)
5. [Post installations](#post-installations)
   1. [Mailhog](#MailHog)
6. [Use Linux-style aliases](#use-linux-style-aliases)
7. [File sizes](#file-sizes)
8. [XDebug](#xdebug)
9. [Redis](#redis)
10. [Troubleshooting](#troubleshooting)

### Background

Read the full story by [@ronilaukkarinen](https://github.com/ronilaukkarinen): **[Moving from Vagrant to a LEMP stack directly on a Macbook Pro (for WordPress development)](https://medium.com/@rolle/moving-from-vagrant-to-a-lemp-stack-directly-on-a-macbook-pro-e935b1bc5a38)**

### Features

- PHP 7.4
- nginx 1.19.2
- Super lightweight
- Native packages
- Always on system service
- HTTPS support
- Consistent with production setup
- Works even [on Windows](https://github.com/digitoimistodude/windows-lemp-setup)

### Requirements

- [Homebrew](https://brew.sh/)
- macOS, preferably 10.14.2 (Mojave)
- wget
- [mkcert](https://github.com/FiloSottile/mkcert)

### Installation

1. Run oneliner installation script `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/digitoimistodude/macos-lemp-setup/master/install.sh)"`
2. Link PHP executable like this: **Run:** `sudo find / -name 'php'`. When you spot link that looks like this (yours might be different version) */usr/local/Cellar/php@7.4/7.4.23/bin/php*, symlink it to correct location to override MacOS's own file: `sudo ln -s /usr/local/Cellar/php@7.4/7.4.23/bin/php /usr/local/bin/php`
3. Use PHP path from correct location by adding to your ~/.bash_profile file, `sudo nano ~/.bash_profile` (change your PHP version accordingly)
   ``` shell
   export PATH="$(brew --prefix php@7.2)/bin:$PATH"
   export PATH="$(brew --prefix php@7.3)/bin:$PATH"
   export PATH="$(brew --prefix php@7.4)/bin:$PATH"
   ```
4. Check the version with `php --version`, it should match the linked file.
5. Brew should have already handled other links, you can test the correct versions with `sudo mysql --version` (if it's something like _mysql  Ver 15.1 Distrib 10.5.5-MariaDB, for osx10.15 (x86_64) using readline 5.1_ it's the correct one) and `sudo nginx -v` (if it's something like nginx version: nginx/1.19.3 it's the correct one)
6. Add `export PATH="$(brew --prefix php@7.4)/bin:$PATH"` to .bash_profile (or to your zsh profile or to whatever term profile you are currently using)
7. Go through [post installations](#post-installations)
8. Enjoy! If you use [dudestack](https://github.com/digitoimistodude/dudestack), please check instructions from [its own repo](https://github.com/digitoimistodude/dudestack).

### Post installations

You may want to add your user and group correctly to `/usr/local/etc/php/7.4/php-fpm.d/www.conf` and set these to the bottom:

````
catch_workers_output = yes
php_flag[display_errors] = On
php_admin_value[error_log] = /var/log/fpm7.4-php.www.log 
slowlog = /var/log/fpm7.4-php.slow.log 
php_admin_flag[log_errors] = On
php_admin_value[memory_limit] = 1024M
request_slowlog_timeout = 10
php_admin_value[upload_max_filesize] = 100M
php_admin_value[post_max_size] = 100M
````

Default vhost for your site (/etc/nginx/sites-enabled/sitename.test) could be something like:

```` nginx
server {
    listen 80;
    root /var/www/example;
    index index.html index.htm index.php;
    server_name example.test www.example.test;
    include php7.conf;
    include global/wordpress.conf;
}
````

Default my.cnf would be something like this (already added by install.sh in `/usr/local/etc/my.cnf`:

````
#
# This group is read both both by the client and the server
# use it for options that affect everything
#
[client-server]

#
# include all files from the config directory
#
!includedir /usr/local/etc/my.cnf.d

[mysqld]
innodb_log_file_size = 32M
innodb_buffer_pool_size = 1024M
innodb_log_buffer_size = 4M
slow_query_log = 1
query_cache_limit = 512K
query_cache_size = 128M
skip-name-resolve
````

For mysql, <b>remember to run `sudo mysql_secure_installation`</b>, answer as suggested, add/change root password, remove test users etc. <b>Only exception!</b> Answer with <kbd>n</kbd> to the question <code>Disallow root login remotely? [Y/n]</code>. Your logs can be found at `/usr/local/var/mysql/yourcomputername.err` (where yourcomputername is obviously your hostname).

After that, get to know [dudestack](https://github.com/digitoimistodude/dudestack) to get everything up and running smoothly. Current version of dudestack supports macOS LEMP stack.

You should remember to add vhosts to your /etc/hosts file, for example: `127.0.0.1 site.test`.

### Use Linux-style aliases

Add this to */usr/local/bin/service* and chmod it +x:

```` bash
#!/bin/bash
# Alias for unix type of commands
brew services "$2" "$1";
````

Now you are able to restart nginx and mysql unix style like this:

```` bash
sudo service nginx restart
sudo service mariadb restart
````

#### MailHog

E-mails won't be sent on local environment because there is no email server configured. This is where [MailHog](https://github.com/mailhog/MailHog) comes in.

MailHog should be pre-installed but if not, run following:

``` bash
brew update && brew install mailhog
```

Ensure you have the latest [air-helper](https://github.com/digitoimistodude/air-helper) or [MailHog for WordPress](https://wordpress.org/plugins/wp-mailhog-smtp/) activated to enable MailHog routing for local environment.

Then just run:

``` bash
mailhog
```

You should now get a log in command line and web interface is available in http://0.0.0.0:8025/.

### File sizes

You might want to increase file sizes for development environment in case you need to test compression plugins and other stuff in WordPress. To do so, edit `/usr/local/etc/php/7.4/php-fpm.d/www.conf` and `/usr/local/etc/php/7.4/php.ini` and change all **memory_limit**, **post_max_size** and **upload_max_filesize** to something that is not so limited, for example **500M**.

Please note, you also need to change **client_max_body_size** to the same amount in `/etc/nginx/nginx.conf`. After this, restart php-fpm with `sudo brew services restart php@7.4` and nginx with `sudo brew services restart nginx`.

### Certificates for localhost

First things first, if you haven't done it yet, generate general dhparam:

```` bash
sudo su -
cd /etc/ssl/certs
sudo openssl dhparam -dsaparam -out dhparam.pem 4096
````

Generating certificates for dev environment is easiest with [mkcert](https://github.com/FiloSottile/mkcert). After installing mkcert, just run:

```` bash
mkdir -p /var/www/certs && cd /var/www/certs && mkcert "project.test"
````

Then edit your vhost as following (change all from *project* to your project name):

```` nginx
server {
    listen 443 ssl http2;
    root /var/www/project;
    index index.php;    
    server_name project.test;

    include php7.conf;
    include global/wordpress.conf;

    ssl_certificate /var/www/certs/project.test.pem;
    ssl_certificate_key /var/www/certs/project.test-key.pem;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;
    ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_stapling_verify on;
    add_header Strict-Transport-Security max-age=15768000;
}

server {
    listen 80;
    server_name project.test;
    return 301 https://$host$request_uri;
}
````

Test with `sudo nginx -t` and if everything is OK, restart nginx.

### XDebug

1. Check your PHP version with `php --version` and location with `which php`. If the location points to `/usr/bin/php`, you are mistakenly using macOS built-in PHP. Change PHP path to correct location by adding to your ~/.bash_profile file, `sudo nano ~/.bash_profile` (change your PHP version accordingly):
   ``` shell
   export PATH="$(brew --prefix php@7.2)/bin:$PATH"
   export PATH="$(brew --prefix php@7.3)/bin:$PATH"
   export PATH="$(brew --prefix php@7.4)/bin:$PATH"
   ```
2. Search pecl `find -L "$(brew --prefix php@7.4)" -name pecl -o -name pear`
3. Symlink pecl based on result, for example `sudo ln -s /usr/local/opt/php@7.4/bin/pecl /usr/local/bin/pecl`
4. Add executable permissions `sudo chmod +x /usr/local/bin/pecl`
5. Install xdebug `pecl install xdebug`
6. Check `php --version`, it should display something like this:

``` shell
$ php --version
PHP 7.4.23 (cli) (built: Aug 27 2021 09:20:14) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Xdebug v3.0.3, Copyright (c) 2002-2021, by Derick Rethans
    with Zend OPcache v7.4.23, Copyright (c), by Zend Technologies
```

7. Check where your php.ini file is with `php --ini`
8. Edit php.ini, for example `sudo nano `
9. Make sure these are on the first lines:

```
zend_extension="xdebug.so"
xdebug.mode=debug
xdebug.client_port=9003
xdebug.client_host=127.0.0.1
xdebug.remote_handler=dbgp
xdebug.start_with_request=yes
xdebug.discover_client_host=0
xdebug.show_error_trace = 1
xdebug.max_nesting_level=250
xdebug.var_display_max_depth=10
xdebug.log=/var/log/xdebug.log
```

10. Save and close with <kbd>ctrl</kbd> + <kbd>O</kbd> and <kbd>ctrl</kbd> + <kbd>X</kbd>
11. Make sure the log exists `sudo touch /var/log/xdebug.log && sudo chmod 777 /var/log/xdebug.log`
12. Restart services (requires [Linux-style aliases](#use-linux-style-aliases)) `sudo service php@7.4 restart && sudo service nginx restart`
13. Install [PHP Debug VSCode plugin](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug)
14. Add following to launch.json (<kbd>cmd</kbd> + + <kbd>shift</kbd> + <kbd>P</kbd>, "Open launch.json"):

``` json
{
  "version": "0.2.0",
  "configurations": [
    {
      //"debugServer": 4711, // Uncomment for debugging the adapter
      "name": "Listen for Xdebug",
      "type": "php",
      "request": "launch",
      "port": 9003,
      "log": true
    },
    {
      //"debugServer": 4711, // Uncomment for debugging the adapter
      "name": "Launch",
      "request": "launch",
      "type": "php",
      "program": "${file}",
      "cwd": "${workspaceRoot}",
      "externalConsole": false
    }
  ]
}
```
15. Xdebug should now work on your editor
16. PHPCS doesn't need xdebug but will warn about it not working... this causes error in [phpcs-vscode](https://marketplace.visualstudio.com/items?itemName=ikappas.phpcs) because it depends on outputted phpcs json that is not valid with the warning _"Xdebug: [Step Debug] Could not connect to debugging client. Tried: 127.0.0.1:9003 (through xdebug.client_host/xdebug.client_port) :-(_". This can be easily fixed by installing a bash "wrapper":
17. Rename current phpcs with `sudo mv /usr/local/bin/phpcs /usr/local/bin/phpcs.bak`
18. Install new with `sudo nano /usr/local/bin/phpcs`:

``` bash
#!/bin/bash
XDEBUG_MODE=off /Users/rolle/Projects/phpcs/bin/phpcs "$@"
```

19. Add permissions `sudo chmod +x /usr/local/bin/phpcs`
20. Make sure VSCode settings.json has this setting:

``` json
"phpcs.executablePath": "/usr/local/bin/phpcs",
```

### Redis

Redis is an open source, in-memory data structure store, used as a database, cache. We are going to install Redis and php-redis.

Before installation, make sure you do not use PHP provided by macOS. You should be using PHP installed by homebrew. If you are having problems with testing php-redis after installation, it is most probably caused bacuse of using wrong PHP. See (Troubleshooting: Testing which version of PHP you run)(#testing-which-version-of-php-you-run) for more information.

1. Check that `pecl` command works
2. Run `brew update` first
3. Install Redis, `brew install redis`
4. Start Redis `brew services start redis`, this will also make sure that Redis is always started on reboot
5. Test if Redis server is running `redis-cli ping`, expected response is `PONG`
6. Install PHP igbinary extension `pecl install igbinary-3.2.1`
6. Install PHP Redis extention `pecl install redis-5.3.4`. When asked about enabling some supports, answer `no`.
7. Restart nginx and php-redis should be available, you can test it with `php -r "if (new Redis() == true){ echo \"\r\n OK \r\n\"; }"` command, expected response is `OK`

### Troubleshooting

**Testing which version of PHP you run**

Test with `php --version` what version of PHP you are using, if the command returns something like `PHP is included in macOS for compatibility with legacy software` and especially when `which php` is showing /usr/bin/php then you are using macOS built-in version (which will be removed in the future anyway) and things most probably won't work as expected.

To fix this, run command `sudo ln -s /usr/local/Cellar/php@7.4/7.4.23/bin/php /usr/local/bin/php` which symlinks the homebrew version to be used instead of macOS version OR use bashrc export as defined [here in step 4](https://github.com/digitoimistodude/macos-lemp-setup#installation).

#### PHP or mysql not working at all

If you have something like this in your /var/log/nginx/error.log:

```
2019/08/12 14:09:04 [crit] 639#0: *129 open() "/usr/local/var/run/nginx/client_body_temp/0000000005" failed (13: Permission denied), client: 127.0.0.1, server: project.test, request: "POST /wp/wp-admin/async-upload.php HTTP/1.1", host: "project.test", referrer: "http://project.test/wp/wp-admin/upload.php"
```

If you cannot login to mysql from other than localhost, please answer with <kbd>n</kbd> to the question <code>Disallow root login remotely? [Y/n]</code> when running <code>sudo mysql_secure_installation</code>.

**Make sure you run nginx and php-fpm on your root user and mariadb on your regular user**. This is important. Stop nginx from running on your default user by `brew services stop nginx` and run it with sudo `sudo brew services start nginx`.

<code>sudo brew services list</code> should look like this:

``` shell
~ sudo brew services list
Name       Status  User  Plist
dnsmasq    started root  /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist
mariadb    started rolle /Users/rolle/Library/LaunchAgents/homebrew.mxcl.mariadb.plist
nginx      started root  /Library/LaunchDaemons/homebrew.mxcl.nginx.plist
php@7.4    started root  /Library/LaunchDaemons/homebrew.mxcl.php@7.4.plist
```

You may have "unknown" or "error" as status or different PHP version, that is not a problem if ther server runs. **User** should be like in the list above. Then everything should work.

#### MySQL/MariaDb issues

If you get problems like:

```
ERROR 2002 (HY000): Can't connect to MySQL server on '127.0.0.1' (36)
```

It seems you have messed up with your root password. Try resetting root password with by adding this to your home directory (for example /Users/rolle/mysql-init):

Try resetting root password with (add new password in place of _YOUR_NEW_PASSWORD_):

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'YOUR_NEW_PASSWORD';
```

Then kill all mysql processes:


```
sudo ps xa |grep mysql
kill -9 <pid>
```


Then run:

```
mysqld --init-file=/Users/rolle/mysql-init &
```

After this:

```
sudo mysql_secure_installation
```

Answer:

```
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] n
 ... skipping.

You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] n
 ... skipping.

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] n
 ... skipping.

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

If you are still having problems connecting with WordPress and prompting `Access denied for user 'root'@'127.0.0.1'`, try this in `mysql -u root -p`:

``` sql
GRANT ALL PRIVILEGES ON *.* TO root@localhost IDENTIFIED BY 'YOUR_MYSQL_ROOT_PASSWORD' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO root@127.0.0.1 IDENTIFIED BY 'YOUR_MYSQL_ROOT_PASSWORD' WITH GRANT OPTION;
```
