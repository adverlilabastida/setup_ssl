# Apache 2.0.58 Install from Source

Reference: *[http://www.crucialp.com/resources/...](http://www.crucialp.com/resources/tutorials/server-administration/apache-2-php-4-5-mod_php-linux-redhat-freebsd-apache2-php4-php5/)*

1. Download apache source tarball
    ```
    wget https://archive.apache.org/dist/httpd/httpd-2.0.58.tar.gz
    ```
    
1. Extract ```httpd-2.0.58.tar.gz```
    ```
    tar -xvf httpd-2.0.58.tar.gz && cd httpd-2.0.58
    ```

1. Configure apache source
    ```
    ./configure --prefix=/home/apps/apache --enable-mods-shared=all --enable-ssl
    ```
    
1. Run make and make install
    ```
    make && make install
    ```

1. Create apache binaries symbolic link
    ```
    ln -s /home/apps/apache/bin/apachectl /usr/bin
    ln -s /home/apps/apache/bin/httpd /usr/bin
    ```

1. Finally run apache
    ```
    httpd -k start
    ```


# Generate self-signed ssl certificate

1. Make sure to have openssl installed if you already have skip this step
    ```
    yum -y install openssl
    ```

1. Generate private key
    ```
    openssl genrsa -out cafe24.key 2048
    ```

1. Generate signing-request
    > Note: Fillup the inputs accordingly
    ```
    openssl req -new -key cafe24.key -out cafe24.csr
    ```
    ![ssl](https://user-images.githubusercontent.com/5668664/35431048-e5215610-02b5-11e8-90f8-7009853848b3.gif)

1. Generate self-signed key
    ```
    openssl x509 -req -days 365 -in cafe24.csr -signkey cafe24.key -out cafe24.crt
    ```

1. Copy your self-signed certificate to ```/etc/pki/tls/certs```
    ```
    cp *.crt /etc/pki/tls/certs
    ```
    
1. Copy your certificate key to ```/etc/pki/tls/private```
    ```
    cp *.key /etc/pki/tls/private
    ```
    

# Creating apache's virtualhost(s)
1. ```prepend``` the following line to your ```/home/apps/apache/conf/httpd.conf```
    ```
    Include sites/*.conf
    ```

1. Create the ```sites``` directory
    ```
    mkdir /home/apps/apache/sites
    ```

1. Create ```cafe24.conf``` inside your ```/home/apps/apache/sites```
    touch /home/apps/apache/sites/cafe24.conf

1. Update ```cafe24.conf``` accordingly
    > Not necessarily same as below but this should work
    ```
    Listen 443

    <VirtualHost *:443>
        SSLEngine on
        # Make sure this cert exists
        SSLCertificateFile /etc/pki/tls/certs/cafe24_cloud.crt
        # Make sure this cert key exists
        SSLCertificateKeyFile /etc/pki/tls/private/cafe24_cloud.key
        ErrorLog /home/cafe24corp_error.log
        # Make sure this directory exists
        DocumentRoot /home/hosting/program/www
        ServerName cafe24.dev
        ServerAlias www.cafe24.dev
        <Directory "/home/hosting">
            Options +Indexes +MultiViews
            AllowOverride All
        </Directory>
    </VirtualHost>
    ```

1. Update your windows hostfile ```%windir%/system32/drivers/etc/hosts``` with your virtualbox ip ```192.168.56.nn cafe24.dev www.cafe24.dev```
    ```
    192.168.56.nn cafe24.dev www.cafe24.dev
    ```

1. Add php handler to your ```/home/apps/apache/conf/httpd.conf```
    ```
    ScriptAlias /handler/ /usr/bin/
    AddHandler php5214 .php
    Action php5214 /handler/php-cgi
    ```

1. Restart your apache and your good to go
    ```
    httpd -k restart
    ```
