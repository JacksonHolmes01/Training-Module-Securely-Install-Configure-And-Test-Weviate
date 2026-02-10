# Create the Project Folder

In this step, you will create a folder on your computer that will hold all the files for this lab.

This folder will become your **working directory**, meaning it is the place where:
- Docker will look for the `docker-compose.yml` file
- You will run all commands for this tutorial
- All project files will live together in one place

---

## Open a terminal

If you do not already have a terminal open, open one now.

(If you need help with this, return to the previous page on opening a terminal.)

---

## Create the project folder

Run the following command:

```bash
mkdir weaviate-secure-lab
```

### What this does
- `mkdir` means “make directory”
- `weaviate-secure-lab` is the name of the folder you are creating

After running this command, a new folder named `weaviate-secure-lab` exists on your computer.

---

## Move into the project folder

Now run:

```bash
cd weaviate-secure-lab
```

### What this does
- `cd` means “change directory”
- This tells your terminal to start working **inside** the project folder

All commands you run next should be executed from this directory.

---

## What success looks like

After running the commands above:
- The folder `weaviate-secure-lab` exists on your system
- Your terminal is currently inside that folder

If you run:

```bash
pwd
```

You should see a path that **ends with**:

```
weaviate-secure-lab
```

This confirms you are in the correct location.

---

## Important note

From this point forward:
- You should run all tutorial commands **inside this folder**
- The `docker-compose.yml` file will be placed here in the next step

If you close your terminal later, make sure to return to this folder before continuing.

---
Next: [Install Weaviate (Docker Compose)](04-compose-install.md)
