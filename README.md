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

## Install and Execution (the Reverse Proxy server side)
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
Follow the [sites_available_example](https://github.com/PeterTsungYu/reverse_proxy/blob/main/sites_available_example.txt).
There are some variable that you should fill in.
- `<your sub.domain.name here>`. It is the domain name you would like to use. Ex. service.goggle.comt.
> No need to prefix with either http or https. Those are part of the configuration.
- `<your page or leave empty as a root page>`. It is the name of your service page or your route, which will appear at the end of your url. After setting up, it will be a complete URL as service.goggle.comt/`<your route>`.
> Leave it empty if it is gonna be a root webpage.
- `<your designated port number>`. It is the port you would like to forward from the server side. It could be one of the available ports on the server side. Ex. 3322, 5333.

5. After editing the route file, execute
```
# soft link and restart
cd /etc/nginx/sites-enabled
sudo ln -s ../sites-available/<your config_file_name> .
sudo systemctl restart nginx

# check status
sudo systemctl status nginx 

# restart gracefully
sudo systemctl reload nginx

# stop the service 
sudo systemctl stop nginx
```

6. (Optional) Make your service SSL-certified. And, it is for free!
```
 # Let's encrypt for https SSL
sudo apt install certbot python3-certbot-nginx
# Then follow the prompts and steps
## (Recommended) choose (2) to enable redirecting http to https

# <your sub.domain.name here>. It is the domain name you would like to use. Ex. service.goggle.comt.
## No need to prefix with either http or https.
sudo certbot --nginx -d <your sub.domain.name here>

# (Recommended) Auto-renew the SSL via crontab
sudo crontab -e 
# Add the below line at the end of the open file (0 分、0 秒、1 日、任何月、任何天（星期幾），也就是不管每個月的一號的 00:00 執行後面的指令)
## Add the following: 0 0 1 * * certbot renew
```
If the certbot is executed successfully, you will find inserted lines managed by certbot in the config file at /etc/nginx/sites-enabled.

## Install and Execution (the service of running machine side)
### Remote ssh tunnel systemd service
1. Generate a ssh key pair
```
ssh-keygen
cat id_rsa.pub
# copy the public key, and place into the remote server

# init the ssh connection to store known_host
ssh <user_name>@<remote_domain/ip>
```

2. 


