# Create Test Data

In this step, you will create a small **test dataset** in Weaviate.

This matters because a Vector DB is only “working” in a meaningful way if you can:

1. **Define a schema / collection** (what kind of objects exist?)
2. **Insert objects** (store data)
3. **Read objects back** (verify data is actually stored)
4. (Later) **Back up the data** and restore it

This section creates a simple collection called `Note` with one field:

- `text` (a string field)

We will then insert a few notes and confirm they exist.

---

## Before you begin

### 1) Make sure Weaviate is running

From the project root, run:

```bash
docker ps
```

### What success looks like

You should see a container named `weaviate` with a status like `Up ...`.

If you do **not** see it, start Weaviate:

```bash
docker compose up -d
```

---

### 2) Make sure you are using the admin key

Creating schema and inserting data are **write operations**, so they must be done with the **admin** API key.

In this lab:
- Admin key: `admin-key-123`
- Viewer key: `viewer-key-123`

---

## Step 1: Check if the `Note` collection already exists (recommended)

Before creating anything, check your current schema:

```bash
curl -s http://localhost:8080/v1/schema \
  -H "Authorization: Bearer admin-key-123"
```

### What success looks like

You will see JSON output. If you see:

```json
{"classes":[]}
```

that means there are no classes yet.

If you already see `"class":"Note"` somewhere in the output, the class already exists and you should skip the creation step (or delete/reset first).

---

## Step 2: Create the `Note` class (schema)

This command creates a class named `Note` with one property: `text`.

> **Important:** Some Weaviate versions handle schema writes differently.
> If your schema creation fails with an endpoint error (405/422), use the Python approach listed in the “If you hit errors” section below.

### Create the schema using curl

```bash
curl -i -X POST http://localhost:8080/v1/schema \
  -H "Authorization: Bearer admin-key-123" \
  -H "Content-Type: application/json" \
  -d '{
    "classes": [
      {
        "class": "Note",
        "vectorizer": "none",
        "properties": [
          { "name": "text", "dataType": ["text"] }
        ]
      }
    ]
  }'
```

---

### What this command does

- `POST /v1/schema` asks Weaviate to create schema
- `Authorization: Bearer admin-key-123` authenticates as the admin user
- The JSON body defines:
  - a class called `Note`
  - vectorizer set to `"none"` (no external embedding module needed)
  - one field: `text`

---

### What success looks like

You should see an HTTP response status like:

- `HTTP/1.1 200 OK`
- or `HTTP/1.1 201 Created`

If you see a 200/201, the schema exists and you can continue.

---

## Step 3: Verify the schema exists

```bash
curl -s http://localhost:8080/v1/schema \
  -H "Authorization: Bearer admin-key-123"
```

### What success looks like

You should see `"class":"Note"` somewhere in the output.

If you do not, stop here and troubleshoot before continuing.

---

## Step 4: Insert test objects (Notes)

Now we will insert a few objects into the `Note` class.

### Insert Note 1

```bash
curl -i -X POST http://localhost:8080/v1/objects \
  -H "Authorization: Bearer admin-key-123" \
  -H "Content-Type: application/json" \
  -d '{
    "class": "Note",
    "properties": {
      "text": "Weaviate is running securely with API keys and RBAC."
    }
  }'
```

### Insert Note 2

```bash
curl -i -X POST http://localhost:8080/v1/objects \
  -H "Authorization: Bearer admin-key-123" \
  -H "Content-Type: application/json" \
  -d '{
    "class": "Note",
    "properties": {
      "text": "The viewer user should be able to read but not write."
    }
  }'
```

### Insert Note 3

```bash
curl -i -X POST http://localhost:8080/v1/objects \
  -H "Authorization: Bearer admin-key-123" \
  -H "Content-Type: application/json" \
  -d '{
    "class": "Note",
    "properties": {
      "text": "Backups protect against accidental deletion and corruption."
    }
  }'
```

---

### What success looks like

Each insert should return a status like:

- `HTTP/1.1 200 OK`
- or `HTTP/1.1 201 Created`

The response body will include an `id` (a UUID for the object).

If inserts fail, do not continue until inserts succeed.

---

## Step 5: Read the data back (admin)

Now verify that objects are actually stored.

```bash
curl -s "http://localhost:8080/v1/objects?class=Note&limit=10" \
  -H "Authorization: Bearer admin-key-123"
```

### What success looks like

You should see JSON containing objects with the `text` values you inserted.

This confirms:
- schema exists
- data is persisted
- API key authentication works for reads/writes

---

## Step 6: Optional security check — viewer can read but cannot write

### Viewer read (should succeed)

```bash
curl -s "http://localhost:8080/v1/objects?class=Note&limit=10" \
  -H "Authorization: Bearer viewer-key-123"
```

If RBAC is configured correctly, the viewer should see results.

### Viewer write (should fail)

```bash
curl -i -X POST http://localhost:8080/v1/objects \
  -H "Authorization: Bearer viewer-key-123" \
  -H "Content-Type: application/json" \
  -d '{
    "class": "Note",
    "properties": {
      "text": "Viewer should NOT be able to insert this."
    }
  }'
```

**Expected result:** viewer insert is rejected (usually `403 Forbidden`).

If the viewer can write, RBAC is not working and you should stop.

---

## If you hit schema creation errors (common beginner issue)

Sometimes `/v1/schema` behaves differently depending on Weaviate version and configuration.

If your schema creation step returns:
- `405 Method Not Allowed`
- `422 Unprocessable Entity`
- or other confusing schema endpoint errors

Use the official client approach instead.

### Option: Create schema using the Python RBAC script

If you already ran the RBAC script in the previous section, it should have created the `Note` collection for you.

To confirm:

```bash
python scripts/rbac_test.py
```

Then re-run the verification step:

```bash
curl -s http://localhost:8080/v1/schema \
  -H "Authorization: Bearer admin-key-123"
```

If `Note` exists, skip the curl schema creation and just insert objects.

---

## Summary: What you achieved

At this point, you have:

- created (or verified) a schema (`Note`)
- inserted real objects (test data)
- verified reads work
- optionally confirmed RBAC (viewer read-only)

This gives you a working dataset that you can now back up in the next lesson.

---

Next: weaviate-secure-lab/docs/09-Download-Data.md
