### **`git checkout`**

The `git checkout` command is used to switch between branches or to move to a specific commit. Here's how it works:

1. **Switch to a Branch**:
    
    - When you want to switch to another branch, use:
        
        git checkout branch_name
        
    - This moves the `HEAD` (the pointer to your current location) to the latest commit of the specified branch (`branch_name`).
        
    - Your working directory will update to match the state of that branch.
        
2. **Checkout a Specific Commit**:
    
    - If you want to move to a specific commit (not a branch), use:
        
        git checkout target_commit_hash
        
    - This **detaches the `HEAD`**, meaning it no longer points to a branch but directly to the commit (`target_commit_hash`).
        
    - Your working directory will update to match the state of that commit.
        
    - **Note**: If you make changes and commit them in this detached `HEAD` state, those commits won't belong to any branch unless you explicitly create a new branch.
        

---
------------------<<<<<<<<<<<<<>>>>>>>>>>>>>>>>>-----------------

### **`git reset`**

The `git reset` command is used to move the `HEAD` (and optionally the working directory and staging area) to a specific commit. It’s often used to undo changes or go back to a previous state. It has three modes: `--hard`, `--soft`, and `--mixed` (default).

1. **`git reset --hard target_commit_hash`**:
    
    - This is the **most aggressive mode**.
        
    - It moves the `HEAD` to the target commit (`target_commit_hash`).
        
    - It also updates the **staging area** and **working directory** to match the target commit.
        
    - **Effect**: Any changes or commits after the target commit are **permanently discarded**.
        
    - Use this when you want to completely undo all changes and go back to a previous state.
        
    
    Example:
    
    git reset --hard target_commit_hash
    
2. **`git reset --mixed target_commit_hash`** (default mode):
    
    - This is the **default mode**.
        
    - It moves the `HEAD` to the target commit (`target_commit_hash`).
        
    - It updates the **staging area** to match the target commit.
        
    - However, the **working directory** remains unchanged. This means any changes made after the target commit will still be present in your files, but they will be **unstaged**.
        
    - **Effect**: You can see all the changes since the target commit, and you can decide what to do with them (e.g., stage and commit them again).
        
    
    Example:
    
    git reset --mixed target_commit_hash
    
3. **`git reset --soft target_commit_hash`**:
    
    - This is the **least aggressive mode**.
        
    - It moves the `HEAD` to the target commit (`target_commit_hash`).
        
    - However, it **does not change the staging area or working directory**.
        
    - **Effect**: All changes since the target commit are preserved and staged, ready to be committed again.
        
    - Use this when you want to undo commits but keep the changes staged for a new commit.
        
    
    Example:
    
    git reset --soft target_commit_hash
    

---

### **Key Differences Between `git checkout` and `git reset`**

|Feature|`git checkout`|`git reset`|
|---|---|---|
|**Purpose**|Switch branches or move to a commit.|Move `HEAD` (and optionally staging area/working directory) to a commit.|
|**Effect on `HEAD`**|Moves `HEAD` to a branch or commit.|Moves `HEAD` to a commit.|
|**Effect on Working Dir**|Updates to match the target commit/branch.|Updates only in `--hard` mode.|
|**Effect on History**|Does not alter commit history.|Can alter commit history (e.g., `--hard` deletes commits).|
|**Detached `HEAD`**|Happens when checking out a commit.|Does not detach `HEAD`.|

---

### **When to Use What?**

- Use **`git checkout`** when:
    
    - You want to switch branches.
        
    - You want to explore a specific commit without altering anything.
        
- Use **`git reset`** when:
    
    - You want to undo commits or go back to a previous state.
        
    - You want to unstage changes or discard changes entirely.
        

---

### **Example Walkthrough**

Suppose you have the following commit history:

commit 5: Add feature C
commit 4: Add feature B
commit 3: Add feature A
commit 2: Fix bug
commit 1: Initial commit

1. **Using `git checkout`**:
    
    - To switch to `commit 3`:
        
        git checkout commit_3_hash
        
    - Your working directory will now match the state of `commit 3`.
        
    - If you make changes and commit them, you’ll be in a detached `HEAD` state.
        
2. **Using `git reset`**:
    
    - To undo `commit 4` and `commit 5` and go back to `commit 3`:
        
        git reset --hard commit_3_hash
        
    - This will move the `HEAD` to `commit 3`, and your working directory and staging area will also match `commit 3`.
        
    - **Warning**: This will permanently discard `commit 4` and `commit 5`.
        

---

### **Final Notes**

- **`git checkout`** is for **navigating** (switching branches or exploring commits).
    
- **`git reset`** is for **undoing** (moving `HEAD` and optionally discarding changes).
    
- Always be cautious with `git reset --hard` as it can permanently delete changes.



------------------<<<<<<<<<<<<<>>>>>>>>>>>>>>>>>-----------------


### **`git revert`**

The `git revert` command is used to **undo the changes introduced by a specific commit** by creating a new commit that reverses those changes. Unlike `git reset`, which rewrites history, `git revert` is a **safe and non-destructive** way to undo changes because it preserves the commit history. This makes it ideal for collaborative workflows where rewriting history could cause issues for others.

---

### **How Does `git revert` Work?**

1. **Identifies the Commit**:
    
    - You specify the commit you want to undo (e.g., `git revert commit_hash`).
        
    - Git analyses the changes made in that commit.
        
2. **Creates a New Commit**:
    
    - Git creates a new commit that **reverses the changes** introduced by the specified commit.
        
    - This new commit is added to the commit history, so the original commit remains intact.
        
3. **Preserves History**:
    
    - Since `git revert` adds a new commit, it does not alter the existing commit history. This is why it’s considered a safe operation, especially in shared repositories.


EX:

### **our Commit History Before Revert**

commit 5: Add feature C
commit 4: Add feature B
commit 3: Add feature A
commit 2: Fix bug
commit 1: Initial commit

- You want to **revert `commit 4`** (Add feature B).
    
- After running `git revert commit_4_hash`, Git creates a new commit (`commit 6`) that **undoes the changes made in `commit 4`**.
    

---

### **Your Commit History After Revert**

commit 6: Revert "Add feature B"
commit 5: Add feature C
commit 4: Add feature B
commit 3: Add feature A
commit 2: Fix bug
commit 1: Initial commit

---

### **What Happens to Your Project Directory?**

After `commit 6` (the revert commit), your project directory will reflect the state of the repository **as if `commit 4` never happened**, but **all other commits (`commit 1`, `commit 2`, `commit 3`, and `commit 5`) will still be applied**.

#### Key Points:

1. **Changes from `commit 4` are removed**:
    
    - If `commit 4` added a file, that file will be deleted.
        
    - If `commit 4` modified a file, those modifications will be reverted.
        
    - If `commit 4` deleted a file, that file will be restored.
        
2. **Changes from other commits (`commit 1`, `commit 2`, `commit 3`, and `commit 5`) are preserved**:
    
    - Any changes made in these commits will still be present in your project directory.
        
3. **Your project directory will reflect the state of `commit 6`**:
    
    - This means the directory will look like `commit 5` (Add feature C) **minus the changes introduced by `commit 4`**.