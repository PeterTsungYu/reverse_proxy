# reverse_proxy
Nginx is a web server (for HTTP/HTTPS or others).
It could serve as a web server or a reverse proxy.
A Reverse Proxy is to add a layer between the host and clients. The info of the host will be hidden behind the reverse proxy. Ex. Hostname, Host IP, Host info, ….
A reverse proxy could map the domain name to a certain port of the host.

To put it in a nutshell, a reverse proxy is a server to manage several devices behind the server itself. 
In this project, a Google Cloud Machine (a GCP compute engine, to be exact) is used as the reverse proxy Nginx server. 
It manages all the devices, which are a handful of Rasberry Pis in this project, to render webpages of running services to the requesting users. 
One benefit is that you could access those web services via your designated domain names. 
The other benefit is that even though the service is exposed, the principle of the reverse proxy is to hide all the information of machines behind the proxy.

## Install and Execution
1. Setup a DNS to get a domain name for reverse proxy
A domain name is a premise. One should be able to setup a cloud server or any similar kind with a public domain name. 
In this project, a cloud server is created in the GCP (Google Cloud Platform), and a domain name is registered in GoDaddy.

2. Install nginx on the server
```
# install nginx
sudo apt update
sudo apt upgrade
sudo apt install nginx
```

3. Edit nginx route file inside the folder of /etc/nginx/sites-available 
```
cd /etc/nginx/sites-available
touch <your config_file_name>
```

4. Write into the config file you just created
Follow the 

# soft link and restart
$ cd /etc/nginx/sites-enabled

# locate in /etc/nginx/sites-enabled
$ sudo ln -s ../sites-available/<config_name> .
$ sudo service nginx restart
(or) sudo systemctl restart nginx 
# check status
$ sudo systemctl status nginx 

# Let's encrypt for https SSL
$ sudo apt install certbot python3-certbot-nginx
# choose (2) to enable redirecting http to https

$ sudo certbot --nginx -d rvproxy.fun2go.energy

# auto-renew the SSL via crontab
$ sudo crontab -e 
# add below to the final line, 0 分、0 秒、1 日、任何月、任何天（星期幾），也就是不管每個月的一號的 00:00 執行後面的指令
# 0 0 1 * * certbot renew
```

