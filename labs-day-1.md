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
Review your image and container listings before and after running the `hello-world` image. How have things changed?

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

Lab 1.5: Attach and Detach
------------------------------

In this lab, we will learn to attach and detach from running containers' stdout

1. Run Centos image in detached mode:

```
docker container run -d centos ping 8.8.8.8 -c 50 
```

2. Attach to the newly created container and observe the output:

```
docker container attach <container id>
```
	
3. Try to detach from the container by entering CTRL+P+Q at once. 
Nothing happens...exit using CTRL+C.

4. Check your running containers and notice that your container is no longer running:

```
docker container ls
```
The CTRL+C combination killed the container process.

5. Run another container, but this time with `-it` passed:

```
docker container run -d -it centos ping 8.8.8.8 -c 50
```

6. Attach and detach from container:

```
docker container attach <container id>
```
CTRL+P+Q

7. Show your container still running in the background:

```
docker container ls
```

Lab 1.6: Start and Stop
------------------------------

In this lab, we will learn how to start and stop containers.

1. Run Tomcat:7 image in detached mode:

```
docker container run -d tomcat:7
```

2. Follow the logs of the newly created container and observe the output:

```
docker container logs -f <container id>
```
You should see output of Tomcat server starting up.
(CTRL+C)
	
3. Stop the container

```
docker container ls
docker container stop <container id>
```

4. Start the container and attach to stdout:

```
docker container start -a <container id>
```
We're back up and running! You will see Tomcat server go through its startup routine again. 
Bonus: We saved our filesystem, but not memory and running processes. Reasearch `docker checkpoint`

5. Detach and exit from container using CTRL+C

6. Start container again, but without `-a`:

```
docker container start <container id>
```
Your container is now running again, but this time in the background.
```
docker container ls
```

Lab 2.1: Docker Host Mounts
-----------------------

In this lab, we will learn how to use host mounts as a volume.

1. Run a container with attached host volume:

```
docker container run --rm -it -v /tmp:/myloc/tmp --name mycontainer alpine
```
The `-v` flag is for volume. What are the other new flags?

2. Create a temporary file in the container and exit:

```
touch /myloc/tmp/test.txt
exit
```

3. View the contents of `test.txt` on the host filesystem:

```
ls /tmp
```

Lab 2.2: Docker Named Volumes
-----------------------

In this lab, we will learn how to use Docker Named volumes.

1. View available Docker Volumes on your host:

```
docker volume ls
``` 

2. Create a new Named volume:

```
docker volume create myvolume
```

3. Inspect your new volume:

```
docker volume inspect myvolume
```
Take note the `"Mountpoint"`.

4. Run a new container with your Named volume mounted: 

```
docker container run --rm -it -v myvolume:/myloc/tmp --name mycontainer alpine
```

5. Create a temporary file in the container and exit:

```
touch /myloc/tmp/test.txt
exit
```

6. View the contents of Volume "myvolume" on the host:

```
ls <mountpoint from step 6>/volumes/myvolume/_data
```

Lab 2.3: Re-Run Nginx With Content
----------------------------------

In this lab, we will run Nginx with the content of our local host directory.

1. Create a new directory "content" with file "index.html" on your host. Add some text to the file ("hello" maybe?).

2. Run NGINX container using a host volume mount
`docker container run -P -d -v $(pwd)/content:/usr/share/nginx/html nginx`

Did it work? What happenend to the content that was already at the path /usr/share/nginx/html?

Lab 2.4: Exploring the Default Bridge Network
-----------------------------------

1. Make sure there are no existing containers running:

`docker container rm $(docker container ls -qa)`

2. Launch a Centos container called "centos1" in detached mode:

```
docker container run -d - it --name centos1 centos
```

3. Inspect the bridge network and note the container IP address:

```
docker network inspect bridge
```
If you look at the "Containers" filed you will find the IPv4 address.

4. Launch another Centos container called "centos2" and get terminal access:

```
docker container run -it --name centos2 centos bash
```

5. Ping your "centos1" container using the IP address from Step 3.

```
ping <ip address>
```

6. Try and ping "centos1" using the container name:

```
ping centos1
```
It doesn't work!

Lab 2.5: Create a bridge network
-----------------------------------

1. Create a new bridge network called "my_bridge":

```
docker network create --driver bridge my_bridge
```

2. Verify and inspect your new network:

```
docker network ls
docker network inspect my_bridge
```

3. Launch a Centos container in the background on your "my_bridge" network named "centos3":

```
docker container run -d -it --network my_bridge --name centos3 centos
```

4. Launch another Centos container on your "my_bridge" network and connect to it:

```
docker container run -it --network my_bridge --name centos4 centos
```

5. Open the `/etc/hosts` file:

```
cat /etc/hosts
```
Do you see centos3? Why didn't we in the previous exercise?

6. Ping "centos3" using the container name:

```
ping centos3
```

7. Ping "centos1" and "centos2" using the container name:

```
ping centos1
ping centos2
```

Lab 2.6 Manual Port Mapping
--------------------------------------

In this lab, we will learn about manually exposing and publishing ports.

1. Exposing Ports  
Run NGINX without any port mapping:  
```
docker container run -d --name nginx-exposed nginx
```

2. Show nginx exposed port 80:

```
docker container ls -a
```
Why can't we access it?

3. Run an nginx container and map port 80 on the container to port 8080 on your host. Map port 443 on the container to port 4443 on the host:

```
docker container run -d -p 8080:80 -p 4443:443 nginx
```

4. Verify open ports:

```
docker container ls
```

Lab 2.7 Auto Port Mapping
---------------------------- 

1. Publishing Ports  
Run Nginx and let Docker choose a high port:  
`docker container run -d -p 80 --name nginx-high nginx`  

2. Run Nginx and let Docker determine exposed ports and mapping: 
`docker container run -d -P --name nginx-auto nginx`

3. Show open ports:

```
docker container ls
```

For communication within the network, use the exposed port.
For communication external to the network, use the published port.  

Lab 2.8 Inter-container Communication
-----------------------------------------

Let's take what we've learned in Section 2 apply it in this lab. 

Run an NGINX container and view your custom content from another container (using curl or similar).

Lab 3.1: Build a Simple Python App
----------------------------------

In this lab, we will build a very simple python application.

1. Go to bootcamp/simple-python-app

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
