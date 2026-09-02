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

