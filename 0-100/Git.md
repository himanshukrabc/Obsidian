---
aliases: []
---
- Local Repo : these are replication of the private copy of the main git repo.
- push : send changes to git repo
- pull : copy changes to local repo
#### Workflow of git
1. modify files in local repo
2. add files to staging area
3. commit changes to the main repo\

#### Blobs
All files are stored in Binary Large Object format. Metadata is required to make sense of the file. Name = SHA1 hash of the file.
#### Trees
It represents a directory. Stores directories and blobs through their SHA1 references.
#### Commits
Commit Objects represent states of the repo. They are arranged in a linked list.
Each object has parent pointer which can be used to traverse back through the history of the repo.
If a commit has multiple parents then it is formed by merging 2 branches.\
## Git Commands
- #### init
	initializes git folder in the local repo. Now it can be pushed into git repo.
	`git inti`

- #### clone
	to copy a remote repo to a local repository
	`git clone __repo_url__`

- #### add
	add all changes to the staging area
	`git add filename`
	`git add .`

- #### status
	This command displays the state of the local directory and the staging area. Which files are committed, modified etc.
	- red -> modified, unstaged(untracked)
	- green -> staged
	- after commit nothing appears
	`git status`

- #### remote
	sets the directory to which the changes need to be pushed.
	`git remote add origin __remote__`
	`git remote -v` - shows the current push and pull origins

- #### commit
	commit the changes from the staging area.
	It stores a snapshot of your local repo. You can revert to a snapshot after making changes.
	`git commit -m __message__`

- #### push
	After commit the changes are still in the staging area. 
	Push all changes to the remote repo(origin specifies)
	`git push origin __branch_name__`

- #### pull
	pull all the changes from the remote repo``
	`git pull origin __branch_name__`

- #### stash
	create a stash of the current changes to revert back to last commit. The stash can be popped to return to the stash state.
	`git stash`
	`git stash pop`
- #### branch
	tells the current branch,
	### Branches
	These are copies of the main repo created to develop new features.
	The main repo is usually used for production env. The dev env runs of branches.
	- `Merging` - used to merge branches together.
	- `Conflicts` - codes may conflict between branches due to changes during merge
	`git branch`

- #### checkout
	change to a different branch
	`git checkout -b __branchname`

- #### diff
	shows the difference of local repo from last commit.
	`git diff`

- #### merge
	`git merge __mainbranchname`
	Usually you dont merge. You create a pull request and someone reviews and merges the code.

- #### Creating a pull request
	- Go to repo
	- There should be a button for creating a pull request

- #### Handling Merge Conflicts
	- Happens when you try to push code to a repo which has already been changed.
	- Since you didn't pull the latest code, it will lead to conflicts.
	- Types of conflicts -
		1. While starting merge - because of files in the staging area.
		2. During merge - Due to conflict between the local branch and the branch being merged to.
	- This has to be resolved manually. 	  ![[Pasted image 20240519150733.png]]
	- 