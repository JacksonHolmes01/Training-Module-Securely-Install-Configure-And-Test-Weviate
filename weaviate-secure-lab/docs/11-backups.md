# Backups (Practice)

In this lab, filesystem backups are enabled so you can safely practice
creating and restoring backups without relying on external cloud storage.

Backups are critical in production environments because they protect against:

- Accidental data deletion
- Schema corruption
- Misconfiguration
- Container failure
- Host machine failure

Even in a local lab environment, it is important to understand:

- How backups are triggered
- Where backup files are stored
- How restoration works
- How backup permissions relate to RBAC

---

## How backups are configured in this lab

In `docker-compose.yml`, the following settings enable the filesystem backup module:

```
ENABLE_MODULES: "backup-filesystem"
BACKUP_FILESYSTEM_PATH: "/tmp/backups"
```

And this volume maps the container backup directory to your local project:

```
volumes:
  - ./backups:/tmp/backups
```

This means:

- Backups created inside Weaviate
- Are stored in `/tmp/backups` inside the container
- And appear locally inside the `backups/` folder in your project

You can verify that folder exists:

```bash
ls
```

You should see:

```
backups
docker-compose.yml
README.md
```

---

## Step 1 — Verify backup module is enabled

Run:

```bash
curl -s http://localhost:8080/v1/meta \
  -H "Authorization: Bearer admin-key-123"
```

Look for a section in the output similar to:

```json
"modules": {
  "backup-filesystem": { ... }
}
```

If you see `backup-filesystem`, the module is active.

If you do not see it:

- Check `docker-compose.yml`
- Restart Weaviate:

```bash
docker compose down
docker compose up -d
```

---

## Step 2 — Create a backup

Backups are collection-based.

This example creates a backup for the `Note` collection.

```bash
curl -X POST http://localhost:8080/v1/backups/filesystem \
  -H "Authorization: Bearer admin-key-123" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "note-backup-1",
    "include": ["Note"]
  }'
```

### What this does

- `id`: unique name for this backup
- `include`: list of collections to back up
- Only an admin can create backups

---

## Step 3 — Verify backup status

```bash
curl -s http://localhost:8080/v1/backups/filesystem/note-backup-1 \
  -H "Authorization: Bearer admin-key-123"
```

You should see a status such as:

```
SUCCESS
```

---

## Step 4 — Confirm files exist locally

Because we mapped volumes, backup files appear in your project folder.

Check:

```bash
ls backups
```

You should see a folder named:

```
note-backup-1
```

This confirms:

- The backup was written
- The filesystem module works
- Volume mapping is correct

---

## Step 5 — Restore from backup (optional practice)

To restore:

```bash
curl -X POST http://localhost:8080/v1/backups/filesystem/note-backup-1/restore \
  -H "Authorization: Bearer admin-key-123"
```

Restore requires:

- Admin privileges
- Existing backup files

---

## Why this matters

Backups are a core part of secure deployment:

- RBAC protects data access
- Authentication protects identity
- Backups protect availability and integrity

In real-world environments:

- Backups are automated
- Backup storage is encrypted
- Restore procedures are regularly tested
- Access to backups is restricted

This lab gives you hands-on exposure to the core mechanics
without requiring external infrastructure.

---

## Common issues

### 1. 401 Unauthorized
Check your API key.

### 2. 403 Forbidden
Make sure you are using the admin key.

### 3. Backup folder is empty
Check:

- Volume mapping in `docker-compose.yml`
- That the backup status returned `SUCCESS`

---

Next: [Stop / Start / Reset](10-stop-start-reset.md)
