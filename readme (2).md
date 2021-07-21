# Schedule automatic start and stop dockerized selenium web scrapper image
***
## Usecases
How dockerized web scrapper image helps?
1.	Usually, performing the web scrapping repeatedly number of times which leads to the increasing memory usage of chrome driver and have possibilities of crash. To avoid this problem completely removing containers from memory after web scrapping stores results locally.
2.	As web scrapping in regular interval of time, we can use event driven cloud computing service like Amazon Lambda which has pay-per-use basis. This will also reduce overall cost.
***
## Steps followed
1.	Create Dockerfile for python selenium script.
2.	Written docker-compose file where docker image dependent on selenium chrome driver image (base image doesn’t have chrome driver installed).
3.  Combine all commands sequentially in shell script file.
4.	Create .cron file which schedules the shell script at specified time.
5.  Run .cron file.
***
## Dockerfile for python selenium script
Files structure
```
.
|-app
|  |- etfgi
|  |- prenews
|  |- bot.py
|- Dockerfile
|- docker-compose.yml
|- requirements.txt
|- script.sh
|- sch.cron
```
Dockerfile
```
# python base image
FROM python:3.7-alpine
# Set Environment varible 
ENV PYTHONUNBUFFERED 1 
COPY requirements.txt /requirements.txt
# install the required libraries
RUN pip install -r /requirements.txt
RUN mkdir /app
COPY ./app /app/
WORKDIR /app/
```
## Docker compose file
selenium/standalone-chrome starts first then selenium script image. \
urllink runtime argument 
containers used the shared volumes i.e. data will be stored local host machine(in our case /app directory)
```
version: '3'
services: 
    selenium:
        image: selenium/standalone-chrome
        ports:
            - 4444:4444
    app:
        build: 
            context: .
        volumes: 
        - ./app:/app
        environment: 
            urllink: ${urllink}
        command: sh -c "python3 bot.py -src=${urllink}"
        depends_on:
            - selenium
```

## Script file

**export will assigns the value to runtime argument that defined in compose file** \
export urllink=value \

**Build and run containers sequentially, stop all containers when any one of container fails** \
docker-compose -f filepath/docker-compose.yml up --build --abort-on-container-exit  \

**Stop and Remove the all containers and networks** \
docker-compose -f filepath/docker-compose.yml down 

Script .sh file 
```
#!/bin/bash
export urllink=prnews
docker-compose -f /home/docker/job1/docker-compose.yml up --build --abort-on-container-exit
docker-compose -f /home/docker/job1/docker-compose.yml down
```
## Crontab used for schedule the container
.cron file
```
*/1 * * * * echo "hello" >> /home/docker/job1/log3.txt
*/30 * * * * bash /home/docker/job1/script1.sh >> /home/docker/job1/log1.txt
*/35 * * * * bash /home/docker/job1/script2.sh >> /home/docker/job1/log2.txt
```
At 30 min interval run script1.sh and store the log in log1.txt file \
At 35 min interval run script2.sh and store the log in log2.txt file \
Use link https://crontab.guru/ 

## Run crontab 
```
    $ crontab sch.cron
```
## Now task will run at specified regular interval unless we stop crontab 

