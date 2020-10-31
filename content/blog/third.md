+++ 
comments = true 
date = "2020-10-02T15:59:13-04:00"
slug = "" 
tags = ["engineering", "programming"]
categories = ["engineering"]
showpagemeta = true
showcomments = true
title = "Docker, Celery and Queues"
description = "Practical setup for Docker and Celery while having different queues"
+++


There are a lot of blogs on how to setup docker and celery for development, but none of them are really practical or like a real world example of development.   
So I'm gonna show how I setup celery with docker while I have a few different celery queues for development.   
   
Setup Celery and Docker when you only have `one queue` :   
### Dockerfile
```dockerfile
FROM python:3.8  
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8 PYTHONUNBUFFERED=1

WORKDIR /  
COPY requirements.txt ./  
RUN pip install --no-cache-dir -r requirements.txt  
RUN rm requirements.txt  

COPY . /  
WORKDIR /app
```
### docker-compose.yml
```docker-compose
version: '3'
services: 
  worker:
    build: .
    image: &img worker 
    command: [celery, worker, --app=worker.app, --pool=gevent, --concurrency=20, --loglevel=INFO]
    environment: &env      
      - CELERY_BROKER_URL=amqp://guest:guest@rabbitmq:5672
    depends_on:
      - rabbitmq
    restart: 'no'
    volumes:
      - ./app:/app 

  rabbitmq:
    hostname: rabbit
    image: rabbitmq:3.8.4-alpine
    environment:
      - RABBITMQ_DEFAULT_USER=guest
      - RABBITMQ_DEFAULT_PASS=guest
    ports:
      - 5672:5672
    container_name: rabbit
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq/data/
```
Now you just start up the stack  
```bash
docker-compose up -d
```
If you need to add another celery worker  
```bash
docker-compose up -d --scale worker=2
```

Now this was pretty simple, but what should we do when we have `several celery queues` and we want a celery worker for each queue?   
There are two ways to achieve this :  

## creating a celery worker for each queue :  

### worker.Dockerfile
```dockerfile
FROM python:3.8
RUN pip install celery, django-celery-beat
COPY worker_entrypoint.sh /usr/bin/worker_entrypoint.sh
WORKDIR /usr/src/app/
CMD bash /usr/bin/worker_entrypoint.sh
```
### worker_entrypoint.sh
```sh
echo "<---Waiting for rabbit to start--->"
sleep 30
echo "<---Starting Worker--->"
echo "QUEUES" $QUEUES
watchmedo auto-restart \
--directory "/usr/src/app/" \
--pattern "*.py" \
--recursive \
-- celery --app=worker.app --pool=gevent --concurrency=20 --loglevel=INFO -n $WORKER_NAME -Q $QUEUES
```

### docker-compose.yml
```docker-compose
version: '3'
services:

  worker:
    image: &worker
    build:
      context: .
      dockerfile: worker.Dockerfile
    environment: &env      
      - CELERY_BROKER_URL=amqp://guest:guest@rabbitmq:5672
      - QUEUES=celery
    depends_on:
      - rabbitmq
    restart: 'no'
    volumes:
      - ./app:/app 

  recommendation_worker:
    image: &worker
    build:
      context: .
      dockerfile: worker.Dockerfile
    environment: &env
      - QUEUES=recommendation_worker
    depends_on:
      - rabbitmq

  periodic_worker:
    image: &worker
    build:
      context: .
      dockerfile: worker.Dockerfile
    environment: &env
      - QUEUES=periodic_worker
    depends_on:
      - rabbitmq

  beat:
    image: &worker
    build:
      context: .
      dockerfile: worker.Dockerfile
    command: [celery --app=worker.app beat]
    depends_on:
      - rabbitmq

  rabbitmq:
    hostname: rabbit
    image: rabbitmq:3.8.4-alpine
    environment:
      - RABBITMQ_DEFAULT_USER=guest
      - RABBITMQ_DEFAULT_PASS=guest
    ports:
      - 5672:5672
    container_name: rabbit
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq/data/

```

Now this works fine in production which is on a server machine. But for development purposes, it is really costly in terms of `memory usage`, `cpu usage`, and `docker build time` on your development machine. So what I've done for development is this :   

### docker-compose.yml
```docker-compose

  worker:
    image: &worker
    build:
      context: .
      dockerfile: worker.Dockerfile
    environment: &env      
      - CELERY_BROKER_URL=amqp://guest:guest@rabbitmq:5672
    depends_on:
      - rabbitmq
    volumes:
      - ./app:/app 

  rabbitmq:
    hostname: rabbit
    image: rabbitmq:3.8.4-alpine
    environment:
      - RABBITMQ_DEFAULT_USER=guest
      - RABBITMQ_DEFAULT_PASS=guest
    ports:
      - 5672:5672
    container_name: rabbit
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq/data/
```

### worker_entrypoint.sh
```sh
echo "<---Waiting for rabbit to start--->"
sleep 30
echo "<---Starting Worker--->"
echo "QUEUES" $QUEUES
watchmedo auto-restart \
--directory "/usr/src/app/" \
--pattern "*.py" \
--recursive \
-- celery  --app=worker.app --beat \
--pool=gevent --concurrency=20 --loglevel=INFO -Q $QUEUES
```

### worker.Dockerfile
```dockerfile
FROM python:3.8
RUN pip install celery, django-celery-beat
COPY worker_entrypoint.sh /usr/bin/worker_entrypoint.sh
WORKDIR /usr/src/app/
ENV QUEUES celery,recommendation_worker,periodic_worker
CMD bash /usr/bin/worker_entrypoint.sh
```

*Note* : This is only meant for development purposes

This way you handle whatever queues you have with only one worker, the celery beat is also on one line command with the celery worker.   

#### Advantages:  
- less building time
- mush less cpu and memory usage
- less logs when running docker
- less space used
- less code

#### Disadvantages:  
- it's only suited for development stage   


If you need more workers you could scale the number of your workers:  
```bash
docker-compose up -d --scale worker=5
```