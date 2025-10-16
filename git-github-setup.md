# Step-by-Step Guide: Setting Up Git and GitHub (with SSH)

---

## Install Git

### Windows

1. Go to [https://git-scm.com/downloads](https://git-scm.com/downloads)
2. Download **Git for Windows**.
3. Run the installer — just keep clicking **Next**, but make sure these options are selected:

   * *“Git from the command line and also from 3rd-party software”*
   * *Use Visual Studio Code as Git’s default editor* (if you use VS Code)
4. After installation, open **Git Bash** or **Command Prompt** and type:

```bash
git --version
```

If you see something like `git version 2.xx.x`, it worked.

---

### Linux (Ubuntu / Debian)

```bash
sudo apt update
sudo apt install git -y
git --version
```

---

### macOS

```bash
brew install git
git --version
```

---

## Configure Git (your identity)

Git needs to know who you are to mark your commits.

```bash
git config --global user.name "Your Full Name"
git config --global user.email "youremail@example.com"
```

To check your configuration:

```bash
git config --list
```

---

## Create a GitHub Account

1. Go to [https://github.com/](https://github.com/)
2. Click **Sign Up**.
3. Choose:

   * A username
   * An email
   * A password
4. Verify your email and you’re in! 🎊

---

## (Optional but Recommended) Configure SSH Access

SSH keys let you push and pull code **without entering your password every time**.

### 4.1 Generate SSH key

In your terminal:

```bash
ssh-keygen -t ed25519 -C "youremail@example.com"
```

If your system doesn’t support `ed25519`, use:

```bash
ssh-keygen -t rsa -b 4096 -C "youremail@example.com"
```

When prompted for a file location, press **Enter** to accept the default.
When asked for a passphrase, you can leave it blank (or add one for extra security).

---

### Start the SSH agent

```bash
eval "$(ssh-agent -s)"
```

Then add your new key:

```bash
ssh-add ~/.ssh/id_ed25519
```

---

### Add SSH key to GitHub

1. Copy the public key:

   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```

   Copy the full output (starts with `ssh-ed25519 ...`).

2. Go to GitHub → top-right corner → **Settings → SSH and GPG keys**

3. Click **New SSH key** → give it a name (e.g., “My Laptop”) → paste your key → **Save**

---

### 4.4 Test the connection

```bash
ssh -T git@github.com
```

If you see:

```
Hi your-username! You've successfully authenticated...
```

You’re connected securely

---

## Create a New Repository on GitHub

1. Click the **+** icon (top-right) → **New repository**
2. Choose a name (e.g., `my-first-repo`)
3. Select:

   * **Public** (for open source) or **Private**
   * Optionally check “Add a README.md”
4. Click **Create repository**

---

## Connect Local Folder to GitHub Repo

Inside your project folder:

```bash
git init
git remote add origin git@github.com:your-username/my-first-repo.git
```

Add and commit your files:

```bash
git add .
git commit -m "Initial commit"
```

Push to GitHub:

```bash
git branch -M main
git push -u origin main
```

Check your repository on GitHub — your code should be live!

---

## Everyday Git Commands (Cheat Sheet)

| Command                   | Description                             |
| ------------------------- | --------------------------------------- |
| `git status`              | Check changes in your working directory |
| `git add .`               | Stage all changes                       |
| `git commit -m "Message"` | Save staged changes                     |
| `git push`                | Upload commits to GitHub                |
| `git pull`                | Fetch and merge the latest changes      |
| `git log --oneline`       | View commit history                     |
| `git clone <url>`         | Download a repo from GitHub             |