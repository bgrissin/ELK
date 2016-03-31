# Lab 10 : Setting up an ELK stack

> **Difficulty**: Easy

> **Time**: 15 minutes

> **Tasks**:
>
* [Introduction](#introduction)
* [Task 1: Log into hub.docker.com (#task-2-Hub Login)
* [Task 2: verify docker-compose.yml (#task-3-Docker-compose)
* [Task 3: Connect and browse logs](#task-4-browse-logs)

## Introduction

One of the more popular OSS logging solutions is ELK. It is comprised of Elasticsearch, Logstash and Kibana (ELK). This lab will show you how to deploy ELK that can be then be used for logging and monitoring setup within Docker Universal Control Plane (DUCP).

## Task 1: Login to the offical Docker Hub repository

We want to use official scanned images from hub that provides us the best protection available from potential image vulnerabilities.

1. Log into hub so that when images are pulled they are coming from the official Docker repository.

            $ docker login https://index.docker.io/v1/
              Username: JoeUser
              Password:
              Email: JoeUser@example.com
              WARNING: login credentials saved in /Users/JoeUser/.docker/config.json
              Login Succeeded

## Task 2: Docker-compose 

1. The following docker-compose.yml file consumes three images - elastisearch, logstash, and kibana.  These are all official images that are pulled from the offical Docker hub repository.  These images are scanned which gives additional levels of assurance against known CVE's and vulnerabilities within the images.
		
version: "2"

services:
  elasticsearch:
    image: elasticsearch:latest
    command: elasticsearch -Des.network.host=0.0.0.0
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - ucp-elasticsearch-data:/usr/share/elasticsearch/data
  logstash:
    image: logstash:latest
    command: logstash sh -c "logstash -e 'input { syslog { } } output { stdout { } elasticsearch { hosts => [ \"es\" ] } } filter { json { source => \"message\" } }'"
    ports:
      - "514:514"
    links:
      - elasticsearch
  kibana:
    build: kibana/
    ports:
      - "5601:5601"
    links:
      - elasticsearch

volumes:
  ucp-elasticsearch-data:
    external: true


2. Verify all three containers are running.

		docker ps
		CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                    NAMES
		922b7b4fb477        kibana              "/docker-entrypoint.s"   42 seconds ago       Up 42 seconds       0.0.0.0:5601->5601/tcp   kibana
		04fd46e25ab1        logstash            "/docker-entrypoint.s"   About a minute ago   Up About a minute   0.0.0.0:514->514/tcp     logstash
		5a561ce45cce        elasticsearch       "/docker-entrypoint.s"   3 minutes ago        Up 3 minutes        9200/tcp, 9300/tcp       elasticsearch


## Task 3: Connect and browse logs

Now that the three containers are up and running, you can view logs.



1. Point your web browser to your node on port 5601

	Example: `http://ec2-52-90-128-138.compute-1.amazonaws.com:5601/`
	
	From here you can browse and search log/event entries. 

	>**Note:** When deploying in production environments, you should secure kibana (not described in this doc).

2. Perform some example searches

	- Show all the modifications on the system

		`type:"api" AND (tags:"post" OR tags:"put" OR tags:"delete")`

	- Show all access from a given user

		`username:"admin"`

	- Show all authentication failures on the system

		`type:"auth fail" `


Now that we have a logging solution setup, we can later instruct DUCP to use our logging solution to aggregate logging.

## Conclusion

At this point you have successfully set up a simple ELK stack and ran common log queries.


## Clean up

If you plan to do another lab, you need to cleanup your AWS EC2 instances. Cleanup removes any environment variables, configuration changes, Docker images, and running containers. To do a clean up, log into each EC2 instance and run the following:


	$ source /home/ubuntu/cleanup.sh


## Related information
