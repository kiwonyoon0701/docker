**Docker Install**

```
sudo apt-get upgrade
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository  "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
sudo usermod ubuntu -aG docker
logout and login with ubuntu user
docker run hello-world


```

**Docker Run Test**

```
ubuntu@ip-172-31-52-63:/home/ubuntu> docker run -dt --name ubuntu01 ubuntu bash
ubuntu@ip-172-31-52-63:/home/ubuntu> docker exec -it ubuntu01 bash
root@a92c674be00e:/# ls

```

**nginx 시작 및 기본 동작 curl local**

```
ubuntu@ip-172-31-52-63:/home/ubuntu> docker run -dt -it --name web01 alpine
ubuntu@ip-172-31-52-63:/home/ubuntu> docker exec web01 apk add nginx
ubuntu@ip-172-31-52-63:/home/ubuntu> docker exec web01 ls /etc/nginx/http.d/
default.conf
ubuntu@ip-172-31-52-63:/home/ubuntu> docker cp web01:/etc/nginx/http.d/default.conf .
ubuntu@ip-172-31-52-63:/home/ubuntu> cat default.conf

ubuntu@ip-172-31-52-63:/home/ubuntu> vi default.conf
ubuntu@ip-172-31-52-63:/home/ubuntu> cat default.conf
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/;
}


ubuntu@ip-172-31-52-63:/home/ubuntu> docker cp ./default.conf web01:/etc/nginx/http.d/default.conf

ubuntu@ip-172-31-52-63:/home/ubuntu> docker exec web01 cat /etc/nginx/http.d/default.conf
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/;
}

ubuntu@ip-172-31-52-63:/home/ubuntu> docker stats web01
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O     PIDS
0a219dc2d2b0   web01     0.00%     2.527MiB / 15.35GiB   0.02%     2.77MB / 19.8kB   0B / 32.8kB   1


ubuntu@ip-172-31-52-63:/home/ubuntu> vi index.html
ubuntu@ip-172-31-52-63:/home/ubuntu> cat index.html
Welcome to the world!!
ubuntu@ip-172-31-52-63:/home/ubuntu> docker cp index.html web01:/var/www/

ubuntu@ip-172-31-52-63:/home/ubuntu> docker exec -dt web01 nginx -g 'pid /tmp/nginx.pid; daemon off;'
ubuntu@ip-172-31-52-63:/home/ubuntu> docker inspect web01 | grep -i IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.3",
                    "IPAddress": "172.17.0.3",
ubuntu@ip-172-31-52-63:/home/ubuntu> curl 172.17.0.3
Welcome to the world!!


ubuntu@ip-172-31-52-63:/home/ubuntu> docker exec web01 ps -efa |grep nginx
   65 root      0:00 nginx: master process nginx -g pid /tmp/nginx.pid; daemon off;
   71 nginx     0:00 nginx: worker process
   72 nginx     0:00 nginx: worker process
   73 nginx     0:00 nginx: worker process
   74 nginx     0:00 nginx: worker process

ubuntu@ip-172-31-52-63:/home/ubuntu> docker top web01
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                4632                4610                0                   Aug25               pts/0               00:00:00            /bin/sh
root                5201                4610                0                   00:32               ?                   00:00:00            nginx: master process nginx -g pid /tmp/nginx.pid; daemon off;
systemd+            5207                5201                0                   00:32               ?                   00:00:00            nginx: worker process
systemd+            5208                5201                0                   00:32               ?                   00:00:00            nginx: worker process
systemd+            5209                5201                0                   00:32               ?                   00:00:00            nginx: worker process
systemd+            5210                5201                0                   00:32               ?                   00:00:00            nginx: worker process

kiwony@kiwonymac.com:/Users/kiwony> curl 3.38.44.138
curl: (7) Failed to connect to 3.38.44.138 port 80: Connection refused


```

**web-base image생성 및 internect access**

```
ubuntu@ip-172-31-52-63:/home/ubuntu> docker commit web01 web-base
ubuntu@ip-172-31-52-63:/home/ubuntu> docker images
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
web-base     latest    437ea1a2c2d9   About a minute ago   11.3MB
alpine       latest    021b3423115f   2 weeks ago          5.6MB

ubuntu@ip-172-31-52-63:/home/ubuntu> docker run -dt -p 80:80 --name web-public web-base
bb0aa7942818b1aa855e74f00d8e356562082d4e4b42545a0827862f771a3492
ubuntu@ip-172-31-52-63:/home/ubuntu> docker ps
CONTAINER ID   IMAGE      COMMAND     CREATED         STATUS         PORTS                               NAMES
bb0aa7942818   web-base   "/bin/sh"   3 seconds ago   Up 2 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp   web-public
0a219dc2d2b0   alpine     "/bin/sh"   15 hours ago    Up 15 hours                                        web01

ubuntu@ip-172-31-52-63:/home/ubuntu> docker exec -dt web-public nginx -g 'pid /tmp/nginx.pid; daemon off;'

kiwony@kiwonymac.com:/Users/kiwony> curl 3.38.44.138
Welcome to the world!!


```

**Node image 생성 및 실행 **

```
ubuntu@ip-172-31-52-63:/home/ubuntu> wget --content-disposition 'https://github.com/linuxacademy/content-Introduction-to-Containers-and-Docker/raw/master/lessonfiles/demo-app.tar'

ubuntu@ip-172-31-52-63:/home/ubuntu> tar -xf demo-app.tar
ubuntu@ip-172-31-52-63:/home/ubuntu> cd app/
ubuntu@ip-172-31-52-63:/home/ubuntu/app> ls
index.js  node_modules  nodesource_setup.sh  package-lock.json  package.json  public  views

ubuntu@ip-172-31-52-63:/home/ubuntu/app> vi Dockerfile
ubuntu@ip-172-31-52-63:/home/ubuntu/app> cat Dockerfile
FROM node:10-alpine
CMD mkdir -p /home/node/app/node_modules && chown node:node -R /home/node/app
WORKDIR /home/node/app
COPY package*.json ./
RUN npm config set registry http://registry.npmjs.org/
RUN npm install
COPY --chown=node:node . .
USER node
expose 8080
CMD [ "node", "index.js" ]


ubuntu@ip-172-31-52-63:/home/ubuntu/app> docker build . -t appimage
ubuntu@ip-172-31-52-63:/home/ubuntu/app> docker images
REPOSITORY   TAG         IMAGE ID       CREATED          SIZE
appimage     latest      54e479275ef5   24 seconds ago   87.9MB
web-base     latest      437ea1a2c2d9   20 minutes ago   11.3MB
alpine       latest      021b3423115f   2 weeks ago      5.6MB
node         10-alpine   aa67ba258e18   4 months ago     82.7MB

ubuntu@ip-172-31-52-63:/home/ubuntu/app> docker run -dt --name app -p 8080:8080 appimage
72adcfaaddcb3fe09918f9a6d29f2c432dab0ce8bdbb598d27a525e96604db85
ubuntu@ip-172-31-52-63:/home/ubuntu/app> docker ps
CONTAINER ID   IMAGE      COMMAND                  CREATED         STATUS         PORTS                                       NAMES
72adcfaaddcb   appimage   "docker-entrypoint.s…"   3 seconds ago   Up 2 seconds   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   app

ubuntu@ip-172-31-52-63:/home/ubuntu/app> docker ps
CONTAINER ID   IMAGE      COMMAND                  CREATED         STATUS         PORTS                                       NAMES
72adcfaaddcb   appimage   "docker-entrypoint.s…"   3 seconds ago   Up 2 seconds   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   app
ubuntu@ip-172-31-52-63:/home/ubuntu/app> docker inspect app |grep -i IPAD
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",

ubuntu@ip-172-31-52-63:/home/ubuntu/app> curl 172.17.0.2:8080
<html>
<head>
  <title>Forethought: A To-Do List</title>
  <link href='https://fonts.googleapis.com/css?family=Duru Sans' rel='stylesheet'>
  <link href="/style.css" rel="stylesheet">
</head>

<body>
  <div class="container">
    <h1>Forethought</h1>
    <h2>Do This!</h2>
    <form action="/addtask" method="POST">
      <input class="newtask" test="text" name="newtask" placeholder="Add a task!">
      <button class="addbutton">Add</button><br />

      <br /><button class="completed" formaction="/removetask" type="submit">Complete Checked Tasks</button>
    </form>
    <h2>Completed</h2>

    <p class="credit">learn how to make a similar app in <a href="https://medium.com/@atingenkay/creating-a-todo-app-with-node-js-express-8fa51f39b16f">this article</a>; also see the <a href="https://github.com/missating/nodejs-todo">github</a>!</a>
  </div>
</body>
</html>

kiwony@kiwonymac.com:/Users/kiwony> curl 3.38.44.138:8080
<html>
<head>
  <title>Forethought: A To-Do List</title>
  <link href='https://fonts.googleapis.com/css?family=Duru Sans' rel='stylesheet'>
  <link href="/style.css" rel="stylesheet">
</head>

<body>
  <div class="container">
    <h1>Forethought</h1>
    <h2>Do This!</h2>
    <form action="/addtask" method="POST">
      <input class="newtask" test="text" name="newtask" placeholder="Add a task!">
      <button class="addbutton">Add</button><br />

      <br /><button class="completed" formaction="/removetask" type="submit">Complete Checked Tasks</button>
    </form>
    <h2>Completed</h2>

    <p class="credit">learn how to make a similar app in <a href="https://medium.com/@atingenkay/creating-a-todo-app-with-node-js-express-8fa51f39b16f">this article</a>; also see the <a href="https://github.com/missating/nodejs-todo">github</a>!</a>
  </div>
</body>
</html>

```

**dockerhub login 및 upload image**

```
ubuntu@ip-172-31-52-63:/home/ubuntu/app> docker login --username=kiwonyoon0701
Password:
WARNING! Your password will be stored unencrypted in /home/ubuntu/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

ubuntu@ip-172-31-52-63:/home/ubuntu/app> docker tag appimage kiwonyoon0701/demo-app:latest
ubuntu@ip-172-31-52-63:/home/ubuntu/app> docker images
REPOSITORY               TAG         IMAGE ID       CREATED          SIZE
kiwonyoon0701/demo-app   latest      54e479275ef5   4 minutes ago    87.9MB
appimage                 latest      54e479275ef5   4 minutes ago    87.9MB
web-base                 latest      437ea1a2c2d9   23 minutes ago   11.3MB
alpine                   latest      021b3423115f   2 weeks ago      5.6MB
node                     10-alpine   aa67ba258e18   4 months ago     82.7MB
ubuntu@ip-172-31-52-63:/home/ubuntu/app> docker push kiwonyoon0701/demo-app

```

**v2 docker 생성 및 upload & deploy**

```
ubuntu@ip-172-31-52-63:/home/ubuntu/app/views> vi index.ejs
ubuntu@ip-172-31-52-63:/home/ubuntu/app/views> grep "V2" index.ejs
    <h1>Forethought-V2*************</h1>

ubuntu@ip-172-31-52-63:/home/ubuntu/app> docker build . -t appimage-v2
ubuntu@ip-172-31-52-63:/home/ubuntu/app> docker tag appimage-v2 kiwonyoon0701/appimage-v2
ubuntu@ip-172-31-52-63:/home/ubuntu/app> docker images
REPOSITORY                  TAG         IMAGE ID       CREATED              SIZE
kiwonyoon0701/appimage-v2   latest      e0d02de69035   About a minute ago   87.9MB
appimage-v2                 latest      e0d02de69035   About a minute ago   87.9MB
kiwonyoon0701/demo-app      latest      54e479275ef5   10 minutes ago       87.9MB
appimage                    latest      54e479275ef5   10 minutes ago       87.9MB
web-base                    latest      437ea1a2c2d9   30 minutes ago       11.3MB
alpine                      latest      021b3423115f   2 weeks ago          5.6MB
node                        10-alpine   aa67ba258e18   4 months ago         82.7MB

ubuntu@ip-172-31-52-63:/home/ubuntu/app> docker run -dt --name app2 -p 8080:8080 kiwonyoon0701/appimage-v2
63ff5fad7568d98a1d9f6d8324157a61c40f9cc08e84a4d4c05d2493f2d39218
ubuntu@ip-172-31-52-63:/home/ubuntu/app> docker ps
CONTAINER ID   IMAGE                       COMMAND                  CREATED         STATUS         PORTS                                       NAMES
63ff5fad7568   kiwonyoon0701/appimage-v2   "docker-entrypoint.s…"   3 seconds ago   Up 2 seconds   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   app2
ubuntu@ip-172-31-52-63:/home/ubuntu/app> docker inspect app2|grep -i IPAD
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
ubuntu@ip-172-31-52-63:/home/ubuntu/app> curl 172.17.0.2:8080
<html>
<head>
  <title>Forethought: A To-Do List</title>
  <link href='https://fonts.googleapis.com/css?family=Duru Sans' rel='stylesheet'>
  <link href="/style.css" rel="stylesheet">
</head>

<body>
  <div class="container">
    <h1>Forethought-V2*************</h1>
    <h2>Do This!</h2>
    <form action="/addtask" method="POST">
      <input class="newtask" test="text" name="newtask" placeholder="Add a task!">
      <button class="addbutton">Add</button><br />

      <br /><button class="completed" formaction="/removetask" type="submit">Complete Checked Tasks</button>
    </form>
    <h2>Completed</h2>

    <p class="credit">learn how to make a similar app in <a href="https://medium.com/@atingenkay/creating-a-todo-app-with-node-js-express-8fa51f39b16f">this article</a>; also see the <a href="https://github.com/missating/nodejs-todo">github</a>!</a>
  </div>
</body>
</html>






```
