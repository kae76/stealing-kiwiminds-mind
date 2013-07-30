Assumptions
* LAMP stack is up and running
sudo available
* MySQL root credentials available (or enough credentials to create databases and users)
* drush available
Using nano as text editor (I don't understand vim)
* Website hosting directory is /var/www
Site structure will be
/var
/www
/example.com
/www
/db


Create database and user and give correct permissions
Download codebase (drupal, installation profile, etc)
Ensure correct file permissions on files
Create new apache config file
Edit /etc/hosts file so that specified URL is served locally
Enable site in apache
Restart apache for changes to take place
Install site
Import existing DB
New site
Housekeeping

1. Create database
```
mysql -u root -p
mysql > create database example.com;
mysql > GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER,  LOCK TABLES, CREATE TEMPORARY TABLES ON `databasename`.* TO  'username'@'localhost' IDENTIFIED BY 'password'; 
// you could grant all, but this is from https://drupal.org/documentation/install/create-database
```

2. Download codebase
`cd /var/www`
// Download the latest version of drupal. The following line can be replaced by downloading a tarball such as Open Atrium
`drush dl drupal`
// Create a directory to put the site in
mkdir example.com
// Create a directory to put your DB dumps in
mkdir db
// Move your freshly downloaded codebase to example.com/www
mv drupal-7.22 example.com/www

3. Ensure correct file permissions on files
// I have a script that I run to achieve this. It may not need to be done, depending on your setup. https://gist.github.com/kiwimind/6113953
sudo bash /path/to/fix-perms.sh --drupal_path=/var/www/example.com/www --drupal_user=karen // <= use the user that you're logged in to your VM as

4. Create new apache config file
cd /etc/apache2/sites-available
 `sudo nano sitename.conf`
// bare minimum apache config file
```
<VirtualHost *:80>
ServerName example.com
ServerAlias *.example.com
DocumentRoot /var/www/example
RewriteEngine On
</VirtualHost>
```

5. Edit hosts file so that specified URL is served locally - I like to use dev.example.com so that you're working on the correct TLD (so if your finished site will be at www.karenleech.co.uk, use dev.karenleech.co.uk in dev
`sudo nano /etc/hosts`
// Add in an entry to this file similar to the following
`127.0.0.1  dev.example.com`

6. Enable site in apache
// 2 ways of doing this
// Either
sudo a2ensite sitename.conf // use name as created in step 4.
// Or
```
cd /etc/apache2/sites-enabled
sudo ln -s ../sites-available/sitename.conf
```

6.1 Restart apache for changes to take place
// Sometimes a reload is enough, but restart on dev box makes no difference
// 2 ways of doing this
// Either
`sudo service apache2 restart`
// Or
`sudo /etc/init.d/apache2 restart`

7. Install site
7.1 Import existing DB
// Edit sites/default/settings.php to enter your database credentials
`drush sqlc < your_db_dump.sql`
7.2 New site
// I have found using drush `site-install` to be more reliable than using the UI
// If you're happy to use the UI, then at this point visit dev.example.com and install, otherwise the following will install site in one line operation \o/
// [installation profile] can be any of the drupal default profiles, like standard or minimal, or something like openatrium depending on what you downloaded in step 2.
`drush site-install [installation profile] --db-url=mysql://db_user:db_password@localhost/db_name`
// There are other flags that can be used such as the follwing. They are all optional.
```
--account-name=your_admin_login_name
--account-pass=your_admin_login_password
--account-mail=your_admin_login@mailaccount.com
```
// Also possible to specify site name, site email address etc. More details at http://drush.ws/#site-install

// Check your new site! dev.example.com
