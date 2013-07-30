Notes pulled together across IRC channel #drupal-uk & http://openetherpad.org/p/kae76_sapping_kiwiminds..mind

[Kiwimind](https://github.com/kiwimind) <-- is the master mind!

Caveat ...A work in progress... use with care!

####Assumptions
* LAMP stack is up and running
sudo available
* MySQL root credentials available (or enough credentials to create databases and users)
* drush available
* Using nano as text editor
* Website hosting directory is /var/www
Site structure will be  
/var  
\- /www  
-- /example.com  
--- /www  
--- /db  


TOC: *note to link up later*

1. Create database and user and give correct permissions
2. Download codebase (drupal, installation profile, etc)
3. Ensure correct file permissions on files
4. Create new apache config file
5. Edit /etc/hosts file so that specified URL is served locally
6. Enable site in apache  
 6.1 Restart apache for changes to take place
7. Install site  
 7.1 Import existing DB  
 7.2 New site  
8. Housekeeping

---

_1. Create database_
```
mysql -u root -p
mysql> create database example.com;
mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER,  LOCK TABLES, CREATE TEMPORARY TABLES ON `databasename`.* TO  'username'@'localhost' IDENTIFIED BY 'password'; 
// you could grant all, but this is from https://drupal.org/documentation/install/create-database
```

_2. Download codebase_  
`cd /var/www`  
// Download the latest version of drupal. The following line can be replaced by downloading a tarball such as Open Atrium  
`drush dl drupal`  
// Create a directory to put the site in  
`mkdir example.com`  
// Create a directory to put your DB dumps in  
`mkdir db`  
// Move your freshly downloaded codebase to example.com/www  
`mv drupal-7.22 example.com/www`  

_3. Ensure correct file permissions on files_  
// I have a script that I run to achieve this. It may not need to be done, depending on your setup. https://gist.github.com/kiwimind/6113953  
`sudo bash /path/to/fix-perms.sh --drupal_path=/var/www/example.com/www --drupal_user=karen`  
// ^ use the user that you're logged in to your VM as  

_4. Create new apache config file_  
```
cd /etc/apache2/sites-available
sudo nano sitename.conf
```
// bare minimum apache config file
```
<VirtualHost *:80>
ServerName example.com
ServerAlias *.example.com
DocumentRoot /var/www/example
RewriteEngine On
</VirtualHost>
```

_5. Edit hosts file so that specified URL is served locally_  
I like to use dev.example.com so that you're working on the correct TLD (so if your finished site will be at www.karenleech.co.uk, use dev.karenleech.co.uk in dev
`sudo nano /etc/hosts`
// Add in an entry to this file similar to the following
`127.0.0.1  dev.example.com`

_6. Enable site in apache_  
// 2 ways of doing this  
// Either  
`sudo a2ensite sitename.conf // use name as created in step 4.`  
// Or  
```
cd /etc/apache2/sites-enabled
sudo ln -s ../sites-available/sitename.conf
```

_6.1 Restart apache for changes to take place_  
// Sometimes a reload is enough, but restart on dev box makes no difference  
// 2 ways of doing this  
// Either  
`sudo service apache2 restart`  
// Or  
`sudo /etc/init.d/apache2 restart`  

_7. Install site_  
7.1 Import existing DB  
// Edit sites/default/settings.php to enter your database credentials  
`drush sqlc < your_db_dump.sql`  
_7.2 New site_  
// I have found using drush `site-install` to be more reliable than using the UI  
// If you're happy to use the UI, then at this point visit dev.example.com and install, otherwise the following will install site in one line operation \o/
// [installation profile] can be any of the drupal default profiles, like standard or minimal, or something like openatrium depending on what you downloaded in step 2.
`drush site-install [installation profile] --db-url=mysql://db_user:db_password@localhost/db_name`
// There are other flags that can be used such as the following. They are all optional.
```
--account-name=your_admin_login_name
--account-pass=your_admin_login_password
--account-mail=your_admin_login@mailaccount.com
```
// Also possible to specify site name, site email address etc. More details at http://drush.ws/#site-install

// Check your new site! dev.example.com
