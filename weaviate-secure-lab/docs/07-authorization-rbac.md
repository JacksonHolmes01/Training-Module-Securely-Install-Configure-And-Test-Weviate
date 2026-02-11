# Authorization (RBAC)

In the previous section, you verified **authentication** (API keys). Authentication answers:

> **Who is making this request?**

This section verifies **authorization** using RBAC (Role-Based Access Control). Authorization answers:

> **What is this authenticated user allowed to do?**

In a secure deployment, not every authenticated user should have “admin” access. RBAC ensures that:
- some users can only **read**
- only trusted users can **write or change configuration**
- high-impact actions require explicit permission

In this lab, we have two users:
- **Admin** (`admin-user` / `admin-key-123`) — can manage schema and data
- **Viewer** (`viewer-user` / `viewer-key-123`) — should be restricted (read-only)

---

## Why we are not using curl for schema writes in this lab

Weaviate’s schema write endpoints and allowed HTTP methods can vary across versions.

You already saw this in practice:
- some endpoints reject `POST` with `405 Method Not Allowed`
- `PUT` may behave like “update only” and fail with “class not found”

That is confusing for beginners and causes RBAC tests to fail for the wrong reason.

To keep this lab beginner-friendly and stable, we will:
- use **curl** for “read / auth checks”
- use the **official Weaviate client** for “schema creation + write operations”

This makes the RBAC tests about **permissions**, not about version-specific REST details.

---

## What we are going to test

We will test two things:

1. **Viewer cannot create a collection (schema write)**  
2. **Admin can create a collection, then viewer can read but not write data**

That proves RBAC is actually enforcing different permissions for different users.

---

## Step 1: Confirm the viewer can read (should succeed)

This is a safe read-only call.

```bash
curl -i http://localhost:8080/v1/schema \
  -H "Authorization: Bearer viewer-key-123"
```

### What success looks like

You should see:

- `HTTP/1.1 200 OK`

and JSON like:

```json
{"classes":[]}
```

That is expected if you have not created any collections yet.

---

## Step 2: Run the RBAC test script (recommended)

This script will:
- attempt schema creation as the viewer (should fail)
- create the collection as the admin (should succeed)
- attempt a data write as the viewer (should fail)
- do a read as the viewer (should succeed)

### Create a Python virtual environment (recommended)
## RBAC Testing Using a Python Script (Step-by-Step)

In this section, we use a **small Python script** to test authorization (RBAC).

This approach is used because:
- Weaviate’s REST schema write endpoints differ across versions
- The official client handles these differences automatically
- This keeps the lab focused on **security concepts**, not API quirks

You do **not** need to know Python to complete this step.  
You only need to follow the instructions exactly.

---

## What this script will do

The script will automatically test all RBAC rules by:

1. Attempting a **schema write as the viewer** (should fail)
2. Creating a collection as the **admin** (should succeed)
3. Attempting a **data write as the viewer** (should fail)
4. Attempting a **read as the viewer** (should succeed)

At the end, it prints **PASS / FAIL** messages so you can clearly see whether RBAC is working.

---

## Step 1: Make sure you are in the project root

All commands below must be run from the **root of the repository**.

You should see files like:

```bash
ls
```

Expected output includes:

```
README.md
docker-compose.yml
docs
```

If you do not see these files, stop and navigate to the correct folder before continuing.

---

## Step 2: Create a Python virtual environment

A **virtual environment** is an isolated Python workspace that:
- keeps dependencies for this project separate
- avoids breaking other Python projects on your machine

### macOS / Linux

Run:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

After activation, your terminal prompt should change and include something like:

```
(.venv)
```

### Windows (PowerShell)

Run:

```powershell
py -m venv .venv
.venv\Scripts\Activate.ps1
```

If you see a PowerShell execution policy warning, follow the on-screen instructions to allow the script to run.

---

## Step 3: Install the Weaviate Python client

With the virtual environment active, run:

```bash
pip install weaviate-client
```

This installs the official Weaviate client library that the script will use.

If this step fails, make sure:
- your virtual environment is activated
- `pip` is available (`pip --version`)

---

## Step 4: Create the script file

We will now **create a new file** that contains the RBAC test code.

### Create the scripts folder (if it does not exist)

From the project root, run:

```bash
mkdir -p scripts
```

This creates a folder named `scripts` where helper code lives.

---

### Create the Python file

Create a new file named:

```
scripts/rbac_test.py
```

How you do this depends on your setup:

#### Option A: Using a terminal editor (nano)

Run:

```bash
nano scripts/rbac_test.py
```

- Paste the provided Python code into the file
- Press `CTRL + O` → Enter (save)
- Press `CTRL + X` (exit)

#### Option B: Using VS Code or another editor

1. Open the project folder in your editor
2. Navigate to the `scripts` folder
3. Create a new file named `rbac_test.py`
4. Paste the provided code
5. Save the file

---

## Step 5: Paste the RBAC test code

Paste **exactly** the following code into `scripts/rbac_test.py`:

```python
import sys
import weaviate

# Admin (root) credentials — must match docker-compose.yml
ADMIN_KEY = "admin-key-123"

# We will create a dynamic DB user for the viewer so RBAC can grant read permissions
VIEWER_USER_ID = "viewer-user"

# Collection used for testing
COLLECTION_NAME = "Note"

def connect(api_key: str):
    """
    Connect to local Weaviate using an API key.
    """
    return weaviate.connect_to_local(
        headers={"Authorization": f"Bearer {api_key}"},
        host="localhost",
        port=8080,
        grpc_port=50051,
    )

def print_result(label: str, ok: bool, detail: str = ""):
    status = "PASS" if ok else "FAIL"
    line = f"[{status}] {label}"
    if detail:
        line += f" — {detail}"
    print(line)

def ensure_viewer_user_and_role(admin_client):
    """
    Creates (or reuses) a DB user 'viewer-user' and assigns the built-in 'viewer' role.
    Returns the viewer user's API key.
    """
    # Check if viewer already exists
    viewer_key = None
    try:
        users = admin_client.users.db.list_all()
        existing = [u for u in users if u.user_id == VIEWER_USER_ID]
        if existing:
            # If user exists, rotate key so we can keep the lab simple and deterministic
            viewer_key = admin_client.users.db.rotate_key(user_id=VIEWER_USER_ID)
        else:
            # Create user and get generated API key
            viewer_key = admin_client.users.db.create(user_id=VIEWER_USER_ID)
    except Exception as e:
        raise RuntimeError(f"Could not create/list viewer DB user: {e}")

    # Assign built-in 'viewer' role (read-only). If already assigned, this should be harmless.
    try:
        admin_client.users.db.assign_roles(
            user_id=VIEWER_USER_ID,
            roles=["viewer"],
        )
    except Exception as e:
        raise RuntimeError(f"Could not assign 'viewer' role: {e}")

    return viewer_key

def main():
    admin = None
    viewer = None

    try:
        # 1) Connect as admin/root
        admin = connect(ADMIN_KEY)

        # 2) Ensure viewer DB user exists + has viewer role
        viewer_key = ensure_viewer_user_and_role(admin)
        print("\nViewer API key created/rotated for this run:")
        print(viewer_key)
        print("Keep this key if you want to re-run viewer tests without rotating.\n")

        # 3) Viewer cannot create collection (schema write) — should FAIL
        try:
            viewer = connect(viewer_key)
            viewer.collections.create(
                name=COLLECTION_NAME,
                vectorizer_config=None,
            )
            print_result("Viewer cannot create collection (schema write)", False, "viewer was able to create schema")
        except Exception:
            print_result("Viewer cannot create collection (schema write)", True)

        # 4) Admin can create collection — should SUCCEED
        try:
            # Delete existing collection if present (ignore errors)
            try:
                admin.collections.delete(COLLECTION_NAME)
            except Exception:
                pass

            admin.collections.create(
                name=COLLECTION_NAME,
                vectorizer_config=None,
            )
            print_result("Admin can create collection (schema write)", True)
        except Exception as e:
            print_result("Admin can create collection (schema write)", False, str(e))
            sys.exit(1)

        # 5) Insert ONE object as admin so reads have something to fetch
        try:
            col_admin = admin.collections.get(COLLECTION_NAME)
            col_admin.data.insert({"text": "hello from admin"})
            print_result("Admin can write data", True)
        except Exception as e:
            print_result("Admin can write data", False, str(e))
            sys.exit(1)

        # 6) Viewer cannot write data — should FAIL
        try:
            # reconnect viewer (fresh client)
            try:
                if viewer:
                    viewer.close()
            except Exception:
                pass

            viewer = connect(viewer_key)
            col_viewer = viewer.collections.get(COLLECTION_NAME)
            col_viewer.data.insert({"text": "viewer should not write"})
            print_result("Viewer cannot write data", False, "viewer was able to insert data")
        except Exception:
            print_result("Viewer cannot write data", True)

        # 7) Viewer CAN read data — should SUCCEED
        try:
            results = col_viewer.query.fetch_objects(limit=1)
            # If this call returns without exception, viewer read permissions work.
            # We don't require a non-empty result, but we inserted one above anyway.
            print_result("Viewer can read data", True)
        except Exception as e:
            print_result("Viewer can read data", False, str(e))
            sys.exit(1)

        print("\nRBAC test complete.\n")

    finally:
        # Always close clients to avoid ResourceWarnings and socket leaks
        try:
            if viewer:
                viewer.close()
        except Exception:
            pass
        try:
            if admin:
                admin.close()
        except Exception:
            pass

if __name__ == "__main__":
    main()
```
## Important notes for beginners

- **CTRL**, not Command (⌘), is used on macOS
- The arrow keys move the cursor
- You can paste text normally (right-click or Cmd+V / Ctrl+V depending on terminal)

Nano behaves the same on:
- macOS
- Linux
- Windows (WSL, Git Bash, PowerShell)

---

## What success looks like

After exiting Nano, run:

```bash
ls scripts
```

You should see:
```
rbac_test.py
```
---

## Step 6: Run the RBAC test

From the project root, run:

```bash
python scripts/rbac_test.py
```

---

## What success looks like

You should see output similar to:

```
[PASS] Viewer cannot create collection
[PASS] Admin can create collection
[PASS] Viewer cannot write data
[PASS] Viewer can read data

RBAC test complete.
```

If you see **PASS** for all four checks, RBAC is working correctly.

---

## If something fails

- Make sure Docker and Weaviate are running:
  ```bash
  docker ps
  ```
- Make sure your virtual environment is activated
- Make sure the API keys in the script match `docker-compose.yml`

If a step fails, **do not continue** until it is resolved.

---

Next: [Create Test Data](08-create-test-data.md)
## Step 4: Run the RBAC test

From the repository root:

```bash
python scripts/rbac_test.py
```

---

## What success looks like

You should see output where:

- Viewer schema write is **PASS** (meaning viewer was blocked)
- Admin schema write is **PASS**
- Viewer data write is **PASS** (meaning viewer was blocked)
- Viewer read is **PASS**

Example structure (your exact error text may differ):

```
[PASS] Viewer cannot create collection (schema write) — <permission denied ...>
[PASS] Admin can create collection
[PASS] Viewer cannot write data — <permission denied ...>
[PASS] Viewer can read data
```

If you see those PASS results, RBAC is enforcing different permissions correctly.

---

## If something fails

If the script fails unexpectedly:

1. Confirm Weaviate is running:
   ```bash
   docker ps
   ```

2. Confirm schema is reachable:
   ```bash
   curl -i http://localhost:8080/v1/schema -H "Authorization: Bearer admin-key-123"
   ```

3. If you changed keys/users in `docker-compose.yml`, update them in the script constants at the top.

---

Next: [Create Test Data](08-create-test-data.md)
