# Local Setup Guide and Docker compose guide – React + Express + MySQL

This document explains, step‑by‑step, how to run a simple three‑tier application locally first then on containers:

* Frontend: React
* Backend: Express (Node.js)
* Database: MySQL

The goal is to run everything on your laptop before moving to containers.

---

## Architecture (Local Machine)

Browser  →  React (localhost:3000)
→
Express API (localhost:3500)
→
MySQL Server (localhost:3306)

In local development, **localhost means your laptop**.

---

## 1. Install MySQL Server

MySQL is a background service that stores your data and listens on port 3306.

```bash
sudo apt update
sudo apt install mysql-server -y
```

Start and enable the service:

```bash
sudo systemctl start mysql
sudo systemctl enable mysql
```

Verify:

```bash
systemctl status mysql
```

You should see: active (running).

---

## 2. Set MySQL root password (important)

Login using the local system account:

```bash
sudo mysql
```

Inside the MySQL shell, run:

```sql
ALTER USER 'root'@'localhost'
IDENTIFIED WITH mysql_native_password BY 'mysql123';

FLUSH PRIVILEGES;
```

Exit:

```sql
exit;
```

Test login using the new password:

```bash
mysql -u root -p
```

---

## 3. Create the application database and tables

Login:

```bash
mysql -u root -p
```

Create database and tables:

```sql
CREATE DATABASE school;

USE school;

CREATE TABLE student (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(40),
  roll_number INT,
  class VARCHAR(16)
);

CREATE TABLE teacher (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(40),
  subject VARCHAR(40),
  class VARCHAR(16)
);
```

Verify:

```sql
SHOW TABLES;
```

---

## 4. If you are using Node.js 18 version, you can Upgrade Node.js from 18 to 20+

Check current version:

```bash
node -v
```

Remove old Node.js:

```bash
sudo apt remove nodejs -y
```

Add Node 20 repository:

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
```

Install Node 20:

```bash
sudo apt install nodejs -y
```

Verify:

```bash
node -v
npm -v
```

---

## 5. Backend (Express) setup

Go to backend folder:

```bash
cd backend
```

Install dependencies:

```bash
npm install
```

you will find `.env` inside the backend directory:

```env
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=mysql123
DB_NAME=school
DB_PORT=3306

```

Explanation:

* DB_HOST=localhost → MySQL is running on your laptop
* Express will listen on port 3500

Start the backend:

```bash
npm start
```

```bash
curl http://localhost:3500
```

---

## 6. Frontend (React) setup

Go to frontend folder:

```bash
cd frontend
```

Install dependencies:

```bash
npm install
```

you will find `.env` file inside the frontend directory:

```env
REACT_APP_API_URL=http://localhost:3500
```

Explanation:
This URL is used by the browser. The browser must be able to reach the backend.

Start the frontend:

```bash
npm start
```

Open in your browser:

[http://localhost:3000](http://localhost:3000)

---

## 7. How communication works locally

* React runs on: localhost:3000
* Express runs on: localhost:3500
* MySQL runs on: localhost:3306

Flow:

Browser → React → Express → MySQL

---

## 8. Very short EC2 note (for later)

If you later move this to one EC2 instance:

* MySQL will still be accessed by the backend using:
  localhost:3306

* Your browser will access React and Express using the EC2 public IP:

Example:

Frontend API URL becomes:

http://EC2_PUBLIC_IP:3500

Backend DB host remains:

localhost

---

Always make sure this local setup works before using Docker


# Dockerizing and Deploying Three-Tier Full Stack Applications

- Deployed multi-tier full-stack applications using Docker and Docker Compose, ensuring seamless integration and scalability.
- Configured custom networks and volumes for efficient container communication and data persistence across Node.js and React.js applications.
- Managed MySQL databases within Docker containers, ensuring data integrity and availability through effective volume management.


## Prerequisites

Before you begin, make sure you have the following installed:

- [Docker](https://www.docker.com/get-started)
- [Docker Compose](https://docs.docker.com/compose/install/)

## Project Structure

- **frontend**: Dockerfile and related files for the React.js frontend.
- **backend**: Dockerfile and related files for the Node.js backend.
- **docker-compose.yml**: Docker Compose configuration file.
- **student-teacher-app**: Code for the frontend application.
- **backend**: Code for the backend application.

## Deployment Steps

1. **Clone the Repository:**
   ```bash
   git clone <repository-url>
   cd <repository-directory>
   ```

2. **Build and Run Docker Containers:**
    Use Docker Compose to build and run all containers:
    ```bash
    docker-compose up -d
    ```

3. **Access the Application:**

    Open your favourite browser and visit http://localhost:80. Explore the MERN stack application!

## Create Tables Manually by Exec Into MySQL Container

This section explains how to enter the running MySQL container and create the required tables for the application.

### Step 1 – Make sure the database container is running

From your project root:

```bash
    docker ps
```

You should see a container named:


mysql-container


### Step 2 – Exec into the MySQL container

Run:

```bash
    docker exec -it mysql-container mysql -u root -p
```

When prompted, enter the root password you defined in docker‑compose:


mysql123


You should now see:


mysql>


### Step 3 – Select the database

The database is created automatically because of the following setting in docker‑compose:


MYSQL_DATABASE=school


Select it:

```sql
USE school;
```

### Step 4 – Create the tables

Run the following SQL commands:

```sql
CREATE TABLE IF NOT EXISTS student (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(40),
  roll_number INT,
  class VARCHAR(16)
);

CREATE TABLE IF NOT EXISTS teacher (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(40),
  subject VARCHAR(40),
  class VARCHAR(16)
);

```

### Step 5 – Verify the tables

```sql
SHOW TABLES;
```
You should see:


student
teacher


### Step 6 – Exit the database

```sql
exit
```

### Important note about persistence

Because your MySQL service uses this volume:


- mysql-data:/var/lib/mysql


The tables are stored permanently.

You only need to create the tables once.

If the container is deleted and recreated using the same volume, the tables will still be available.

The tables will only be lost if you remove the volume, for example:

```bash
docker compose down -v
```
