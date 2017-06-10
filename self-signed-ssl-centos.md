# SSL Configuration CentOs 7
To enable SSL on Apache HTTP Server use the following command to install SSL Module and OpenSSL tool-kit which is needed for SSL/TLS support.
```
yum install mod_ssl openssl
```
After SSL module has been installed, restart HTTPD daemon and add a new Firewall rule to ensure that SSL port – 443 – it’s opened to outside connections on your machine in listen state.

#### FOR CENTOS 7

```
systemctl restart httpd
firewall-cmd --add-service=https  

firewall-cmd --permanent  --add-service=https 
```
#### FOR CENTOS 6
```
service httpd restart
``` 
In case you get error while restarting apache due to serverName

-  Add ServerName in the file httpd.conf
```
cd /etc/httpd/conf
nano httpd.conf
```
-  Modify the serverName with localhost
```
ServerName localhost:80
service httpd restart
```

-  Enable firewall
You can see the list of iptables using the following command 
```
# sudo iptables -L
``` 
 - Add/enable the service through firewall using the following. 
```
# sudo system-config-firewall-tui
``` 

- The previous SSL communication between the server and client was done using a default Certificate and Key automatically generated on installation. In order to generate new private keys and self-signed certificates pairs create the following bash script on a executable system path ($PATH).
For this tutorial /usr/local/bin/ path was chosen, make sure the script has executable bit set and, then, use it as a command to create new SSL pairs on /etc/httpd/ssl/ as Certificates and Keys default location.
```
# nano /usr/local/bin/apache_ssl
```
-  Add the following file content in the editor and save.
#
```
#!/bin/bash
mkdir /etc/httpd/ssl
cd /etc/httpd/ssl

echo -e "Enter your virtual host FQDN: \nThis will generate the default name for Apache SSL Certificate and Key!"
read cert

openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out $cert.key
chmod 600 $cert.key
openssl req -new -key $cert.key -out $cert.csr
openssl x509 -req -days 365 -in $cert.csr -signkey $cert.key -out $cert.crt

echo -e " The Certificate and Key for $cert has been generated!\nPlease link it to Apache SSL available website!"
ls -all /etc/httpd/ssl
exit 0
```

 - Now make this script executable and launch it to generate a new pair of Certificate and Key for your Apache SSL Virtual Host.
```# chmod +x /usr/local/bin/apache_ssl
cd /usr/local/bin/
# ./apache_ssl
```
Fill it with your information and pay attention to Common Name value to match your server name ```“local.example.com”```. 

 
- After the Certificate and Key are generated, the script will present a long listing of all your Apache SSL pairs stored in ```/etc/httpd/ssl/``` location.

- If the Certificate is not issued by a trusted CA – Certification Authority or the hostname from certificate does not match the hostname who establish the connection, an error should appear on your browser and you must manually accept the certificate.

###### Modify the following file based on your key, crt, root directory and server name as highlighted
File Location: ```/etc/httpd/conf.d/ssl.conf```
 
 
```
Listen 443 https

<VirtualHost securelocal.example.com:443>
 
DocumentRoot "/var/www/html/local.example.com"
ServerName local.example.com:443
 
ErrorLog logs/ssl_error_log
TransferLog logs/ssl_access_log
LogLevel warn
 
SSLEngine on
 
SSLCertificateFile /etc/httpd/ssl/local.example.com.crt

SSLCertificateKeyFile /etc/httpd/ssl/local.example.com.key

<Directory "/var/www/html/local.example.com">
SSLOptions +StdEnvVars
Options Indexes FollowSymLinks MultiViews
AllowOverride All
Order allow,deny
allow from all
</Directory>
</VirtualHost>                                  
```
