# Git Assignment

## Question 1 

`git merge` and `git rebase` are both used to combine changes from different branches, but they do it differently and produce different commit histories.

### `git merge`
`git merge` combines the history of another branch with the current branch. If the two branches have diverged, Git normally creates a new merge commit with both branch histories as its parents. For example:

```bash
git checkout main
git merge feature
```
Here, `git checkout main` switches to the `main` branch, and `git merge feature` combines the commits from `feature` into `main`.
If `main` has not changed since `feature` was created, Git may perform a fast-forward merge instead of creating a merge commit. In that case, Git simply moves the `main` branch pointer forward.
A merge is appropriate when the branch has already been shared with other developers because it does not rewrite the existing commits or change their commit IDs.

### `git rebase`
`git rebase` moves the commits of one branch and reapplies them on top of another base commit. For example:

```bash
git checkout feature
git rebase main
```
Git takes the commits that exist on `feature` but not on `main` and reapplies them on top of the current `main` history.
The main advantage is that it produces a more linear history, which can make the project history easier to read.
However, rebase rewrites commits. The reapplied commits receive new commit IDs because their parent commits have changed.

I would use merge when the branch is already shared with other developers or when preserving the existing commit history is important.
I would use rebase for a private feature branch before sharing it, when I want to update it with the latest changes from `main` and keep the history linear. I would avoid rebasing commits that other developers have already based their work on, because rewriting shared history can cause synchronization problems.

---

# Question 2 

A hunk is a section of related changes shown by Git when displaying a diff. Git groups nearby added, removed, or modified lines into hunks so that changes can be reviewed and staged as separate units. For example, if I modify three unrelated parts of `app.js`, Git may display them as three different hunks.

Instead of staging the entire file:

```bash
git add app.js
```
I can use interactive patch mode:

```bash
git add -p app.js
```
The `-p` option is short for `--patch`. It tells Git to interactively go through the changes in the file and ask me what to do with each hunk.
Git provides several choices:
- `y` — stage the current hunk.
- `n` — do not stage the current hunk.
- `s` — split the current hunk into smaller hunks, when possible.
- `e` — manually edit the patch so that I can choose individual lines.
- `q` — quit the interactive staging process.
For example, if `app.js` contains three separate changes and I only want the first and third changes in my next commit, I can select `y` for the first hunk, `n` for the second, and `y` for the third.
After staging the selected changes, I can verify exactly what is in the staging area with:

```bash
git diff --staged
```
This shows the changes that are currently staged and will be included in the next commit.

Interactive staging helps create focused and coherent commits. For example, suppose I fixed a bug and also made some unrelated formatting changes in the same file. I can stage only the bug fix and create a commit specifically for that fix. This makes commits easier to understand, review, revert, and maintain.

---

# Question 3

`git bisect` is used to find the commit that introduced a bug when I know that an earlier commit was working correctly and a later commit is broken. It uses a binary search through the commit history instead of checking every commit one by one.

Start bisect

```bash
git bisect start
```
This starts the bisect process.

Mark the current commit as bad
If the current version contains the bug:

```bash
git bisect bad
```
This tells Git that the current commit is known to be bad.

Identify a known good commit
Suppose I know that commit `a1b2c3d` did not contain the bug:

```bash
git bisect good a1b2c3d
```
Now Git knows that:
- `a1b2c3d` is good.
- The current commit is bad.
Git can therefore search between these two commits.

Test the commit selected by Git
Git checks out a commit somewhere in the middle of the remaining range.
I run the relevant test.
If the test passes:

```bash
git bisect good
```
If the test fails:

```bash
git bisect bad
```
Git uses the result to eliminate half of the possible commits. For example, if the selected commit is good, the problematic commit must be after it. If the selected commit is bad, the problematic commit must be before it or at that commit. Git continues selecting commits from the remaining range until it finds the first bad commit.

Identify the problematic commit
Eventually Git reports something similar to:

```text
<commit-hash> is the first bad commit
```
I can then inspect that commit using:

```bash
git show <bad-commit-hash>
```
`git show` displays the commit information and the changes introduced by that commit, allowing me to determine what change caused the bug.

Finish the bisect process
After identifying the problematic commit:

```bash
git bisect reset
```
This ends the bisect process and returns the repository to the branch/commit state from before the bisect operation.

Suppose an application had 100 commits. A test passed two weeks ago but currently fails, and I do not know which commit caused the problem.Checking all 100 commits manually would be inefficient. `git bisect` repeatedly divides the search range, so only a small number of commits need to be tested.
Therefore, `git bisect` is useful for efficiently finding the commit that introduced a regression when I know a good commit and a bad commit.

---

# Question 4

## 4.1 
A) Check the repository status
I would first use:

```bash
git status
```
This shows the current branch and the state of the working tree and staging area.
It can show modified files, staged files, unstaged modifications, untracked files, and other information about the repository state.

B) Check changes that are not staged
I would use:

```bash
git diff
```
This compares the working tree with the staging area. Therefore, it shows modifications that have been made to tracked files but have not yet been staged.

C) Check changes that are staged
I would use:

```bash
git diff --staged
```
The `--staged` option compares the staging area with the last commit (`HEAD`). Therefore, it shows exactly what is currently prepared for the next commit.

```bash
git diff
```
shows Working tree → Staging area
while:

```bash
git diff --staged
```
shows Staging area → HEAD
This distinction is important because the same file can contain both staged and unstaged changes.

---

## 4.2 

Suppose I run:

```bash
git commit -m "Update README"
```
without using `-a`.
Git commits only the changes currently in the staging area. Therefore. the changes that were staged before the second edit will be included in the commit. The changes made after staging will remain as unstaged changes in the working tree.
The second set of changes is not automatically included because a normal `git commit` takes its content from the staging area.
The `-a` option is not appropriate for understanding or replacing the staging process here because the question specifically asks what happens with a normal commit without `-a`. After the commit, the working tree can therefore still contain the second set of modifications.

---

## 4.3 

To prevent files inside the `tmp/` directory from being added to future commits, I would add the following rule to the `.gitignore` file at the repository root:

```gitignore
/tmp/
```
The leading `/` makes the rule refer to the `tmp/` directory at the root of the repository. This means that untracked temporary files inside that directory will be ignored by Git.

`.gitignore` only affects files that are not already tracked by Git. If a file has already been added and committed, adding its path to `.gitignore` does not automatically make Git stop tracking it. To stop tracking an already tracked `tmp/` directory while keeping the files on the local filesystem, I can use:

```bash
git rm -r --cached tmp/
```
The `--cached` option removes the files from Git's index/staging area but does not delete the files from the working directory.
After committing this change, the files will remain locally and the `.gitignore` rule will prevent them from being added again.

---

# Question 5 

## 5.1 

The conflict occurred because both branches started from the same commit and then modified the same line of `README.md` in different ways. For example, suppose the common version was:

```text
Project Description
```
The `main` branch changed it to:

```text
Main branch description
```
while `feature-description` changed the same line to:

```text
Feature branch description
```

When Git tries to merge the two branches, it cannot automatically determine which version should be kept. Therefore, Git reports a merge conflict.
Two people can modify the same file without creating a conflict. A conflict usually occurs when Git cannot automatically combine the changes, especially when both branches modify overlapping parts of the same file. For example, if one developer changes lines 10–15 and another changes completely different lines 50–55, Git can usually combine those changes automatically. Therefore, editing the same file does not automatically mean there will be a conflict. The important issue is whether the changes overlap in a way that Git cannot resolve automatically.

---

## 5.2 

First, I can check the repository status:

```bash
git status
```
Git will report files that are in a conflicted/unmerged state.
I can also list only the files that currently have unresolved merge conflicts using:

```bash
git diff --name-only --diff-filter=U
```
The `--name-only` option shows only file names, while `--diff-filter=U` filters the result to files with an unmerged status.
Inside a conflicted file, Git places conflict markers around the two versions. These markers indicate:
the beginning of the version from the current branch. Separates the two versions. Marks the end of the incoming branch's version. The final file must not contain these conflict markers.

---

## 5.3 

Since the question says that both changes must be preserved, I would manually edit `README.md` and combine the two pieces of information into one correct final version.

Check the conflicted files

```bash
git status
```
or:

```bash
git diff --name-only --diff-filter=U
```

Open and resolve the file
I manually edit `README.md`, preserve the information from both branches, and remove:

Stage the resolved file
After resolving the conflict:

```bash
git add README.md
```
This tells Git that the conflict in `README.md` has been resolved.

Verify the merge state
I can run:

```bash
git status
```
The file should no longer appear as unmerged.
I can also check for remaining conflict markers in the file. If this command produces no output, those conflict markers are not present in `README.md`.
I can also use:

```bash
git diff --check
```
to check for common whitespace errors in the changes.

Complete the merge
Once the conflict has been resolved and the file has been staged:

```bash
git commit
```
Git creates the merge commit and completes the merge operation.

---

## 5.4 
If I decide that the merge should not be performed yet, I can abort the merge with:

```bash
git merge --abort
```
This cancels the merge operation currently in progress and attempts to restore the working tree and index to their state before the merge started.
This is different from simply deleting the conflict markers manually. `git merge --abort` is specifically intended to abandon the current merge operation.

---

# Question 6

## 6.1 

I would use:

```bash
git restore notes.txt
```
This restores `notes.txt` in the working tree from the version currently in the staging area.
Because the question states that the staging-area version is identical to the latest commit, this effectively restores `notes.txt` to the last committed version.

The result is that the unwanted working-tree changes in `notes.txt` are discarded. The staging area is unchanged and other files are not affected. No new commit is created and the commit history is unchanged.
Therefore, `git restore notes.txt` is appropriate because I only want to discard unstaged changes to this particular file.

---

## 6.2 

I would use:

```bash
git restore --staged notes.txt
```
The `--staged` option tells Git to restore the version in the staging area rather than discarding the working-tree changes.

After running this command, `notes.txt` is removed from the staging area. My modifications remain in the working tree and the file is still modified.
---

## 6.3 

The appropriate command is:

```bash
git revert <bad-commit-hash>
```
`git revert` does not remove the original commit from history. Instead, Git creates a new commit that applies the inverse of the changes introduced by the bad commit.
The original bad commit remains in the history, but its changes are reversed by the new revert commit.

A force push can rewrite the remote branch history. Since the other team members have already received the bad commit, rewriting the history can cause their local repositories to diverge from the remote repository and may require them to recover or synchronize their work manually.

`git revert` is safer for a shared branch because it preserves the existing history and adds a new commit that reverses the unwanted changes. Therefore, for this scenario, the appropriate solution is:

```bash
git revert <bad-commit-hash>
```
because it undoes the effect of the bad commit without rewriting shared history.

---

# AI Tools Disclosure

I used an AI assistant to help me review Git command usage, check command options and arguments, and organize the explanations in this README.
