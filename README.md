# Docker Tutorials


## 1. Docker Images

``` powershell
docker images
```

লোকাল কম্পিউটারের সব Docker Image দেখায়।

``` powershell
docker pull python:latest
```

Docker Hub থেকে Python Image ডাউনলোড করে।

``` powershell
docker rmi python:latest
```

লোকাল কম্পিউটার থেকে Python Image মুছে দেয়। Image কোনো container ব্যবহার
করলে আগে সেই container সরাতে হতে পারে।

## 2. Docker Containers

``` powershell
docker ps
```

চলমান Container-এর তালিকা দেখায়।

``` powershell
docker ps -a
```

চলমান ও বন্ধ---সব Container দেখায়।

``` powershell
docker run python:latest
```

Image থেকে নতুন Container তৈরি করে চালায়। Container-এর নাম সাধারণত Docker
নিজে দেয়।

``` powershell
docker run --name py-c1 python:3.12-slim
```

`py-c1` নাম দিয়ে Container তৈরি ও চালায়।

``` powershell
docker run -d --name py-c1 python:3.12-slim
```

Container-কে background-এ চালায় (Detached Mode)।

``` powershell
docker run -it --name py-c1 python:3.12-slim
```

Interactive terminal চালু করে। Python shell পেতে সাধারণত
`python:3.12-slim` image-এর সঙ্গে `python` command দিতে হয়।

``` powershell
docker exec -it py-c1 bash
```

চলমান Container-এর ভেতরে Bash shell চালু করে। Image-এ Bash না থাকলে `sh`
ব্যবহার করা যায়।

``` powershell
docker start -ai py-c1
```

বন্ধ Container চালু করে এবং তার মূল process-এর সঙ্গে terminal যুক্ত করে।

``` powershell
docker stop py-c1
```

চলমান Container বন্ধ করে।

``` powershell
docker rm py-c1
```

বন্ধ Container মুছে দেয়।

## 3. Dockerize a Python File

### Folder Structure

``` text
python-docker-app/
├── app.py
└── Dockerfile
```

-   `app.py` --- Python code রাখে।
-   `Dockerfile` --- Docker Image তৈরির নির্দেশনা রাখে।

### `app.py`

``` python
print("Hello from Docker!")
```

### `Dockerfile`

``` dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

**ব্যাখ্যা:** - `FROM` --- Base image নির্ধারণ করে। - `WORKDIR` ---
Container-এর working directory নির্ধারণ করে। - `COPY` --- Host থেকে file
image-এর মধ্যে কপি করে। - `CMD` --- Container চালু হলে default command
চালায়।

### Image Build

Project folder থেকে চালাও:

``` powershell
docker build -t my-python-app:v1 .
```

এটি `Dockerfile` ব্যবহার করে `my-python-app:v1` image তৈরি করে।

### Container Run

``` powershell
docker run --name py-app my-python-app:v1
```

Output:

``` text
Hello from Docker!
```

Container-এর অবস্থা দেখতে:

``` powershell
docker ps -a
```

**Note:** Python script শেষ হলে Container সাধারণত বন্ধ হয়ে যায়, কিন্তু
Container নিজে থেকে মুছে যায় না।

## 4. Dockerize a Django Project

### Project তৈরি

`uv` ব্যবহার করে Django project তৈরি করো:

``` powershell
mkdir docker_django
cd docker_django

uv init --bare
uv add django

uv run django-admin startproject config .
```

### Folder Structure

``` text
docker_django/
├── config/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
├── manage.py
├── pyproject.toml
├── uv.lock
├── Dockerfile
└── .dockerignore
```

### `Dockerfile`

``` dockerfile
FROM python:3.12-slim

WORKDIR /app

ENV UV_PROJECT_ENVIRONMENT=/opt/venv


RUN pip install uv

COPY pyproject.toml uv.lock ./
RUN uv sync

COPY . .

EXPOSE 8000

CMD ["uv", "run", "python", "manage.py", "runserver", "0.0.0.0:8000"]
```

`uv.lock` আগে তৈরি থাকতে হবে। `uv sync` project dependencies install
করে।

### `.dockerignore`

``` text
.venv/
__pycache__/
*.py[cod]
.git/
.env
```

### Image Build

``` powershell
docker build -t docker-django:v1 .
```

### Container Run

``` powershell
docker run --name django-c1 -p 8000:8000 docker-django:v1
```

-   `--name django-c1` --- Container-এর নাম নির্ধারণ করে।
-   `-p 8000:8000` --- Host-এর port `8000`-কে Container-এর port
    `8000`-এর সঙ্গে যুক্ত করে।
-   `docker-django:v1` --- চালানোর image।

Background-এ চালাতে:

``` powershell
docker run -d --name django-c1 -p 8000:8000 docker-django:v1
```

Browser: http://localhost:8000

Logs দেখতে:

``` powershell
docker logs django-c1
```

## 5. Docker Volumes

Docker-এর mount ব্যবহার করে Host-এর files Container-এ যুক্ত করা যায় অথবা
Container-এর data আলাদা storage-এ রাখা যায়।

### 5.1 Virtual Environment-এর path

Dockerfile-এ `WORKDIR /app`-এর পরে:

``` dockerfile
ENV UV_PROJECT_ENVIRONMENT=/opt/venv
```

এতে `uv`-এর virtual environment `/opt/venv`-এ থাকবে, `/app/.venv`-এ নয়।
Host project folder `/app`-এ bind mount করলে এতে environment ঢেকে যাওয়ার
সমস্যা এড়ানো যায়।

### 5.2 Bind Mount

``` powershell
docker run --rm --name dj-c1 -p 8000:8000 -v "${PWD}:/app" docker-django:v1
```

-   `--rm` --- Container বন্ধ হয়ে exit করলে Container-টি মুছে দেয়।
-   `--name dj-c1` --- Container-এর নাম নির্ধারণ করে।
-   `-p 8000:8000` --- Host ও Container-এর port যুক্ত করে।
-   `-v "${PWD}:/app"` --- বর্তমান Host directory-কে Container-এর
    `/app`-এ mount করে।

**কেন ব্যবহার করি?** Host-এ source code পরিবর্তন করলে Container-এর মধ্যে
সেই পরিবর্তন দেখা যায়। Django development server সাধারণত code change
detect করে reload হয়।

### 5.3 Named Volume

Volume নিজে তৈরি করা যায়:

``` powershell
docker volume create django_venv
```

অথবা `docker run`-এ ব্যবহার করলে Docker না থাকলে তৈরি করে:

``` powershell
docker run --rm --name dj-c1 `
  -p 8000:8000 `
  -v "${PWD}:/app" `
  -v django_venv:/opt/venv `
  docker-django:v1
```

`django_venv` হলো Named Volume; এটি Container-এর `/opt/venv` path-এ
mount হয়।

### 5.4 Anonymous Volume

``` powershell
docker run --rm --name example-c1 -v /data python:3.12-slim
```

`/data`-এর জন্য Docker একটি নাম-স্বয়ংক্রিয়ভাবে-নির্ধারিত Volume mount করে।
`--rm`-সহ Container সরালে তার anonymous volume সাধারণত সরিয়ে দেওয়া হয়।

### 5.5 Volume-এর পার্থক্য

  -----------------------------------------------------------------------
  বিষয়              Bind Mount        Named Volume      Anonymous Volume
  ----------------- ----------------- ----------------- -----------------
  Storage           Host-এর নির্দিষ্ট   Docker পরিচালিত   Docker পরিচালিত
                    path                                

  নাম               Host path দিয়ে    নিজে নাম দাও      Docker নাম তৈরি
                    চিহ্নিত                              করে

  সাধারণ ব্যবহার     Development code  পুনর্ব্যবহারযোগ্য     নাম না দিয়ে mount
                                      data              

  Container সরালে   Host files থাকে   সাধারণত থাকে      `--rm` থাকলে
                                                        সাধারণত সরানো হয়
  -----------------------------------------------------------------------

### 5.6 দরকারি Volume Commands

``` powershell
# সব Volume দেখো
docker volume ls

# Volume-এর বিস্তারিত তথ্য
docker volume inspect django_venv

# নির্দিষ্ট Volume মুছে ফেলো
docker volume rm django_venv

# ব্যবহার না হওয়া Volume পরিষ্কার করো
docker volume prune
```

**সতর্কতা:** `docker volume prune` ব্যবহার না হওয়া Volume মুছে দিতে পারে।
গুরুত্বপূর্ণ data থাকলে আগে যাচাই করো।

## 6. Dockerize a Next.js Project

### Folder Structure

``` text
docker_nextjs/
├── app/
│   ├── page.tsx
│   ├── layout.tsx
│   └── globals.css
├── public/
├── Dockerfile
├── .dockerignore
├── package.json
├── package-lock.json
└── next.config.mjs
```

`node_modules/` ও `.next/` local development-এ তৈরি হতে পারে; এগুলো
সাধারণত source control-এ commit করা হয় না।

### `Dockerfile`

``` dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "run", "dev"]
```

### `.dockerignore`

``` text
node_modules/
.next/
.git/
.env*
```

### Image Build

``` powershell
docker build -t next-docker:v1 .
```

### Container Run: Bind Mount + Named Volume + Anonymous Volume

``` powershell
docker run --rm --name next-c1 `
  -p 3000:3000 `
  -v "${PWD}:/app" `
  -v next_node_modules:/app/node_modules `
  -v /app/.next `
  next-docker:v1
```

  ---------------------------------------------------------------------------------------
  Mount                                   Type                    কাজ
  --------------------------------------- ----------------------- -----------------------
  `${PWD}:/app`                           Bind Mount              Host-এর code
                                                                  Container-এ যুক্ত করে

  `next_node_modules:/app/node_modules`   Named Volume            Dependencies-এর জন্য
                                                                  পুনর্ব্যবহারযোগ্য storage

  `/app/.next`                            Anonymous Volume        Next.js cache ও
                                                                  generated files আলাদা
                                                                  রাখে
  ---------------------------------------------------------------------------------------

Browser: http://localhost:3000

**Note:** Named Volume আগে তৈরি করা বাধ্যতামূলক নয়। Docker volume না থাকলে
`docker run`-এর সময় তৈরি করতে পারে। Dependency পরিবর্তন হলে পুরোনো
`node_modules` volume-এ নতুন packages install করার প্রয়োজন হতে পারে।

## 7. Docker Compose --- Django + Next.js

Docker Compose দিয়ে একটি YAML file থেকে একাধিক service configure ও
চালানো যায়।

### Folder Structure

``` text
project/
├── backend/
│   ├── config/
│   ├── manage.py
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── pyproject.toml
│   └── uv.lock
├── frontend/
│   ├── app/
│   ├── public/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── package.json
│   └── package-lock.json
└── compose.yaml
```

### `compose.yaml`

``` yaml
services:
  backend:
    build: ./backend
    container_name: django-backend
    ports:
      - "8000:8000"
    volumes:
      - ./backend:/app
      - django_venv:/opt/venv

  frontend:
    build: ./frontend
    container_name: next-frontend
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - next_node_modules:/app/node_modules
      - next_cache:/app/.next
    depends_on:
      - backend

volumes:
  django_venv:
  next_node_modules:
  next_cache:
```

**ব্যাখ্যা:** - `services` --- কোন কোন application/container চালানো হবে। -
`build` --- Dockerfile থাকা directory নির্ধারণ করে। - `ports` --- Host ও
Container-এর port যুক্ত করে। - `volumes` --- Bind Mount বা Docker-managed
storage configure করে। - `depends_on` --- service startup order নির্ধারণে
সাহায্য করে; backend পুরোপুরি ready হওয়ার নিশ্চয়তা দেয় না। - Root-level
`volumes` --- Named Volume declare করে।

### Compose Commands

``` powershell
# Configuration যাচাই
docker compose config

# Image build করে service চালু
docker compose up --build

# Background-এ চালু
docker compose up -d

# Logs দেখা
docker compose logs -f

# Container বন্ধ ও সরানো
docker compose down
```

Named Volume-সহ সব Compose volume মুছতে:

``` powershell
docker compose down -v
```

**সতর্কতা:** `down -v` গুরুত্বপূর্ণ data থাকা volume-ও মুছে দিতে পারে।

### Application URLs

-   Frontend (Next.js): http://localhost:3000
-   Backend (Django): http://localhost:8000
-   Django API (যদি `api/hello/` route তৈরি করা থাকে):
    http://localhost:8000/api/hello/

**মনে রাখবে:** `Dockerfile` image তৈরির নির্দেশনা দেয়; `compose.yaml` এক
বা একাধিক service-এর build, port, volume ও startup configuration নির্ধারণ
করে।
