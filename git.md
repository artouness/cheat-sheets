## 12 steps to push your local changes back to GitHub. Each step has the exact command(s) and a one-line explanation.

### 1. Check working tree and current branch

```bash
git status
git branch --show-current
```

Shows unstaged/staged changes and prints the name of the branch you’re on.

### 2. Stage a single file (example)

```bash
git add path/to/file
```

Stages only the specified file for commit.

### 3. Stage all changes (tracked + untracked + deletions)

```bash
git add -A
```

Stages all changed, new, and deleted files in the repo.

### 4. Commit your staged changes with a message

```bash
git commit -m "Describe what you changed"
```

Creates a commit from the staged snapshot with a concise message.

### 5. Verify the remote named `origin` (or see it’s missing)

```bash
git remote -v
```

Lists remotes and their URLs so you can confirm `origin` exists or not.

### 6. Add `origin` if missing (SSH and HTTPS examples)

```bash
git remote add origin git@github.com:USERNAME/REPO.git
# or (HTTPS)
git remote add origin https://github.com/USERNAME/REPO.git
```

Adds the GitHub repo as `origin` — replace `USERNAME/REPO` with your values.

### 7. Push current branch to the remote (replace branch if needed)

```bash
git push origin $(git branch --show-current)
# or explicitly
git push origin main
```

Pushes your current local branch to the remote branch of the same name (or push `main` explicitly).

### 8. Create a new local branch and push it, setting upstream (`-u`)

```bash
git checkout -b myfeature
# after commits...
git push -u origin myfeature
```

Creates `myfeature` locally, then pushes it and links the local branch to origin/myfeature.

### 9. Authentication note (HTTPS vs SSH)

```bash
# SSH key (generate example)
ssh-keygen -t ed25519 -C "you@example.com"
```

HTTPS: supply a GitHub Personal Access Token (PAT) as your password or use a credential helper; SSH: add your public key to GitHub and push via `git@github.com:...` for passwordless auth.

### 10. Fix non-fast-forward by rebasing (safe pull)

```bash
git fetch origin
git rebase origin/main
# or shorter
git pull --rebase origin main
```

Fetch remote changes and replay your commits on top to avoid unnecessary merge commits (replace `main` with the remote branch).

### 11. Simple conflict flow (resolve, continue, or abort)

```bash
git status
# edit conflicted files, then:
git add path/to/conflicted-file
git rebase --continue
# if you want to stop:
git rebase --abort
```

When a conflict appears: edit files, `git add` them, then `git rebase --continue` (or `git merge --continue` if you were merging); use `--abort` to back out.

### 12. Force push warning + safer alternative, and quick verifications + GitHub URL tip

```bash
# Safer forced update:
git push --force-with-lease origin main
# Dangerous (do NOT use unless you know what you're doing):
git push --force origin main

# Quick verifications:
git log --oneline -n 5
git remote -v
```

`--force-with-lease` protects against silently overwriting others’ work (avoid plain `--force`); use `git log` to see recent commits and `git remote -v` to check remotes — to view a commit on GitHub, copy the commit SHA from `git log` and open `https://github.com/USERNAME/REPO/commit/<sha>` or click **Commits** on the repo page.
