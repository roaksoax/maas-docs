Title: SSL
TODO:  Critical: review required
       modify Apache redirection wording if this redirection is ever removed


# SSL

MAAS doesn't support SSL natively and needs to be enabled in web server
software independently (e.g. Apache, Nginx) which users access directly.

To enable SSL for endusers, you can use example configurations:

## nginx

```
server {
 listen 443 ssl;

 server_name _;
 ssl_certificate /etc/nginx/ssl/nginx.crt;
 ssl_certificate_key /etc/nginx/ssl/nginx.key;

 location / {
  proxy_pass http://localhost:5240;
  include /etc/nginx/proxy_params;
 }

 location /MAAS/ws {
  proxy_pass http://localhost:5240/MAAS/ws;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "Upgrade";
 }

}
```


## Apache2

```
<VirtualHost *:443>
 SSLEngine On

 SSLCertificateFile /etc/apache2/ssl/apache2.crt
 SSLCertificateKeyFile /etc/apache2/ssl/apache2.key

 RewriteEngine On
        RewriteCond %{REQUEST_URI} ^/MAAS/ws [NC]
        RewriteRule /(.*) ws://localhost:5240/MAAS/ws [P,L]

        ProxyPreserveHost On
        ProxyPass / http://localhost:5240/
        ProxyPassReverse / http://localhost:5240/
</VirtualHost>
```

Once the above is done, change the MAAS URL:

```bash
sudo maas-region local_config_set --maas-url https://localhost:5240/MAAS
sudo systemctl restart maas-regiond
```

Note that currently Apache is employed for redirecting TCP port 80 to 5240.
This may be removed in future versions of MAAS.
