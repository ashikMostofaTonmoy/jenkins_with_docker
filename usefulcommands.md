<!--  -->
###use the docker file
Open up a terminal window.

- Create a bridge network in Docker using the following docker network create command:
```
docker network create jenkins
```

- write docker file
```
FROM jenkins/jenkins:2.387.1
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"
```

- build the image . This incorporates the blueocean plugin

```
docker build -t myjenkins-blueocean:2.387.1-1 .
```

Run your own myjenkins-blueocean:2.387.1-1 image as a container in Docker using the following docker run command:



```
docker run \
  --name jenkins-blueocean \
  --restart=on-failure \
  --detach \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:2.387.1-1 
```

- go to the location cause it is runing locally
```
http://localhost:8080/
http://127.0.0.1:8080/
```

- you could access your docker container (through a separate terminal/command prompt window) with a docker exec command like:

```
docker exec -it jenkins-blueocean bash
```
- then go to 
```
cat /var/jenkins_home/secrets/initialAdminPassword
```

- alternatively as you are running Jenkins in Docker using the official jenkins/jenkins image you can use 
```
sudo docker exec ${CONTAINER_ID or CONTAINER_NAME} cat /var/jenkins_home/secrets/initialAdminPassword
```
to print the password in the console without having to exec into the container.

