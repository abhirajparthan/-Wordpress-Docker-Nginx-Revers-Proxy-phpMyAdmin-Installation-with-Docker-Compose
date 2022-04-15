# Wordpress-Docker-phpMyAdmin-Installation-with-Docker-Compose



WordPress is the most commonly used management system. It's written in PHP, stores data in a MySQL database, and is usually executed behind an Apache website. In this tutorial, I will install WordPress by using multiple docker containers. WordPress itself is in one container and the MySQL database in another container, Also installing the phpMyAdmin container for database management. Then we will install Nginx on the host machine as a reverse proxy for the WordPress container. 

------
### Prerequisites
1. Docker and Docker Compose installed on your server.
2. Need a registered Domain Name for WordPress hosting (My Domain is abhiraj.ga ). 
3. Point your domain to your server IP ( A record ).

------
### Diagram

![Screenshot from 2022-04-11 00-14-55](https://user-images.githubusercontent.com/103326353/162634786-192c9b27-67b2-4325-8338-ab94b2d197a7.png)

------
### Table of Contents
1. Create A project directory for WordPress installation
        - Project directry for holding the YML file, apache config etc..

2. Mysql Container 
        - We need WordPress environment variables such as the MySQL root password, users, and database

3. WordPress Container 
        - We need a WordPress image to create a WordPress website. WordPress image includes the Apache server that we need to execute PHP code.

4. phpMyAdmin Container 
        - Its for viewing database tables and columns. 

5. Reversproxy
        -  Reverse proxy for the WordPress container to communicate with users

6. SSL
        - We already purchase the SSL. ( I have SSL certificate for ny domain "abhiraj.ga" )


Now, let’s get into it!

----
### 1 - Create A project directory for WordPress installation.

My project directory name is "wp-Installation". I have created the project directory using the MKDIR command and navigate to it.

~~~
mkdir wp-Installation && cd wp-Installation
~~~

Next create a docker-compose.yml file for writing the code. This file will setup Wordpress, MySQL & PHPMyAdmin with a single command.

~~~
vi  docker-compose.yml
~~~

Then write the codes in the docker-compose.yaml file.


### 2 - Mysql Container 

Here we are discusing the creation of MySQL container. We are using the MYSQL 5.6 image with container name 'database' and the network 'wpnet'. We are mounted the /var/lib/mysql to the volume "database-mysql". The service definitions for your setup will be in the docker-compose.yml file and service definitions define how each container will run. We are using the version 3 for build.

~~~
version: "3"
       
services:

  database:
     
    image: mysql:5.6
    container_name: database
    networks:
      - wpnet
    volumes:
      - database-mysql:/var/lib/mysql/
    environment:
        
      - MYSQL_ROOT_PASSWORD=t33th123!
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wpuser
      - MYSQL_PASSWORD=wpuser@123
~~~

### 3 - WordPress Container creation  

Here we are discusing the creation of WordPress container. We are using the latest version of wordpress image with container name 'wordpressblog' and the network 'wpnet'. We are Mounted the /var/www/html to the volume "wordpress-data". Other things we alreday defin the "WORDPRESS_DB_HOST=database". This means, in the first step we have created a database container with the tab "database". So the DB host loading from the databse container ( Here we are using the Bridge Network for docker container communication. ).  

~~~
wordpress:
    
    image: wordpress:latest
    container_name: wordpress
    networks:
      - wpnet
    volumes:
      - wordpress-data:/var/www/html/
    environment:
      - WORDPRESS_DB_HOST=database
      - WORDPRESS_DB_USER=wpuser
      - WORDPRESS_DB_PASSWORD=wpuser@123
      - WORDPRESS_DB_NAME=wordpress
    ports:
      - "8080:80"
~~~

### 4 - phpMyAdmin Container Creation.

Here we are discusing the creation of phpMyAdmin container. Its help to handle the database tables and coulms easily. We are using the same network for this containet. Here we added the 2 volumes for mysql and wordpress. the volumes are database-mysql, wordpress-data, 

~~~
  phpmyadmin:

    image: phpmyadmin/phpmyadmin
    container_name: wpphpmyadmin
    restart: always
    networks:
      - wpnet
    ports:
      - '8000:80'
    environment:
      - PMA_HOST:database
      - MYSQL_ROOT_PASSWORD:t33th123!
    depends_on:
      - database      

networks:
  
  wpnet:        
    
volumes:
    
  database-mysql:
  wordpress-data: 
~~~

Now, we have completed the docker-compose.yml file. The file look like this ( I have added the all content in the below snipt )

~~~
version: "3"    
    
services:

  database:
     
    image: mysql:5.6
    container_name: database
    networks:
      - wpnet
    volumes:
      - database-mysql:/var/lib/mysql/
    environment:
        
      - MYSQL_ROOT_PASSWORD=t33th123!
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wpuser
      - MYSQL_PASSWORD=wpuser@123
    
  wordpress:
    
    image: wordpress:latest
    container_name: wordpress
    networks:
      - wpnet
    volumes:
      - wordpress-data:/var/www/html/
    environment:
      - WORDPRESS_DB_HOST=database
      - WORDPRESS_DB_USER=wpuser
      - WORDPRESS_DB_PASSWORD=wpuser@123
      - WORDPRESS_DB_NAME=wordpress
    ports:
      - "8080:80"
    
  phpmyadmin:

    image: phpmyadmin/phpmyadmin
    container_name: wpphpmyadmin
    restart: always
    networks:
      - wpnet
    ports:
      - '8000:80'
    environment:
      - PMA_HOST:database
      - MYSQL_ROOT_PASSWORD:t33th123!
    depends_on:
      - database      


networks:
  
  wpnet:
        
    
volumes:
    
  database-mysql:
  wordpress-data:
~~~ 

----
## Testing Docker compose file "docker-compose.yml"

The YML file is ready to initialize the defined Docker container. Run the following command to set this container.

~~~
docker-compose up -d
~~~
Note: Ensure you are running the command above from the directory where your YML file is located.

This will download all the environs required by WordPress,Mysql, phpMyAdmin installation and you can see the out put like this.


![Screenshot from 2022-04-11 00-42-08](https://user-images.githubusercontent.com/103326353/162635841-61a75bdf-3766-40af-9bbe-9ad079ea7dba.png)


### 5 - Install and Configure Nginx as Reverse Proxy

In this step, we will install the Nginx web server on the host system. We will configure Nginx as a reverse proxy for the Docker container 'WordPress' on port 8080. 

Install and restart the Nginx on the host system.
~~~
yum install nginx

systemctl restart nginx

systemctl enable nginx
~~~

Next, go to the Nginx directory and create a new virtual host configuration for the WordPress container. Here my domain name is abhiraj.ga. You can add your domain as server_name in the below nginx config file.

~~~
cd /etc/nginx/conf.d
vi wordpress.conf
~~~

Paste virtual host configuration below:

~~~
server {
    server_name your_domain_name;
    
    listen 443 ssl; 
    ssl_certificate /var/ssl/certificate.crt;
    ssl_certificate_key  /var/ssl/private.key;


    location /  {
        proxy_set_header X-Real-IP  $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;
       proxy_pass http://3.14.28.205:8080;
    }

}

server {
    if ($host = your_domain_name) {
        return 301 https://$host$request_uri;
    } 

    listen 80;
    server_name your_domain_name;
    return 404;
~~~

Save the file and exit. Also please restart the Nginx service after checking the nginx syntax error. Like this
~~~
nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

systemctl restart nginx
~~~

### 6 - SSL for the domain abhiraj.ga

Here we are using the paid SSL. The SSL certificates are moved to the location "/var/ssl/". I have already defined the SSL file location in the Nginx configuration file ( 5 th step ). Like this

~~~
    ssl_certificate /var/ssl/certificate.crt;
    ssl_certificate_key  /var/ssl/private.key;
~~~

-----
Then, Open your web browser and visit your domain name on the nginx configuration (my domain is Abhiraj.ga ) and you will be redirected to the WordPress installation. Like this.

![Screenshot from 2022-04-11 14-45-57](https://user-images.githubusercontent.com/103326353/162706518-79a3efcf-4300-4b00-9555-e5204d16e1d2.png)

-----
Also you can check the PHPMyAdmin connection using your domain name with port like this http://abhiraj.ga:8000. Here we didn't create a reverse proxy for phpMyadmin, If you want you can set up this same as Nginx conf that I have made in the 5 the step.


![Screenshot from 2022-04-11 15-14-49](https://user-images.githubusercontent.com/103326353/162712534-91bf3a72-8c02-4142-b29e-d8c028087538.png)

-----
Now you can install WordPress. After the installation, we have configured the new theme and plugins for testing.


![Screenshot from 2022-04-11 14-55-29](https://user-images.githubusercontent.com/103326353/162709109-8fc9c907-9fd5-4b14-8795-c5016bdcd04a.png)


Now the wordpress installation complted with docker.


---
## Conclusion

In this tutorial, I have shown you set up a Wordpress Installation with Docker Compose file. Now the WordPress up and running. This is an easier way to set up the WordPress using docker.


---
### ⚙️ Connect with Me

 <p align="center">
<a href="mailto:aparthan275@gmail.com"><img src="https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white"/></a>
<a href="https://www.instagram.com/_r.e.b.e.l.z_33/"><img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/abhiraj-parthan-82038b191"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a> 
<a href="https://www.wppredirect.tk/go/?p=918893532145&m=Abhiraj%20Parthan."><img src="https://img.shields.io/badge/WhatsApp-25D366?style=for-the-badge&logo=whatsapp&logoColor=white"/></a>
  </a></p>
</div>

