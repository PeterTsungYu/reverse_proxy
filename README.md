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
1. Generate a ssh key pair in your running machine
In this project, the RPi 3B+ is used.
```
# in the terminal, enter the command
ssh-keygen

# locate the file of the public ssh key  
## In Rpi, it is at the path of ~/.ssh
cat id_rsa.pub
# Then, copy the public key
```

After acquiring the pub key, place the public key to the server where the key is used as ssh service.
In this project, the public key is configured through the GCP cloud engine.
Once the ssh public key is configured, you could make a ssh connection between the server and the machine.

```
# init the ssh connection to store the known_host
ssh <user_name in the server>@<server_domain/ip>
```
- `<user_name in the server>`. It is the user name used in your server.
> In this project, the name is called "pi" as the RPi 3B+ is used as the machine.
- `<server_domain/ip>`. It is the domain name / public IP of the server. It is recommended that the IP should be a static or permanent IP for further usage. 
> This is a critical step to go through before further configurations. It is going to book the server information into the machine. So that both sides are now known to each other.

> By initializing the ssh connection, the server EDSCA key print will be stored into the known_host file.
This is the crucial part at first. 

> If the service is gonna connecting to a subdomain.domain name, then user should do the ssh through the same ==subdomain.domain name==

>> Otherwise, the local machine is not gonna make any connection with the unknown host. Then the ssh is gonna be blocked by the remote host, even the PING (ICMP) as well. 
>> 
>> To restore from the misconnection, it is better to restart both local and remote machines, and temporarily stop all the connection-wised services. Then, start all over again.

2. Create a service file in the path of /etc/systemd/system (still on the machine side)
> (not in /lib/systemd/system, which is for downloaded packages) 
```
cd /etc/systemd/system
touch <your service file>.service
```
After creating a blank service file, paste the [the example of the service file](https://github.com/PeterTsungYu/reverse_proxy/blob/main/destination_ssh_remote_example.txt) to `<your service file>.service`.
Fill in your variables inside the file.
- /etc/default/`<your env_config file>`. It is a env variable file that it will be configured in the below instructions.
> The env variables will populate all the ${sign}'s that are in the line of defining `ExecStart=`. 
- `<your user_name>`. It is your user name on the machine. It is recommended to have the same name here and there in the server so that it is not confusing.
> In this project, the name is as "pi" since RPi 3B+ is used.

> try restart every 60s (RestartSec=60)
max 20 tries (-o ServerAliveCountMax=20) with an interval of 15s

3. Create a env_variables file in the path of /etc/default
```
cd /etc/default
touch <your env_variable file>
```
After creating a blank file, paste the [the example of the env_variable file](https://github.com/PeterTsungYu/reverse_proxy/blob/main/ssh_remote_env_example.txt) to `<your env_variable file>`.
Fill in your variables inside the file.
- `<path to your ssh public key file>`. It is the path of your ssh public key, which is the one you place into the server to be able to establish the ssh connection. See the [previous](https://github.com/PeterTsungYu/reverse_proxy/tree/readme#remote-ssh-tunnel-systemd-service) first step.
> In this project, the path in RPi 3B+ is at ~/.ssh/id_rsa.pub
- LOCAL_ADDR=localhost. In this project, the service will be located in the machine itself.
- `<your service port number>`. It is the port number that your service is running on the machine. It would typically be the port that a service used.
> Ex. Flask app is running on port 5000.
- `<your remote port number>`. It is the port number that your server is mapping to. 
Previously, the above-mentioned step set a port number in the server. It should be the same port number here to map each other.
So you will have one port number from the server side and one port number from the machine. Those two are mapping to each other.
See the [previous](https://github.com/PeterTsungYu/reverse_proxy/tree/readme#install-and-execution-the-reverse-proxy-server-side) fourth step. 
- `<your remote user_name>`. It is the user_name you used in the server. It is recommended to have the same name here and there in the server so that it is not confusing.
- `<your remote domain name>`. It is the server's domain name / IP. It is recommended that the IP should be a static or permanent IP for further usage.

4. Finally, start the systemctl service. Good luck there!
```
# start the service
sudo systemctl start <your service file>.service

# status
systemctl status <your service file>.service

# stop 
sudo systemctl stop <your service file>.service
```
If execute successfully, you now can access the service running on your machine by visiting the public URL such as `service.goggle.comt/<your route>`.
It will be a HTTPS connection, tho.
