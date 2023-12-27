# Load Balancing with Nginx 

in this project I am to configure a webserver that will be used as a load balancer

#### Step 1

I created three EC2 instances one to be used as a load balancer and the two others to be used by the load balancer.

to do so I first created two other servers namely 

`server load 1`

`server load 2`

and I configured them as follows

I opened port 8000 to allow traffic from anywhere 

I opened the instance summary of the `server load 1` as shown below

![server load 1 instance summary](<img/4 instance summary.png>)

next, I clicked on the security tab as seen below

![security tab](<img/5 security groups option.png>)

after I clicked on the security groups to open another webpage where I clicked on the following steps 

`"actions" >> "edit inbound rules"` 

as shown below

![edit inbound rules](<img/edit inbound rules.png>)


I added a new rule and chose the following parameters 

`custom TCP`
`port = 8000`
`IPV v4`

as shown below 

![add port 8000 ](<img/6 add port 8000 to security groups.png>)

Next, I ssh into my server through my terminal

as shown below

![connect to server instance](<img/7 connect to server instance.png>)


next, I installed apache using the command below 

`sudo apt update -y &&  sudo apt install apache2 -y
`

![install apache sotware](<img/8 install apache webserver.png>)

next, I verify the apache is running using the code below

`sudo systemctl status apache2`

![verify apache is running](<img/9 verify apache is runnng correctly.png>)

The next step is for me to configure the apache to serve the webpage on port `8000`

I did so using the command below

`sudo vi /etc/apache2/ports.conf`

![add port 8000](<img/10 add port 8000.png>)

next is to open file `000-default.conf` and change virtual host port form `80` to `8000` using the command below

`sudo vi /etc/apache2/sites-available/000-default.conf`

![change virtual port to 8000](<img/11 change virtual port to 8000.png>)

next, I restarted apache to load the new configuration using the code below

`sudo systemctl restart apache2`

![restart apache](<img/12 restart apache.png>)

next, I opened a new file `index.html` using the code below

`sudo vi index.html`

![create a new index file](<img/13 crate a new index file.png>)

after switching to **vi** editor I typed the code below

>
         <!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my EC2 instance</h1>
            <p>Public IP: YOUR_PUBLIC_IP</p>
        </body>
        </html>


![type code](<img/14 code with my EC2 public ip addess.png>)

next, I changed the file ownership of the index.html file using the code below

`sudo chown www-data:www-data ./index.html`

![change file ownership](<img/15 change file ownership of the index file.png>)

next, I override the default html file of Apache Webserver suing the code below

`sudo cp -f ./index.html /var/www/html/index.html`

![replace the default html file](<img/16 replace the default index file with the new one created.png>)

After I restart the webserver to load the new configuration using the command below

`sudo systemctl restart apache2`

![restart the apache webserver](<img/17 restart the apache webserver.png>)

The output of the IP address on a web browser is seen below

![web browser output](<img/18 web browser display of instance.png>)


### At this stage I will repeat this process for the second web server from configuring the inbound rule to restarting the webserver after replacing the old index file with the new one

after configuring the second server I moved on to the third which is my ___LOADBALANCER___

First of all, I configured the inbound rules with the following setting 
`custom tcp`
`port 80`
`anywhere ipv4`

![all traffic allowed](<img/19 load balancer port 80 is open to recieve traffic.png>)

next, I installed Nginx using the code below

`sudo apt update -y && sudo apt install nginx -y`

![install nginx](<img/20 install nginx in loadbalancer.png>)

next, I verified that Nginx is running using the code below

`sudo systemctl status nginx`

![verfy nginx is running](<img/21 verify nginx is running.png>)

next, I opened Nginx configuration file using the code below

`sudo vi /etc/nginx/conf.d/loadbalancer.conf`

![edit nginx configuration file](<img/22 code to open nginx configuration file.png>)

next I typed the code below

>         
        upstream backend_servers {

            # your are to replace the public IP and Port to that of your webservers
            server 127.0.0.1:8000; # public IP and port for webserser 1
            server 127.0.0.1:8000; # public IP and port for webserver 2

        }

        server {
            listen 80;
            server_name <your load balancer's public IP addres>; # provide your load balancers public IP address

            location / {
                proxy_pass http://backend_servers;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
        }
    
![code configuration Nginx](<img/23 nginx code configuration.png>)


next, I test my configuration using the code below

`sudo nginx -t`

![code configuration Nginx test](<img/24 test nginx code.png>)

![code configuration Nginx 2](<img/25 nginx test code is ok.png>)


After I restart nginx to load the new configuration using the command below

`sudo systemctl restart nginx`

![restart nginx server](<img/26 restart nginx server.png>)

after this step, I pasted the public address of my Nginx server on my browser and it gave the output below

![loadbalancing successful](<img/27 load balancing successful.png>)

continuous refreshing the the webpage will output the IP address of the two web servers 








