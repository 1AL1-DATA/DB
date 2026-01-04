
# 1. Navigate to project
cd ~/games-multi-user

# 2. Start containers
docker compose up -d

# 3. Check status
docker ps

# 4. Access services
#    Metabase: http://127.0.0.1:3000
#    Database: localhost:3306
Windows:
powershell
# 1. Open PowerShell as Administrator
cd C:\Users\YourUsername\games-multi-user

# 2. Start Docker Desktop (make sure it's running!)

# 3. Start containers
docker compose up -d

# 4. Check status
docker ps

# 5. Access services
#    Open browser to: http://localhost:3000
#    Database: localhost:3306
🔐 Credentials
All passwords are in .env file:

bash
# View passwords (Linux/Mac)
cat ~/games-multi-user/.env

# View passwords (Windows PowerShell)
Get-Content .env
Default Admin Access:
Metabase: admin@games.local / (check .env for METABASE_ADMIN_PASSWORD)

Database Root: root / (check .env for DB_ROOT_PASSWORD)

📊 Metabase Setup (First Time)
Log in as admin: http://localhost:3000

Email: admin@games.local

Password: From .env (METABASE_ADMIN_PASSWORD)

Create lecturer user:

Go to Admin → People → Add someone

Email: lecturer@university.edu

Password: Use LECTURER_METABASE_PASSWORD from .env

Group: All users (read-only)

Configure data:

Database connection should be auto-configured

Explore games database tables

Create dashboards for different user roles

🛠️ Management Commands
Start/Stop:
bash
# Start all services
docker compose up -d

# Stop all services
docker compose down

# Stop but keep volumes
docker compose stop

# View logs
docker compose logs -f

# Restart
docker compose restart
Database Operations:
bash
# Connect to database
mysql -h 127.0.0.1 -u root -p

# Backup database
docker exec games-db-multi mysqldump -u root -p games > backup_$(date +%Y%m%d).sql

# Restore backup
docker exec -i games-db-multi mysql -u root -p games < backup_file.sql
User Testing:
bash
# Test lecturer access (read-only)
mysql -h 127.0.0.1 -u lecturer -p

# Test analyst access (read/write)
mysql -h 127.0.0.1 -u analyst -p
🔒 Security Features
Localhost only: Services bound to 127.0.0.1

Role-based access: Different permissions per user

Secure passwords: Random 24-character passwords

File permissions: .env set to chmod 600

Container security: no-new-privileges:true

⚠️ Troubleshooting
Port Conflicts:
bash
# Check if ports are in use
netstat -ano | findstr :3000
netstat -ano | findstr :3306

# Change ports in docker-compose.yml if needed
# Edit: "127.0.0.1:NEW_PORT:3306"
Docker Issues:
bash
# Check if Docker is running
docker version

# Check container logs
docker logs games-db-multi
docker logs games-metabase-multi

# Rebuild containers
docker compose down
docker compose up -d --build
Database Issues:
bash
# Check if database initialized
docker exec games-db-multi ls /docker-entrypoint-initdb.d/

# Check MariaDB logs
docker exec games-db-multi tail -f /var/log/mysql/error.log
📁 File Locations
Linux/Mac:
Project: ~/games-multi-user/

Database data: ~/games-multi-user/mysql_data/

Metabase data: ~/games-multi-user/metabase_data/

Windows:
Project: C:\Users\[Username]\games-multi-user\

Database data: C:\Users\[Username]\games-multi-user\mysql_data\

Metabase data: C:\Users\[Username]\games-multi-user\metabase_data\

🔄 Migration from Local MariaDB
If migrating from existing MariaDB installation:

Backup: mysqldump -u root -p games > games_backup.sql

Stop local MariaDB: sudo systemctl stop mariadb

Copy backup to ~/games-multi-user/

Run docker compose up -d

🎯 What's Been Accomplished
✅ Containerized setup: MariaDB + Metabase in Docker

✅ Multi-user security: 4 distinct user roles with appropriate permissions

✅ Data migration: games database restored from backup

✅ Secure configuration: Passwords, localhost binding, file permissions

✅ Documentation: This README + access instructions

📞 Support
Quick Checks:
bash
# Is everything running?
docker ps

# Can I access Metabase?
curl -I http://localhost:3000

# Is database responding?
mysql -h 127.0.0.1 -u root -p -e "SELECT 1;"
Common Issues Solved:
✅ Port conflicts (3306/3000)

✅ Password generation issues

✅ Volume permissions

✅ Database initialization

Remember:

Passwords are in .env file

Services only accessible from local machine

Lecturer Metabase account needs manual creation

Regular backups recommended

Happy Analyzing! 📈

# Create a network from within docker

? docker network create metabase-network ?
docker ps # Check whats running
docker logs games-metabase-multi --tail 50 #check why metabase stuck on loading 

docker exec -it mariadb mysql -uroot -p # --old Fix permissions in the running MariaDB container

docker exec -it games-db-multi mariadb -uroot -p # --new Correct 
# Accessing mariadb games within the docker to execute SQL
# Adding the correct user to db
DB_ROOT_PASSWORD=Jw02Ct6SWevXSn2BEf7eSeFa
CREATE USER 'metabase_app'@'%' IDENTIFIED BY 'pARquxzH9FWmQG5Q0pzQjb3H'; sometimes it will need to be modified with specific host adress instead of %
GRANT ALL PRIVILEGES ON games.* TO 'metabase_app'@'%';
FLUSH PRIVILEGES;

USE core_g SOURCE /home/a/games-multi-user -> pwd for docker copy backup into it.

## Both your Metabase and MariaDB containers are running on the same host, and you mapped the ports to 127.0.0.1. That means:
## From your host machine, Metabase can connect to MariaDB at 127.0.0.1:3306.

Field	Value
Database type	MySQL (or MariaDB compatible)
Host	games-db-multi
Port	3306
Database name	games2
Username	root
Password	Jw02Ct6SWevXSn2BEf7eSeFa

# Had to load the db into the container again. Since original db was taken over by metabase.
docker cp /home/a/games-multi-user/games_backup.sql games-db-multi:/tmp/games_backup.sql # copying the sqlalchemy made db backup into docker staging folder.

# If you want to restart the apps
docker start games-db-multi
docker start games-metabase-multi

Absolutely! Let’s go step by step with **all the Docker commands we’ve been using**, fully annotated so you understand what each does. I’ll include examples specific to your `MariaDB` and `Metabase` setup.

#Below a list of usefull docker commands.
---

## **1. List running containers**

```bash
docker ps
```

* Shows all **currently running containers**
* Columns include:

  * `CONTAINER ID` → unique short ID for each container
  * `IMAGE` → which Docker image the container is using
  * `NAMES` → friendly name you can reference

**Example output:**

```
CONTAINER ID   IMAGE               NAMES
b511b78a1886   mariadb:11.8.3      games-db-multi
33bd5af7cda6   metabase/metabase    games-metabase-multi
```

---

## **2. List all containers (running or stopped)**

```bash
docker ps -a
```

* Shows **all containers**, including stopped ones
* Useful for checking containers that didn’t start automatically

---

## **3. Start a container**

```bash
docker start games-db-multi
docker start games-metabase-multi
```

* `start` resumes a **stopped container**
* Can start multiple containers at once:

```bash
docker start games-db-multi games-metabase-multi
```

---

## **4. Stop a container**

```bash
docker stop games-db-multi
docker stop games-metabase-multi
```
* Stops multiple containers at once similarly to `start`
---
## **5. Enter a container shell**
```bash
docker exec -it games-db-multi bash
```

* `exec` = execute a command **inside a running container**
* `-it` = interactive terminal
* `bash` = opens a shell inside the container

**Use case:** run `mysql -u root -p` inside MariaDB

---

## **6. Copy files into a container**

```bash
docker cp /home/a/games-multi-user/games_backup.sql games-db-multi:/tmp/games_backup.sql
```

* `docker cp` = copy files **between host and container**
* Syntax: `docker cp <source> <container>:<destination>`
* Use for SQL backups, config files, etc.

---

## **7. Import SQL into MariaDB inside container**

**Option A: Interactive inside container**

```sql
mysql -u root -p
CREATE DATABASE games2;
USE games2;
SOURCE /tmp/games_backup.sql;
```

**Option B: Direct from host db loading **

```bash
docker exec -i games-db-multi mysql -u root -p games2 < /home/a/games-multi-user/games_backup.sql
```

* `-i` = keep STDIN open
* Streams the SQL file from host directly into the container’s MySQL

---

## **8. Update container restart policy**

```bash
docker update --restart unless-stopped games-db-multi
docker update --restart unless-stopped games-metabase-multi
```

* Ensures containers **restart automatically after a reboot**
* `unless-stopped` = restarts automatically unless you manually stop it
* Other options:

  * `no` → default, does **not** restart automatically
  * `always` → always restart, even if manually stopped

---

## **9. Check container logs**

```bash
docker logs -f games-db-multi
docker logs -f games-metabase-multi
```

* Shows live logs (`-f` = follow)
* Helps debug startup errors or check status

---

## **10. Inspect container environment**

```bash
docker inspect games-db-multi
```

* Shows full JSON details of the container
* Useful to find:

  * Ports (`3306`)
  * Environment variables (`MYSQL_ROOT_PASSWORD`)
  * Volumes and mounts

---

## **11. (Optional) Remove a container**

```bash
docker rm games-db-multi
```

* Deletes a container **after stopping it**
* Doesn’t delete the image, just the container instance

---

## **12. (Optional) Run a new container**

```bash
docker run -d --name games-db-multi -e MYSQL_ROOT_PASSWORD=my-secret -p 3306:3306 mariadb:11.8.3
```

* `run` = create and start a new container
* `-d` = detached mode (runs in background)
* `--name` = friendly container name
* `-e` = environment variables
* `-p` = map host port → container port

---


sudo mysql -u root -S /var/run/mysqld/mysqld3307.sock #access mariadb on port 3307
a@debian:~$ docker exec -it games-metabase-multi sh 

mkdir games-multi-user
cd games-multi-user

# Extract MySQL data
tar xzvf mysql_data_backup.tar.gz

# Extract docker-compose and init scripts
tar xzvf docker_compose_and_init.tar.gz

# Load images
docker load -i metabase_image.tar
