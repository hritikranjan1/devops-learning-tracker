# Part D — Review and Walkthrough

## Block Merge Comments

### 1. Tests can fail but pipeline still passes

```yaml
run: pytest tests/ || true
```

**Comment:**

> **Block:** `|| true` ignores test failures. Even if tests fail, the pipeline will continue and deploy to production. This should be removed.

Use:

```yaml
run: pytest tests/
```

---

### 2. Secret is printed in logs

```bash
echo "Authenticating with token $REGISTRY_TOKEN"
```

**Comment:**

> **Block:** The registry token should not be printed in the CI logs. Secrets should stay hidden.

Remove the `echo` command.

---

### 3. Docker registry login is missing 

The workflow has `REGISTRY_TOKEN`, but it does not actually use it to log in to the Docker registry.

**Comment:**

> **Block:** The workflow should log in to the Docker registry before pushing the image.
>

> Missing user name -   REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}

---

## Non-Blocking Comments

### 4. `actions/checkout@master`

```yaml
- uses: actions/checkout@master
```

**Comment:**

> **Suggestion:** Use a fixed version instead of `@master` so the workflow does not unexpectedly change.

Because - The master branch can be updated in the future.

For example:

Today → checkout@master → Version A

Next month → checkout@master → Version B

So the same workflow may behave differently later, even though you did not change your workflow.
For a production deployment, we normally want the action version to be predictable.

---

### 5. Using `latest`

```bash
docker build -t registry.example.com/grants:latest .
```

**Comment:**

> **Suggestion:** Use a unique tag such as the Git commit ID. It will make it easier to identify and rollback a deployment.

---

### 6. No deployment health check

After:

```bash
docker compose up -d
```

there is no check to confirm that the application is actually working.

**Comment:**

> **Suggestion:** Add a simple health check after deployment.

---

## Things That Are Fine

### 7. GitHub Secret

```yaml
${{ secrets.REGISTRY_TOKEN }}
```

This is correct. The secret is stored in GitHub Secrets.

