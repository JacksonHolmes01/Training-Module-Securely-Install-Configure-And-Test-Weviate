# Export and Download Data from Weaviate

In previous sections, you:

- Imported external data into Weaviate
- Verified RBAC enforcement
- Confirmed that only authorized users can write data

Now we focus on the opposite direction:

> How do we safely retrieve, download, and export data from Weaviate?

This section covers two distinct but related concepts:

1. **Download (Query + Save Locally)** — Pull data out for analysis or inspection
2. **Export (Structured Extraction)** — Create a reusable file (e.g., JSON) for migration, audit, or backup workflows

These operations are essential for:
- Auditing stored content
- Debugging ingestion
- Migrating environments
- Incident response
- Governance and compliance validation

---

# Part 1 — Simple Data Download (Read + Save)

This demonstrates how to:
- Query Weaviate
- Save results locally as a file

---

## Step 1 — Confirm Data Exists

Before exporting anything, verify that objects exist:

```bash
curl -s http://localhost:8080/v1/objects \
  -H "Authorization: Bearer viewer-key-123" | jq
```

If `jq` is not installed, remove `| jq`.

You should see stored objects from earlier import steps.

---

## Step 2 — Create a Download Script

We will create a script that:
- Connects to Weaviate
- Fetches all objects from the `Note` collection
- Writes them to a local JSON file

---

### Make Sure You Are in the Project Root

```bash
ls
```

You should see:

```
docker-compose.yml
scripts
```

---

### Create the Script File

If the `scripts` folder already exists (it should from earlier RBAC steps), create a new file:

```bash
nano scripts/export_data.py
```

If the folder does not exist:

```bash
mkdir -p scripts
nano scripts/export_data.py
```

---

## How to Save and Exit nano

### macOS / Linux
- Press `CTRL + O`
- Press `Enter`
- Press `CTRL + X`

### Windows (Git Bash or WSL)
Same commands as macOS/Linux.

---

## Step 3 — Paste This Code Into `export_data.py`

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

    try:
        collection = client.collections.get(COLLECTION_NAME)

        objects = collection.query.fetch_objects(limit=100)

        exported = []
        for obj in objects.objects:
            exported.append(obj.properties)

        with open("note_export.json", "w") as f:
            json.dump(exported, f, indent=2)

        print(f"Exported {len(exported)} objects to note_export.json")

    finally:
        client.close()

if __name__ == "__main__":
    main()
```

---

## Step 4 — Run the Export Script

```bash
python scripts/export_data.py
```

You should see:

```
Exported X objects to note_export.json
```

A new file should appear in your project root:

```
note_export.json
```

This is your downloaded dataset.

---

# Part 2 — Export for Migration or Governance

The previous step exported only properties.  
Now we improve it for realistic export use cases.

A production export should preserve:

- Object properties
- Object UUIDs
- Collection name
- Metadata if required

Replace your script with this enhanced version:

```bash
nano scripts/export_data.py
```

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

    try:
        collection = client.collections.get(COLLECTION_NAME)

        objects = collection.query.fetch_objects(limit=100)

        exported = []
        for obj in objects.objects:
            exported.append({
                "uuid": str(obj.uuid),
                "properties": obj.properties
            })

        with open("note_full_export.json", "w") as f:
            json.dump(exported, f, indent=2)

        print(f"Exported {len(exported)} objects to note_full_export.json")

    finally:
        client.close()

if __name__ == "__main__":
    main()
```

Run again:

```bash
python scripts/export_data.py
```

You will now have:

```
note_full_export.json
```

This file is suitable for:

- Migration
- Re-ingestion
- Audit review
- Compliance evidence
- Offline inspection

---

# Download vs Backup (Important Distinction)

This section performs:

**Download / Logical Export**
- Extracting objects via API
- Producing JSON files

This is not the same as:

**System Backup**
- Full database snapshot
- Index files
- Metadata
- Shard structure

Backups are covered in the next section.

---

# What You Have Now Proven

You have demonstrated:

- Data can be retrieved securely
- Exports can be controlled via RBAC
- Data can be serialized for reuse
- Authorization protects read access
- Export is separate from backup

---

Next: `09-backups.md`
