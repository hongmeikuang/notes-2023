# git/repo/gerrit

# git 

Git是一个开源的分布式版本控制系统，用以有效、高速的处理从很小到非常大的项目版本管理。

## git 命令

git 是分布式版本控制系统；分为

    1、本地仓库：开发人员自己电脑上的Git仓库
    
    2、远程仓库：远程服务器上的Git仓库

clone：克隆，将远程仓库复制到本地

push：推送，将本地仓库代码上传到远程仓库

pull：拉取，将远程仓库代码下载到本地

git clean 

git clean -d -fx:强制删除本地修改代码

git push origin --delete my-branch   release的时候把不需要的仓库push了代码，所以在openlinux删掉这个分支

```
git push origin --delete buildroot-openlinux-202307-a113
```



git工作流程：

    1、将代码从远程仓库下载到本地仓库
    2、在本地仓库上checkout 代码然后进行代码修改
    3、在提交前先将代码提交到暂存区
    4、提交到本地仓库，本地仓库中保存修改的各个版本
    5、完成修改后，需要和团队成员共享代码，将代码push到远程仓库

相关git命令：
git init：把当前目录变成git仓库可以管理的仓库（把当前目录变成工作区）

git add ：把文件添加到仓库

git add . ：添加当前文件所有文件及文件夹到仓库

git commit -m "comment"把文件提交到仓库

git status:查看仓库当前的状态

git diff：查看修改之后的不同

git branch：查看分支

 git --no-pager log --oneline  -1000 > ~/work/1.txt

无分页1000条提交记录并写入1.txt

git restore <file>:直接丢弃工作区的修改，因为改乱了工作区某个文件的内容

git restore --staged <file>:当修改了某个工作区的内容并添加到了暂存区，但想丢弃修改，执行该命令后再执行上一步操作

git revert + id  将id所提交从当前分支剔除

git pull amlogic master 拉去主分支 

git cherry-pick commitid 把刚才你做好的改动拿到当前这个分支来。

## 工作中修改代码提交步骤
1、在buildroot_upload 目录下，保证分支正确

2、通过git branch -a，找到remotes/m/master对应的分支
3、git checkout -t amlogic/master(根据情况修改)

4、beyoundcompare(是一个软件)对比合并后

5、git add .:添加当前文件夹到仓库

6、git commit -s：查看具体修改内容

7、如果想要再次修改时：git commit --amend 

8、如果该仓库是第一次提交需要 add review
```
git remote add review ssh://hongmei.kuang@scgit.amlogic.com:29418/linux/amlogic/test_tools
```
仓库地址根据情况修改

9、然后push 到review ,分支名根据情况调整
```
git push review HEAD:refs/for/master
```
10、然后把提交后的url发送给hanliang.xiong和peipeng.zhao,发送时，对自己修改的内容进行简单的描述


## 缺少commit msg 

拷贝~/work/a113x2/code2/buildroot/.git/hooks/目录下的commit msg 到当前目录下的.git/hooks/目录

## git add代码之后需要切换分支提交的方法

在原来的分支已执行以下操作

```
git add .
git commit -s
```

但发现需要push代码的并不是这个分支，可以进行如下操作简便

先在当前分支

```
git format-patch HEAD~1
```

然后切换到正确的分支

```
git checkout master
git am 0001-buildroot-Remove-the-libplayer.-1-1.patch
```

最后执行

```
git push review HEAD:refs/for/master
```

以上便可以将代码快捷纠正push到正确的分支。

# repo 

Repo是谷歌用Python脚本写的调用git的一个脚本。主要是用来下载、管理Android项目的软件仓库（也就是说Repo是用来管理给Git管理的一个个仓库的）

repo是一个基于GIT的工具，它的主要目的是为了管理多个代码仓库，也就是多个GIT。

**repo 是用来管理git仓库的**

## 管理代码改动都是由 git 完成的，repo 在整个系统中主要担任了什么角色呢？

repo 在实际使用中主要担任2个角色：

- 和主代码服务器（gerrit）进行交互
  
- 根据前面提到的一个xml（manifest.xml）来管理多个 git 仓库

```
repo init -u url -b branchname

repo init -m manifest-name
```
- 在当前目录里面下载安装 repo:因为最初从网上下载的那个 repo 文件并不是一个完整的 repo，它主要负责初始化工作，并且在初始化完成以后将命令移交给完整的 repo 来执行。
  
- 根据命令中指定的地址（-u url）去下载项目的管理文件 manifest.xml。
manifest.xml 是用 git 管理起来的，在这里 -b branchname 就是指的 manifest.xml 的相应 branch。

- -m :指定所需要的manifests库中的清单文件。默认情况下，会使用maniftests/default.xml
  

初始化的时候，repo 会在当前目录下面建立一个 .repo 目录，然后把刚才提到的从网上下载下来的所有文件都放在这个目录里面。

```
repo sync
```
同步所有的项目

但如果只想改动某个项目时，常用命令为：
```
repo sync project
```
在执行 repo sync (project)的时候，它干的第一件事就是先把 .repo/manifests/ 下面的内容同步到最新，然后再根据最新的 manifest.xml 里面指定的属性来执行下一步操作，下一步的操作主要有2种情况：
- 第一次下载一个项目:
  
  在 .repo/manifest.xml 里面找到你想要下载的项目，然后使用这个命令下载整个项目，相当于 git clone。

  git 通常是把代码仓库放在相应的文件夹下面的 .git 目录里面，但是 repo 在管理 git 仓库的时候，为了统一管理，是将所有的 .git 目录都按照相应的目录结构放到了 .repo/projects/ 下面。

- 同步项目到最新:

  在当前项目的目录下面，直接 `repo sync . `就可以同步当前项目。

  repo sync 在这种情况下，相当于执行了的 git pull (git fetch & git merge)，从远端取回最新代码，并将当前 branch 更新到最新。所以你需要主要你当前所在的 branch。

**在任何情况下，在执行 repo sync 的时候，它干的第一件事就是先把 .repo/manifests/ 下面的内容同步到最新，然后再根据最新的 manifest.xml 里面指定的属性来执行下一步操作。**

```
repo upload project-list
```
将本地的代码上传到远程服务器.

upload命令首先会找出本地分支从上一次同步操作以来发生的改动，然后会将这些改动生成Patch文件，上传至Gerrit服务器。 如果没有指定PROJECT_LIST，那么upload会找出所有git库的改动;如果某个git库有多个分支，upload会提供一个交互界面，提示选择其中若干个分支进行上传操作。

```
repo forall PROJECT-LIST -c <COMMAND>
```

管理多个库时使用

例如

```
repo forall frameworks/base packages/apps/Mms -c "git status"
```
表示对platform/frameworks/base和platform/packages/apps/Mms同时执行git status命令。 如果没有指定PROJECT_LIST，那么，会对repo管理的所有git库都同时执行命令。

该命令的还有一些其他参数：

-r, –regex： 通过指定一个正则表达式，只有匹配的PROJECT，才会执行指定的命令

-p：输出结果中，打印PROJECT的名称

```
repo prune [PEOJECT-LIST]
```
删除指定PROJECT中，已经合并的分支。  删除要舍弃的分支

```
repo start <branch-name> [<PROJECT-LIST>]
```
在指定的PROJECT的上，切换到<BRANCH_NAME>指定的分支。可以使用–all参数对所有的PROJECT都执行分支切换操作。 该命令实际上是对git checkout命令的封装，<BRANCH_NAME>是自定义的，它将追踪manifest中指定的分支名。

```
repo status [PROJECT-LIST]
```
status用于查看多个git库的状态。实际上，是对git status命令的封装。

## manifest.xml

一个 repo 所管理的所有项目都记录在这个 .repo/manifest.xml，而这个文件通常是指向 .repo/manifests/default.xml。

- default 标签
```
<default remote="origin" revision="master" />
```
一是指定了使用哪一个远程的 git 仓库，

二是指定了所有它所管理的 git 的默认远程 branch。

- project 标签
```
<project path="aaa/bbb/ccc" name="ddd/eee" revision="release-3" />
```
指定了 repo 所管理的项目的位置和相应的名字。


# gerrit

gerrit是个基于网页的代码review工具，也是基于GIT的一个工具。GIT本身是个分布式的版本控制工具，Gerrit作为一个强大的review工具的同时，也加强了GIT集中化管理代码的能力

