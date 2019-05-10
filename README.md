# item_catalog_final

## Connecting to the site
(SSH key submitted with this project)
```
ssh grader@18.212.186.88 -p 22 -i grader
```

## Software installed

apache2
(mod wsgi whatever)
postgresql
python-flask
python-jinja2
python-pip
python-psycopg2
python-sqlalchemy
python-werkzeug
readline

## Configurations

Setting up your Linux box (Ubuntu)
Packages
Update (collects info on what’s available to update)
$ sudo apt-get update
Upgrade (actually updates the software)
$ sudo apt-get upgrade
Remove anything unnecessary
$ sudo apt-get autoremove
Install new packages (e.g. search for packages at packages.ubuntu.com)
$ sudo apt-get install <package>
Take a look at /etc/passwd
Definition of columns:
user : x : userid : groupid : [user description] : user’s home : user’s default shell
Create a new user
 First generate key pair - see below section.
Might want to name the key the same as the new username.
Add user.
$ sudo adduser <username>
EXAMPLE
User: grader
Password: udacity201905
View permissions of users (perform this as root user).
$ sudo cat /etc/sudoers
Shows  #includedir /etc/sudoers.d
$ sudo ls /etc/sudoers.d
Shows root user. Want to add the new user here.
$ sudo cp /etc/sudoers.d/vagrant /etc/sudoers.d/<username>
Edit.
$ sudo nano /etc/sudoers.d/<username> 
Replace existing user with “<username>”. New user now has sudo permissions.
Force a user to reset their password on next login.
$ sudo passwd -e <username>
Reboot the machine.
$ sudo reboot
SSH back in using the ubuntu user.
$ ssh -i <.pem file> ubuntu@<public IP of your instance>
SU into the new user:
$ su <username>
Enter password; then enter new password.
Jump down to “For a new user name, you’ll need to do these steps” in next section.
SSH in as user.
$ ssh <username>@127.0.0.1 -i <path to specific private key>
Key Based Authentication
ALWAYS GENERATE KEYS LOCALLY.
Create the pair.
$ ssh-keygen
Save it to a directory and filename (default is ~/.ssh/id_rsa, but you can change the filename to something more specific, e.g. “udacity_linux_course”).
Private and public keys will be generated .
For a new user name, you’ll need to do these steps:
Install the public key on your remote server.
In the home dir, make “.ssh” dir.
$ mkdir .ssh
Make an “authorized_keys” file. Each line in this file can have a public key for each user that’s going to log in.
$ touch .ssh/authorized_keys
$ nano .ssh/authorized_keys
Paste in your public key. (Use: $ pbcopy < keyname.pub)
Set permissions on “authorized_keys” file and the “.ssh” directory.
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
You can now login like so, using the -i flag pointing to your private key file.
$ ssh <username>@127.0.0.1 -p 2222 -i ~/.ssh/<filename>
Force key based authentication for all users.
Edit the /etc/ssh/sshd_config file.
$ sudo vim /etc/ssh/sshd_config
Change “PasswordAuthentication” to “no”.
Restart the sshd service.
$ sudo service ssh restart
Disable remote login of the ROOT user
Reference
Edit /etc/ssh/sshd_config
Change this line:
#PermitRootLogin yes
To this:
PermitRootLogin no
File permissions
Owner / Group / Everyone   USER / GROUP
rwx / rwx / rwx  (read-write-execute)
Octal form (e.g. when you do chmod 777):
r = 4
w = 2
x = 1 

Firewalls
Ports
HTTP: 80
HTTPS: 443
SSH: 22
FTP: 21
POP3: 110
SMTP: 25
You should only listen on the ports required for your application to function correctly. You can configure this using a firewall.
Ubuntu comes pre-installed with “ufw”. You have to make it active.
Check status.
$ sudo ufw status
Start by blocking all incoming and allowing all outgoing -- on all ports.
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
Allow ssh.
$ sudo ufw allow ssh
Allow SSH on port 2222.
$ sudo ufw allow 2222/tcp
Allow http on port 80. (reference)
$ sudo ufw allow http
Allow ntp on port 123.
$ sudo ufw allow ntp
Now turn it on the firewall!
$ sudo ufw enable
Check status.
$ sudo ufw status
Apache web server
Install apache2
$ sudo apt-get install apache2
Start it (reference)
$ sudo service apache2 start
Test that the server is working by opening your browser and going to the public IP address of your instance. You should see the default ubuntu apache page sayin “It works!”.z
For debugging, check error log:
$ sudo vim /var/log/apache2/error.log

Apache mod-wsgi (web server gateway interface)
Install it.
$ sudo apt-get install libapache2-mod-wsgi
You then need to configure Apache to handle requests using the WSGI module. You’ll do this by editing the /etc/apache2/sites-enabled/000-default.conf file. This file tells Apache how to respond to requests, where to find the files for a particular site and much more. You can read up on everything this file can do within the Apache documentation.
Make a test wsgi app. 
Edit your config file:
$ sudo vim /etc/apache2/sites-enabled/000-default.conf
Add the following line at the end of the <VirtualHost *:80> block, right before the closing </VirtualHost> line: WSGIScriptAlias / /var/www/html/myapp.wsgi
Finally, restart Apache with this command:
$ sudo apache2ctl restart
WSGI is a specification that describes how a web server communicates with web applications. Most if not all Python web frameworks are WSGI compliant, including Flask and Django; but to quickly test if you have your Apache configuration correct you’ll write a very basic WSGI application.
You just defined the name of the file you need to write within your Apache configuration by using the WSGIScriptAlias directive. Despite having the extension .wsgi, these are just Python applications. Create the /var/www/html/myapp.wsgi file using the command:
$ sudo nano /var/www/html/myapp.wsgi
Within this file, write the following application:
def application(environ, start_response):
    status = '200 OK'
    output = 'Hello Udacity!'


    response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
    start_response(status, response_headers)


    return [output]
This application will simply print return Hello Udacity! along with the required HTTP response headers. After saving this file you can reload http://localhost:8080 to see your application run in all its glory!

Postgres
Install it.
$ sudo apt-get install postgresql
Create a user.
EXAMPLE (user name is “grader”):
$ sudo -u postgres createuser -P grader
Password: udacity
Make this user superuser (reference):
$ ALTER USER grader WITH SUPERUSER;
If you run into trouble, run:
$ sudo su - postgres
postgres$ psql
Then you’ll be running psql as “postgres” superuser.
Create an empty database:
=# CREATE DATABASE catalog;
Dump sql file:
$ psql catalog < catalog.sql
Make a demo wsgi app that connects to your postgres DB.
Add this line to your /etc/apache2/sites-enabled/000-default.conf file to point to your demo wsgi file:
/var/www/html/test_postgres.wsgi
Created this file at /var/www/html/test_postgres.wsgi
https://github.com/pseudoyim/item_catalog/blob/master/test_postgres.wsgi.py
Then restart apache2:
$ sudo apache2ctl restart
To debug, you can check the error log:
/var/log/apache2/error.log
You may have to convert unicode to bytestring. (reference) Example:
b = mystring.encode()



## Third-party resources used to complete this project

The `item_catalog.wsgi` file should be one level up, in the same directory as `item_catalog`.
```
item_catalog/
|__item_catalog.wsgi
|__item_catalog/
    |__(rest of this repo's contents)
```
