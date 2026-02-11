# Conclusion — Operational Control, Security Model, and Command Reference

You have now completed a full secure Weaviate deployment lifecycle.

This lab was not just about “getting Weaviate running.” It was about understanding and enforcing authentication, authorization, resource control, secure ingestion, controlled export, backups, and operational lifecycle management.

This section serves as both a conclusion and an operational glossary.

---

# What You Have Built

You have successfully:

- Installed Weaviate using Docker Compose
- Disabled anonymous access
- Enforced API key authentication
- Implemented RBAC (role-based access control)
- Restricted write permissions to admins
- Verified ingestion is protected
- Imported external data securely
- Exported data safely
- Created filesystem backups
- Applied CPU and memory limits to protect the host machine

This is a secure, production-style deployment.

---

# Security Model Review

## Authentication

Authentication answers:

> Who is making this request?

In this lab:

- API keys identify users
- Anonymous access is disabled
- Requests without credentials return `401 Unauthorized`

Authentication ensures every request is tied to a known identity.

---

## Authorization (RBAC)

Authorization answers:

> What is this authenticated user allowed to do?

In this lab:

- Admin can create collections and ingest data
- Viewer can read but cannot write
- Unauthorized actions return `403 Forbidden`

This enforces the principle of least privilege.

---

# Operational Command Glossary

Weaviate runs inside Docker. These commands control the lifecycle of your deployment.

---

## Start Weaviate

`docker compose up -d`
Starts containers in detached mode (background).
Creates network if needed.
Recreates containers if configuration changed.

Use when:
- First launching
- Restarting after edits
- Returning to the lab

## Check Running Containers
`docker ps`
Lists active containers.
Confirms Weaviate is running.
Shows port mappings and container status.

## View Logs (Troubleshooting)
`docker compose logs weaviate`
Displays container logs.

For live streaming logs:
`docker compose logs -f weaviate`
Press CTRL + C to exit log streaming.

## Stop Weaviate (Temporary Stop)
`docker compose stop`
Stops containers but preserves:
- Data volumes
- Network configuration
- Container definitions

Use when pausing work.

## Stop and Remove Containers
`docker compose down`
Stops and removes:
- Containers
- Network

Keeps:
- Persistent volumes (your data remains)

Use when:
- Resetting runtime environment
- Applying configuration changes

## Full Reset (Delete All Data)
`docker compose down -v`
Stops containers and removes:
- Containers
- Network
- Volumes (ALL DATA)

This permanently deletes stored data.
Use only when you want a complete clean reset.

## Pull Updated Images
`docker compose pull`
Downloads updated container images without starting them.

Use when:
- Upgrading Weaviate version
- Syncing with repository changes

## Restart After Changes
`docker compose down`
`docker compose up -d`

Required after editing:
- docker-compose.yml
- Authentication settings
- RBAC settings
- Resource limits

---

# Operational Best Practices

In real deployments, you should:

- Use TLS (HTTPS) instead of plain HTTP
- Store API keys in environment variables or secret managers
- Rotate API keys regularly
- Restrict exposed ports to localhost or internal networks
- Monitor logs and metrics
- Separate development and production environments
- Perform regular backups and test restore procedures

---

# Final Takeaways

You now understand:

- How secure installation differs from default installation
- How authentication and authorization work together
- Why least privilege matters
- Why ingestion must be protected
- Why export must be controlled
- Why backups are mandatory
- How to safely stop, start, and reset the system
- How containerization enforces isolation

This concludes the Secure Weaviate Deployment Lab.

You have moved beyond “running a vector database” and into operating one securely.
