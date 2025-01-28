

### 1. **`git clone`**

- **What it does**: Creates a local copy of a remote repository on your machine.
    
- **When to use**: When you want to download a repository from GitHub (or any remote server) to your PC for the first time.
    
- **How it works**:
    
    - It copies the entire repository, including all branches, commits, and files.
        
    - It automatically sets up a remote called `origin` pointing to the repository you cloned.
        
- **Example**:
    
    git clone https://github.com/username/repository.git
    
    This will create a folder named `repository` on your PC with all the code and Git history.
    

---

### 2. **`git remote add`**

- **What it does**: Adds a connection to a remote repository (like GitHub) to your local repository.
    
- **When to use**: When you already have a local repository and want to link it to a remote repository (e.g., after initializing a new Git repo locally or if you want to add multiple remotes).
    
- **How it works**:
    
    - It adds a reference to a remote repository so you can push or pull changes to/from it.
        
    - You can add multiple remotes (e.g., `origin`, `upstream`) to your local repository.
        
- **Example**:
    
    git remote add origin https://github.com/username/repository.git
    
    This links your local repository to the remote repository at the specified URL. You can then push or pull changes using `origin`.
    

---

### 3. **`git pull`**

- **What it does**: Fetches changes from a remote repository and merges them into your current branch.
    
- **When to use**: When you want to update your local repository with the latest changes from the remote repository.
    
- **How it works**:
    
    - It combines two commands: `git fetch` (downloads changes from the remote) and `git merge` (merges those changes into your current branch).
        
- **Example**:
    
    git pull origin main
    
    This fetches the latest changes from the `main` branch of the `origin` remote and merges them into your current branch.




-> git fetch downloads the changes from the destination branch only. it doesn't change the working directory. to change the working directory, we have to use git merge after git fetch.