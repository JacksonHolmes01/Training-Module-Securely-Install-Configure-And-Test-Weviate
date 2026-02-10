# Get the Project Files from GitHub

In this step, you will download the project files from GitHub onto your computer.

This is an important step. Files that exist on GitHub do **not** automatically exist on your local machine. You must download them before Docker Compose can use them.

This repository already contains the `docker-compose.yml` file and all documentation. Cloning the repository ensures everything is placed in the correct location.

---

## Open a terminal

If you do not already have a terminal open, open one now.

---

## Choose where to store the project

Decide where you want to keep this project on your computer. A common choice is your home directory.

If you are unsure, you can continue without changing directories.

---

## Clone the repository

Run the following command, replacing the URL with the repository URL if needed:

```bash
git clone https://github.com/YOUR_USERNAME/weaviate-secure-lab.git
```

### What this does
- `git clone` downloads a copy of the repository
- A new folder named `weaviate-secure-lab` is created
- All project files are placed inside that folder, including:
  - `docker-compose.yml`
  - the `docs/` folder
  - supporting files

---

## Move into the project folder

Now run:

```bash
cd weaviate-secure-lab
```

All commands for the rest of this tutorial must be run from inside this folder.

---

## What success looks like

If you run:

```bash
ls
```

You should see files and folders such as:

```
docker-compose.yml
docs
README.md
```

This confirms the project files are available on your machine.

If you do **not** see `docker-compose.yml`, do not continue yet.

---

## Important note

If you skip this step and create an empty folder manually, Docker Compose will fail because the configuration file will not exist locally.

Always clone the repository before running Docker commands.

From this point forward:
- You should run all tutorial commands **inside this folder**
- The `docker-compose.yml` file will be placed here in the next step

If you close your terminal later, make sure to return to this folder before continuing.

---
Next: [Install Weaviate (Docker Compose)](04-compose-install.md)
