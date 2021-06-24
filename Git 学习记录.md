### Git 学习记录

下载一个 gitjk 插件，能提示上一条 git 指令怎么原样撤销

- git init 

  在当前目录下创建 git 仓库

- git add a.txt

  添加文件到暂存区

- git add .

  提交新文件(new)和被修改(modified)文件，不包括被删除(deleted)文件

- git add -u

  提交被修改(modified)和被删除(deleted)文件，不包括新文件(new)

- git add -A

  提交所有变化

- git status 

  查看当前工作区分支、本地文件修改和暂存区情况

- git reset a.txt

  将文件移除暂存区

- git cat-file -p sha-1 值

  查看指定的 sha-1 值对应的文件的内容

- git ls-files --stage

  查看 index 暂存区

- git commit -m '提交信息'

  将暂存区的内容提交到本地仓库 并添加提交信息

- git branch

  查看所有分支

- git branch name

  创建新的分支

- git branch -D name

  删除分支

- git branch -merged 

  查看哪些分支合并进了当前分支

- git checkout branch

  切换分支

- git checkout -b name

  创建分支并切换为工作分支

- git checkout sha-1 值

  回退到某个固定版本

-  git merge xxx

  将指定分支和当前分支合并

- git merge -abort

  合并失败后 放弃合并

- git log 

  查看日志

-  git rebase name

  找到当前分支和指定分支的共同父节点，然后使用指定的分支作为当前分支的前快照，形成一个新的快照链。

  注意，这样可能会影响相关提交的信息。

- git remote add origin xxxxxx.git

  创建远程仓库 origin 是起一个别名 可以创建多个远程仓库

- git push -u origin main

  推送到远程仓库

- git clone xxx.git 

  拉取项目

- git pull origin main

  拉取远程 main 分支和当前分支合并

- git pull origin main:xxx

  拉取远程 main 分支和指定分支合并

- git stash

   https://www.cnblogs.com/tocy/p/git-stash-reference.html