
# Docker — Building an environment with multiple PHP versions on a single Nginx server

A practical use case based in Nginx ports

![Docker | Nginx | PHP — Photo by [Scott Rodgerson](https://unsplash.com/es/@scottrodgerson?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/es/fotos/un-monton-de-cables-azules-conectados-entre-si-PSpf_XgOM5w?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)](https://cdn-images-1.medium.com/max/9014/1*pGnB8GwfjYSvHqQCx6_OXQ.png)*Docker | Nginx | PHP — Photo by [Scott Rodgerson](https://unsplash.com/es/@scottrodgerson?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/es/fotos/un-monton-de-cables-azules-conectados-entre-si-PSpf_XgOM5w?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)*

We often work with systems that run on different software versions. Therefore, we need tools that allow us to compile / interpret / execute our code according to the context where we are.

In this article we will see how to create a development environment with Docker that will allow:

* Use multiple PHP containers with different versions (in this guide we will install FPM 7.4 and 8.2).

* Configure an Nginx container as a web server to redirect requests to the PHP container that should act as interpreter based on the URL and port.

👉* I will explain everything in detail, but I understand that you know Docker and how it works.*

### Getting started with Docker 🐳

You need to have Docker installed on your machine. After that, if you don’t have a Docker Desktop environment, you can use [Portainer](https://www.portainer.io/) for quick management of all your containers.

### Docker Compose 📑

Docker Compose allows you to manage and define multi-container stacks using a YAML file called *docker-compose.yml*. In this file we will specify the images, ports, volumes and other instructions for each container.

    version: "3.9"
    services:
      nginx:
        image: nginx:alpine    
        ports:
          - "8001:80"
          - "8002:82"
        volumes:
          - ./sites/:/var/www/html/
    
      php74:
        image: php:7.4-fpm
        volumes:
          - ./sites/:/var/www/html/
    
      php82:
        image: php:8.2-fpm
        volumes:
          - ./sites/:/var/www/html/
    
    networks:
      multi_php_nginx:
    
    name: multi_php_nginx

* Nginx will be configured to listen on ports 80 and 82, and based on them it will redirect the requests to the corresponding PHP container.

* Each container will be mapped to the */sites/* folder, which will contain the directories corresponding to each website:
> *./sites/
— site_74/index.php*
> *— site_82/index.php*

### Running containers ➡️

To create and start the containers, open a terminal inside the folder containing the *docker-compose.yml* file and run the following command:

    $ docker compose up -d

Finally, we have our stack of containers ready:

![Containers view on Portainer](https://cdn-images-1.medium.com/max/2606/1*oBG_r3ilNL2ZeY8hw_QiTw.png)*Containers view on Portainer*

### Nginx configuration ⚡

We already have active in our *localhost* the Nginx service listening on the defined ports. We only need to modify the server configuration to redirect to the correct folder and PHP container.

You can run commands in a running container by executing the following command in your local terminal:

    $ docker exec -it -u root multi_php_nginx-nginx-1 sh

Let’s edit the Nginx default configuration file:

    $ nano /etc/nginx/nginx.conf

    user  nginx;
    worker_processes  auto;
    
    error_log  /var/log/nginx/error.log notice;
    pid        /var/run/nginx.pid;
    
    events {
        worker_connections  1024;
    }
    
    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;
    
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';
    
        access_log  /var/log/nginx/access.log  main;
    
        sendfile        on;
        keepalive_timeout  65;
    
        include /etc/nginx/conf.d/*.conf;
    }

We need to add the following code inside *http *block:

    # PHP 7.4 service
    server {
        listen 80;
        server_name localhost;
        root /var/www/html/site_74/;
        index index.php index.html index.htm;
    
        location ~ \.php$ {
            fastcgi_pass php74:9000;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }
    }
    
    # PHP 8.2 service
    server {
        listen 82;
        server_name localhost;
        root /var/www/html/site_82/;
        index index.php index.html index.htm;
    
        location ~ \.php$ {
            fastcgi_pass php82:9000;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }
    }

Now save the changes and restart the Nginx service.
> Remember that you can also map the configuration file within Docker Compose.

### How it works ❓

When you make a request to *localhost:8001*, Nginx will listen on port 80 and define the* /site_74/* folder as the root web directory.

Also, the *fastcgi_pass php74:9000* line will specify the location of the PHP service to which Nginx will send requests for processing.

The same thing will happen when calling to *localhost:8002* and the PHP 8.2 service.

### Conclusión ✅

Obviously, there are many ways to do this. Probably better (and also worse), but it’s a way to see how we can develop under different versions of PHP.

I hope I have explained myself well and that it has been useful to you.

Feel free to contact me if you have any questions or another solution😄.

You can follow me on [LinkedIn](https://www.linkedin.com/in/josegarciarodriguez/) and [GitHub](https://github.com/m4n50n).

Thank you very much for coming here.
