1. How many images are available on this host?

9

```shell
docker image ls

REPOSITORY                      TAG       IMAGE ID       CREATED        SIZE
mysql                           latest    7ce93a845a8a   5 weeks ago    586MB
alpine                          latest    324bc02ae123   5 weeks ago    7.79MB
nginx                           latest    a72860cb95fd   2 months ago   188MB
nginx                           alpine    1ae23480369f   2 months ago   43.2MB
ubuntu                          latest    35a88802559d   2 months ago   78MB
redis                           latest    6c00f344e3ef   3 months ago   116MB
postgres                        latest    07a4ee949b9e   3 months ago   432MB
kodekloud/simple-webapp-mysql   latest    129dd9f67367   5 years ago    96.6MB
kodekloud/simple-webapp         latest    c6e3cd9aae36   5 years ago    84.8MB

$ docker image ls | wc -l
10
```

2. What is the size of the ubuntu image?

78MB

```shell
$ docker image ls ubuntu              
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
ubuntu       latest    35a88802559d   2 months ago   78MB
```

3. We just pulled a new image. What is the tag on the newly pulled NGINX image?

1.14-alpine

```shell
$ docker image ls nginx 
REPOSITORY   TAG           IMAGE ID       CREATED        SIZE
nginx        latest        a72860cb95fd   2 months ago   188MB
nginx        alpine        1ae23480369f   2 months ago   43.2MB
nginx        1.14-alpine   8a2fb25a19f5   5 years ago    16MB
```

4. We just downloaded the code of an application. What is the base image used in the Dockerfile?

Inspect the Dockerfile in the webapp-color directory.

python:3.6

```shell
$ cat Dockerfile 
```

```
FROM python:3.6

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]
```

5. To what location within the container is the application code copied to during a Docker build?

Inspect the Dockerfile in the webapp-color directory.

/opt

6. When a container is created using the image built with this Dockerfile, what is the command used to RUN the application inside it.

Inspect the Dockerfile in the webapp-color directory.

python app.py

7. What port is the web application run within the container?

Inspect the Dockerfile in the webapp-color directory.

8080

8. Build a docker image using the Dockerfile and name it webapp-color. No tag to be specified.
- Image Name: webapp-color

```shell
docker build -h

docker build --tag=webapp-color .
```

9. Run an instance of the image webapp-color and publish port 8080 on the container to 8282 on the host.

- Container with image 'webapp-color'
- Container Port: 8080
- Host Port: 8282

```shell
docker run --help
docker run -p 8282:8080 webapp-color
```

10. Access the application by clicking on the tab named HOST:8282 above your terminal.

After you are done, you may stop the running container by hitting CTRL + C if you wish to.

11. What is the base Operating System used by the python:3.6 image?

If required, run an instance of the image to figure it out.

```shell
docker ps     
CONTAINER ID   IMAGE          COMMAND           CREATED         STATUS         PORTS                    NAMES
94db58651d02   webapp-color   "python app.py"   9 seconds ago   Up 7 seconds   0.0.0.0:8282->8080/tcp   nice_hypatia

docker exec --help
Usage:  docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

docker exec -it 94db58651d02 cat /etc/os-release

PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

12. What is the approximate size of the webapp-color image?

```shell
$ docker image ls webapp-color
REPOSITORY     TAG       IMAGE ID       CREATED         SIZE
webapp-color   latest    c336cb8e2748   5 minutes ago   913MB
```

13. That's really BIG for a Docker Image. Docker images are supposed to be small and light weight. Let us try to trim it down.

14. Build a new smaller docker image by modifying the same Dockerfile and name it webapp-color and tag it lite.
Hint: Find a smaller base image for python:3.6. Make sure the final image is less than 150MB.

- Name: webapp-color:lite
- Image size less than 150MB.

```shell
docker build --tag=webapp-color:lite .

docker image ls webapp-color
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
webapp-color   lite      0296b38b4572   15 seconds ago   51.9MB
webapp-color   latest    c336cb8e2748   11 minutes ago   913MB
```

15. Run an instance of the new image webapp-color:lite and publish port 8080 on the container to 8383 on the host.
- Container with image 'webapp-color:lite'
- Container to publish port 8080 to 8383

```shell
docker run -p 8383:8080 webapp-color:lite
```
