# lab-08

Данная лабораторная работа посвящена созданию собственного локального сервера и работы с ним при помощи клиент-серверного приложения

# Task 1
To begin with, let's make a server. Let's use a ready-made solution in `Python`:

`server.py` contents:
```
import http.server
import socketserver

handler = http.server.SimpleHTTPRequestHandler
with socketserver.TCPServer(("", 1234), handler) as httpd:
   httpd.serve_forever()
```

`client.py` contents
```
import urllib.request

fp = urllib.request.urlopen("http://localhost:1234/")

encodedContent = fp.read()
decodedContent = encodedContent.decode("utf8")

print(decodedContent)

fp.close()
```

Next, we will make a `Dockerfile` for each part of the application
`Dockerfile` contents for server:
```
FROM python:latest

ADD server.py /server/
ADD index.html /server/

WORKDIR /server/
```

`Dockerfile` contents for client:
```
FROM python:latest

ADD client.py /client/

WORKDIR /client/
```

We will also make an `html` file with the contents of the output in the directory `server.py `

`index.html` contents:
```
Hello from server!
```

Now we will make a `docker-compose.yml` for building our Docker

`docker-compose.yml` contents:
```
version: "3"

services:

  server:

    build: server/
    command: python ./server.py
    ports:
      - 1234:1234

  client:

    build: client/
    command: python ./client.py
    network_mode: host
    depends_on:
      - server
```

And finally, let's create a `.yml` file for our application to work in GitHub Actions

`Action.yml` contents:
```
name: Docker Compose

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Login to DockerHub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build Docker images
      run: |
        docker-compose build

    - name: Push Docker images
      run: |
        docker-compose push

    - name: Deploy with Docker Compose
      run: |
        docker-compose up -d
```
