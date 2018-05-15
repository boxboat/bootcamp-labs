BoxBoat Boot Camp Labs
=====================

Lab 1.1: Running Your First Docker Container
--------------------------------------------

In this lab, we will examine the Docker CLI and execute our first container.

1. Search Docker Hub using the CLI:

```
docker container ls
docker image ls
docker search hello-world
```
What's an "Official" image?

2. Pull the busybox image from Docker Hub:

```
docker image ls
docker image pull hello-world
docker image ls
docker container ls
```
We have an image! Why do we not have any containers?

3. Run the hello-world Docker image:

```
docker container ls
docker image ls
docker container run hello-world
docker container ls
docker container ls -a
docker image ls
```
Lab 1.2 Terminal Access
---------------------------------------------
1. Create a container using the Ubuntu 16.04 image and connect to STDIN and a terminal:

`docker run -it ubuntu:16.04 bash`

This will run the container, attach to standard input stream, and get a pseudo-terminal. For the container process, we specify bash to get the terminal.

2. Create a file using the touch command:

```
touch test
ls
```
You should see the file created in the root directory of the container. Now exit:
```
exit
```

3. Run the container once again:
```
docker run -it ubuntu:16.04 bash

ls
```
Where did our file go?

Lab 1.3: Docker Container Detached Mode and Exec
------------------------------------------------
In this lab, we will learn how to interact with an already running Docker container.

1. Run the NGINX image (a web server that services http requests external to the host) and connect to STDIN and a terminal:

```
docker container ls
docker container run -it --name nginx nginx
```	
ctrl+c to exit.

2. Run the NGINX image in detached mode:
```
docker container ls
docker container run -d --name nginx nginx
docker container ls
```
Notice the port mapping - port 80 has been mapped to a random port on the host. 
Go to server ip:random port and see the NGINX welcome page.

3. Gain terminal access to detached container:

```
docker container exec -it nginx bash
exit
docker container ls
docker rm $(docker kill nginx)
```

Lab 1.4: Processes On The Host
------------------------------

In this lab, we will look at the processes running on the host

```
ps aux
docker run -d --name nginx nginx
ps aux

docker run -it ubuntu bash
ps aux
```
Are the process ids different? What about users and groups?

Lab 2.1: Intro to Docker Volumes
-----------------------

In this lab, we will learn more about Docker volumes.

1. Host Volumes:

```
docker container run --rm -it -v /tmp:/myloc/tmp --name alpine alpine
touch /myloc/tmp/test.txt
exit
ls /tmp
```

2. Named Volumes:
```
docker volume ls
docker volume create myvolume
docker volume inspect myvolume
docker container run --rm -it -v myvolume:/myloc/tmp --name alpine alpine
touch /myloc/tmp/test.txt
exit
ls /var/lib/docker/volumes/myvolume/_data
```

Lab 2.2: Re-Run Nginx With Content
----------------------------------

In this lab, we will run Nginx with the content of our local host directory.

1. Create a new directory "content" with file "index.html" on your host. Add some test to the file ("hello" maybe?).

2. Run NGINX container using a host volume mount
`docker container run -P -d -v $(pwd)/content:/usr/share/nginx/html nginx`

Did it work? What happenend to the content that was already at the path /usr/share/nginx/html?

Lab 2.3: Intro to Docker Networking
-----------------------------------

In this lab, we will get more practice with Docker networking.

1. Create a user-defined network:
```
docker network ls
docker network create -d bridge test
docker network inspect test
docker network ls
```

2. Put 2 containers on the same bridge network so they can communicate via DNS

```
docker container run --network test -d --name web1 nginx
docker container run --network test -it --name web2 ubuntu bash

$ apt-get update
$ apt-get install -y iputils-ping
$ ping web1
```

Lab 2.4: Exposing and Publishing Ports
--------------------------------------

In this lab, we will learn about exposing and publishing ports.
After you run each container, look at how the ports are exposed.

1. Exposing Ports  
Run NGINX without any port mapping:  
`docker container run -d --name nginx-exposed nginx`  

2. Publishing Ports  
Run Nginx and let Docker choose a high port:  
`docker container run -d -p 80 --name nginx-high nginx`  

3. Run Nginx and choose the high port: 
`docker container run -d -p 8080:80 --name nginx-user nginx`  

4. Run Nginx and let Docker determine exposed ports and mapping: 
`docker container run -d -P --name nginx-auto nginx`

For communication within the network, use the exposed port.
For communication external to the network, use the published port.  

Lab 2.5: Inter-container Communication
-----------------------------------------

Let's take what we've learned in Section 2 apply it in this lab. 

Run an NGINX container and view your custom content from another container (using curl or similar).

Lab 3.1: Build a Simple Python App
----------------------------------

In this lab, we will build a very simple python application.

1. Go to bootcamp-dotcms/simple-python-app

2. Execute the following to build the Docker Image:

`docker build –t python-app:latest .`

Where did our image go?

`docker image ls`

Where is it actually stored on the host?

```
ls /var/lib/docker/image/overlay2/imagedb/content/..somehash../`
ls /var/lib/docker/image/overlay2/layerdb/...somehash.../
````

Lab 3.2: Multi-Stage Builds
---------------------------

In this lab, we will show how Multi-Stage Builds can reduce application size  

Go to to multi-stage-builds directory  
In here, we have 2 subdirectories, small and large  
Each subdirectory contains a dockerfile, let's build them and compare  

Lab 4.1: WordPress
------------------

In this lab, we will launch the WordPress application.  
It contains 2 components: front-end (PHP) and backend (database)  

First, we'll create the proper network:  
	wordpress-net  

Next, we will create 2 volumes:  
	db-data  
	wordpress-data  

Next, we will launch the database container:	
Create a docker container run command with the following configuration:
```
        name: db
	image: mysql:5.7
	volumes: db-data:/var/lib/mysql
	network: wordpress-net
	environment variables:
	  MYSQL_ROOT_PASSWORD: wordpress
      	  MYSQL_DATABASE: wordpress
      	  MYSQL_USER: wordpress
      	  MYSQL_PASSWORD: wordpress
```	

Next, we will launch the WordPress container:
Create a docker container run command with the following configuration:
```
        name: wordpress
	image: wordpress:latest
	volumes: wordpress-data:/var/www/html
	network: wordpress-net
	ports: 8001:80
	environment variables:
	  WORDPRESS_DB_HOST: db:3306
      	  WORDPRESS_DB_PASSWORD: wordpress
```
