# Verify Weaviate is Running

In this step, you will verify that the Weaviate vector database is:
- running correctly
- reachable on your local machine
- enforcing authentication as expected

This step is critical. A container that is “running” is not necessarily **working** or **secure**. You will explicitly test both conditions.

---

## Before you begin

Make sure:
- Docker is running
- The Weaviate container is started
- You are in the **repository root**, where `docker-compose.yml` is located

If needed, confirm with:

```bash
docker ps
```

You should see a container named `weaviate` with a status of `Up`.

---

## Why we test unauthenticated vs authenticated access

Before working with data, schemas, or permissions, we must confirm that Weaviate is enforcing authentication correctly.

A running service is not necessarily a **secure** service.

This step verifies two critical security properties:

1. **Unauthorized users cannot access the API**
2. **Authorized users can access the API**

Both must be true for the system to be considered safely configured.

---

### What risk this test is addressing

Vector databases often store:
- proprietary documents
- embeddings derived from sensitive text
- internal knowledge bases
- model inputs and outputs

If anonymous access is enabled by mistake:
- anyone with network access could query metadata
- schema and configuration details could be exposed
- data leakage becomes possible without authentication

Testing unauthenticated access ensures this risk is mitigated.

---

### Why failure is the expected outcome

In the first test, we intentionally make a request **without credentials**.

If Weaviate is configured securely:
- the request should be rejected
- the API should return `401 Unauthorized`
- the error message should indicate authentication is required

A failure here is **success** from a security standpoint.

If this request succeeds, the system is misconfigured and must not be used.

---

### Why we test authenticated access next

After confirming that anonymous access is blocked, we then verify that:

- valid API keys are accepted
- authorized users can successfully access the API
- the authentication mechanism works end to end

This confirms that security controls are not only present, but usable.

---

### What this proves before continuing

By completing both tests, you have proven that:

- Weaviate is running and reachable
- authentication is enforced
- access is intentionally restricted
- the system is safe to continue configuring

Only after these checks pass should you proceed to defining users, roles, and permissions.

---

The next sections build directly on this trust boundary.


## Why we use `/v1/meta`

The `/v1/meta` endpoint:
- does not modify data
- does not require a schema
- returns basic information about the running Weaviate instance

This makes it ideal for testing connectivity and authentication without risk.

---

## Test without authentication (expected to fail)

Run the following command **without** any API key:

```bash
curl -i http://localhost:8080/v1/meta
```

### What this command does
- `curl` sends an HTTP request
- `-i` includes HTTP response headers
- The request targets your local Weaviate instance

---

### What success looks like (failure is expected)

You should see an HTTP response similar to:

```
HTTP/1.1 401 Unauthorized
```

This means:
- Weaviate is running
- Weaviate is reachable
- Anonymous access is correctly disabled

If this request **succeeds**, your security configuration is wrong and must be fixed before continuing.

---

## Test with admin API key (expected to succeed)

Now test the same endpoint using a valid API key.

Run:

```bash
curl -s http://localhost:8080/v1/meta \
  -H "Authorization: Bearer admin-key-123"
```

### What this command does
- Adds an `Authorization` header
- Uses the admin API key defined in `docker-compose.yml`
- Authenticates you as an authorized user

---

### What success looks like

You should see a JSON response similar to:

```json
{
  "hostname": "http://localhost:8080",
  "version": "1.x.x",
  "modules": { ... }
}
```

This confirms:
- Weaviate is running
- Authentication is enforced
- Valid API keys work
- The admin user has access

---

## If something goes wrong

### Error: `Connection refused`
This usually means:
- The container is not running
- The wrong port is being used

**Fix**
- Run `docker ps`
- Restart with `docker compose up -d`

---

### Error: `401 Unauthorized` with API key
This means:
- The API key is wrong
- The header was formatted incorrectly

**Fix**
- Double-check the key in `docker-compose.yml`
- Make sure the header starts with `Bearer`

---

### No response or hanging request
This may mean:
- Docker is running but Weaviate failed to start

**Fix**
- Run:
  ```bash
  docker compose logs
  ```
- Look for startup errors
- Do not continue until resolved

---

## Why this step matters

At this point, you have proven:
- The vector database is running
- Network access is correct
- Anonymous access is blocked
- Authenticated access works

These checks form the foundation for all later steps involving:
- API keys
- role-based access control
- data ingestion
- backups

If any test in this section fails, stop and fix it before continuing.

---

Next: [Authentication (API Keys)](06-authentication.md)
