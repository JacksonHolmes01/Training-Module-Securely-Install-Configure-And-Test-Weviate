# Get the Project Files from GitHub

In this step, you will download the project files from GitHub onto your computer.

This is an important step. Files that exist on GitHub do **not** automatically exist on your local machine. You must download them before Docker Compose can use them.

This repository already contains the `docker-compose.yml` file and all documentation. Cloning the repository ensures everything is placed in the correct location.

---

## Open a terminal

If you do not already have a terminal open, open one now.

---

## Choose where to store the project

Before cloning the repository, you need to decide **where on your computer** the project folder should live.

If you are not sure where to put it, your **home directory** is a safe default.

When you open a terminal, you usually start in your home directory automatically.

You can confirm this by running:

```bash
pwd
```

If the output looks something like:

```
/Users/your-username
```

or

```
/home/your-username
```

then you are already in your home directory and do not need to change anything.

---

### Optional: create a projects folder (recommended but not required)

If you want to keep your projects organized, you can create a folder named `projects` and work inside it.

Run:

```bash
mkdir -p ~/projects
cd ~/projects
```

After running these commands, your project will be stored inside the `projects` folder.

If you skip this step, the project will be created directly in your home directory, which is also fine.

---

Once you are in the directory where you want the project to live, continue to the next step.


---

## Clone the repository (important note)

In the command below, `YOUR_USERNAME` is a **placeholder**, not a real value.

You **must replace it** with the actual GitHub username that owns the repository.

If you copy the command exactly as written with `YOUR_USERNAME`, cloning will fail.

---

## Get the correct repository URL

1. Open the repository on GitHub in your web browser
2. Click the green **Code** button
3. Make sure **HTTPS** is selected
4. Copy the URL shown

It will look similar to this:

```
https://github.com/actual-username/weaviate-secure-lab.git
```

---

## Clone the repository

Replace `actual-username` with the real username and run:

```bash
git clone https://github.com/actual-username/weaviate-secure-lab.git
```

This command downloads the repository and creates a folder named `weaviate-secure-lab` on your computer.

---

## What success looks like

If the clone is successful, you will see output similar to:

```
Cloning into 'weaviate-secure-lab'...
```

and a new folder named `weaviate-secure-lab` will appear in your current directory.

---

## If you see an authentication error

If you see an error like:

```
remote: Invalid username or token.
Password authentication is not supported for Git operations.
```

This usually means **one of the following**:

- The repository URL is incorrect
- `YOUR_USERNAME` was not replaced
- The repository is private

If the repository is private, you must authenticate with GitHub before cloning.

---

## If the repository is private (optional)

The easiest way to authenticate is using the GitHub CLI.

1. Install GitHub CLI:  
   https://cli.github.com/
2. Log in:
   ```bash
   gh auth login
   ```
3. Clone the repository:
   ```bash
   gh repo clone actual-username/weaviate-secure-lab
   ```

---

If cloning succeeds and the `weaviate-secure-lab` folder exists, you can continue to the next step.


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
