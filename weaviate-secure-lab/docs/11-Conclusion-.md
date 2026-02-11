# Conclusion — Operational Control, Security Model, and Command Reference

You have now completed a full secure Weaviate deployment lifecycle.

This lab was not just about “getting Weaviate running.”  
It was about understanding and enforcing:

- Authentication (Who are you?)
- Authorization (What are you allowed to do?)
- Resource isolation (How much can you consume?)
- Data ingestion controls
- Export controls
- Backup discipline
- Operational lifecycle management

This final section ties everything together and serves as a practical command glossary for managing your deployment.

---

# What You Have Built

By completing this lab, you have:

- Installed Weaviate using Docker Compose
- Disabled anonymous access
- Enforced API key authentication
- Implemented RBAC (role-based access control)
- Restricted write permissions to admins
- Verified ingestion is protected
- Practiced importing external data
- Exported data safely
- Created filesystem backups
- Controlled system resources (CPU / memory limits)

This is a secure, controlled, production-style deployment model.

---

# Authentication vs Authorization (Final Review)

## Authentication
Authentication answers:

> Who is making this request?

In this lab:
- API keys identify users.
- Anonymous access is disabled.
- Every request must include a valid key.

## Authorization (RBAC)
Authorization answers:

> What is this authenticated user allowed to do?

In this lab:
- Admin can manage schema and ingest data.
- Viewer can read but cannot write.
- Write attempts from viewer are blocked.

This is the principle of least privilege in action.

---

# Operational Control — Docker Lifecycle Commands

Weaviate runs inside a container managed by Docker Compose.

Understanding how to control it is essential.

---

## Start Weaviate

```bash
docker compose up -d
