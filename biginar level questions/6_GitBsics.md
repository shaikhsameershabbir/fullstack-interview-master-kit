# Git Basics – Complete Beginner Interview Reference

---

## 🛠️ Section 1: Core Concepts & Fundamentals

---

### 1. What is Git?

**Definition:** **Git** is a **Distributed Version Control System (DVCS)** that tracks changes in source code during software development. It allows multiple developers to work on the same project simultaneously without overwriting each other's work.

**Key Features:**
-   **Distributed:** Every developer has a full copy of the project history on their local machine, not just a snapshot.
-   **Speed:** Most operations are local, making them very fast.
-   **Data Integrity:** Every change is checksummed (using SHA-1) before it is stored.
-   **Branching & Merging:** Git has a powerful and lightweight branching model.

**Why use it?**
-   To keep track of code history (who changed what and when).
-   To revert to previous versions if something breaks.
-   To collaborate with team members seamlessly.

---

### 2. What is a Repository (Repo)?

**Definition:** A **Repository** is a central location where all the files of a project and their **entire revision history** are stored. It is basically the ".git" folder inside your project directory.

**Types of Repositories:**
1.  **Local Repository:** Stored on your personal computer. You do your work and commit changes here.
2.  **Remote Repository:** Stored on a server (like GitHub, GitLab, or Bitbucket). It allows teams to share code and collaborate.

---

### 3. What is `git clone`?

**Definition:** `git clone` is a command used to create a **copy of an existing remote repository** onto your local machine.

**What it does:**
-   Downloads all the files from the remote repo.
-   Downloads the entire commit history.
-   Sets up a "remote" reference (usually named `origin`) so you can pull and push changes easily.

**Syntax:**
```bash
git clone <repository-url>
```

---

### 4. What is `git pull`?

**Definition:** `git pull` is used to **fetch and download** content from a remote repository and immediately update your local repository to match that content.

**How it works:**
It is actually a combination of two commands:
1.  **`git fetch`:** Downloads the changes from the remote repo but doesn't change your local files.
2.  **`git merge`:** Merges those downloaded changes into your current local branch.

**Syntax:**
```bash
git pull origin <branch-name>
```

---

### 5. What is `git push`?

**Definition:** `git push` is used to **upload** your local repository commits to a remote repository. It is how you share your changes with the rest of the team.

**Syntax:**
```bash
git push origin <branch-name>
```

---

### 6. What is `git branch`?

**Definition:** A **Branch** is essentially a pointer to a specific commit. It allows you to **diverge from the main line of development** and work on a new feature, bug fix, or experiment without affecting the stable code (usually the `main` or `master` branch).

**Key Commands:**
-   `git branch`: List all local branches.
-   `git branch <name>`: Create a new branch.
-   `git checkout <name>` (or `git switch <name>`): Move to a different branch.

**Analogy:**
Think of a tree. The trunk is the `main` branch. A branch is a side-shoot where you can grow new leaves (code) without touching the trunk.

---

### 7. What is `git merge`?

**Definition:** `git merge` is the process of **combining the changes** from one branch into another. 

**Common Workflow:**
1.  Work on a feature in a `feature` branch.
2.  Switch back to the `main` branch.
3.  Run `git merge feature` to bring those changes onto the `main` branch.

**Types of Merges:**
-   **Fast-forward:** If the target branch hasn't moved since you branched off, Git simply moves the pointer forward.
-   **Three-way merge:** If both branches have different new commits, Git creates a new "merge commit" to tie them together.

---

### 8. What is `git rebase`?

**Definition:** `git rebase` is an alternative to merging. It **moves or combines a sequence of commits** to a new base commit. It effectively "rewrites" history by taking your changes and placing them on top of the latest changes from another branch.

**Merge vs Rebase:**
-   **Merge:** Keeps the history as it happened (shows exactly when branches diverged and merged). Can get "messy" with many merge commits.
-   **Rebase:** Creates a **clean, linear history**. It looks like the work was done in a straight line.

**Golden Rule:** Never rebase branches that have been pushed to a public/shared repository.

---

### 9. What is a Merge Conflict?

**Definition:** A **Merge Conflict** occurs when Git cannot automatically resolve differences in code between two commits being merged. This usually happens when two people **change the same line(s) in the same file**, or one person deletes a file that another person is modifying.

**How to resolve:**
1.  Git stops the merge and marks the files as "conflicted".
2.  You open the files and look for markers:
    ```
    <<<<<<< HEAD
    Your version of the code
    =======
    Their version of the code
    >>>>>>> branch-name
    ```
3.  You manually choose which code to keep (or combine both).
4.  Remove the markers.
5.  `git add` and `git commit` to finish the merge.

---

### 10. What is a Pull Request (PR)?

**Definition:** A **Pull Request** (also called a Merge Request in GitLab) is a way to **notify team members** that you have completed a feature or fix and want to merge your changes from your branch into the main repository.

**Purpose:**
-   **Code Review:** Others can look at your code, leave comments, and suggest improvements.
-   **Discussion:** A central place to talk about the changes.
-   **Testing:** Automated tests (CI/CD) usually run on PRs to ensure the code doesn't break the build.
-   **Approval:** A lead developer can "Approve" the PR before it is merged.

---
## 📈 Section 2: Essential Commands & Workflow

---

### 11. What is the difference between `git init` and `git clone`?

-   **`git init`:** Used to **initialize a new repository** from scratch in an existing folder. It creates a `.git` subfolder with all the necessary Git metadata.
-   **`git clone`:** Used to **copy an existing repository** from a remote server to your local machine.

---

### 12. What is the "Staging Area" (Index)?

**Definition:** The **Staging Area** is an intermediate step between your Working Directory (where you edit files) and the Repository (where your changes are committed). It allows you to **group specific changes** to be included in the next commit.

**The Three Stages:**
1.  **Working Directory:** You modify files.
2.  **Staging Area:** You `git add` files you want to commit.
3.  **Repository:** You `git commit` to save the staged changes permanently.

---

### 13. What is the difference between `git add` and `git commit`?

-   **`git add <file>`:** Moves changes from the working directory to the **staging area**. It tells Git, "I want these changes to be part of the next snapshot."
-   **`git commit -m "msg"`:** Saves the changes from the staging area to the **local repository**. It creates a permanent snapshot of the project with a message describing the changes.

---

### 14. What is `git status`?

**Definition:** `git status` shows the **current state** of your working directory and staging area. It tells you:
-   Which branch you are currently on.
-   Which files have been modified but not yet staged.
-   Which files are in the staging area and ready to be committed.
-   Which files are untracked (not yet monitored by Git).

---

### 15. What is `git log`?

**Definition:** `git log` displays a **list of all the commits** made in the current branch. It shows the commit hash (ID), the author, the date, and the commit message.

**Useful flags:**
-   `git log --oneline`: Shows a condensed, one-line version of each commit.
-   `git log --graph`: Shows a visual text-based representation of the commit history and branches.

---

### 16. What is `git diff`?

**Definition:** `git diff` is used to **compare the differences** between different versions of your project. 
-   `git diff`: Shows changes in the working directory that are NOT yet staged.
-   `git diff --staged`: Shows changes in the staging area vs the last commit.
-   `git diff <branch1> <branch2>`: Compares two different branches.

---

### 17. What is `git checkout` vs `git switch`?

-   **`git checkout`:** Prior to Git 2.23, this command was used for **both** switching branches and restoring files. It was often confusing.
-   **`git switch`:** A newer, safer command solely for **switching between branches**.
-   **`git restore`:** A newer command solely for **undoing changes** to files.

---

### 18. What is `git stash`?

**Definition:** `git stash` temporarily **hides (shelves)** your uncommitted changes (both staged and unstaged) so you can work on something else, and then come back and re-apply them later.

**Common use case:**
You are halfway through a feature, but a bug needs fixing immediately on the `main` branch. You can `git stash` your work, switch to `main`, fix the bug, then switch back and run `git stash pop` to get your work back.

---

## 🤝 Section 3: Collaboration & Best Practices

---

### 19. What is the difference between `git reset` and `git revert`?

-   **`git reset`:** Moves the current branch pointer backward to a specific commit. It "undoes" commits and can modify the staging area and working directory. **Use it for local changes.**
-   **`git revert`:** Creates a **new commit** that does the exact opposite of a specific commit. It doesn't modify existing history. **Use it for shared/remote branches.**

---

### 20. What is a "Remote"?

**Definition:** A **Remote** is a version of your project that is hosted on the internet or network somewhere. `origin` is the default name Git gives to the server you cloned from.

**Commands:**
-   `git remote -v`: Shows the URL of the remote repo.
-   `git remote add <name> <url>`: Adds a new remote connection.

---

### 21. What is the purpose of `.gitignore`?

**Definition:** A text file that tells Git **which files or folders to ignore** (not track). It prevents sensitive or temporary files from being committed to the repository.

**Common things to ignore:**
-   `node_modules/` (dependency folders).
-   `.env` (sensitive API keys/secrets).
-   `dist/` or `build/` (compiled code).
-   `.DS_Store` (OS-specific files).

---

### 22. Forking vs Branching?

-   **Branching:** Creating a new pointer within the **same repository**. Used for different features of the same project.
-   **Forking:** Creating a **complete copy of the entire repository** under your own account. Commonly used in open-source projects to propose changes ("contribute") to someone else's repo.

---

### 23. SSH vs HTTPS for Git?

-   **HTTPS:** Uses a username and password (or a personal access token) for authentication. Easier to set up initially.
-   **SSH:** Uses an encrypted key pair (Public and Private). More secure and doesn't require entering credentials every time once set up.

---

### 24. How to undo the last commit?

-   **If you haven't pushed:** `git reset --soft HEAD~1`. This removes the last commit but keeps your changes in the staging area.
-   **If you have pushed:** `git revert HEAD`. This creates a new commit that undoes the previous one.

---

### 30. Common Git Best Practices Checklist:

```
✅ Commit early and often (modular commits).
✅ Write clear and descriptive commit messages (e.g., "Fix: resolve login crash").
✅ Never push sensitive information (.env, API keys).
✅ Pull from the remote before you start working to avoid conflicts.
✅ Use branches for EVERY new feature or bug fix.
✅ Resolve merge conflicts immediately.
✅ Delete branches once they are merged.
✅ Always use a .gitignore file.
```

---

*End of Git Basics Reference*

---

### 21. What is the difference between `git checkout` and `git reset`?

-   **`git checkout`:** Primarily used to **switch** between branches or restore files from a specific commit without moving the branch pointer.
-   **`git reset`:** A more powerful (and dangerous) command that **moves the branch pointer** itself to a previous commit. It can also unstage files.

---

### 22. What is `git cherry-pick`?

**Definition:** `git cherry-pick` allows you to apply the changes from **one specific commit** from another branch into your current branch. It’s useful when you want a bug fix that was done on a feature branch but don’t want to merge the entire branch yet.

```bash
git cherry-pick <commit-hash>
```

---

### 23. What is `git show`?

**Definition:** `git show` is used to see the **details of a specific object** (usually a commit). It shows the commit message, the author, and a "diff" of the changes introduced by that commit.

```bash
git show <commit-hash>
```

---

### 24. What is a "Bare" Repository?

**Definition:** A **Bare Repository** is a Git repository that **does not have a working directory**. It only contains the `.git` data.

**Why use it?**
Bare repositories are used as **central servers** (like GitHub) where people push and pull code. Since no one actually edits files *on* the server, a working directory is not needed.

---

### 25. What is `git am` and `git apply`?

-   **`git apply`:** Applies a patch file (generated by `git diff`) to your code without creating a commit.
-   **`git am` (Apply Mailbox):** Applies a series of patches (usually from an email or `git format-patch`) and **automatically creates commits** for each one, preserving the original author and message.

---

*End of Git Basics Reference*
