# DockerForDjango
**Docker for Django + PostgreSQL**  
This guide explains how to run a **Django application** with **PostgreSQL** completely inside Docker.  
You won’t need to install Python or PostgreSQL locally — only **Docker** and **Docker Compose**.

---

## **Why use Docker?**
- Same environment for all developers (no “works on my machine” issues).
- No need to install PostgreSQL locally.
- Easy to deploy anywhere with Docker.
- Database stored in volumes for persistence.

---

## 📌 **Setup for Windows**

### **1. Install Docker Desktop**
Download and install from: [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)  
Enable **WSL 2** backend when prompted.

---

### **2. Create Project Folder**
```powershell
mkdir django_docker_postgres
cd django_docker_postgres
```
### **3. Folder Structure**
django_docker_postgres/
│
├── Dockerfile
├── docker-compose.yml
├── .env
├── requirements.txt
└── app/                

### **4. Create Dockerfile**

```
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /code

RUN apt-get update && apt-get install -y libpq-dev gcc

COPY requirements.txt /code/
RUN pip install --no-cache-dir -r requirements.txt

COPY . /code/

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

### **5. Create docker-compose.yml**

```
version: '3.9'

services:
  web:
    build: .
    container_name: django_web
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    env_file:
      - .env
    depends_on:
      - db

  db:
    image: postgres:16
    container_name: postgres_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```
### **6. Create .env**
```
POSTGRES_DB=django_db
POSTGRES_USER=django_user
POSTGRES_PASSWORD=django_pass
```

### **7. Create requirements.txt**
```
Django>=5.0
psycopg2-binary>=2.9

```
### **8. Update Django settings.py**
```
import os

ALLOWED_HOSTS = ["127.0.0.1", "localhost", "192.168.x.x"]  # Replace with your IP

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('POSTGRES_DB'),
        'USER': os.environ.get('POSTGRES_USER'),
        'PASSWORD': os.environ.get('POSTGRES_PASSWORD'),
        'HOST': 'db',
        'PORT': '5432',
    }
}
```
### **9. Build & Run**
```
docker compose up --build
```
### **10. Run Migrations**
```
docker compose run web python manage.py migrate
docker compose run web python manage.py makemigrations
docker compose run web python manage.py createsuperuser
```
### **11. Access the App**
Open in browser:
```
http://127.0.0.1:8000
```

### **12. Useful Commands**
```
docker compose up              # Start containers
docker compose down            # Stop containers
docker compose ps              # List containers
docker compose logs -f web     # View Django logs
docker compose exec web bash   # Enter Django container
```


### **Quick Setup for Linux**
```
sudo apt update
sudo apt install docker.io docker-compose-plugin -y
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```

---

Then follow Steps 2–12 from the Windows guide — commands are the same.











