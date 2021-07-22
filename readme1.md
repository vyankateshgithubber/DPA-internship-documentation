# Create and Deploy the Docker Image of ML API across multiple servers
***
## Step by step procedure
1. Create the Dockerfile given ML API 
2. Build it to create docker image
3. Create the container of docker image and check it working
4. Tagging of Docker image (in this format username/imagename:tag)
5. Push the tagged docker image to dockerhub
5. Create three virtual docker-host in your local system
6. Any one of virtual docker-host machine initialize the docker swarm act as manager node
7. Join rest two docker-machine as worker nodes
8. Pull the docker image from dockerhub on all three docker-machines
9. Create the service wiht that docker image with specified replicas
10. Scale up or down the docker service replicas and verify on each machine
11. Remove service to stop process

***

## Create file with name as **Dockerfile** 
We used the python base image and also used the multistages in Dockerfile to reduce the size of docker image.Stage as the compile-image which will have libraries installed and build-image as to build and run the ML API.
```
# Base Image as python
FROM python:3.7-slim AS compile-image

# Install the required updates 
RUN apt-get update
RUN apt-get install -y --no-install-recommends build-essential gcc
RUN python -m venv /opt/venv

# Make sure we use the virtualenv:
ENV PATH="/opt/venv/bin:$PATH"

# Install the required libraries in requirements.txt file 
RUN /opt/venv/bin/python -m pip install --upgrade pip
COPY /app/requirements.txt  requirements.txt
RUN pip3 install -r requirements.txt

# New python base image 
FROM python:3.7-slim AS build-image

# copy the required files from previous stage 
COPY --from=compile-image /opt/venv /opt/venv

# COPY flair model to cache path of flair library
COPY /.flair/ /root/.flair/models/
# copy the api files to container file system 
WORKDIR /new_app
COPY /app/ /new_app/
ENV PATH="/opt/venv/bin:$PATH"

# RUN this command while running the docker image
CMD ["uvicorn","main:app","--host", "0.0.0.0" ,"--port" ,"8080"]

```
## Build the Docker image
mlapi is image name
Make sure that you are in same directory as in Dockerfile 
```
    docker build -t mlapi .
```
## Run and verify the Container is running properly 
Run container in intractive mode and exposed to port 8080, firstly container load "ner" file from cache then we can access the api
```
    docker run -it -p 8080:8080 mlapi
```
## Tagging the docker image in format user_name/image_name:tag
Tagging helps in versioning of docker images \
docker tag image-ID user_name/image_name:tag
```
    docker tag mlapi 09052001/mlapi:v1
```
## Push the docker image to dockerHub
To push docker image,it must  have name like user_id/imagename
```
    docker push 09052001/mlapi:v1
```
## Creating the three virtual docker-host in your local system
### Install docker machine 
**Requirements**  
1. Hyper-V installed
2. Virtualization must be enabled
3. Check wheather any external network switch available in Hyper-V if not then setup new external network switch with proper and correct external network.
4. After setup reboot the computer.
   for Windows system, install Git bash run as administrator
   Run following command to install docker-machine
```
    $ base=https://github.com/docker/machine/releases/download/v0.16.0 \
      && mkdir -p "$HOME/bin" \
      && curl -L $base/docker-machine-Windows-x86_64.exe > "$HOME/bin/docker-machine.exe" \
      && chmod +x "$HOME/bin/docker-machine.exe"
```
### Create three virtual docker host machine as manager1,worker1 and worker2
RUN commands on the Git Bash as administrator
Use the Microsoft Hyper-V driver and reference the new virtual switch you created.
```
    $ docker-machine create -d hyperv --hyperv-virtual-switch <NameOfVirtualSwitch> <nameOfNode>
```
In our case,
```
    $ docker-machine create -d hyperv --hyperv-virtual-switch "Primary Virtual Switch" manager1
```
```
    $ docker-machine create -d hyperv --hyperv-virtual-switch "Primary Virtual Switch" worker1
```
```
    $ docker-machine create -d hyperv --hyperv-virtual-switch "Primary Virtual Switch" worker1
```
### Check the list of machines
```
    docker-machine ls
```
### To start the machine 
```
    docker-machine start machine_name 
```
### To stop the machine 
```
    docker-machine stop machine_name 
```
### Log into or run a command on a machine with SSH.
```
    docker-machine ssh machine_name
```

### Get the IP of the machine
```
    docker-machine ip machine_name
```
## Swarm mode intalization in the virtual docker host
SSH in manger1 machine 
```
    $ docker-machine ssh manager1
```
Intailize the swarm mode with manager1 as manager node
```
    $ docker swarm init --advertise-addr manager1_ip
```
Join the other nodes as worker.
Run command which returned following command on the all nodes which you want as worker node 
```
    $ docker swarm join-token worker
```
## Node list
```   
    $ docker node ls
```    
## To change state of tbe node in swarm to active or pause or drain status
```
    $ docker node update --availability status node_name

```
## Storing the docker image of virtual docker host machine 
This step is necessary only when the docker image size very big.
Pull the docker image from dockerHub 
```    
    $ docker pull 09052001/mlapi:v1
```
## Creating the service name "web" of the docker image of 3 replicas and port mapped to 8080 
```
    $ docker service create --replicas number -p target:source --name servicename imagename
```
```
    $ docker service create --replicas 3 -p 8080:8080 --name web 09052001/mlapi:v1
```
## List the service available
```
    $ docker service ls
```
## List the status of task running on nodes of service
```
    $ docker service ps servicename
```
In our case,
```
    $ docker service ps web
```
## To scale up or down the services
```
    $ docker service scale servicename=NumberofContainer
```
## Removing service stop  all container and erase them completely
```
    $ docker service rm serviceName
```
## To leave the swarm mode ssh into that machine 
```
    $ docker swarm leave
```
## To leave the manager it must be forceful action
```   
    $ docker swarn leave --force
```
then exit
## To stop the docker-machine 
```
    $ docker-machine stop nodename
```  
## To remove machine completely
```    
    $ docker-machine kill nodename
```

##   Image and conatiner Details 
**Docker Image** = 2.66GB \
**Container layer Size** = 3.25MB \
**Container memory usage stats**
***

CONTAINER ID |  NAME  |  CPU % |    MEM USAGE/ LIMIT |   MEM %   |  NET I/O  |  BLOCK I/O  | PIDS| 
----|---|---|----|----|----|----|----
14bb20323619|   kind_haslett |  0.46%   |  817.3MiB / 6.13GiB |  13.02%   | 9.87kB / 7.26kB |  0B / 0B   |  11 |
***