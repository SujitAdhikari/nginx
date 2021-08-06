# nginx
NGINX (Web Server,Reverse proxy,Load balancer)

#### Package Installation: 
[sujit@server ~]$ sudo  yum module list nginx
[sujit@server ~]$ sudo yum module enable nginx:stream_version
[sujit@server ~]$ sudo dnf install nginx

#### Start and enable the newly-installed nginx service.
[sujit@server ~]$ sudo systemctl enable --now nginx

#### Confirm the nginx service is up and running:
[sujit@server ~]$ sudo systemctl status nginx


#### Modify the firewall:
```
[root@utility ~]# firewall-cmd --permanent --add-service=http
success
[root@utility ~]# firewall-cmd --permanent --add-service=https
success
[root@utility ~]# firewall-cmd --reload
success
```
#### Configure the server:
Host file(DNS Entry):  
Vim /etc/hosts 
```
192.168.10.135 example.com
```

#### Verify the installation:
$ sudo nginx -v
nginx version: nginx/1.6.3

#### Create Index.html file:
```
sudo mkdir -p /var/www/example.com/html
sudo chown -R $USER:$USER /var/www/example.com/html
vim /var/www/example.com/html/index.html
```
```
<html>
    <head>
        <title>Welcome to Example.com!</title>
    </head>
    <body>
        <h1>Success!  The Example.com is working!</h1>
    </body>
</html>
```

#### Update server’s SELinux security contexts:
```
semanage fcontext -a -t httpd_sys_content_t "/var/www/example.com/html/(/.*)?"
restorecon -Rv /var/www/example.com/html/
```
#### Backup Original config file:
```
cp -v /etc/nginx/nginx.conf /etc/nginx/nginx.conf.original
vim /etc/nginx/nginx.conf 
```
``` 
server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  example.com;
        root         /var/www/example.com/html;
```
#### Setting up virtual domain/host:
```
sudo useradd -d /home/vhostusr -m -k /dev/null -s /usr/sbin/nologin vhostusr
passwd -l vhostusr

mkdir /var/www/vhost1.com/html/ -p
semanage fcontext -a -t httpd_sys_content_t "/var/www/example.com/html/(/.*)?"
restorecon -Rv /var/www/example.com/html

mkdir /var/www/vhost1.com/html/ -p
semanage fcontext -a -t httpd_sys_content_t "/var/www/vhost1.com/html/(/.*)?"
restorecon -Rv /var/www/vhost1.com/html

mkdir /var/www/vhost2.com/html/ -p
semanage fcontext -a -t httpd_sys_content_t "/var/www/vhost2.com/html/(/.*)?"
restorecon -Rv /var/www/vhost2.com/html

vim /etc/nginx/conf.d/vhost1.com.conf
```
```
server {
   ## Listen to TCP port 80 ##
        listen 80;
        listen [::]:80;
 
   ## Set document web root to directory ##
        root /var/www/vhost1.com/html;
        index index.html;
 
   ## Set virtual domain/host name here ##
        server_name vhost1.com www.vhost1.com;
 
   ## Set default access and error log file ##
        access_log  /var/log/nginx/vhost1.com_access.log  main;
        error_log /var/log/nginx/vhost1.com_error.log;
 
   ## Set default error ##
        location / {
                try_files $uri $uri/ =404;
        }
}
```

vim /etc/nginx/conf.d/vhost2.com.conf
```
server {
   ## Listen to TCP port 80 ##
        listen 80;
        listen [::]:80;

   ## Set document web root to directory ##
        root /var/www/vhost2.com/html;
        index index.html;

   ## Set virtual domain/host name here ##
        server_name vhost2.com www.vhost2.com;

   ## Set default access and error log file ##
        access_log  /var/log/nginx/vhost2.com_access.log  main;
        error_log /var/log/nginx/vhost2.com_error.log;

   ## Set default error ##
        location / {
                try_files $uri $uri/ =404;
        }
}
```
#### Creating Sample Pages for Each Site:
vhost1.com:

vim /var/www/vhost1.com/html/index.html
```
<html>
    <head>
        <title>Welcome to vhost1.com!</title>
    </head>
    <body>
        <h1>Success!  The vhost1.com server block is working!</h1>
    </body>
</html>
```
vhost2.com:
```
vim /var/www/vhost1.com/html/index.html

<html>
    <head>
        <title>Welcome to vhost1.com!</title>
    </head>
    <body>
        <h1>Success!  The vhost1.com server block is working!</h1>
    </body>
</html>
```
```
 chmod -R 0555 /var/www/example.com/html
 chmod -R 0555 /var/www/vhost1.com/html/
 chmod -R 0555 /var/www/vhost2.com/html/
 
 chown -R vhostusr:vhostusr /var/www/example.com/html
 chown -R vhostusr:vhostusr /var/www/vhost1.com/html/
 chown -R vhostusr:vhostusr /var/www/vhost2.com/html/
```
#### DNS Server Entry:

[root@utility conf.d]# vim /etc/hosts
```
192.168.10.155 vhost2.com
192.168.10.155 vhost1.com
192.168.10.155 example.com
```

 $ sudo nginx -t
 If not erros means reload the Nginx server:
 $ sudo nginx -s reload
## or ##
 $ sudo systemcl reload nginx

**Verify:**
```
$ curl example.com
$ curl vhost1.com
$ curl vhost2.com
```
#### Enable HTTPS:
---------------------
Generate Self-Signed SSL/TLS certificate:
---------------------------------
[root@server ~]# openssl req -newkey rsa:4096 -nodes -keyout /etc/pki/tls/private/test.key -x509 -days 365 -out /etc/pki/tls/certs/test.crt

NOTE: To improve the SSL/TLS security by ensuring a secure cryptographic key exchange, generate Diffie-Hellman (DH) keys parameters.

[root@server ~]# openssl dhparam -out /etc/pki/tls/certs/test.pem 4096

NOTE: If you want to redirect HTTP traffic to HTTPS, you can simply add the line below under the Nginx HTTP configuration section.
return 301 https://$host$request_uri;

 vim /etc/nginx/conf.d/test.com.conf
```
server {
    listen 80;
    server_name test.com www.test.com;
    #Redirect HTTP to HTTPS site
    return 301 https://test.com$request_uri;
}


# Settings for a TLS enabled server.
#
server {
        listen       443 ssl http2 default_server;
        listen       [::]:80;
        server_name  test.com;
        root         /test/html;
        ssl_protocols TLSv1.3; # Enable TLS v1.3 only
        ssl_certificate "/etc/pki/tls/certs/test.crt";
        ssl_certificate_key "/etc/pki/tls/private/test.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
 	ssl_ciphers PROFILE=SYSTEM;
        ssl_prefer_server_ciphers on;
       # Add DH parameters
        ssl_dhparam /etc/pki/tls/certs/test.pem;

        access_log  /var/log/nginx/test.com_access.log  main;
        error_log /var/log/nginx/test.com_error.log;

        include /etc/nginx/default.d/*.conf;
        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
}
```

#### Load Balancer:
--------------------------
 [root@utility ~]# cat /etc/nginx/conf.d/domain.exampel.com.conf
```
upstream servers {
        server srv1.example.com;
        server srv2.example.com;
    }

    server {
        listen      80 default_server;
        listen      [::]:80 default_server;
        server_name domain.example.com www.domain.example.com;

        location / {
                proxy_redirect      off;
                proxy_set_header    X-Real-IP $remote_addr;
                proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header    Host $http_host;
                proxy_pass http://servers;
        }
}

```

 $ sudo nginx -t
 If not erros means reload the Nginx server:
 $ sudo nginx -s reload
 ## or ##
 $ sudo systemcl reload nginx

#### Verify:
 Verify that TCP port 80 or 443 opened using ss command command:
 $ sudo ss -tulpn | grep 80

Reverse proxy on another server:
---------------------------------
[root@srv ~]# vim /etc/nginx/conf.d/example.com.conf
```
upstream backend2 {
        server google.com;
    }

upstream backend3 {
        server 192.168.10.149;
    }

    server {
        listen      80;
        listen      [::]:80;
        server_name example.com example.akij.com;

        location /app {
                proxy_redirect      off;
                proxy_set_header    X-Real-IP $remote_addr;
                proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header    Host $http_host;
                proxy_pass http://backend2;
        }


        location / {
                proxy_redirect      off;
                proxy_set_header    X-Real-IP $remote_addr;
                proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header    Host $http_host;
                proxy_pass http://backend3;
        }
```
#Monitoring:


#Additional Command:
```
Search and info for Nginx package:
$ sudo yum search nginx
$ sudo yum list nginx
$ sudo yum info nginx

$ sudo yum module list nginx
$ sudo yum module enable nginx:1.16
## verify it version set to 1.16 ##
$ sudo yum module list nginx

$ sudo systemctl reload nginx ## <-- reload the server ##
```

Troubleshooting:

Reference:  
 https://linuxhint.com/install_nginx_centos8/  
 https://www.cyberciti.biz/faq/how-to-install-and-use-nginx-on-centos-8/#Testing  
 https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/  
