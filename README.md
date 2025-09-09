# Septembre2025

Track the activities of the month


# Bjets

## Codes

## Plots

## Documentation


# EFT

## Codes

## Plots

## Documentation

# ITK

## Codes

## Plots

## Documentation


# Thesis

## Chapters

## Plots


# Teaching

## TD

## TP


# Links


# Comments


# Git

## ✅ Safe option (merge the remote changes into your local branch)

```bash
git pull origin main
```
This fetches the changes from GitHub and merges them into your local branch.
If there are conflicts, Git will ask you to resolve them.


1. Fetch and merge the remote changes
   Run
   ```bash
     git pull origin main
   ```

   -  This will fetch the commits from the remote (origin/main) and merge them into your local main.
   -  If your local repo has no conflicting changes, Git will merge automatically.
   -  If there are conflicts, Git will stop and tell you which files need resolving.
2. (If conflicts happen) Resolve them
   -  Open each conflicting file – you’ll see conflict markers like this:
     ```bash
     <<<<<<< HEAD
    your local code
    =======
    code from GitHub
    >>>>>>> origin/main
   ```
   -  Edit the file to keep what you want, then remove the conflict markers.
   -  After fixing all conflicts, run:
     ```bash
     git add <file>
   ```
     (for each file you resolved).
   -  Then continue the merge:
     ```bash
     git commit
   ```
     (Git will open an editor with a merge commit message – just save & close).
3. Push your changes
   Now your local branch is up-to-date with the remote, so you can safely push:
   ```bash
     git push origin main
   ```
✅ After this, both your local and remote main branches will be in sync, with all commits preserved.


## 🔄 Rebase option (keep a cleaner history)

Instead of merging, you can rebase your changes on top of what’s in GitHub:

```bash
git pull --rebase origin main
```
This fetches the changes from GitHub and merges them into your local branch.
If there are conflicts, Git will ask you to resolve them.

Then push:

```bash
git push origin main
```

## ⚠️ Force push (overwrites remote branch)

If you are **absolutely sure** that you want to overwrite what’s on GitHub with your local branch, you can force push:

```bash
git push origin main --force
```

⚠️ This will delete commits that exist only on GitHub, so use it carefully (especially if others are working on the repo).
