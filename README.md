This guide walks you through using Nginx as a load balancer and proxy server.


What you’ll build
________________
You’ll set up 3 nginx servers, one of them works as a load balancer.


What you’ll need
________________
1. 3 virtual machines (use virtual box or VMware tools)
* Virtual box guide: https://www.nakivo.com/blog/use-virtualbox-quick-overview/
* VMware guide: https://kb.vmware.com/s/article/1018415
* Or any other documentation. I used Ubuntu 20.4 in this project so I suggest you use this distro, in order to come along well with this tutorial.
2. Docker and docker compose installed on your machines.


Set up the project
________________
We have almost similar installation process for each nginx server so do these steps in all of your machines:


1. In your desired directory create 2 directory for our volumes. First one for the nginx.conf file that I named it “nginx_home”. Second one for our data or the index.html file which I named it “templates”. 


2. You can edit your index.html file as you wish. Mine is “Hello from 192.168… ” for one of my machines.


3. You have to create a nginx.conf file and copy it into the nginx_home directory. You can use default nginx.conf which exists on the internet or you can use mine in this repository.


4. Create your docker-compose.yml file for installing nginx in the same directory. You can see docker compose files in all of the directories in this repository (they are similar).
I used the latest version of nginx image on port 80 and then mounted the local src directories to the /etc/nginx/nginx.conf and /etc/nginx/templates inside the container.
If you have another service on your 80 port, use another port like 81 or … .

5. Now run this command in your docker compose file directory:

        $ docker-compose up -d


If everything goes well, you can search localhost:<port> in the browser of each machine and see nginx server is up and running.
Now as you used volumes in your docker-compose file, you can change nginx.conf and index.html files as you wish and restart your docker container in order to see changes.
         
        $ docker restart nginx

As I wanted to use one of my nginx servers as a load balancer, I added some lines to the http context of the nginx.conf file of my load balancer machine like this : 

       http {        
       # we use upstream context for load balancing
        upstream backend {
                server 192.168.88.127;        #first nginx server running on port 80
                server 172.16.30.16;            #second nginx server running on port 80
        }
        server {
                listen 80;
                server_name localhost;
     # this location uses backend upstream we made and do round robin 
                location / {
                        proxy_pass http://backend;  
                }
                location /nginx192 {
                        proxy_pass http://192.168.88.127/;
                }
                location /nginx172 {
                        proxy_pass http://172.16.30.16/;
                }
        }}


Now If you search http://localhost:/nginx172 in your browser in the load balance machine you get the result of 172.16.30.16 machine.
	-----> hello from 172.16...

And If you search http://localhost:/nginx192 in your browser in the load balance machine you get the result of 192.168.88.127 machine.
	-----> hello from 192.168....

And If you search http://localhost in your browser in the load balance machine you get the result of 172.16.30.16 machine and 192.168.88,127 machine randomly (round robin).
