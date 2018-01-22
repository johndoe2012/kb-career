# Git Remote Management

## Remote Repository

Git remote repository is maintained by multiple collaborators, therefore usually hosted on the Internet. Status of remote can be different from local repository of individual collaborator, and needs to be synced from time to time. 

- `git remote -v`: list all the remote repos currently associated with local repo. It shows the short name, url, and permission as fetch or push. 
- `git remote show <shortname>`: inspect specified remote repo. It shows all the branches it has and they are tracked or not. 
- `git remote add <shortname> <url>`: add a new remote repo to local repo. 
- `git remote rename <oldshortname> <newshortname>`: rename a remote repo.
- `git remote remove <shortname>`: remove a remote repo. 

### Import Existing Repository to a Server

We have an existing local Git repository, and want to copy it to a remote server. The repository on a server is a **bare repository, one without working directory**. 

1. `cd` to the **parent level** of local Git repository. 
2. Run `git clone --bare localrepo barerepo.git`. A bare repo named `barerepo.git` is created as a clone of local project. 
3. Copy the bare repository onto server, `scp -r barerepo.git <server url>:<path>/barerepo.git`.
4. **Do another clone** is an easy way to make local directory track the remote one. Run `git clone <server url>:<path>/barerepo.git localproj`. 
5. On server, `cd` to `<path>/bareproject.git` and run `git init --bare --shared` to **set proper write permission** to the repo on server.

### Create New Git Repository on a Server

We want to create a new empty Git repository on a server.

1. On server, `cd` to the **parent level** of project root. 
2. Run `git init --bare barerepo.git`. A Git directory named `bareproject.git` is created. 
3. On local machine, Run `git clone <server url>:<path>/barerepo.git localrepo`. 

## Branch Synchronization

### Remote Branch and Tracking Branch

***Remote branch*** is actually a **local reference** to the state of branches on remote repo. It usually takes the form of `<remote repo>/<remote branch>`, such as `origin/master`. You **cannot modify it except for syncing** it with the remote repo. Local repo can set up multiple remote branches with different remote repos, just as local branches. 

***Tracking branch*** is a **local branch** that tracks a remote branch, sometimes called **upstream**. You can **make commits to it as usual, merge it with remote branch**, and sync the changes to on a remote branch to the remote repo. 

### Create Remote Branches

#### Clone

`git clone` is a special case of synchronization. It will automatically do the following: 

- create local repo and add a remote repo called `origin`
- set up a remote branch called  `origin/master`
- create a tracking branch called `master` which points to the same commit as `origin/master`. 

![remote branch clone](./res/remote-branches-clone.png)

#### Fetch

Use `git fetch origin` to create remote branches for remote repo `origin` to existing repo. 

![remote branch fetch](./res/remote-branches-fetch.png)

Use `git fetch teamone` to add another remote repo and its remote branches. 

![remote branch fetch multiple](./res/remote-branches-fetch2.png)

### Update Remote Branches

Your remote branch `origin/master` shows a cache of `master` branch on remote repo since last time you synchronize. When someone pushed changes to remote `master`, it becomes out of sync with your remote branch `origin/master`. 

**Use `git fetch --all; git branch -vv` regularly** to keep your remote branch up-to-date. 

### Track Remote Branches

When you do a `git fetch` that brings down new remote branches, you do **NOT automatically have a local editable copies** of them. A local branch must be created to track the remote branch. 

For instance you fetched `origin/hotfix` and want to extend your work based on it, you have two options: 

- Use `git merge origin/hotfix master` to merge `origin/hotfix` into your work then making further changes. 
- Use `git checkout -b hotfix origin/hotfix` to create a new local branch that automatically tracks the remote branch. 
- Use `git branch -u origin/hotfix` to make current local branch tracking the remote branch. Then merge the remote branch `git merge origin/hotfix`. 

> **[info] Note:** you can merge or rebase local branch with remote branch the same way as another local branch. 

### Push to Remote

When local work is done then you want to update it to the remote repository. This requires you to explicitly push to remote repository to synchronize your work. 

- Use `git push origin master` to sync local `master` branch with remote branch with the same name on repo `origin`. 
- Use `git push origin master:remotemaster` to sync with remote branch of different name. 


> **[info] NOTE:**  `git push` will create a local branch in remote repo. 

> **[warning] WARN:** a `git push` fails if it results in a `non-fast-forward` merge in the remote repo. Make sure you **do a fetch first, and merge** local branch with latest remote branch, before push. 

#### Force Push

Use `git push --force` to override the `non-fast-forward` merge failure. It will **delete any remote changes** that may have occurred since last sync, and make the remote branch match your local branch. 

The only occasion you should ever use force push is when you pushed incorrect commits but fixed them with `git commit --amend` and `git rebase -i`, AND you make sure no one has checked out those incorrect commits you pushed. 


#### Delete

Use `git push <shortname> --delete <remotebr>` to delete a remote branch. 

## Distributed Workflow

### Small Private Team

In a small private team developers are equal and all have rights to push to the public repo. Merges are done client-side at commit time before pushed to server. 

The typical workflow is:

- **clone** the repo to local.
- create topic **branch** and work on it for a while. 
- **merge** the topic branch to local master branch. 
- **fetch** `origin/master`, then **merge** it into local `master`.
- **test** the code still working properly. 
- **push** to `origin/master`. 

### Managed Private Team

In a larger managed team we have small groups working in parallel on different features, and a manager that integrates group-based contributions. All groups have rights to create topic branches on the repo, but only the manager has rights to merge the topic branches to repo `master`. 

The typical workflow is following: 

- **clone** the repo to local.
- create a topic **branch** `featureA`, and work on it for a while. 
- **push** `featureA` to `origin` by creating a new remote topic **branch** `origin/featureA`. 
- **inform** group members about the addition. 
- create a new topic **branch** `featureB` based on `origin/master`, and work on it for a while. 
- be notified that changes about feature B has been made on `origin/featureBee`. 
- **fetch** `origin` to check out the changes. 
- **merge** changes from `origin/featureBee` onto our local work on `featureB`. 
- push to `origin/featureBee` and make `featureB` tracks `origin/featureBee`. 
- inform manager to merge `origin/featureA` and `origin/featureBee` to `origin/master`. 
- fetch `origin/master`. merge it to local `master`. 

### Forked Public Project

When contributing to public project, you do not have the permission to push or create branches on remote repo. 

The typical workflow is following: 

- **clone** the repo to local. 
- create a topic **branch**, and work on it for a while. 
- **fork** the project on hosting site to create your own writable fork of the project. 
- **add** the fork repo as second **remote** (first is the origin repo you cannot write). 
- **push** the topic branch to fork repo. Note **do not merge** to `master` then push `master`. 
- initiate a **pull request** to the maintainer. 