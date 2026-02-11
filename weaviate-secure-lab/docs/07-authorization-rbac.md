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
"""
RBAC test script for Weaviate (API keys + RBAC)

What this script proves:
1) Viewer cannot create a collection (schema write)        -> should FAIL (PASS = blocked)
2) Admin can create a collection                           -> should SUCCEED
3) Viewer cannot write data                                -> should FAIL (PASS = blocked)
4) Viewer CAN read data                                    -> should SUCCEED

Why we do a small "setup" step:
- RBAC only works if the viewer user has a role that grants read permissions.
- In this lab, we assign the predefined "viewer" role to `viewer-user` at runtime.
  (This avoids confusing REST endpoint differences for schema writes.)

Expected outcome:
- PASS, PASS, PASS, PASS
"""

import sys
from typing import Optional

import weaviate


WEAVIATE_HOST = "localhost"
WEAVIATE_HTTP_PORT = 8080
WEAVIATE_GRPC_PORT = 50051

ADMIN_USER_ID = "admin-user"
ADMIN_KEY = "admin-key-123"

VIEWER_USER_ID = "viewer-user"
VIEWER_KEY = "viewer-key-123"

COLLECTION_NAME = "Note"


def connect_with_key(api_key: str) -> weaviate.WeaviateClient:
    """
    Connect to local Weaviate using an API key via Authorization: Bearer <key>.
    """
    client = weaviate.connect_to_local(
        host=WEAVIATE_HOST,
        port=WEAVIATE_HTTP_PORT,
        grpc_port=WEAVIATE_GRPC_PORT,
        headers={"Authorization": f"Bearer {api_key}"},
    )
    return client


def print_result(label: str, ok: bool, detail: str = "") -> None:
    status = "PASS" if ok else "FAIL"
    msg = f"[{status}] {label}"
    if detail:
        msg += f" — {detail}"
    print(msg)


def safe_close(client: Optional[weaviate.WeaviateClient]) -> None:
    try:
        if client is not None:
            client.close()
    except Exception:
        pass


def ensure_viewer_has_viewer_role(admin: weaviate.WeaviateClient) -> None:
    """
    Make sure `viewer-user` has the predefined `viewer` role.

    Note:
    - `viewer-user` may be a db_env_user (created via AUTHENTICATION_APIKEY_USERS).
      Even then, assigning roles should work via the RBAC API.
    - If DB user management is disabled, this will fail. That's OK: the lab should
      enable RBAC and keep users defined via AUTHENTICATION_APIKEY_USERS.

    This step is required so the viewer can READ data but not WRITE data.
    """
    # Confirm the user exists (helps give a clear error if the compose keys/users are wrong)
    try:
        users = admin.users.db.list_all()
    except Exception as e:
        raise RuntimeError(
            "Could not list users. RBAC user management endpoints might be unavailable.\n"
            "Confirm RBAC is enabled and Weaviate is running."
        ) from e

    found = any(getattr(u, "user_id", None) == VIEWER_USER_ID for u in users)
    if not found:
        raise RuntimeError(
            f"Viewer user '{VIEWER_USER_ID}' not found.\n"
            "Fix: ensure docker-compose.yml includes:\n"
            f'  AUTHENTICATION_APIKEY_USERS: "{ADMIN_USER_ID},{VIEWER_USER_ID}"\n'
            f'  AUTHENTICATION_APIKEY_ALLOWED_KEYS: "{ADMIN_KEY},{VIEWER_KEY}"'
        )

    # Assign the predefined "viewer" role (grants read permissions).
    # This is the missing step when you see:
    #   forbidden action ... insufficient permissions to read_data
    try:
        admin.users.db.assign_roles(user_id=VIEWER_USER_ID, role_names=["viewer"])
    except Exception as e:
        raise RuntimeError(
            f"Could not assign RBAC role 'viewer' to user '{VIEWER_USER_ID}'.\n"
            "Fix: confirm RBAC is enabled and the admin user is a root user:\n"
            f'  AUTHORIZATION_RBAC_ENABLED: "true"\n'
            f'  AUTHORIZATION_RBAC_ROOT_USERS: "{ADMIN_USER_ID}"\n'
        ) from e


def ensure_clean_collection(admin: weaviate.WeaviateClient) -> None:
    """
    Ensure a known-good collection exists for the test.
    We delete it if it exists, then recreate it.
    """
    try:
        if admin.collections.exists(COLLECTION_NAME):
            admin.collections.delete(COLLECTION_NAME)
    except Exception:
        # If delete fails, we still try to create fresh; creation may fail if it already exists
        pass

    # Create with no external vectorizer dependency
    admin.collections.create(
        name=COLLECTION_NAME,
        vectorizer_config=None,
        properties=[
            weaviate.classes.config.Property(
                name="text",
                data_type=weaviate.classes.config.DataType.TEXT,
            )
        ],
    )


def main() -> None:
    admin = None
    viewer = None

    try:
        # --- Connect as admin ---
        admin = connect_with_key(ADMIN_KEY)

        # --- Ensure viewer has a read-only role (critical) ---
        ensure_viewer_has_viewer_role(admin)

        # --- 1) Viewer cannot create collection (schema write) ---
        try:
            viewer = connect_with_key(VIEWER_KEY)
            viewer.collections.create(
                name="ShouldFail_CreateByViewer",
                vectorizer_config=None,
            )
            print_result("Viewer cannot create collection", False, "viewer was able to create schema (RBAC not enforced)")
        except Exception:
            print_result("Viewer cannot create collection", True)
        finally:
            safe_close(viewer)
            viewer = None

        # --- 2) Admin can create collection (schema write) ---
        try:
            ensure_clean_collection(admin)
            print_result("Admin can create collection", True)
        except Exception as e:
            print_result("Admin can create collection", False, str(e))
            sys.exit(1)

        # Insert one object as admin so there's something to read
        try:
            col_admin = admin.collections.get(COLLECTION_NAME)
            col_admin.data.insert({"text": "hello"})
        except Exception:
            # Not fatal for RBAC itself, but read test is nicer with data
            pass

        # --- 3) Viewer cannot write data (should fail) ---
        try:
            viewer = connect_with_key(VIEWER_KEY)
            col_viewer = viewer.collections.get(COLLECTION_NAME)
            col_viewer.data.insert({"text": "viewer-write-should-fail"})
            print_result("Viewer cannot write data", False, "viewer was able to insert data (RBAC not enforced)")
        except Exception:
            print_result("Viewer cannot write data", True)
        finally:
            safe_close(viewer)
            viewer = None

        # --- 4) Viewer CAN read data (should succeed) ---
        try:
            viewer = connect_with_key(VIEWER_KEY)
            col_viewer = viewer.collections.get(COLLECTION_NAME)
            _ = col_viewer.query.fetch_objects(limit=5)
            print_result("Viewer can read data", True)
        except Exception as e:
            print_result("Viewer can read data", False, str(e))
            sys.exit(1)
        finally:
            safe_close(viewer)
            viewer = None

        print("\nRBAC test complete.")

    finally:
        safe_close(viewer)
        safe_close(admin)


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
