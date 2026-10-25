# Docker Commands — Short Bangla Notes

## 1. Docker Images

1. `docker images`  
   => লোকাল কম্পিউটারের সব Docker Image দেখায়।

2. `docker pull python:latest`  
   => Docker Hub থেকে Python Image ডাউনলোড করে।

3. `docker rmi python:latest`  
   => লোকাল কম্পিউটার থেকে Python Image মুছে দেয়।

## 2. Docker Containers

1. `docker ps`  
   => সব Running Container-এর তালিকা দেখায়।

2. `docker ps -a`  
   => Running ও Stopped সব Container দেখায়।

3. `docker run python:latest`  
   => নতুন Container তৈরি ও চালু করে; সাধারণত Random Name দেয়।

4. `docker run --name py-c1 python:3.12-slim`  
   => নির্দিষ্ট নাম `py-c1` দিয়ে Container তৈরি ও চালু করে।

5. `docker run -d --name py-c1 python:3.12-slim`  
   => Container-কে Background-এ চালায় (Detached Mode)।

6. `docker run -it --name py-c1 python:3.12-slim`  
   => Interactive Mode-এ Python Shell চালু করে।

7. `docker exec -it py-c1 bash`  
   => Running Container-এর ভিতরে Bash Shell চালু করে।

8. `docker start -ai py-c1`  
   => Stopped Container চালু করে এবং তার মূল Process-এর সঙ্গে যুক্ত হয়।

9. `docker stop py-c1`  
   => Running Container বন্ধ করে।

10. `docker rm py-c1`  
   => Stopped Container মুছে দেয়।


## 3. Dockerize a Python File

### Step 1: Create the Project Folder

Create a project folder named `python-docker-app`.

### Project Folder Structure

```text
python-docker-app/
├── app.py
└── Dockerfile
```

- `app.py` => Python code রাখবে।
- `Dockerfile` => Docker Image তৈরির নির্দেশনা রাখবে।

### Step 2: Create a Python File

Create a file named `app.py`.

```python
print("Hello from Docker!")
```

### Step 3: Create a Dockerfile

Create a file named `Dockerfile` (no extension).

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

**Explanation:**
- `FROM` => Python Image নির্ধারণ করে।
- `WORKDIR` => Working Directory সেট করে।
- `COPY` => Python File Container-এ কপি করে।
- `CMD` => Container চালু হলে Python File রান করে।

### Step 4: Build the Docker Image

Run this command inside the project folder:

```bash
docker build -t my-python-app:v1 .
```

=> `Dockerfile` ব্যবহার করে `my-python-app` নামে Image তৈরি করে।

### Step 5: Run the Docker Container

```bash
docker run --name py-app my-python-app:v1
```

=> Image থেকে Container তৈরি করে Python File রান করে।

### Step 6: Check the Output

```text
Hello from Docker!
```

### Step 7: Check the Container

```bash
docker ps -a
```

=> Running ও Stopped সব Container দেখায়।

**Note:** Python Script শেষ হলে Container সাধারণত বন্ধ হয়ে যায়, কিন্তু Container মুছে যায় না.




# 4. Dockerize a Django Project

## 1. Django Project তৈরি করা

প্রথমে `uv` ব্যবহার করে Django Project তৈরি করব।

```bash
mkdir docker_django
cd docker_django

uv init --bare
uv add django

uv run django-admin startproject config .
```


## 2. Project Folder Structure

```text
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


## 3. Dockerfile তৈরি করা

Project-এর Root Folder-এ `Dockerfile` নামে একটি ফাইল তৈরি করো। ফাইলটির কোনো Extension থাকবে না।

```dockerfile
FROM python:3.12-slim

WORKDIR /app

RUN pip install uv

COPY . .

RUN uv sync

EXPOSE 8000

CMD ["uv", "run", "python", "manage.py", "runserver", "0.0.0.0:8000"]
```

## 4. `.dockerignore` তৈরি করা

Project-এর Root Folder-এ `.dockerignore` নামে একটি ফাইল তৈরি করো।

```dockerignore
.venv/
__pycache__/
*.py[cod]
.git/
.env
```



## 5. Docker Image তৈরি করা

Project Folder-এর ভিতরে Terminal খুলে চালাও:

```bash
docker build -t docker-django:v1 .
```

```

## 6. Docker Container চালু করা

```bash
docker run -d --name django-c1 -p 8000:8000 docker-django:v1
```

### Command-এর ব্যাখ্যা

- `docker run` => নতুন Container তৈরি করে এবং চালু করে।
- `-d` => Container-কে Background-এ চালায়।
- `--name django-c1` => Container-এর নাম `django-c1` রাখে।
- `-p 8000:8000` => Host-এর Port `8000`-কে Container-এর Port `8000`-এর সঙ্গে যুক্ত করে।
- `docker-django:v1` => যে Image থেকে Container তৈরি হবে।

### Detached Mode (`-d`) কী?

`-d` ব্যবহার করলে Container Background-এ চলে এবং Terminal অন্য কাজের জন্য ব্যবহার করা যায়।

`-d` ছাড়া চালালে:

```bash
docker run --name django-c1 -p 8000:8000 docker-django:v1
```

Terminal-এ Server-এর Log দেখা যাবে এবং Terminal ওই Process-এর সঙ্গে যুক্ত থাকবে।

`-d`-সহ চালালে:

```bash
docker run -d --name django-c1 -p 8000:8000 docker-django:v1
```

Container Background-এ চলবে।

## 7. Django Application খোলা

Browser-এ এই URL খোলো:

`http://localhost:8000`

Container ঠিকভাবে চালু হলে Django-এর Welcome Page দেখতে পাবে।


### Container-এর Log দেখা

```bash
docker logs django-c1
```

Django Server-এর Output ও Error Log দেখায়।



# Docker Volumes

Docker Volume ব্যবহার করে Host Machine এবং Container-এর মধ্যে ফাইল share করা যায় অথবা Container-এর data সংরক্ষণ করা যায়।

## 1. Dockerfile Configuration

```dockerfile
WORKDIR /app

ENV UV_PROJECT_ENVIRONMENT=/opt/venv
```

- `WORKDIR /app` — Container-এর working directory নির্ধারণ করে।
- `ENV UV_PROJECT_ENVIRONMENT=/opt/venv` — `uv`-এর virtual environment `/opt/venv`-এ রাখে, যাতে project folder mount করলেও virtual environment `/app`-এর মধ্যে না থাকে।

## 2. Bind Mount

```powershell
docker run --rm --name dj-c1 -p 3000:8000 -v "${PWD}:/app" docker-django:v1
```

### ব্যাখ্যা

- `--rm` — Container বন্ধ হলে container-টি স্বয়ংক্রিয়ভাবে মুছে যায়।
- `--name dj-c1` — Container-এর নাম নির্ধারণ করে।
- `-p 3000:8000` — Host-এর port `3000`-কে container-এর port `8000`-এর সঙ্গে যুক্ত করে।
- `-v "${PWD}:/app"` — বর্তমান Windows project directory-কে container-এর `/app` directory-এর সঙ্গে যুক্ত করে।
- `docker-django:v1` — যে Docker image থেকে container তৈরি হবে।

**সুবিধা:** Host machine-এ Python code পরিবর্তন করলে Django development server সাধারণত স্বয়ংক্রিয়ভাবে reload হয়। প্রতিবার image rebuild করতে হয় না।

## 3. Named Volume

### Volume তৈরি

```powershell
docker volume create django_venv
```

### Volume ব্যবহার

```powershell
docker run --rm --name dj-c1 -p 3000:8000 -v "${PWD}:/app" -v django_venv:/opt/venv docker-django:v1
```

এখানে `django_venv` একটি Named Volume, যা container-এর `/opt/venv` directory-এর সঙ্গে যুক্ত।

- Volume-এর নির্দিষ্ট নাম থাকে।
- Docker storage পরিচালনা করে।
- Container মুছে গেলেও Volume-এর data সাধারণত থেকে যায়।
- Virtual environment সংরক্ষণে ব্যবহার করা যায়।

## 4. Anonymous Volume

```powershell
docker run --rm --name dj-c1 -v /data docker-django:v1
```

- Docker নিজে Volume তৈরি করে।
- Volume-এর নাম ব্যবহারকারী নির্ধারণ করে না।
- Container-এর `/data` directory-তে Volume mount হয়।
- `docker volume ls` দিয়ে Volume দেখা যায়।

**নোট:** `--rm` দিয়ে container সরালে তার সঙ্গে যুক্ত anonymous volume-ও সাধারণত মুছে যায়। Named Volume সাধারণত থেকে যায়।

## 5. তিন ধরনের Volume-এর পার্থক্য

| বিষয় | Bind Mount | Named Volume | Anonymous Volume |
|---|---|---|---|
| Storage পরিচালনা | Host directory | Docker | Docker |
| পরিচিতি | Host path | নির্দিষ্ট নাম | Docker-প্রদত্ত নাম |
| Host-এর code edit | সরাসরি প্রতিফলিত হয় | সরাসরি নয় | সরাসরি নয় |
| সাধারণ ব্যবহার | Development | Database ও persistent data | স্বয়ংক্রিয় storage |

## 6. দরকারি Docker Commands

### সব Volume দেখা

```powershell
docker volume ls
```

### Volume-এর বিস্তারিত তথ্য

```powershell
docker volume inspect django_venv
```

### নির্দিষ্ট Volume মুছে ফেলা

```powershell
docker volume rm django_venv
```

### ব্যবহার না হওয়া Volume পরিষ্কার করা

```powershell
docker volume prune
```

**সতর্কতা:** `docker volume prune` ব্যবহার না হওয়া Volume মুছে দিতে পারে। গুরুত্বপূর্ণ data থাকলে আগে যাচাই করবে।

## 7. গুরুত্বপূর্ণ বিষয়

- **Bind Mount:** Host-এর নির্দিষ্ট directory container-এ যুক্ত করে।
- **Named Volume:** Docker পরিচালিত storage-এ data সংরক্ষণ করে।
- **Anonymous Volume:** Docker পরিচালিত storage ব্যবহার করে, কিন্তু Volume-এর নাম আগে থেকে নির্ধারণ করা হয় না।
- **Port Mapping:** `-p 3000:8000` Volume নয়; এটি Host ও Container-এর network port যুক্ত করে।

### Practice Task

1. `django_data` নামে একটি Named Volume তৈরি করো।
2. Volume-টি container-এর `/data` directory-তে mount করো।
3. `/data`-এর মধ্যে একটি file তৈরি করো।
4. Container বন্ধ করে মুছে ফেলো।
5. একই Named Volume নতুন container-এ mount করে পরীক্ষা করো file-টি এখনও আছে কি না।

এভাবে Container এবং Volume-এর lifecycle-এর পার্থক্য বুঝতে পারবে।


# Dockerize Next.js Project

## 1. Folder Structure

```text
next-docker/
├── app/
│   ├── page.tsx
│   ├── layout.tsx
│   └── globals.css
├── public/
├── node_modules/
├── .next/
├── Dockerfile
├── .dockerignore
├── package.json
├── package-lock.json
└── next.config.ts
```

## 2. Dockerfile

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "run", "dev"]
```

## 3. `.dockerignore`

```text
node_modules
.next
.git
.env*
```

## 4. Build Image

```powershell
docker build -t next-docker:v1 .
```

## 5. Run Container with 3 Volumes

```powershell
docker run --rm --name next-c1 `
  -p 3000:3000 `
  -v "${PWD}:/app" `
  -v next_node_modules:/app/node_modules `
  -v /app/.next `
  next-docker:v1
```

## 6. Volume Types

| Volume | Type | Purpose |
|---|---|---|
| `${PWD}:/app` | Bind Mount | Host-এর code container-এ যুক্ত করে |
| `next_node_modules:/app/node_modules` | Named Volume | Dependencies সংরক্ষণ করে |
| `/app/.next` | Anonymous Volume | Next.js cache ও generated files আলাদা রাখে |

**Note:** Named Volume আগে তৈরি করা বাধ্যতামূলক নয়। Docker স্বয়ংক্রিয়ভাবে তৈরি করে।

Open in browser: http://localhost:3000


# Docker Compose — Django + Next.js

## ১. Folder Structure

```text
project/
├── backend/
│   ├── config/
│   ├── manage.py
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── pyproject.toml
│   └── uv.lock
│
├── frontend/
│   ├── app/
│   ├── public/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── package.json
│   └── package-lock.json
│
└── compose.yaml
```

## ২. `compose.yaml`

```yaml
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

## ৩. Docker Compose Commands

```powershell
# Configuration যাচাই
docker compose config

# Image build ও container চালু
docker compose up --build

# Background-এ চালু
docker compose up -d

# Logs দেখা
docker compose logs -f

# Container বন্ধ ও সরানো
docker compose down
```

## ৪. Application URLs

- **Frontend (Next.js):** http://localhost:3000
- **Backend (Django):** http://localhost:8000
- **Django API:** http://localhost:8000/api/hello/

## ৫. Volume-এর ব্যবহার

| Volume | ধরন | কাজ |
|---|---|---|
| `./backend:/app` | Bind Mount | Django-এর source code যুক্ত করে |
| `django_venv:/opt/venv` | Named Volume | Python virtual environment সংরক্ষণ করে |
| `./frontend:/app` | Bind Mount | Next.js-এর source code যুক্ত করে |
| `next_node_modules:/app/node_modules` | Named Volume | Node.js dependencies সংরক্ষণ করে |
| `next_cache:/app/.next` | Named Volume | Next.js-এর generated files ও cache রাখে |

## ৬. গুরুত্বপূর্ণ বিষয়

- `build` — কোন folder-এর `Dockerfile` ব্যবহার করে image তৈরি হবে, তা নির্ধারণ করে।
- `ports` — Host ও container-এর port সংযুক্ত করে।
- `volumes` — Host-এর file যুক্ত করে অথবা Docker-managed storage ব্যবহার করে।
- `depends_on` — Backend-কে আগে চালু করার চেষ্টা করে; তবে Django API সম্পূর্ণ প্রস্তুত হওয়ার নিশ্চয়তা দেয় না।
- `docker compose down` — Container সরায়, কিন্তু Named Volume সাধারণত রেখে দেয়।
- `docker compose down -v` — Compose-এর Named Volume-গুলোও মুছে দেয়; তাই গুরুত্বপূর্ণ data থাকলে সতর্ক থাকবে।










