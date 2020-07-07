# Docker Nginx Microservices Certbot
Example how to configure nginx to work with docker microservices - vault, apigateway, identityService, grafana, seq and many others

# 1. Connect to VM over SHH
`ssh mr0fka.cloudapp.azure.com`

# 2. Install Nginx
`sudo apt-get update`  
`sudo apt-get install nginx`  

Now we have opened HTTP port on VM and you can see default nginx page when typing `mr0fka.cloudapp.azure.com` in your browser.  
Let’s then create a website configuration in Nginx under `/etc/nginx/sites-available` named as our server `mr0fka.cloudapp.azure.com`.

And paste following default configuration:

```
server {                                                        
  listen 80 default_server;
  listen [::]:80 default_server;
                                                              
  server_name mr0fka.cloudapp.azure.com; 
                                                              
  root /var/www/html;                                   
  index index.html index.htm index.nginx-debian.html;
  
  location / {                                                 
    try_files $uri $uri/ =404;                           
  }                                                            
}
```
OR 

```
server {                                                        
  listen 80 default_server;
  listen [::]:80 default_server;
                                                              
  server_name mr0fka.cloudapp.azure.com; 
                                                              
  root /var/www/html;                                   
  index index.html index.htm index.nginx-debian.html;
  
   # GRAFANA
 location / {
   proxy_pass http://localhost:3000/;                           
}                                                            

 # API Gateway
 location /hive/api/ {
   proxy_pass http://localhost:5000/api/; 
}

 # SEQ
 location /seq/ {
   proxy_pass http://localhost:5341/;
}

 # VAULT
 location /vault/ {
   proxy_pass http://localhost:8200/;

   proxy_set_header Host $host;
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   proxy_set_header Accept-Encoding ""; # needed for sub_filter to work with gzip enabled (https://stackoverflow.com/a/36274259/3375325)

   proxy_redirect /ui/ /vault/ui/;

   sub_filter '<head>' '<head><base href="/vault/">';
   sub_filter '"/ui/' '"ui/';
   sub_filter_once off;
 }

location /v1 {
    proxy_pass http://localhost:8200;
}

 # IDENTITY SERVICE
 location /identity/ {
   proxy_pass  http://localhost:5999/;
   proxy_http_version 1.1;
   proxy_set_header Upgrade $http_upgrade;
   proxy_set_header Connection keep-alive;
   proxy_set_header Host $host;
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   proxy_set_header X-Forwarded-Proto $scheme;
   proxy_cache_bypass $http_upgrade;
}                                                           
}

```
Now we need remove default setup and create link to our configuration file

`sudo rm /etc/nginx/sites-enabled/default`  
`sudo ln -s /etc/nginx/sites-available/mr0fka.cloudapp.azure.com /etc/nginx/sites-enabled/mr0fka.cloudapp.azure.com`  

Next we should reload nginx to apply new configuration using following command:
`sudo service nginx reload`  

# 3. Install Certbot
We will use Certbot with Nginx configuration which is an implementation of the ACME protocol for Letsencrypt.

`sudo apt-get install software-properties-common`  
`sudo add-apt-repository ppa:certbot/certbot`  
`sudo apt-get update`  
`sudo apt-get install python-certbot-nginx`  
`sudo certbot --nginx`  

After running all commands, the wizard will ask for an email address to notify you when certificate is close to expiry and then ask for the server.  
Now if you go to `http://mr0fka.cloudapp.azure.com` it should automiticaly redirect to `HTTPS`.

