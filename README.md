HTACCESS FOR SECURITY AUDIT
=============================


## Reflective Cross Site Scripting

**Recommendation:**

The best way to prevent cross-site scripting is to make sure that the web application does not take use of user input in the subsequent HTML pages, without validating or sanitizing it. Validation implies verification of the user input to determine if the input is valid, according to its purpose.


**Solution:**

Use the following function  
```php
htmlspecialchars() 
```

## Directory Listing

**Recommendation:**

Restrict the access to the directory listings in the web server configuration.

**Solution:**


Enable .htaccess (By default, .htaccess isn’t enable. To enable it you will need to edit the configuration file. )

Use a text editor to open your configuration file:

```
sudo vi /etc/apache2/sites-available/example.com.conf
```

After the VirtualHost block () add:

```html
…
</VirtualHost>
<Directory /var/www/html/example.com/public_html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
</Directory>
```

Save the file, then restart apache:
```
sudo service apache2 restart
```

Navigate to your site’s root directory and create an .htaccess file. Now add the following lines

```
Options -Indexes
```

## Missing Custom Error Pages

**Recommendation:**

Enable Custom Error Pages through your server configuration.

**Solution:**


Create a folder named errors and create custom error pages into that folder.

Add the following lines into .htaccess file

```html
#Error Pages
<IfModule mod_rewrite.c>
ErrorDocument 400 /errors/400.php
ErrorDocument 401 /errors/401.php
ErrorDocument 403 /errors/403.php
ErrorDocument 404 /errors/404.php
ErrorDocument 500 /errors/500.php
ErrorDocument 502 /errors/502.php
ErrorDocument 503 /errors/503.php
</IfModule>
```

## Unwanted HTTP Methods

****Recommendation:****

Disable PUT,TRACE,OPTIONS,DELETE,CONNECT and other unwanted Method through
web server configurations.

**Solution:**


To disable HTTP Methods (HEAD, PUT, DELETE, OPTIONS) Add the following lines in httpd.conf / apache2.conf or in .htaccess file.


```html
<IfModule mod_rewrite.c>
RewriteCond %{REQUEST_METHOD} ^(TRACE|TRACK|PUT|OPTIONS|DELETE|HEAD|CONNECT)
RewriteRule .*$ - [F,L]
</IfModule>
```

Check : 
```
curl --insecure -v -X TRACE https://ip/
```

## PhpMyAdmin Login Page Visible

**Recommendation:**

Disable the login page to access by normal user.

**Solution:**


Disable phpMyAdmin by disabling the module configuration.


```python
sudo a2disconf phpmyadmin.conf
sudo /etc/init.d/apache2 restart
```

Enable it with


```python
a2enconf phpmyadmin.conf
sudo /etc/init.d/apache2 restart
```


## Arbitrary HTTP Methods

**Recommendation:**

Allow only GET and POST methods.

**Solution:**

Add the following lines in httpd.conf / apache2.conf or in .htaccess file.

```html

<LimitExcept GET POST>
Deny from all
</LimitExcept>

```
## Session ID Name Fingerprinting

**Recommendation:**

It is recommended to change the default session ID name of the web development framework to a generic name, such as 'id' or to any name specific to the application.

**Solution:**



Change the value of variable named  session.name in php.ini.

Or ,

Alter this programmatically by calling session_name before any call to session_start or session_register.


```php
session_name('mySessionName');
session_start();
```

## Web Server Version Disclosure

**Recommendation:**

Configure your web server to prevent information leakage from the SERVER header of its HTTP response and if version is disclosed through the error pages, edit/set the default error pages to hide this information.

**Solution:**

Install mod security for apache and then add this in your apache2.conf.



```html

<IfModule security2_module>
    SecRuleEngine on
    ServerTokens Full
    SecServerSignature " "
</IfModule>

```

Restart the server 

```
sudo service apache2 restart
```
How To Install ModSecurity:

Ubuntu | CentOS
--- | ---
sudo apt-get install libapache2-mod-security2 | yum install mod_security
Restart Apache: | Restart Apache by entering the following command:
/etc/init.d/apache2 restart | /etc/init.d/httpd restart
Verify the version of ModSecurity is 2.8.0 or higher: | Verify the version of ModSecurity is 2.8.0 or higher:
apt-cache show libapache2-mod-security2 | yum info mod_security



 
> Note:
> By adding following lines into etc/apache2.conf file , server OS version will hide .
> **ServerSignature Off**
> **ServerTokens Prod**
> But in the header **Server name will be displayed** as apache on its own cannot completely unset the Server header (not even with mod_headers).There is a way to do this using ModSecurity.


## Email Address Disclosure

**Recommendation:**

The emails should be added either encoded or in a fashion that could not be captured by an automated script; like, email [at] example [dot] com


## Missing Security Headers

**Recommendation:**

Implement all security headers


**Solution:**

Implement all security headers. a2enmod headers to install and enable headers.

        
# Extra Security Headers

```html

<IfModule mod_headers.c>
	Header set X-XSS-Protection "1; mode=block"
	Header set X-Content-Type-Options "nosniff"
	Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
	Header set Content-Security-Policy "default-src 'self'; style-src 'self' 'unsafe-inline' *.googleapis.com data:; font-src 'self' *.gstatic.com;script-src 'self'  'unsafe-inline' 'unsafe-eval' *.jquery.com *.google-analytics.com *.googleapis.com data:; connect-src 'self' *.google-analytics.com *.googleapis.com *.gstatic.com data:;img-src * data:;"
	Header set Access-Control-Allow-Origin SAMEORIGIN
	Header set X-Download-Options noopen
	Header set X-Frame-Options Deny
	Header set Cache-Control "private, max-age=0"
	Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure
</IfModule>

# Allow Cross-Domain Fonts
<IfModule mod_headers.c>
	<FilesMatch "\.(eot|otf|ttc|ttf|woff|woff2)$">
		Header set Access-Control-Allow-Origin "*"
	</FilesMatch>
</IfModule>

# Disable Internet Explorer Compatibility View
<IfModule mod_headers.c>
    BrowserMatch MSIE is-msie
    Header set X-UA-Compatible IE=edge env=is-msie
</IfModule>

```


## Disable HTTP Compression

**Recommendation:**

To disable compression in Apache, typically you just need to disable the module mod_deflate. After making the below adjustments, test again with the above manual test to confirm compression is disabled.


**Solution:**


***Debian/Ubuntu:***
```python
sudo a2dismod deflate
sudo /etc/init.d/apache2 restart
```
***Red Hat or CentOS:***
```python
sudo nano /etc/httpd/conf/httpd.conf
```
Comment out this line:
```python
LoadModule deflate_module modules/mod_deflate.so
```
It should now look like this:
```python
#LoadModule deflate_module modules/mod_deflate.so
```
Close and save the file then restart httpd:
```python
sudo /etc/init.d/httpd restart

```
