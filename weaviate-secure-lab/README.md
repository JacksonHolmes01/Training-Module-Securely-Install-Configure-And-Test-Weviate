# Secure Weaviate Lab (Beginner Step-by-Step)

This repository teaches how to securely install, configure, and test a Weaviate vector database using Docker.

It is written for beginners and assumes no prior experience with Weaviate, Docker, or vector databases.

---

## What you will learn
- How to install Weaviate using Docker Compose
- How to configure persistent storage
- How to securely configure authentication (API keys)
- How to configure authorization (RBAC / least privilege)
- How to test that security controls are actually enforced
- How to safely stop, restart, and reset a vector database
- How to think about vector databases as secure infrastructure components

---

## Start here (read in order)

1. [Overview](docs/00-overview.md)
2. [How to Open a Terminal](docs/01-terminal-setup.md)
3. [Install & Verify Docker](docs/02-prereqs-docker.md)
4. [Create the Project Folder](docs/03-project-setup.md)
5. [Install Weaviate (Docker Compose)](docs/04-compose-install.md)
6. [Verify Weaviate is Running](docs/05-verify-running.md)
7. [Authentication (API Keys)](docs/06-authentication.md)
8. [Authorization (RBAC)](docs/07-authorization-rbac.md)
9. [Create Test Data](docs/08-create-test-data.md)
10. [Download Data](https://github.com/JacksonHolmes01/Training-Module-Securely-Install-Configure-And-Test-Weviate/blob/139d16f10644b9e17108c2755994327727b1f80a/weaviate-secure-lab/docs/09-Download-Data.md)
11. [Download Export Data](https://github.com/JacksonHolmes01/Training-Module-Securely-Install-Configure-And-Test-Weviate/blob/4d95bbdd5f0ba75ec6e02505f53710de81cbd286/weaviate-secure-lab/docs/10-Download-Export-Data.md)
12. [Backups](https://github.com/JacksonHolmes01/Training-Module-Securely-Install-Configure-And-Test-Weviate/blob/4f69d95d54e27ee5930a2f8c1c65797eb0c84dcf/weaviate-secure-lab/docs/11-backups.md)

---

## Quick start (after prerequisites)

```bash
docker compose up -d
```
