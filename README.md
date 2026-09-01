# 🚀 DevTrack – DevOps Learning Tracker

A simple three-tier web application built using **HTML, CSS, JavaScript, Python Flask, MySQL, Nginx, and Docker Compose**.

## 🏗️ Architecture

```text
User
  ↓
Nginx (Reverse Proxy)
  ↓
Flask Backend
  ↓
MySQL Database
```

## 🛠️ Technologies

* Frontend: HTML, CSS, JavaScript
* Backend: Python Flask
* Database: MySQL
* Reverse Proxy: Nginx
* Containerization: Docker
* Orchestration: Docker Compose

## ✨ Features

* Add learning topics
* Update learning progress
* Delete topics
* Track learning status
* Simple and responsive dashboard

## 📁 Project Structure

```text
devtrack/
├── frontend/
├── backend/
├── nginx/
├── mysql/
├── docker-compose.yml
├── .env
├── ai-usage.md
└── README.md
```

## 🚀 Run the Project

Make sure Docker and Docker Compose are installed.

```bash
docker compose up -d --build
```

Open:

```text
http://localhost
```

Check containers:

```bash
docker compose ps
```

Stop the application:

```bash
docker compose down
```

## 👨‍💻 Author

**Hritik Ranjan**
QA Engineer | DevOps & Cloud Learner  
GitHub: https://github.com/hritikranjan1  
LinkedIn: https://www.linkedin.com/in/hritikranjan1/  
Blog: https://blogs.hritikranjan.in  
Website: https://hritikranjan.in