# Part C — Decisions under Constraints

## C1. Allocate the 4 GB

I need to divide the 4 GB RAM between the Operating System, Nginx, MariaDB, Redis and Application Workers.

My initial allocation would be:

1. Operating System - 1 GB
2. Nginx - 100 MB
3. MariaDB - 1 GB
4. Redis - 500 MB
5. Application Workers - 1 GB
6. Free/Buffer - 400 MB

If the server starts running out of memory, the first thing I would reduce is the **number of application workers**, because more workers will use more RAM.

I would check the server using:

```bash
free -h
```

I would also monitor the application and database memory usage.

If I see high memory usage, slow application response, workers restarting, or the server using swap, it would show that my RAM allocation needs to be adjusted.

I would check the actual usage first and then change the allocation.

---

## C2. Backups I would be willing to rely on

I would take a **database backup every day**.

I would keep:

* Daily backups for 30 days
* Weekly backups for 3 months
* One copy outside the server, such as cloud/object storage

The budget is limited, so I would choose a low-cost storage option for the external backup instead of keeping all backups on the server.

I would not keep all backups only on the same server. If the server or disk fails, the backups could also be lost.

### My backup targets:

**RPO: 24 hours**

This means I can accept losing up to 24 hours of data in the worst case.

**RTO: 4 hours**

This means I will try to bring the application back within 4 hours.

I would not just assume that the backup is working. Once every month, I would restore a backup in a test environment and check that the database and important application data are available.

We can also plan a **mock drill** on a non-working day to test the complete backup restoration process.

---

## C3. Respond to a colleague

I would not run the migration directly on production just because a backup was taken in the morning.

The migration can fail in the middle. Some database changes may happen while other changes fail. This can make the database and application stop working correctly.

Also, the morning backup may not contain the data created after the backup. If we restore it, we may lose that newer data.

I would suggest running the migration **after working hours**, because during working hours users are using the application. During the migration, the application may not work properly and users may face problems.

### I would do this instead:

1. Take a fresh backup before the migration.
2. Check that the backup was successful.
3. Test the migration first in a test environment.
4. Plan a maintenance window.
5. Inform the users before starting.
6. Run the migration during low-usage hours.
7. Check the application after migration.
8. If something fails, use the tested recovery plan.

This will reduce the chance of data loss and reduce the impact on users.

---

## C4. Deployment without a second server

**With only one VM, I cannot honestly promise true zero-downtime deployment.**

Normally, zero-downtime deployment needs the old and new application versions to run together so users can continue using the old version while the new version is deployed.

Our server has only **2 vCPU and 4 GB RAM**, so running two complete application setups may put too much load on the server.

The best option is to use a **short planned maintenance window**.

I would:

1. Take a fresh backup.
2. Check the backup.
3. Deploy during low-usage hours.
4. Put the application in maintenance mode.
5. Update the application and database.
6. Start the application.
7. Check login and important functions.
8. If there is a problem, use the recovery plan.

I would tell the department that true zero-downtime deployment is not possible with the current single-server setup. We can instead keep the downtime as short as possible by deploying during low-usage hours and having a tested backup and recovery plan.
