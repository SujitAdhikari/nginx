Installing Nginx Web Server on CentOS/RHEL8/AlmaLinux/Rocky Linux:
---------------------

[root@server ~]# dnf module install nginx

Enable and start Nginx service.
--------------------------------
[root@server ~]# systemctl enable --now nginx.service

Allow HTTP and HTTPS services in Linux firewall.
--------------------------------
[root@server ~]# firewall-cmd --permanent --add-service={http,https}

Generate Self-Signed SSL/TLS certificate:
---------------------------------
 [root@server ~]# openssl req -newkey rsa:4096 -nodes -keyout /etc/pki/tls/private/test.key -x509 -days 365 -out /etc/pki/tls/certs/test.crt 

To improve the SSL/TLS security by ensuring a secure cryptographic key exchange, generate Diffie-Hellman (DH) keys parameters.

[root@server ~]# openssl dhparam -out /etc/pki/tls/certs/test.pem 4096


NOTE: If you want to redirect HTTP traffic to HTTPS, you can simply add the line below under the Nginx HTTP configuration section.
return 301 https://$host$request_uri;

 vim /etc/nginx/conf.d/test.com.conf

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
       # ssl_ciphers EECDH+AESGCM:EDH+AESGCM;
 	 ssl_ciphers PROFILE=SYSTEM;
       # ssl_ecdh_curve secp384r1;
        ssl_prefer_server_ciphers on;
       # ssl_session_tickets off;
       # resolver 8.8.8.8 valid=300s;
       # resolver_timeout 5s;
       # add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
       # add_header X-Frame-Options DENY;
       # add_header X-Content-Type-Options nosniff;
       # add_header X-XSS-Protection "1; mode=block";
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

If you are using the certificates from CA, you will be provided with two certificate files, the Intermediate certificate and the server certificate. To use them, you need to put them together in a single certificate file.

cat server.crt intermediate.crt >> /etc/pki/tls/certs/ser-int-cert.crt

Ref:
https://kifarunix.com/configure-nginx-with-ssl-tls-certificates-on-centos-8/

Installing a CA Signed SSL/TLS Certificate in Nginx:
https://www.centlinux.com/2020/07/install-ssl-tls-certificates-nginx-web-server.html
