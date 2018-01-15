# Git Introduction

## Version Control Systems

Version control systems (VCS) is a system that record changes to files over time so that you can

- recall specific versions later
- revert files or entire project back to a previous state
- compare changes over time
- see ownership of changes

There are three types of VCS: 

| VCS         | Description                              | Example                         |
| ----------- | ---------------------------------------- | ------------------------------- |
| Local       | A local version database is used to keep track of every revision. Hard to support collaboration. | `rcs`                           |
| Centralized | A version database is kept at central version for everyone to check out the revisions. Single point of failure. | `CVS`, `Subversion`, `Perforce` |
| Distributed | Everyone gets a complete version database by fully mirroring the repository. Allow collaboration with different groups within the same project | `Git`, `Mercurial`              |

## Git Concepts

### Characteristics

- **Snapshot, Not Differences.** 

  This is the major difference between `Git` and other VCS. 

  - Other VCS stores revisions as **changes made to files** over time. 
  - `Git` takes **snapshot of all files** at the time, and revision is a reference to the snapshot. If a file is not changed, a symbolic link is created instead of a copy. 

- **Nearly Every Operation is Local.** 

  Copy of repository is kept locally so that operations can access and modify the copy easily and fast. To synchronize the local copy with remote copies, user needs to manually ask `git` to do so. 


- **Git has Integrity.** 

  Everything in `git` is check-summed with 40 digits long SHA-1 hash before it is stored, then can be referred by the checksum. 


- **Git Generally Only Adds Data.** 

  Nearly all operations in Git adds data to the database, so that it is hard to lose data. 

### Git Objects

Git is basically a **key-value data store**. Every time we stage some files and make a commit, three types of objects are created:

- it takes **snapshot of each file's content**, called `blob` object. A hash key is assigned to each `blob` object, and the `blob` is stored in a directory named by the key. Note `blob` is not exactly the file, but a snapshot of content. 

- it creates a `tree` object to keep track of mapping between hash key and actual **filenames in the file system**. 

- it creates a `commit` object with **meta data and a pointer to the tree**.

  ![A commit and its tree](./res/objects.png)

When consecutive commits are made, common object includes **a pointer to its parent commit object**. 

![Commits and their parents](./res/commit-objects.png)

### Git Repository

Revision history is stored in Git repository. There are three main sections of a typical Git repository:

- Git directory: usually a **`.git` directory** in project root, where Git stores the metadata and object database for the project. 
- Working directory: the **project root directory**, where last version of files are extracted to for further modification. 
- Staging area: **an index file** inside `.git` directory, which records all files ready to be included in the next commit. 

A bare repository is a Git repository without working directory. It is same as the `.git` directory, and usually named as `<project>.git` by convention. 

#### Import Existing Directory

We have an existing project directory `myproject`, and would like to use Git to manage it. 

```shell
# create a local git repo rooted at myproject/
$ cd myproject
$ git init
Initialized empty Git repository in D:/Downloads/temp/myproject/.git/
$ ls -a
./  ../  .git/
```

> **[info] Note: ** **Nothing has been tracked by Git yet**, for which we need to make an initial commit. Also Git ignores empty directory. **Create at least one single file for tracking**.
>

#### Clone Existing Repository

We have a Git repository somewhere and want to create a local working directory of it. 

```shell
# clone an existing repo with working directory under current directory
$ ls
myproject/  myproject.git/
$ git clone myproject mynewproject
Cloning into 'mynewproject'...
warning: You appear to have cloned an empty repository.
done.
$ ls
mynewproject/  myproject/  myproject.git/
```

>  **[info] Note:** all files are **already been tracked, and unmodified**. 
>

#### Create Bare Repository

Both the `git clone` and `git init` commands has `--bare` option to create a bare repo. Usually `git init --bare` is used on the server. 

```shell
# create a bare git repo named myproject.git
$ cd myproject.git
$ git init --bare
Initialized empty Git repository in D:/Downloads/temp/myproject.git/
$ ls -a
./  ../  config  description  HEAD  hooks/  info/  objects/  refs/
```

### File Status

A file, with respect to Git system, has one of the four status: 

- untracked: newly created file that has not been in any snapshot nor the staging area
- committed/unmodified: file included in last snapshot
- dirty/modified: file changed since last snapshot but has not been staged yet
- staged: modified file marked to be included in next snapshot

![File status](./res/lifecycle.png)

#### Check Working Directory Status

Use `git status` to check status of files in working directory. 

```shell
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   README2

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   README

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        SUMMARY
```

Use `git status -s` to check short status of files. There are two columns for the short status. Left is for staging area, right is for working directory. 

```shell
$ git status -s
 M README            # modified, but not staged
M  lib/simplegit.rb	 # modified, then staged
MM Rakefile          # modified, then staged, then modified again
A  lib/git.rb        # new file, then staged
AM lib/repo.rb       # new file, then staged, then modified again
?? LICENSE.txt       # untracked
```

### Git Branch

A branch in Git is simply a lightweight **movable named pointer to one of the commit objects.** It moves forward every time a new commit object is created. 

- The default branch is `master`. Nothing special about it except for being created by `git init`. 
- Multiple branches can be created to point to the same commit object. 
- **`HEAD` is a special pointer that points to current branch pointer**. If a `HEAD` pointer points to a commit object directly, it becomes a *detached `HEAD`*. 

![A branch and its commit history](./res/branch-pointer.png)

## Git Configurations

### Configuration Levels

We can use `git config` tool to set and get the git variables. There are three levels of configuration: 

- System-wise: Use `git config --system` to set variable stored in `/etc/gitconfig`. 
- User-specific: Use `git config --global` to set variable stored in `~/.gitconfig`.
- Repository-specific: Use `git config` to set variable stored in `<proj-root>/.git/config` .
- Each level overrides same settings at broader level.

Use `git config --list` to check current configurations. 

### Common Settings

#### User Identity

You **MUST set `user.name` and `user.email`** before use.

```bash
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```

#### Editing Behaviour

Change default editing for `git`.

```shell
$ git config --global core.editor vim
$ git config --global color.ui auto
```

#### Alias

Use `git config --global alias.<newcommand> "<old command strings>"` to create alias for old command. 

```shell
$ git config --global alias.unstage "reset HEAD --"
$ git config --global alias.unmodify "checkout --"
$ git config --global alias.last "log -1 HEAD"
$ git config --global alias.tree "log --abbrev-commit --date=relative --branches --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset%n'"
```

### Git Ignore

Add files to `.gitignore` file to prevent Git from automatically adding them or showing them as untracked. 

- `.gitignore` should be **in working directory**, not the `.git` directory. 
- sub working directories can also have `.gitignore` files, which only applies to everything below. It is a good solution for tracking empty directories. 
- Files already tracked by Git are not affected. You must explicitly remove them from repo.

Rules for the `.gitignore` file are: 

- Blank lines or lines starting with `#` are ignored.
- End pattern with `/` to specify directory. It matches **any subdirectory** with the name. 
- Glob pattern
  - `*` matches zero or more characters, `?` matches single character. 
  - `[abc]` matches `a` or `b` or `c`.
  - `[0-9]` matches range 0 to 9.
  - `**` matches nested directory: `a/**/z` would match `a/z`, `a/b/z`, `a/b/c/z`
- Start pattern with `!` to negate it. 

