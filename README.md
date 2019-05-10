# Linux Server Configuration (final project)
This repo contains the source files used to create and configure the web app deployed at www.memorepo.com.

## Connecting to the site
```
(SSH private key submitted with this project; assuming it's saved to local directory ~/.ssh/grader)
$ ssh grader@18.212.186.88 -p 22 -i grader
```

## Software installed
```
Using "sudo apt-get install <package name>":
apache2
libapache2-mod-wsgi
postgresql
python-flask
python-jinja2
python-pip
python-psycopg2
python-sqlalchemy
python-werkzeug
readline
```

## Configurations
Update, upgrade, and remove any unnecessary packages.
```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get autoremove
```
Add new user ("grader").
```
$ sudo adduser grader
```
Add user to /etc/sudoers.d/grader
```
$ ssh -i <.pem file> ubuntu@<public IP of your instance>
```

### Generate SSH keys
Always *generate* keys locally.
Create the pair. Save it to a directory and filename (default is `~/.ssh/id_rsa`, but you can change the filename to something more specific, e.g. `udacity_linux_course`). Private and public keys will be generated.
```
$ ssh-keygen
```
Save the *public* key on your remote server.
In the home directory of remote machine, make an `.ssh` dir.
```
$ mkdir .ssh
```
Make an `authorized_keys` file and save public key here. Each line in this file can have a public key for each user that’s going to log in.
```
$ touch .ssh/authorized_keys
$ nano .ssh/authorized_keys
```
Paste in your public key. (Use: `$ pbcopy < keyname.pub`)
Set permissions on `authorized_keys` file and the `.ssh` directory.
```
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
```
You can now login like so, using the `-i` flag pointing to your private key file.
```
$ ssh <username>@127.0.0.1 -p 2222 -i ~/.ssh/<filename>
```
Force key based authentication for all users.
Edit the `/etc/ssh/sshd_config` file.
```
$ sudo vim /etc/ssh/sshd_config
Change “PasswordAuthentication” to “no”.
```
Restart the sshd service.
```
$ sudo service ssh restart
```
Disable remote login of the ROOT user.
Edit `/etc/ssh/sshd_config`
```
$ sudo vim /etc/ssh/sshd_config
```
Change this line:
```
#PermitRootLogin yes
```
To this:
```
PermitRootLogin no
```
### Apache web server
Install apache2
```
$ sudo apt-get install apache2
```
Start it.
```
$ sudo service apache2 start
```
Test that the server is working by opening your browser and going to the public IP address of your instance. You should see the default ubuntu apache page saying “It works!”.

For debugging, check error log:
```
$ sudo vim /var/log/apache2/error.log
```
### Apache mod-wsgi (web server gateway interface)
Install it.
```
$ sudo apt-get install libapache2-mod-wsgi
```
You then need to configure Apache to handle requests using the WSGI module. You’ll do this by editing the `/etc/apache2/sites-enabled/000-default.conf` file. This file tells Apache how to respond to requests, where to find the files for a particular site, etc.

### Make a wsgi app. 
Make `.wsgi` file for your app.
```
$ sudo nano /var/www/html/item_catalog.wsgi
```
Create and edit your config file:
```
$ sudo vim /etc/apache2/sites-available/item_catalog.conf
```
Edit it like so:
```
<VirtualHost *:80>
                ServerName 18.212.186.88
                ServerAdmin admin@18.212.186.88@
                WSGIScriptAlias / /var/www/udacity_fsnd/item_catalog/item_catalog.wsgi
                <Directory /var/www/udacity_fsnd/item_catalog/item_catalog/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/udacity_fsnd/item_catalog/item_catalog/static
                <Directory /var/www/udacity_fsnd/item_catalog/item_catalog/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Finally, restart Apache with this command:
```
$ sudo apache2ctl restart
```

## Postgres
Install it.
```
$ sudo apt-get install postgresql
```
Create a user.
```
EXAMPLE (user name is “grader”):
$ sudo -u postgres createuser -P grader
```
Make this user superuser (reference):
```
$ ALTER USER grader WITH SUPERUSER;
```
Create `catalog` database and dump sql file:
```
$ psql catalog < catalog.sql
```
Test connection and ability to query the databse. Then restart apache2:
```
$ sudo apache2ctl restart
```
To debug, you can check the error log:
```
$ sudo vim/var/log/apache2/error.log
```

### Organization of code
The `item_catalog.wsgi` file should be saved one level up, in the same directory as `item_catalog`.
```
item_catalog/
|__item_catalog.wsgi
|__item_catalog/
    |__(rest of this repo's contents)
```

## Third-party resources used to complete this project
[Disabling SSH login for root user](https://mediatemple.net/community/products/dv/204643810/how-do-i-disable-ssh-login-for-the-root-user)
[How to set up a ufw firewall on Ubunug](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04)
[Postgresql: How to grant access to users](https://tableplus.io/blog/2018/04/postgresql-how-to-grant-access-to-users.html)
Special thanks to:
[otsop110](https://github.com/otsop110/fullstack-nanodegree-linux-server-configuration#modify-your-app-structure-to-be-ready-for-the-deployment)
[Sean-Holcomb](https://github.com/Sean-Holcomb/Linux-Server-Configuration)
[sarithk](https://github.com/sarithk/LinuxServerConfig)
