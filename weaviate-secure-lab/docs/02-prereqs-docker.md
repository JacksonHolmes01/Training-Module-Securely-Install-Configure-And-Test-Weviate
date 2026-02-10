# Install & Verify Docker

Docker is required to run Weaviate in this lab. Docker allows you to run software in isolated, reproducible containers without manually installing dependencies.

---

## Install Docker

Follow the instructions for **your operating system** using the official Docker documentation.

### macOS
Install **Docker Desktop for Mac**:

https://docs.docker.com/desktop/install/mac-install/

After installation:
- Open **Docker Desktop**
- Wait until it says **“Docker is running”**

---

### Windows (Windows 10 / 11)
Install **Docker Desktop for Windows**:

https://docs.docker.com/desktop/install/windows-install/

Notes:
- You may be prompted to enable **WSL 2** (Windows Subsystem for Linux)
- Follow the installer’s instructions exactly
- After installation, open **Docker Desktop** and wait until it is running

---

### Linux
Install **Docker Engine** and the **Docker Compose plugin**:

https://docs.docker.com/engine/install/

Choose your Linux distribution (Ubuntu, Debian, Fedora, etc.) and follow the steps provided.

> Docker Desktop is optional on Linux. Most Linux users install Docker Engine directly.

---

## Verify Docker Installation

Once Docker is installed and running, open a terminal and run:

```bash
docker --version
docker compose version

Next: [Create the Project Folder](03-project-setup.md)
