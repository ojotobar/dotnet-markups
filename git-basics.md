## 1. What Is Git?

Git is a **version control system**.
It tracks changes in your project, lets you **go back in time**, **collaborate safely**, and **merge work** from multiple people.

Think of it like the “history” feature in a Google Doc, but for code — and way smarter.

---

## 2. Key Concepts

| Concept               | What it means                            | Analogy                                  |
| --------------------- | ---------------------------------------- | ---------------------------------------- |
| **Repository (repo)** | A folder Git is tracking                 | A project folder with memory             |
| **Commit**            | A snapshot of your code                  | Saving your game progress                |
| **Branch**            | A separate timeline                      | Alternate universe of your code          |
| **Merge**             | Combining branches                       | Merging timelines (hopefully peacefully) |
| **Clone**             | Downloading a repo                       | Copying someone’s project locally        |
| **Pull**              | Get latest changes                       | Update your copy                         |
| **Push**              | Send your changes                        | Upload your work                         |
| **Remote**            | A repo stored elsewhere (GitHub, GitLab) | The “cloud copy” of your code            |

---

## 3. Basic Git Workflow

### Step 1 — Initialize or Clone

Create a new repo:

```bash
git init
```

Or clone an existing one:

```bash
git clone https://github.com/username/repo-name.git
```

---

### Step 2 — Stage and Commit Changes

Add files for tracking:

```bash
git add .
```

Commit them:

```bash
git commit -m "Add initial project setup"
```

This takes a “snapshot” of your project at that point in time.

---

### Step 3 — Connect to GitHub

Link your local repo to a GitHub remote:

```bash
git remote add origin https://github.com/username/repo-name.git
```

Push your first commit:

```bash
git push -u origin main
```

---

### Step 4 — Branching and Merging

Create a branch:

```bash
git checkout -b feature/login
```

Do your work, commit your changes:

```bash
git add .
git commit -m "Implement login feature"
```

Switch back to main:

```bash
git checkout main
```

Merge your feature in:

```bash
git merge feature/login
```

---

## 4. Common Commands Cheat Sheet

| Command                   | Description                       |
| ------------------------- | --------------------------------- |
| `git status`              | See what’s changed                |
| `git diff`                | See what lines changed            |
| `git log`                 | View commit history               |
| `git reset --hard HEAD~1` | Undo last commit (careful!)    |
| `git branch`              | List branches                     |
| `git stash`               | Save uncommitted work temporarily |
| `git pull`                | Get latest changes from remote    |
| `git push`                | Send commits to remote            |
| `git remote -v`           | Show connected remotes            |

---

## 5. Branching Strategy (for Teams)

Simple rule of thumb:

* **main** → always stable
* **development** → work-in-progress
* **feature/** → individual tasks (`feature/login`, `feature/register`)
* **hotfix/** → urgent bug fixes

When a feature is done:

1. Merge feature → development
2. Merge development → main when ready for release

---

## 6. Basic Exercise (Try This)

```bash
# 1. Create a new folder and repo
mkdir MyFirstRepo && cd MyFirstRepo
git init

# 2. Add a file
echo "Hello Git!" > readme.txt
git add .
git commit -m "Add readme"

# 3. Create and switch to a new branch
git checkout -b feature-update
echo "Version 2" >> readme.txt
git add .
git commit -m "Add version 2"

# 4. Switch back to main and merge
git checkout main
git merge feature-update
```
---

## 7. Key GitHub Actions

* **Fork** → copy someone’s repo to your account.
* **Pull request (PR)** → propose merging your branch into another branch.
* **Issues** → track bugs or features.
* **Actions** → automate builds/deployments.

---

## 🔑 8. Golden Rules for Beginners

1. Commit often, with clear messages.
2. Never commit secrets (passwords, API keys).
3. Pull before you push.
4. Branch for features — don’t work directly on `main`.
5. Use `.gitignore` to skip unwanted files (e.g., `bin`, `obj`, `.env`).

---