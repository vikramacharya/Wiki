# HTTPS Self-signed on MAC
As part of a series on migrating a website to support HTTPS everywhere I needed to create a self-signed certificate on Mac OS X. It’s pretty easy. Here is what you need to do:
- Create a host key
- Create a certificate request
- Create the SSL certificate
- For apache, create a nopass host key
 
To get started, we will create a new folder called ssl in our /etc/apache2 folder.
 
```
$ cd /private/etc/apache2
$ sudo mkdir ssl
$ cd ssl
$ sudo ssh-keygen -f securelocal.example.com.key
```

You should get a message: Generating public/private rsa key pair., and then you will be asked to provide a passcode for the host key. You can choose anything for the passcode, or leave it blank.
# Create Certificate Request
Using the host key you created, create the certificate request file. Note that I am using local.example.com as the file naming convention. I recommend that you use something similar based on your website address.
 
```
$ sudo openssl req -new -key securelocal.example.com.key -out securelocal.example.com.csr
 ```
After executing this command, you will be prompted to enter information about the entity/organization/company. As this is a self-signed certificate, this doesn’t really matter. So, enter whatever suits you – except for Common Name (e.g. server FQDN or YOUR name). The FQDN, or fully qualified domain name, must match the website URL. In this example, we would enter local.example.com.

# Create SSL Certificate
This next step on my local development environment is to create a self-signed certificate. This certificate has no authority, and I will get a warning from my browser indicating this. However, there is no need to purchase a SSL certificate for local development. In a production environment you would submit the certificate request (csr) file to the certificate authority, who would then provide you with the SSL certificate.
So, let’s go ahead and create the self-signed certificate. If you followed the steps above, enter the following command to create the SSL certificate using the csr file.
```
 $ sudo openssl x509 -req -days 365 -in securelocal.example.com.csr -signkey securelocal.example.com.key -out securelocal.example.com.crt
```
 
You will be prompted to enter information about your organization, as well as the common name, or fully qualified domain name (FQDN). Be sure that this matches the virtual host (or site) that you are setting up.
 

# Create nopass Host Key
In order to configure Apache we need to create a nopass key. If you are not using this SSL certificate for Apache you may not be required to do this last step.
 
```
$ sudo openssl rsa -in securelocal.example.com.key -out securelocal.example.com.nopass.key
```
If this runs successfully, you should get a simple message: writing RSA key. All set. We should now see several files in our ssl directory that we will use for securing our website.

Now, we are going to go through the steps necessary to set up HTTPS on our local Apache web server:
- Load mod_ssl extension
- Include ssl config file
- Modify ssl config file, adding our new virtual host bound to port 443
- Optionally specify new testing domain in our hosts file
- Test and restart Apache
 
My httpd.conf file is located at: /etc/apache2/httpd.conf
```
$ cd /etc/apache2/
$ sudo nano httpd.conf
```

Find the following line and uncomment it. Just in case you are not sure, comments have a leading pound/hash symbol ( # ) – just remove it.

```
LoadModule ssl_module libexec/apache2/mod_ssl.so
```
While we still have the httpd.conf file open, we also need to uncomment the line that includes the httpd-ssl.conf file.
```
# Secure (SSL/TLS) connections
Include /private/etc/apache2/extra/httpd-ssl.conf
```
Add VirtualHost to httpd-ssl.conf
The last step is to configure a new virtual host that is bound to port 443 (HTTPS). There is already a sample <VirtualHost> record in the httpd-ssl.conf file. I suggest you first remove it, or comment it all out, so that you can just paste in the necessary code at the bottom of the file.
My httpd-ssl.conf file is located at: /etc/apache2/conf/httpd-ssl.conf

You will need to open the file using nano:
```
$ cd /etc/apache2/extra
$ sudo nano httpd-ssl.conf
```
Declare VirtualHost
The first step is to declare a new virtual host using the ```<VirtualHost>``` directive.
```
<VirtualHost *:443>
```
General Virtual Host Settings
Next, within the <VirtualHost> directive, we will declare some basic host settings:
- DocumentRoot: absolute path to the webroot for the site
- ServerName: the fully qualified domain name (FQDN)
- ErrorLog: location of the error log
- CustomLog: location of the access log

# Enable SSL
To enable the SSL engine in Apache we simply add set the setting to “on”.
```#SSL Engine Switch
SSLEngine on
```
Specify certificate and private key
Using the paths as I described at the beginning, we will tell the SSL engine the location of the certificate request file (csr) and the private host key (.key).
 
- Server Certificate:
```/private/etc/apache2/ssl/securelocal.example.com.crt
SSLCertificateFile "/private/etc/apache2/ssl/securelocal.example.com.crt"
```
- #Server Private Key:
```/private/etc/apache2/ssl/securelocal.example.com.key
SSLCertificateKeyFile "/private/etc/apache2/ssl/securelocal.example.com.key"
``` 

# Protocols and ciphers
For our local development server we are not going to worry about the protocols/ciphers and just simply add the following options.
# SSL Engine Options:
```
<FilesMatch "\.(cgi|shtml|phtml|php)$">
    SSLOptions +StdEnvVars
</FilesMatch>
<Directory "/Library/WebServer/CGI-Executables">
    SSLOptions +StdEnvVars
</Directory>
 
<Directory "/Users/vikram/Documents/workspace/securelocal.example.com">
        Require all granted
        SSLOptions +StdEnvVars
</Directory>

DocumentRoot "/Users/vikram/Documents/workspace/securelocal.example.com"
ServerName securelocal.example.com
ErrorLog "/private/var/log/apache2/securelocal.example.com-error_log"
CustomLog "/private/var/log/apache2/securelocal.example.com-access_log" common
```

- Complete Virtual Host
Last, we need to close the <VirtualHost> directive. Our complete virtual host in the httpd-ssl.conf file should look like this:
```
  <VirtualHost *:443>
    #General setup for the virtual host
    DocumentRoot "/Users/vikram/Documents/workspace/securelocal.example.com"
    ServerName securelocal.example.com
    ErrorLog "/private/var/log/apache2/securelocal.example.com-error_log"
    CustomLog "/private/var/log/apache2/securelocal.example.com-access_log" common

    #SSL Engine Switch:
    SSLEngine on

    #Server Certificate:
    SSLCertificateFile "/private/etc/apache2/ssl/securelocal.example.com.crt"

    #Server Private Key:
    SSLCertificateKeyFile "/private/etc/apache2/ssl/securelocal.example.com.key"

    #SSL Engine Options:
    <FilesMatch "\.(cgi|shtml|phtml|php)$">
      SSLOptions +StdEnvVars
    </FilesMatch>
    <Directory "/Library/WebServer/CGI-Executables">
      SSLOptions +StdEnvVars
    </Directory>
    <Directory "/Users/vikram/Documents/workspace/securelocal.example.com">
      Require all granted
      SSLOptions +StdEnvVars
    </Directory>
  </VirtualHost>
```

# Hosts file
In case this is a new website, you should also modify your hosts file to direct the domain name to your local Apache web server. If you already had the domain configured in your hosts file, then you can skip this step.
The hosts file is located at: /etc/hosts
Note that the file does not have an extension. Let’s open this up in nano.
```$ cd /etc
$ sudo nano hosts
```
Then, add a new line for your website. I am using local.example.com.
```
127.0.0.1    securelocal.example.com
```
Test Configuration and Restart Apache
The last step is to test our new configuration, and assuming everything is good, restart the Apache web server.
```
$ sudo apachectl -t
$ sudo apachectl restart
```
All done. You are now serving your website over HTTPS using Apache.
