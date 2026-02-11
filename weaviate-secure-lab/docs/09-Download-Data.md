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

# Step 6 — Confirm RBAC Enforcement (Modify the Script Safely)

In this step, we will intentionally break the import script to prove that RBAC is working correctly.

We will temporarily change the script to use the **viewer API key** instead of the **admin API key**.

This should cause the import to fail.

After testing, we will restore the script back to its correct state.

---

# Why We Are Doing This

This test proves:

- RBAC is actually enforcing write restrictions
- Viewer users cannot ingest data
- Import scripts cannot bypass authorization controls

If the import succeeds as a viewer, your security configuration is incorrect.

---

# Step 6A — Open the Import Script

From the project root, run:

```bash
nano scripts/import_data.py
```

(If you use VS Code or another editor, open the file `scripts/import_data.py`.)

---

# Step 6B — Locate the API Key Line

Near the top of the file, you will see this line:

```python
ADMIN_KEY = "admin-key-123"
```

This tells the script to authenticate as the admin user.

---

# Step 6C — Change the Key to Viewer

Temporarily replace that line with:

```python
ADMIN_KEY = "viewer-key-123"
```

You are now telling the script:

> Authenticate as the viewer user instead of the admin.

---

# Step 6D — Save and Exit nano

If using nano:

1. Press `CTRL + O`  
2. Press `Enter` to confirm  
3. Press `CTRL + X` to exit  

If using another editor, simply save the file.

---

# Step 6E — Run the Script Again

Now run:

```bash
python scripts/import_data.py
```

---

# What Should Happen

The script should fail with a permission or authorization error.

You may see something like:

```
permission denied
forbidden action
insufficient permissions
```

This is expected and correct.

It proves:

- Viewer users cannot write data
- RBAC is protecting ingestion
- The system is enforcing role separation

If the script succeeds, stop and investigate your RBAC configuration.

---

# Step 6F — Restore the Script (Important)

Now we must change the script back to admin mode.

Open the file again:

```bash
nano scripts/import_data.py
```

Find this line:

```python
ADMIN_KEY = "viewer-key-123"
```

Change it back to:

```python
ADMIN_KEY = "admin-key-123"
```

Save and exit:

1. `CTRL + O`
2. `Enter`
3. `CTRL + X`

---

# Step 6G — Confirm It Works Again

Run the script one more time:

```bash
python scripts/import_data.py
```

You should now see:

```
Import complete.
```

This confirms:

- The admin can write data
- The viewer cannot
- RBAC is functioning correctly

---
---

# Importing Data from an External API (Realistic Example)

In the previous steps, we simulated external data by manually creating a JSON file.

Now we will simulate a **real-world ingestion workflow** by downloading data from a public API and importing it into Weaviate.

This demonstrates a realistic pipeline:

External Website/API → Download → Inspect → Import → Verify

---

# Why This Matters

In production systems, data usually comes from:

- Public APIs
- Internal REST services
- Cloud storage
- Data vendors
- Logs or analytics systems

Before importing external data into a database, you must:

1. Download it securely  
2. Inspect it  
3. Validate its structure  
4. Import it with proper authorization  

Never blindly ingest remote data.

---

# Step 1 — Download Sample Data from a Public API

We will use **JSONPlaceholder**, a safe public mock API.

From your project root, run:

```bash
curl -o external_posts.json https://jsonplaceholder.typicode.com/posts?_limit=5
```

This downloads 5 sample posts and saves them locally as:

```
external_posts.json
```

---

# Step 2 — Inspect the Downloaded File

Always inspect external data before importing it.

View the file:

```bash
cat external_posts.json
```

You should see JSON objects similar to:

```json
[
  {
    "userId": 1,
    "id": 1,
    "title": "...",
    "body": "..."
  }
]
```

Notice:

- The field names do not match our `Note` schema.
- Our schema expects a property called `"text"`.

We must transform the data before importing.

This is a critical security and data hygiene step.

---

# Step 3 — Create a New Import Script for External Data

Create a new script:

```bash
nano scripts/import_external_api.py
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

    with open("external_posts.json", "r") as f:
        posts = json.load(f)

    for post in posts:
        transformed = {
            "text": post["title"] + " — " + post["body"]
        }
        collection.data.insert(transformed)

    print("External API import complete.")

    client.close()

if __name__ == "__main__":
    main()
```

---

## Save and Exit nano

1. `CTRL + O`
2. Press `Enter`
3. `CTRL + X`

---

# Step 4 — Run the Import

Make sure your virtual environment is active:

```bash
source .venv/bin/activate
```

Then run:

```bash
python scripts/import_external_api.py
```

---

# What Success Looks Like

You should see:

```
External API import complete.
```

Now verify the data exists:

```bash
curl -s http://localhost:8080/v1/objects?class=Note \
  -H "Authorization: Bearer admin-key-123"
```

You should see the transformed objects added to the collection.

---

# Security Lessons from This Exercise

You have now demonstrated:

- Downloading data from a public API
- Inspecting external content before ingestion
- Transforming data to match schema
- Importing using admin credentials
- Respecting RBAC restrictions
- Verifying successful ingestion

Most importantly:

You did **not** blindly import unknown JSON directly into the database.

You inspected and transformed it first.

This is proper secure ingestion design.

---

This concludes the realistic external data ingestion workflow.

# What You Have Proven

By temporarily modifying the script and restoring it, you have verified:

- Authentication works
- Authorization works
- Write permissions are restricted
- Import operations respect RBAC
- Security controls cannot be bypassed via scripting

This completes the RBAC validation for data ingestion.

# What You Have Proven

You have now demonstrated:

- How to create an external dataset file
- How to write a secure import script
- How to ingest data into Weaviate
- How to validate ingestion success
- How RBAC restricts ingestion to authorized users

This completes the secure data ingestion workflow.

---

Next: [Download & Export Data](https://github.com/JacksonHolmes01/Training-Module-Securely-Install-Configure-And-Test-Weviate/blob/25f8917b989ffce0b49d953ee14da7b224aa201a/weaviate-secure-lab/docs/09-Download-Export-Data.md)
