---
title: Advanced Git
date: 2018-06-25 20:12:34
categories:
- Development Environment
tags:
- Git
---
# Important concepts

![Git concepts](./git concepts.jpg)

## Repositories

Git has two repository types: local and remote.  The local repo is on your computer for only your direct use.  The remote repo is typically elsewhere and for your indirect use.  Git supports multiple remote repositories.

## Committing is a Multi-Step Process

Git is a three step process to share your files with the team:
1. Add. This copies new or updated files to the “stage” or “index” (you will see doc and info that use both terms).
2. Commit. This copies your staged files to the local repo.
3. Push. This copies your files from the local repo to the remote repo (only the changes the remote repos does not have).

## File Diffs in Workspace, Stage, and Repo

Files can exist in three locations with Git.  The same file can have different content in each location:
1. Committed in the repo: the HEAD version, the contents as the file was last committed.
2. Staged in the index: edits made or the file removed, added to the index, ready to commit.
3. Workspace: Work in progress (usually most files are unchanged, having the same content as the committed version).

The Git diff command options allow comparing between the three locations (and more).
* Specifying no option compares the workspace to the staged/index version.
* Specifying –cached or –staged compares the staged/index version with a committed version.
* Additional diff options include comparing with any two files on disk, comparing two commits, and any file with any commit.

## Scenario-based Commands

### 远程操作

#### 从远程主机克隆一个版本库
* git clone <版本库的网址>
* git clone <版本库的网址> <本地目录名>
* git clone --recursive git://github.com/foo/bar.git

#### 管理主机名
* git remote
* git remote -v
* git remote show <主机名>
* git remote add <主机名> <网址>

#### 将远程主机的版本库更新取回本地
* git fetch <远程主机名>
* git fetch <远程主机名> <分支名>

git fetch命令通常用来查看其他人的进程，因为它取回的代码对你本地的开发代码没有影响。

所取回的更新，在本地主机上要用"远程主机名/分支名"的形式读取。比如origin主机的master，就要用origin/master读取。

#### 远程分支
* git branch -r
* git branch -a

取回远程主机的更新以后，可以在它的基础上，使用git checkout命令创建一个新的分支:
* git checkout -b newBrach origin/master

也可以使用git merge命令或者git rebase命令，在本地分支上合并远程分支:
* git merge origin/master
* git rebase origin/master

取回远程主机某个分支的更新，再与本地的指定分支合并:
* git pull <远程主机名> <远程分支名>:<本地分支名>
* git pull origin next 远程分支是与当前分支合并, 等同于先做git fetch，再做git merge

在某些场合，Git会自动在本地分支与远程分支之间，建立一种追踪关系（tracking）。比如，在git clone的时候，所有本地分支默认与远程主机的同名分支，建立追踪关系，也就是说，本地的master分支自动"追踪"origin/master分支。

Git也允许手动建立追踪关系:
* git branch --set-upstream master origin/next

如果当前分支与远程分支存在追踪关系，git pull就可以省略远程分支名:
* git pull origin

如果当前分支只有一个追踪分支，连远程主机名都可以省略:
* git pull

如果合并需要采用rebase模式，可以使用--rebase选项:
* git pull --rebase <远程主机名> <远程分支名>:<本地分支名>

加上参数 -p 就会在本地删除远程已经删除的分支:
* git pull -p

将本地分支的更新，推送到远程主机:
* git push <远程主机名> <本地分支名>:<远程分支名>

如果省略远程分支名，则表示将本地分支推送与之存在"追踪关系"的远程分支（通常两者同名），如果该远程分支不存在，则会被新建:
* git push origin master

如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支:
* git push origin :master

如果当前分支与远程分支之间存在追踪关系，则本地分支和远程分支都可以省略:
* git push origin

如果当前分支只有一个追踪分支，那么主机名都可以省略:
* git push

如果当前分支与多个主机存在追踪关系，则可以使用-u选项指定一个默认主机，这样后面就可以不加任何参数使用git push:
* git push -u origin master

将本地的所有分支都推送到远程主机:
* git push --all origin

git push不会推送标签（tag），除非使用--tags选项:
* git push origin --tags

### Frequently used

check last commit
* git show
* git log -n1 -p

check commit history for the feature branch
* git log develop..feature/REF-123

This will show you all not pushed commits from all branches
* git log --branches --not --remotes

and this must show you all your local commits
* git log origin/master..HEAD

view the changes in a commit
* git show 877897e8a64043cc9277a859c616ec2e4ff57634

To get the exact copy of remote develop branch, all local changes not pushed will be lost
* git fetch --all
* git reset --hard origin/master

To rename a branch
* git branch -m <oldname> <newname>
* git branch -m <newname>

stash
* git stash
* git stash list
* git stash apply
* git stash apply stash@{2}
* git stash drop stash@{2}
* git stash show stash@{2}

remove everything from the index/staging
-- dry run
* git clean -n
-- actually do the job
* git clean -f
-- do the job with both files and folders
* git clean -xf

remove all staged changes
* git reset HEAD -- .

remove all unstaged changes
* git checkout -- .

remove all untracked changes
* git clean -df

garbage collection
* git gc

Use the following command while being on "master", to list merged branches:
* git branch --merged | grep -v "\*"

To get the commit into current branch
* git cherry-pick COMMIT_HASH

## Best Practice

Each commit should be a single logical change. Don't make several logical changes in one commit.
Don't split a single logical change into several commits.

git commit --amend

#### Git commit message format

[guide](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)

## Use different SSH keys for multiple Bitbucket accounts

[guide](https://developer.atlassian.com/blog/2016/04/different-ssh-keys-multiple-bitbucket-accounts/)

## References
1. [使用原理视角看 Git](https://blog.coding.net/blog/principle-of-Git)
2. [git flight rules](https://github.com/k88hudson/git-flight-rules)
3. [Git远程操作详解](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)
4. [Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials)