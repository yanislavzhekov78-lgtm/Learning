
Version control systems (VCSs) are tools used to track changes to source code (or other collections of files and folders). As the name implies, these tools help maintain a history of changes; furthermore, they facilitate collaboration. VCSs track changes to a folder and its contents in a series of snapshots.

Snapshot - each snapshot encapsulates the entire state of files/folders within a top-level directory

Git - the de facto standard for version control

Git thinks of its data more like a series of snapshots of a miniature filesystem. With Git, every time you commit, or save the state of your project, Git basically takes a picture of what all your files look like at that moment and stores a reference to that snapshot. To be efficient, if files have not changed, Git doesn’t store the file again, just a link to the previous identical file it has already stored. Git thinks about its data more like a stream of snapshots.

If you want to see the changes introduced between the current version of a file and the file a 
month ago, Git can look up the file a month ago and do a local difference calculation

Git has three main states that your files can reside in: modified, staged, and committed:
• Modified means that you have changed the file but have not committed it to your database yet.
• Staged means that you have marked a modified file in its current version to go into your next
commit snapshot.
• Committed means that the data is safely stored in your local database.

This leads us to the three main sections of a Git project: the working tree, the staging area, and the
Git directory.

![[Pasted image 20261007085048.png]]

- The working tree is a single checkout of one version of the project. These files are pulled out of the compressed database in the Git directory and placed on disk for you to use or modify.
- The staging area is a file, generally contained in your Git directory, that stores information about what will go into your next commit. 
- The Git directory is where Git stores the metadata and object database for your project. This is the most important part of Git, and it is what is copied when you clone a repository from another computer.

The basic Git workflow goes something like this:
1. You modify files in your working tree.
2. You selectively stage just those changes you want to be part of your next commit, which adds
only those changes to the staging area.
3. You do a commit, which takes the files as they are in the staging area and stores that snapshot
permanently to your Git directory.

## Git's data model

Definitions:
 - "blob" = file (bunch of bytes)
 - "tree" = directory (maps names to blobs or trees)
 - snapshot = top-level tree that is being tracked
- history = directed acyclic graph (DAG) of snapshots
	-  each snapshot in Git refers to a set of “parents”, the snapshots that preceded it
	- set of parents rather than a single parent because a snapshot might descend from multiple parents, for example, due to combining (merging) two parallel branches of development.
- snapshot = commit
- Commits are immutable !

Data models as pseudocode:

```
// a file is a bunch of bytes
type blob = array<byte>

// a directory contains named files and directories
type tree = map<string, tree | blob>

// a commit has parents, metadata, and the top-level tree
type commit = struct {
    parents: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

An “object” is a blob, tree, or commit:

```
type object = blob | tree | commit
```

In Git’s data store, all objects are content-addressed by their SHA-1 hash ()

```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

Blobs, trees, and commits are unified in this way: they are all objects. When they reference other objects, they don’t actually _contain_ them in their on-disk representation, but have a reference to them by their hash.

## References

Now, all snapshots can be identified by their SHA-1 hashes. That’s inconvenient, because humans aren’t good at remembering strings of 40 hexadecimal characters.

Git’s solution to this problem is human-readable names for SHA-1 hashes, called “references”. References are pointers to commits. Unlike objects, which are immutable, references are mutable (can be updated to point to a new commit). For example, the `master` reference usually points to the latest commit in the main branch of development.

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

With this, Git can use human-readable names like “master” to refer to a particular snapshot in the history, instead of a long hexadecimal string.

!!! In Git, that “where we currently are” is a special reference called “HEAD”.

## Repositories

Finally, we can define what (roughly) is a Git _repository_: it is the data `objects` and `references`.
...

# Staging area

This is another concept that’s orthogonal to the data model, but it’s a part of the interface to create commits....
Git accommodates such scenarios by allowing you to specify which modifications should be included in the next snapshot through a mechanism called the “staging area

# Git command-line interface

## Basics

- `git help <command>`: get help for a git command
- `git init`: creates a new git repo, with data stored in the `.git` directory
- `git status`: tells you what’s going on
- `git add <filename>`: adds files to staging area
- `git commit`: creates a new commit
- `git log`: shows a flattened log of history
- `git log --all --graph --decorate`: visualizes history as a DAG
- `git diff <filename>`: show changes you made relative to the staging area
- `git diff <revision> <filename>`: shows differences in a file between snapshots
- `git checkout <revision>`: updates HEAD (and current branch if checking out a branch)

## Branching and merging

- `git branch`: shows branches
- `git branch <name>`: creates a branch
- `git switch <name>`: switches to a branch
- `git checkout -b <name>`: creates a branch and switches to it
    - same as `git branch <name>; git switch <name>`
- `git merge <revision>`: merges into current branch
- `git mergetool`: use a fancy tool to help resolve merge conflicts
- `git rebase`: rebase set of patches onto a new base

## Remotes

- `git remote`: list remotes
- `git remote add <name> <url>`: add a remote
- `git push <remote> <local branch>:<remote branch>`: send objects to remote, and update remote reference
- `git branch --set-upstream-to=<remote>/<remote branch>`: set up correspondence between local and remote branch
- `git fetch`: retrieve objects/references from a remote
- `git pull`: same as `git fetch; git merge`
- `git clone`: download repository from remote

## Undo

- `git commit --amend`: edit a commit’s contents/message
- `git reset <file>`: unstage a file
- `git restore`: discard changes

# Advanced Git

- `git config`: Git is [highly customizable](https://git-scm.com/docs/git-config)
- `git clone --depth=1`: shallow clone, without entire version history
- `git add -p`: interactive staging
- `git rebase -i`: interactive rebasing
- `git blame`: show who last edited which line
- `git stash`: temporarily remove modifications to working directory
- `git bisect`: binary search history (e.g. for regressions)
- `git revert`: create a new commit that reverses the effect of an earlier commit
- `git worktree`: check out multiple branches at the same time
- `.gitignore`: [specify](https://git-scm.com/docs/gitignore) intentionally untracked files to ignore

## Pro Git book 

Remember that each file in your working directory can be in one of two states: tracked or untracked. Tracked files are files that were in the last snapshot, as well as any newly staged files;
they can be unmodified, modified, or staged. In short, tracked files are files that Git knows about.
Untracked files are everything else — any files in your working directory that were not in your last
snapshot and are not in your staging area. When you first clone a repository, all of your files will be
tracked and unmodified because Git just checked them out and you haven’t edited anything.
As you edit files, Git sees them as modified, because you’ve changed them since your last commit.
As you work, you selectively stage these modified files and then commit all those staged changes,
and the cycle repeats.

![[Pasted image 20261001202919.png]]

Let’s say you add a new file to your project, a simple README file. If the file didn’t exist before, and you run `git status` and You can see that your new README file is untracked, because it’s under the “Untracked files” heading in your status output. Untracked basically means that Git sees a file you didn’t have in the previous snapshot (commit), and which hasn’t yet been staged; Git won’t start including it in your commit snapshots until you explicitly tell it to do so. It does this so you don’t accidentally begin including generated binary files or other files that you did not mean to include.

Tracking New Files:
 - In order to begin tracking a new file, you use the command git add. To begin tracking the README file, you can run this: `$ git add README`
 - If you run your status command again, you can see that your README file is now tracked and staged to be committed
 - the git add command takes a path name for either a file or a directory; if it’s a directory, the command adds all the files in that directory recursively.
 - git add is a multipurpose command — you use it to begin tracking new files, to stage files, and to do other things like marking merge-conflicted files as resolved. It may be helpful to think of it more as “add precisely this content to the next commit” rather than “add this file to the project”.
 - If you modify a file after you run git add, you have to run git add again to stage the latest version of the file

`git diff`
- What have you changed but not yet staged? 
- And what have you staged that you are about to commit? 
- To see what you’ve changed but not yet staged, type `git diff` with no other arguments
- If you want to see what you’ve staged that will go into your next commit, you can use `git diff --staged`. This command compares your staged changes to your last commit

`git log`
- lists the commits made in that repository in reverse chronological order; that is, the most recent commits show up first.
- One of the more helpful options is `-p` or `--patch`, which shows the difference (the patch output) introduced in each commit. You can also limit the number of log entries displayed, such as using -2 to show only the last two entries: `$ git log -p -2` ; displays the same information but with a diff directly following each entry. This is very helpful for code review or to quickly browse what happened during a series of commits that a collaborator has added. 
- if you want to see some abbreviated stats for each commit, you can use the `--stat` option...prints below each commit entry a list of modified files, how many files were changed, and how many lines in those files were added and removed. 
- `--pretty` - changes the log output to formats other than the default. A few prebuilt option values are available for you to use. The `oneline` value for this option prints each commit on a single line, which is useful if you’re looking at a lot of commits. In addition, the `short`, `full`, and `fuller` values show the output in roughly the same format but with less or more information, respectively.
- The `oneline` and `format` option values are particularly useful with another log option called `--graph`. This option adds a nice little ASCII graph showing your branch and merge history:
- `-S` option (colloquially referred to as Git’s “pickaxe” option), which takes a string and shows only those commits that changed the number of occurrences of that string: For instance, if you wanted to find the last commit that added or removed a reference to a specific function, you could call: `$ git log -S function_name`

## Undoing Things

One of the common undos takes place when you commit too early and possibly forget to add some files, or you mess up your commit message. If you want to redo that commit, make the additional changes you forgot, stage them, and commit again using the `--amend` option: `$ git commit --amend` - 
As an example, if you commit and then realize you forgot to stage the changes in a file you wanted to add to this commit, you can do something like this:
```
$ git commit -m 'Initial commit'
$ git add forgotten_file
$ git commit --amend
```

Unstaging a Staged File -  `git reset HEAD <file>`  OLD look get restore!!!

Unmodifying a Modified File - `git checkout -- <file>` (old look get restore!!) to discard changes in working directory !!!! It’s important to understand that `git checkout -- <file>` is a dangerous command - Any local changes you made to that file are gone — Git just replaced that file with he last staged or committed version. Don’t ever use this command unless you..

**Undoing things with `git restore`**
- an alternative to git reset (that is used in new git)
- Unstaging a Staged File with git restore: `git restore --staged <file>`
- Unmodifying a Modified File with git restore:  `git restore <file>` - again is dangeroues! 

## Working with Remotes

 Remote repositories are versions of your project that are hosted on the Internet or
network somewhere. You can have several of them, each of which generally is either read-only or
read/write for you. Collaborating with others involves managing these remote repositories and
pushing and pulling data to and from them when you need to share work.

- Showing your remotes: `git remove -v` - showing your remotes the URLs that Git has stored
- Adding remote repositories: 
	- `git remote add <shortname> <url>`
	- Example: `git remote add pb https://github.com/paulboone/ticgit` -> now  you can use the string `pb` on the command line instead of the whole URL
- Fetching - get data from your remote projects
	- `git fetch <remote>`
	- The command goes out to that remote project and pulls down all the data from that remote project that you don’t have yet. After you do this, you should have references to all the branches from that remote, which you can merge in or inspect at any time
	- fetches any new work that has been pushed to that server since you cloned (or last fetched from) 
	- It’s important to note that the git fetch command only downloads the data to your local repository — it doesn’t automatically merge it with any of your work or modify what you’re currently working on. You have to merge it manually into your work when you’re ready.
- Pulling - automatically fetch and then merge that remote branch into your current branch. 
	- `git clone <remote>`
- Pushing:
	- `git push <remote> <branch>`
	- Example: `git push origin master` -  push your master branch to your origin server
	- This command works only if you cloned from a server to which you have write access and if nobody has pushed in the meantime.
- Inspecting a remote -  see more information about a particular remote
	- `git remote show <remote>`

## Tagging

Like most VCSs, Git has the ability to tag specific points in a repository’s history as being important.
Typically, people use this functionality to mark release points (v1.0, v2.0 and so on).
- `git tag` - listing the existing tags in git in alphabetical order
- Lightweight tag - very much like a branch that doesn’t change - it’s just a pointer to a specific commit
- Annotated tags =  however, are stored as full objects in the Git database; they’re checksummed; contain the tagger name, email, and date; have a tagging message;
- Create an annotated tag: `git tag -a v1.4 -m "my version 1.4"`
	- The `-m` specifies a tagging message
- Sharing tags - By default, the git push command doesn’t transfer tags to remote servers. You will have to explicitly push tags to a shared server after you have created them. This process is just like sharing remote branches — you can run `git push origin <tagname>`.
- Checking out tags - If you want to view the versions of files a tag is pointing to, you can do a git checkout of that tag, although this puts your repository in “detached HEAD” state: `git checkout v2.0.0`

# GIT BRANCHING

How git stores data:
When you make a commit, Git stores a commit object that contains a pointer to the snapshot of the content you staged. To visualize this, let’s assume that you have a directory containing three files, and you stage them all and commit. Staging the files computes a checksum for each one (the SHA-1), stores that version of the file in the Git repository (Git refers to them as blobs), and
adds that checksum to the staging area. When you create the commit by running git commit, Git checksums each subdirectory (in this case, just the root project directory) and stores them as a tree object in the Git repository.

First commit:
![[Pasted image 20261008083338.png|443]]

If you make some changes and commit again, the next commit stores a pointer to the commit that
came immediately before it

![[Pasted image 20261008083408.png|489]]

A branch in Git is simply a lightweight movable pointer to one of these commits. The default branch name in Git is master. As you start making commits, you’re given a master branch that points to the last commit you made. 

**CREATING A NEW BRANCH**
- `git branch <name>`
- This creates a new pointer to the same commit you’re currently on.
- Example `git branch testing`
- Create a branch - creates a new pointer for you to move around; оnly created a new branch - it didn’t switch to that !!!
- HEAD -  pointer to the local branch you’re currently on
![[Pasted image 20261008083809.png|323]]

**SWITCHING BRANCHES**
- `git checkout <branch name>`
- From the example: `git branch testing` - this moves HEAD to point to the testing branch
- ![[Pasted image 20261008084014.png|335]]
- now if we do commit: 
- ![[Pasted image 20261008084041.png|343]]
- if you switch to master and rung `git log` there will be no testing branch - git log will only show commit history below the branch you’ve checked out. (need to specify `-all`). When switching the command did two things `git checkout master` - It moved the HEAD pointer back to point to the master branch, and it reverted the files in your working directory back to the snapshot that master points to. 
- CREATE AND SWITCH TO THE NEW BRANCH: `git checkout -b <newbranchname>`
- you can use `git switch` as alternative to `git checkout`

**GIT MERGE**
Example: starting point master -> create and commit branch `iss53` to work on issue 53 -> needed to fix some bug and went back to master and created and commit `hotifx` branch. 
![[Pasted image 20261008085501.png|430]]
- to merge the hotfix branch back into your master:
	- `git checkout master` and then `git merge hotfix`
	- it will say `Fast-forward` : Because the commit C4 pointed to by the branch hotfix you merged in was directly ahead of the commit C2 you’re on, Git simply moves the pointer forward. To phrase that another way, when you try to merge one commit with a commit that can be reached by following the first commit’s history, Git simplifies things by moving the pointer forward because there is no divergent work to merge together 
![[Pasted image 20261008085544.png|445]]
- delete hotfix branch as now we don't need it -> `git branch -d hotfix`
- switch back to branch `iss53` to work further and commit it
- ![[Pasted image 20261008085654.png|594]]
- now you can merge merge your `iss53` branch into `master` -> `git checkout maste` ang then  `git merge iss53` - because the commit on the branch you’re on isn’t a direct ancestor of the branch you’re merging in, Git has to do some work. In this case, Git does a simple three-way merge, using the two snapshots pointed to by the branch tips and the common ancestor of the two.
- ![[Pasted image 20261008085929.png|511]]
- Instead of just moving the branch pointer forward, Git creates a new snapshot that results from this three-way merge and automatically creates a new commit that points to it. This is referred to as a merge commit, and is special in that it has more than one parent
- ![[Pasted image 20261008090034.png|591]]
- Now you can delete iss53: `git branch -d iss53`

**BASIC MERGE CONFLICTS**
- If you changed the same part of the same file differently in the two branches you’re merging, Git won’t be able to merge them cleanly. If your fix for issue #53 modified the same part of a file as the hotfix branch, you’ll get a merge conflict. Git hasn’t automatically created a new merge commit. It has paused the process while you resolve the conflict. Run `git status` and it will show which files
- after resolve manually the conflict run `git add` on each file to mark it as resolved. 
- You can use `git mergetool`

**BRANCH MANAGEMENT**
- `git branch` - with no arguments -> simple listing of your current branches 
- ....

**REBASING**
In Git, there are two main ways to integrate changes from one branch into another: the merge and
the rebase.

![[Pasted image 20261008093313.png|374]]

- you can take the patch of the change that was introduced in C4 and reapply it on top of C3. In Git, this is called rebasing. With the rebase command, you can take all the changes that were committed on one branch and replay them on a different branch
-  `git checkout experiment` and then `git rebase master
![[Pasted image 20261008093455.png|577]]
- now you can `git checkout master` and `git merge experiment`

