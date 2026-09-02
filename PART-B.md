# Part B — Diagnose a Faulty Deployment

## Problem 1 — Application Restarts (S1-The application restarts on its own two or three times a day.)
### 1. Problem
The application has a memory limit of only **256 MB**. (- Config in compose file for app service)

```yaml
deploy:
  resources:
    limits:
      memory: 256M
```
This can cause the application container to stop if it uses more memory means maybe Docker or OS Kill that container after it reached the memory
### 2. Symptom
This can explain **S1**, where the application restarts 2–3 times a day.
### 3. How I would confirm
I would check the container status:
```bash
docker inspect $(docker compose ps -q app) \
--format 'Status={{.State.Status}} ExitCode={{.State.ExitCode}} OOMKilled={{.State.OOMKilled}} RestartCount={{.RestartCount}}'
```
And After I also would also check memory usage:
```bash
docker stats
```
If `OOMKilled=true` and memory usage is close to 256 MB, this confirms the issue.
### 4. Fix
Increase the memory limit based on actual usage, for example:
```yaml
deploy:
  resources:
    limits:
      memory: 512M
```
---

## Problem 2 — Database Data Loss (S2 - After the server was rebooted for operating system patching last month, all data was gone.)
### 1. Problem
The database volume is mounted to:

```yaml
volumes:
  - dbdata:/var/lib/mysql/data
```
MariaDB normally stores its data under:
```text
/var/lib/mysql
```
So the volume should be mounted to `/var/lib/mysql`.

### 2. Symptom
This can explain **S2**, where database data was lost after the server/container was recreated.
The reason `docker compose restart` did not lose data is that it only restarts the existing container.

### 3. How I would confirm

Check the volume mount:
```bash
docker inspect $(docker compose ps -q db) --format '{{json .Mounts}}'
```
Then check where the database files are:
```bash
docker compose exec db ls -lah /var/lib/mysql
```
If the database files are not inside the mounted volume, the volume is not protecting the actual database data.

### 4. Fix
Change:
```yaml
volumes:
  - dbdata:/var/lib/mysql
```
---

## Problem 3 — Database Not Ready After Reboot (S4 - Roughly one server reboot in three, the application starts unable to reach the database and
somebody has to restart it manually. )
### 1. Problem
The application only has:
```yaml
depends_on:
  - db
```
This means Docker starts the DB container before the app, but it does not wait until MariaDB is fully ready to accept connections.
### 2. Symptom
This explains **S4**, where sometimes the application starts but cannot connect to the database.
### 3. How I would confirm
Check both logs with timestamps:
```bash
docker compose logs --timestamps db
docker compose logs --timestamps app
```
If the app tries to connect to the database before MariaDB is ready, this confirms the problem.
### 4. Fix
Add a database healthcheck and make the app wait for the database to become healthy.
Example:
```yaml
healthcheck:
  test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
  interval: 10s
  timeout: 5s
  retries: 5
```
Then:

```yaml
depends_on:
  db:
    condition: service_healthy
```
---


## Problem 4 — HTTPS Links (S3)

### 1. Problem

Nginx is missing some forwarded headers:

```nginx
location / {
    proxy_pass http://app:8000;
    proxy_set_header X-Real-IP $remote_addr;
}
```

The load balancer is using HTTPS, but Nginx may not be passing the original HTTPS information to the application.

### 2. Symptom

This can explain **S3**, where some links redirect to unreachable HTTP addresses.

The browser security warning should also be checked separately at the load balancer level.

### 3. How I would confirm

Open the browser's **Developer Tools → Network**.

After login, check the URL/redirect of the failed link.

If the user starts on:

```text
https://...
```

but the application redirects to:

```text
http://...
```

then this confirms the forwarded protocol issue.

I would also check the active Nginx configuration:

```bash
docker compose exec proxy nginx -T
```

### 4. Fix

Add the required forwarded headers:

```nginx
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
proxy_set_header Host $host;
```

The load balancer should be configured to send the original protocol correctly.

---
