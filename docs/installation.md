## Manual installation

### Requirements
Make sure the server hosting Betterprotect supports the following:

#### PHP
* php7.3 or higher 
* php7.3-curl
* php7.3-json
* php7.3-mysql
* php7.3-opcache
* php7.3-readline
* php7.3-gd
* php7.3-mbstring
* php7.3-xml
* php7.3-ldap

#### Database 
* MariaDB 10.1 or higher (MySQL should work as well)

#### Web server
* Apache, Nginx or IIS (with Websocket Proxy support, e.g. proxy_wstunnel)

### Installation on Debian Buster with Apache and MariaDB

#### Dependencies
````bash
apt install apache2 libapache2-mod-php7.3 php7.3-curl php7.3-json php7.3-mysql php7.3-opcache php7.3-readline php7.3-gd php7.3-mbstring php7.3-xml php7.3-ldap mariadb-server supervisor
````

#### Database
Secure your database server:

````bash
mysql_secure_installation
````

Create a database:

````bash
mysql -u root -p
````

````mysql
CREATE DATABASE betterprotect;
GRANT ALL PRIVILEGES ON betterprotect.* TO 'betterprotect'@'localhost' IDENTIFIED BY 'your-secret-password-here';
````

!!! note "We will need the password later."

#### Webserver
Adjust the marked path if necessary:

````apacheconfig
<VirtualHost *:443>
  ServerName betterprotect.contoso.com

  # !!! Adjust the path if required !!!
  DocumentRoot "/var/www/betterprotect/public"

   # !!! Adjust the path if required !!!
  <Directory "/var/www/betterprotect/public">
    Options Indexes FollowSymLinks MultiViews
    AllowOverride None
    Require all granted
  </Directory>

   # !!! Adjust the path if required !!!
  ErrorLog "/var/log/apache2/betterprotect_ssl_error_ssl.log"

  # !!! Adjust the path if required !!!
  CustomLog "/var/log/apache2/betterprotect_ssl_access_ssl.log" combined

  SSLEngine on
   # !!! Adjust the path if required !!!
  SSLCertificateFile      "/etc/ssl/apache2/publicCert.pem"

   # !!! Adjust the path if required !!!
  SSLCertificateKeyFile   "/etc/ssl/apache2/privateKey.pem"
  SSLCACertificatePath    "/etc/ssl/certs"

  <Location "/ws">
    ProxyPass ws://localhost:6001/
    ProxyPassReverse ws://localhost:6001/
  </Location>

  # !!! Adjust the path if required !!!
  <Directory /var/www/betterprotect/public>
    <IfModule mod_rewrite.c>
      <IfModule mod_negotiation.c>
        Options -MultiViews
      </IfModule>

     RewriteEngine On

     RewriteCond %{REQUEST_FILENAME} !-d
     RewriteRule ^(.*)/$ / [L,R=301]

     RewriteCond %{REQUEST_FILENAME} !-d
     RewriteCond %{REQUEST_FILENAME} !-f
     RewriteRule ^ index.php [L]

     RewriteCond %{HTTP:Authorization} .
     RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
   </IfModule>
  </Directory>
</VirtualHost>
````

!!! note "Location "/ws""
    This part is important for the bidirectional communication between Betterprotect and the client.
    Betterprotect uses Websockets to inform the user about the current status of background tasks. Apache works
    as a reverse proxy for the websocket server.
    
#### Download & Extract
````bash
cd /var/www
wget https://github.com/Hank-IT/Betterprotect/releases/download/v1.5/betterprotect-production-65-d9d353244d32a619b3ae2cc6ab0dca7e93d3eed5.tar.gz
gunzip betterprotect-production-65-d9d353244d32a619b3ae2cc6ab0dca7e93d3eed5.tar.gz
tar -xvf betterprotect-production-65-d9d353244d32a619b3ae2cc6ab0dca7e93d3eed5.tar
````

!!! warning "Releases"
    Always check the release tab on the Github repository for the current version, as the link here might be outdated.
    The one looking like this is required: "betterprotect-production-65-d9d353244d32a619b3ae2cc6ab0dca7e93d3eed5.tar.gz". 
    This is a fully build release, including all dependencies.
    
#### Setup Betterprotect
````bash
cp .env.example .env
vi .env
````

Adjust the following settings in your .env file:

| Setting     | Description                                      |
| ----------- | -------------------------------------------------|
| APP_URL     | The root URL of your Betterprotect installation. |
| APP_DOMAIN  | The domain of your Betterprotect instalation.    | 
| DB_HOST     | The IP or name of your database host.            | 
| DB_PORT     | The port on which your database is listening.    | 
| DB_DATABASE | The name of the database you created earlier.    | 
| DB_USERNAME | The username of the user you created earlier.    | 
| DB_PASSWORD | The password of the user you created earliuer.   | 

Generate the secret key, which is used for encryption of passwords:
````bash
php artisan key:generate
````

#### Setup Supervisor
Betterprotect uses a background worker for long running tasks. Supervisor is used to start and monitor the process.

Create the config for the queue worker ``/etc/supervisor/conf.d/betterprotect-worker.conf`` with the following content:
````
[program:betterprotect-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/betterprotect/artisan queue:work --tries 3 --timeout 120 database --queue task
autostart=true
autorestart=true
user=www-data
numprocs=4
redirect_stderr=true
stdout_logfile=/var/www/betterprotect/storage/logs/betterprotect-worker.log
````

Create the config for the websocket server ``/etc/supervisor/conf.d/betterprotect-websocket.conf`` with the following content:
````
[program:betterprotect-websocket]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/betterprotect/artisan websocket:serve
autostart=true
autorestart=true
user=www-data
numprocs=1
redirect_stderr=true
stdout_logfile=/var/www/betterprotect/storage/logs/betterprotect-websocket.log
````

Restart supervisor to start the processes:
````bash
systemctl restart supervisor
````

#### Create a user
````bash
php artisan user:create
````

You successfully installed Betterprotect 😄