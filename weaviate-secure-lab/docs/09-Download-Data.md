# Importing External Data into Weaviate

In previous sections, you:

- Created a collection (`Note`)
- Verified authentication
- Verified RBAC enforcement

Now we will simulate a realistic workflow:

> Download external data → Import it securely → Verify ingestion

This mirrors real production systems where data comes from APIs, files, logs, or third-party sources.

---

# Why This Step Matters

Data ingestion is one of the highest-risk operations in a database system.

If improperly handled, it can:

- Bypass validation
- Overwrite existing objects
- Insert malformed or malicious data
- Cause performance instability
- Circumvent RBAC if misconfigured

In this section, we will:

- Create an external dataset file
- Write a controlled import script
- Import data as an admin
- Confirm RBAC prevents unauthorized ingestion

---

# Step 1 — Confirm You Are in the Project Root

Run:

```bash
ls
```

You should see:

```
README.md
docker-compose.yml
docs
scripts
```

If not, navigate to the correct folder before continuing.

---

# Step 2 — Create a Sample External Data File

We will simulate downloaded data by creating a JSON file.

Create the file:

```bash
nano sample_data.json
```

Paste the following content exactly:

```json
[
  { "text": "Weaviate is a vector database." },
  { "text": "RBAC protects sensitive operations." },
  { "text": "API keys enforce authentication." }
]
```

---

## Save and Exit nano

### macOS / Linux

1. Press `CTRL + O`  
2. Press `Enter`  
3. Press `CTRL + X`

### Windows (Git Bash / WSL)

Same steps:

1. `CTRL + O`  
2. `Enter`  
3. `CTRL + X`

---

## Confirm File Exists

```bash
ls -lh sample_data.json
```

Verify contents:

```bash
cat sample_data.json
```

You should see the JSON content.

---

# Step 3 — Ensure the Collection Exists

Confirm the `Note` collection exists:

```bash
curl -s http://localhost:8080/v1/schema \
  -H "Authorization: Bearer admin-key-123"
```

You should see `"Note"` in the output.

If not, create it before continuing.

---

# Step 4 — Create the Import Script

Make sure the `scripts` folder exists:

```bash
mkdir -p scripts
```

Create the script file:

```bash
nano scripts/import_data.py
```

Paste the following code exactly:

```python
import json
import weaviate

ADMIN_KEY = "admin-key-123"
COLLECTION_NAME = "Note"

def main():
    client = weaviate.connect_to_local(
        host="localhost",
        port=8080,
        grpc_port=50051,
        headers={"Authorization": f"Bearer {ADMIN_KEY}"},
    )

    collection = client.collections.get(COLLECTION_NAME)

    with open("sample_data.json", "r") as f:
        data = json.load(f)

    for obj in data:
        collection.data.insert(obj)

    print("Import complete.")

    client.close()

if __name__ == "__main__":
    main()
```

---

## Save and Exit nano

1. `CTRL + O`
2. `Enter`
3. `CTRL + X`

---

## Confirm Script Exists

```bash
ls scripts
```

You should see:

```
import_data.py
rbac_test.py
```

---

# Step 5 — Run the Import Script

Activate your virtual environment if needed:

```bash
source .venv/bin/activate
```

Run:

```bash
python scripts/import_data.py
```

---

# What Success Looks Like

You should see:

```
Import complete.
```

Now verify objects were inserted:

```bash
curl -s http://localhost:8080/v1/objects?class=Note \
  -H "Authorization: Bearer admin-key-123"
```

You should see the inserted objects in the JSON output.

---

# Step 6 — Confirm RBAC Enforcement

Edit the script temporarily:

Change:

```
ADMIN_KEY = "admin-key-123"
```

to:

```
ADMIN_KEY = "viewer-key-123"
```

Run again:

```bash
python scripts/import_data.py
```

The script should fail with a permission error.

This confirms:

- Only admins can ingest data
- RBAC protects write operations
- External ingestion cannot bypass authorization

---

# What You Have Proven

You have now demonstrated:

- How to create an external dataset file
- How to write a secure import script
- How to ingest data into Weaviate
- How to validate ingestion success
- How RBAC restricts ingestion to authorized users

This completes the secure data ingestion workflow.

---

Next: Download & Export Data
