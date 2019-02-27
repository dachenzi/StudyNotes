# 1 Gogs安装
Gogs 是一款极易搭建的自助 Git 服务。Gogs 的目标是打造一个最简单、最快速和最轻松的方式搭建自助 Git 服务。使用 Go 语言开发使得 Gogs 能够通过独立的二进制分发，并且支持 Go 语言支持的 所有平台，包括 Linux、Mac OS X、Windows 以及 ARM 平台。
## 1.1 安装准备
> 以下步骤均在CentOS 7.6下测试成功
1. 更新yum源
```bash
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```
2. 安装git
```bash
yum install -y git
```
3. 安装数据库
支持MySQL、Mariadb、PostgreSQL、SQLite等
```bash
yum install -y mariadb-server
```
## 1.2 启动数据库
1. 启动数据库
```bash
systemctl start mariadb-server
```
2. 基础配置
```bash
# 初始化数据库
mysql_secure_installation
```
按照提示删除test库，以及创建数据库的root等


if my firt











# 2 Git介绍
Git是分布式版本控制系统,集中式VS分布式(SVN VS Git),SVN和Git主要的区别在于历史版本维护的位置,SVN和Git主要的区别在于历史版本维护的位置,这样的好处在于：

自己可以在脱机环境查看开发的版本历史。
多人开发时如果充当中央仓库的Git仓库挂了，可以随时创建一个新的中央仓库然后同步就立刻恢复了中央库。
Git命令
Git配置
1
2
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
PS：当然 git config的参数 global 表示当前的主机上所有的git 仓库都会使用这个配置，当然也可以对某个仓库指定不同的用户名和Email地址。

为单独的项目配置git参数，只需要在项目的.git目录下，执行git config 命令(去掉--global参数即可).

1
$ git config user.name 'My name'
PS: 这种方式的用户名Email会存放在.git/config中，当同时存在.gitconfig和config中时，在项目内进行git操作时，config中的配置优先

其他配置：
　　忽略SSL证书 （在ssl证书未经过第三方机构签署）

1
$ git config --global http.sslVerify "false"
PS：实际上，上面这些命令是修改了~/.gitconfig这个文件，我们打开这个文件可以看到先前配置的git相关参数

创建版本库
　　什么是版本库呢，版本库又名仓库(repository)，可以简单的理解为一个目录，这个目录里的文件都会被git管理起来，每个文件的修改，删除，git都可以进行跟踪，以便任何时候可以追踪历史，或者在将来某一个时刻还可以还原。

初始化一个Git仓库
创建一个版本库只需要使用init进行初始化即可。

1
2
3
$ mkdir myfirstrepo
$ cd myfirstrepo
$ git init　
这样就可以把git仓库创建好了，并在该目录下产生.git目录，用于存放git相关的用于跟踪的相关信息，千万不要乱修改，否则可能把git仓库给破坏了。

添加文件到Git仓库
包括两步：

1
2
$ git add <file>
$ git commit -m "description"
git add可以反复多次使用，添加多个文件，git commit可以一次提交很多文件，-m后面输入的是本次提交的说明，可以输入任意内容。

查看工作区状态
1
$ git status
1、当我们在仓库中新建文件，但是没有add时，执行后会有如下提示

1
2
3
4
# Untracked files:
# (use "git add <file>..." to include in what will be committed)
#
#   readme.txt
这时，当我们 使用 add命令添加后

1
2
3
4
# Changes to be committed:
# (use "git rm --cached <file>..." to unstage)
#
#   new file: readme.txt
表示本次新增了新文件readme.txt

2、当我们修改readme.txt文件时，git会告诉我们这个文件被修改了。

1
2
3
4
# Changes to be committed:
# (use "git rm --cached <file>..." to unstage)
#
#   modified: readme.txt
但是不会告诉我们哪里被修改了，这时可以使用diff命令来查看详细的修改内容

1
2
3
4
5
6
7
diff --git a/readme.txt b/readme.txt
index 860d0e4..e757091 100644
--- a/readme.txt
+++ b/readme.txt
@@ -1 +1 @@
-Hello World 2018
+HEllO World    # 第一行由Hello World 2018 变为了 HELLO World
查看修改内容
1
2
3
$ git diff file           # 是工作区(work dict)和暂存区(stage)的比较  （工作区的文件和add后的文件）
$ git diff --cached file  # 是暂存区(stage)和分支(master)的比较  （add后的文件和commit后的文件）
$ git diff HEAD -- file   # 是工作区和分支的比较
查看提交日志
　　当我们不断的对文件进行修改，然后不断的提交到版本库里，就好比玩游戏，每通过一关就会把当前的关卡保存，如果某一关没过去，那么还可以读取前一关的状态，继续开始。Git也一样，每当你修改文件到一定程度，就可以保存一个快照，这个快照在Git中成为commit，一旦把文件改乱，或者误删除，还可以从最近一个commit进行恢复。

　　我们不可能知道所有自己每次提交的时间及内容，那么git提供了log命令用于查询这些提交的信息

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
[root@localhost repo]# git log
commit 7521d7483dd263a6302168744a69ae8e3d7f11be
Author: daxin <daxin.li@foxmail.com>
Date: Sat May 19 17:47:50 2018 +0800
 
test2
 
commit e0b08fa7b70251f14cf5d6472260ba8145f105c1
Author: daxin <daxin.li@foxmail.com>
Date: Sat May 19 17:45:22 2018 +0800
 
test1
 
commit e449c011faf28c40398c59ace8c3017bd7fbd572
Author: daxin <daxin.li@foxmail.com>
Date: Sat May 19 17:43:25 2018 +0800
 
Hello World
简化输出：

1
$ git log --pretty=oneline
PS： log还有一个常用的参数-1，表示显示最后一次提交。

1
2
3
4
5
6
[root@daxin-vpn myfirstrepo]# git log -1
commit f1a9599b26ea4e2c44ba06497474e07c2c62781e
Author: dachenzi <daxin.li@foxmail.com>
Date:   Wed May 23 17:31:42 2018 +0800
 
    ignore files
查看命令历史
1
$ git reflog  
例子：通过历史，可以确定commit id，那么有了commit id 就可以进行回退了

1
2
3
4
5
6
7521d74 HEAD@{0}: reset: moving to 7521d
e449c01 HEAD@{1}: reset: moving to HEAD^
e0b08fa HEAD@{2}: reset: moving to HEAD^
7521d74 HEAD@{3}: commit: test
e0b08fa HEAD@{4}: commit: test
e449c01 HEAD@{5}: commit (initial): Hello World
版本回退
1
$ git reset --hard commit_id
PS：commit_id是版本号，是一个用SHA1计算出的序列，唯一标识某一次的提交，在Git中，还有一些特殊的标签，比如用HEAD表示当前版本，上一个版本就是HEAD^，上上一个版本是HEAD^^，往上100个版本写成HEAD~100。根据log得到的commit id，那么我们就可以自用穿越了。

工作区、暂存区和版本库
工作区：一个git项目的根目录(仓库的主目录，我们之前执行git init的目录)就是一个工作区；
版本库：在工作区有一个隐藏目录.git，是Git的版本库。
Git的版本库中存了很多东西，其中最重要的就是称为stage（或者称为index）的暂存区，还有Git自动创建的master，以及指向master的指针HEAD。

理解

还记得我们提交文件时的两步吗？

git add 添加文件，实际上就是把文件修改添加到暂存区
git commit 提交修改，实际上就是把暂存区的所有内容提交到当前分支
撤销修改
当我们在工作区中修改了某些文件以后，发现修改错误，想要恢复成没有修改之前的样子，那么我们可以使用Git来进行撤销修改

丢弃工作区的修改
1
$ git checkout -- <file>
该命令是指将文件在工作区的修改全部撤销，这里有两种情况：

一种是file自修改后还没有被放到暂存区(没有add)，现在，撤销修改就回到和版本库一模一样的状态；
一种是file已经添加到暂存区后(add以后)，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。
总之，就是让这个文件回到最近一次git commit或git add时的状态。

丢弃暂存区的修改
分两步： 第一步，把暂存区的修改撤销掉(unstage)，重新放回工作区：

1
$ git reset HEAD <file>     # 不加文件名，那么可以回滚所有暂存区的文件
第二步，撤销工作区的修改

1
$ git checkout -- <file>　  # *表示通配符，表示回滚所有修改过后的文件
小结：

当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- <file>。
当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令git reset HEAD <file>，就回到了第一步，第二步按第一步操作。
已经提交了不合适的修改到版本库时，想要撤销本次提交，进行版本回退，前提是没有推送到远程库。
删除文件
一般情况下，我们删除文件 可以 简单的使用 rm filename 删除，但是这个时候Git知道我们删除了文件，因此工作区和版本库就不一致了，再使用 git status 命令时，会立刻告诉你哪些文件被删除了。

1.如果真的要删除文件，那么需要使用 git rm file 来提交删除动作到暂存区，然后使用 git commit 进行提交

1
2
3
4
5
$ rm <files>
$ git rm <files>
$ git commit -m 'message'
 
# git rm <files> 包含了rm的动作，所以不用再执行
2.如果是误删，但是这时版本库中还存在，所以我们可以使用如下命令恢复文件

1
$ git checkout -- <file>
3.如果错误使用了git rm file删除文件，先撤销暂存区修改，重新放回工作区，然后再从版本库写回到工作区

1
2
$ git reset head <file>
$ git checkout -- <file>
PS：所以只要文件存在版本库中，工作区无论是修改还是删除，都可以"一键还原"。

远程仓库
　　远程仓库，就是远程存在一个用于存放我们提交的修改的仓库，Github就是一个公有的对外提供Git仓库托管的服务，需要注意的是，仓库内的东西对外都是可见的，所以机密性的东西需要自行处理。
　　使用Github首先需要进行注册，这里就不在进行说明。注册完毕后可以在Github上创建一个仓库，创建的步骤也不在说明。创建完毕后github会提示你有以下两种途径上传你的代码
1、本地初始化git仓库，添加远程仓库，然后提交

1
2
3
4
5
6
echo "# GitNote" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin git@github.com:dachenzi/GitNote.git   # 添加远程仓库，远程仓库的名称命名为origin
git push -u origin master                                   # 提交代码到远程仓库origin的master分支上
2、只是代码提交

1
2
git remote add origin git@github.com:dachenzi/GitNote.git   # origin是仓库名称，多仓库可以自己命名
git push -u origin master
PS：仓库提供两种方式接入，HTTPS和SSH

SSH方式，我们可以生成密钥用于免密码提交
HTTPS的方式，提交的时候需要输入用户名密码(可以配置git保存密码有效期)，建议使用ssh的方式。
关于使用-u选项：

　　如果当前分支与多个主机存在追踪关系，那么这个时候-u选项会指定一个默认主机，这样后面就可以不加任何参数使用git push。上面命令将本地的master分支推送到origin主机，同时指定origin为默认主机，后面就可以不加任何参数使用git push了。

1
2
3
删除远程仓库：       git remote remove origin(仓库名称)
从远程库克隆：       git clone 仓库地址
查看远程库信息：     git remote -v
分支
　　 当我们需要开发一个新的功能或者修改bug，又不想直接修改master分支的代码，那么就需要新开一个分支，我们在新开的分支内的操作不会影响其他分支的内容，其他的版本控制系统都有分支的概念，Git与它们不同的是，在1秒钟之内就能完成创建、切换和删除分支，无论是1个文件或者是1万个文件。

创建及切换分支
　　前面我们已经知道，每次提交，git都会把他们连成一个时间线，这条时间线就是一个分支，截至到目前，只有1条时间线，在Git里，这个分支叫做主分支，即master分支，HEAD严格来说不是指向指向提交，而是指向master，master才是指向提交的，再用HEAD指向master，就能确定当前分支，以及当前分支的提交点，当我们创建新的分支dev时，Git新建了一个指针dev，指向master相同的提交，再把HEAD指向dev，就表示当前分支在dev上。（Git创建一个分支很快，因为除了增加一个dev指针，改改HEAD指向，工作区和文件都没有任何变化。新建了dev分支后，后面的提交master指针将不会在改变，而dev指针会随着提交，其时间线会依次向后。如果我们在dev分支上的任务完成，就可以把dev合并到master上，Git怎么合并呢？最简单的方法是直接把master指向dev的当前提交，就完成了合并，所以Git合并分支也就很快，就改改指针，工作区内容也不变，就算要删除dev分支也很方便，只需要删除指向dev的指针即可，这样我们就只剩下master一个分支

1
2
3
4
5
6
7
8
9
10
11
# 创建分支
$ git branch <branchname>
 
# 查看分支
$ git branch    # git branch命令会列出所有分支，当前分支前面会标一个*号。
 
# 切换分支
$ git checkout <branchname>
 
# 创建+切换分支
$ git checkout -b <branchname>   × 常用
合并分支及删除分支
1
2
$ git merge <branchname>       # 用于合并指定分支到当前分支。
$ git branch -d <branchname>   # -d 表示delete
在做合并的时候常常会出现，比如在创建dev分支后，又对master分支的文件进行了修改，如果改动的地方相同，那么在合并的时候会有冲突提示，这个时候我们需要手动解决冲突之后才能合并。

1
2
3
4
[root@localhost repo]# git merge dev             # 在master分支上合并dev分支
Auto-merging readme.txt
CONFLICT (content): Merge conflict in readme.txt # 提示我们readme.txt 文件在合并的时候产生冲突
Automatic merge failed; fix conflicts and then commit the result.
必须手动解决分支以后再进行提交(git status 也可以告诉我们冲突的文件)。

Git会在冲突的文件中标注出那些部分冲突，必须手动的修改需要保存的文件内容，解决冲突

1
2
3
4
5
6
7
8
[root@localhost repo]# cat readme.txt
Hello world My name is Dahl.
Come on from China
<<<<<<< HEAD 　　　　# 当前分支名
like master - 1 　　# 内容
=======
like dev - 1 　　　　# 内容
>>>>>>> dev    　　# 合并的分支名
冲突解决完毕后

1
2
$ git add readme.txt
$ git commit -m 'conflict fixed'
使用带参数的git log 也可以看到分支合并的情况

1
$ git log --graph --pretty=oneline --abbrev-commit
最后再删除分支，即可。

所以：当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并。完成用git log --graph命令可以看到分支合并图。

普通模式合并分支
　　通常，合并分支时，如果可能，Git会用Fast Forward模式，但这种模式下，删除分支后，会丢掉分支信息，如果要强制禁用Fast Forward模式，Git就会在merge时生成一个新的commit，这样从分支历史上就可以看出分支信息，即在进行merge的时候使用 --no-ff 来表示禁用 Fast Forward

1
$ git merge --no-ff -m "description" <branchname>
切换工作区
　　在软件开发的过程中，难免会有各种各样的bug，在使用Git作为版本控制的环境内，一般用分支来进行bug修复，修复后，合并分支，然后临时分支删除。
　　当遇到线上紧急bug修复时，那么就需要紧急修复，而此时手头的任务还没有完成，还没办法提交，所以这里Git提供了一个stash功能，可以把你目前的工作现场给存档，等到方便的时候再回来处理。

1
2
3
4
5
6
7
8
9
10
11
# 保存工作现场
$ git stash
 
# 查看工作现场
$ git stash list
 
# 恢复工作现场
$ git stash pop
 
# 丢弃一个没有合并过的分支
$ git branch -D <branchname>
抓取分支
　　多人协作时，大家都会往master和dev分支上推送各自的修改。当我们从远程仓库clone时，默认情况下只能看到本地的master分支，如果想要从dev分支开始开发，那么需要在本地创建dev分支，然后对应到远程仓库的dev分支上

1
$ git checkout -b dev origin/dev
PS：本地和远程分支的名称最好一致

　　因为大家都是从dev分支上开发新功能的，后来的人在推送本地代码的时候，由于远程仓库中的代码已经改变，所以会提示冲突

1
2
3
4
5
6
7
8
9
[root@localhost repo]# git push origin dev
To git@github.com:dachenzi/GitNote.git
! [rejected] dev -> dev (fetch first)
error: failed to push some refs to 'git@github.com:dachenzi/GitNote.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first merge the remote changes (e.g.,
hint: 'git pull') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
　　出现这种情况时，很简单，Git 也提示我们了，需要先执行git pull 把最新的提交从origin/dev抓下来，然后在本地合并解决冲突后，再推送

1
2
3
4
5
6
7
8
9
10
[root@localhost repo]# git pull    # 包含：从远程仓库中拉取最新代码，并和本地进行合并，两个动作。
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details
 
    git pull <remote> <branch>
 
If you wish to set tracking information for this branch you can do so with:
 
    git branch --set-upstream-to=origin/<branch> dev
　　出现这种情况，表示没有指定本地dev分支与远程origin/dev分支的链接，根据提示，设置dev和origin/dev的链接：

1
2
[root@localhost repo]# git branch --set-upstream-to=origin/dev dev
Branch dev set up to track remote branch dev from origin.
PS: 由于pull会进行代码合并，所以如果冲突，那么还需要解决冲突后提交(请参照上面解决冲突部分)，然后再推送到远程仓库即可。

标签
　　相当于IP和域名的映射，Commit id 是一串很长的数字，不是那么容易记忆，这时如果能在提交时，对当前版本打上一个tag:v1.0，那么以后我就只需要说到V.10就好了。

新建一个标签
1
$ git tag <tagname>  <commit-id>    # 不加commit-d时，默认为HEAD
PS：如果不加commit-id，默认为HEAD，那么就会为当前分支的HEAD打上标签

1
2
3
4
5
6
7
8
9
10
11
12
# 针对某个commit id 打标签
# 1、查看历史提交信息
$ git log --pretty=oneline --abbrev-commit
 
# 2、针对commit id打标签
$ git tag v1.1 f52c633
 
# 3、查看标签对应的信息
$ git show v1.0
 
# 4、创建带有说明的标签， -a 表示标签名， -m 表示说明性的文字
$ git tag -a 'v1.1' -m 'Version 1.0' f52c633
查看及推送标签
1
2
# 查看标签 
$ git tag
PS：不是按照时间来排序的，想要看详细的内容，可以使用git show

1
$ git show v1.0
推送本地标签
　　默认状态下标签是不会进行推送的，如果想要推送某个标签到远程仓库

1
$ git push origin <tagname>
　　推送所有标签到远程仓库

1
$ git push origin --tags
删除标签
删除本地标签

1
$ git push origin --tags
删除一个远程标签

1
$ git push origin :refs/tags/<tagname>
自定义Git　
　　Git除了上面所说的那些配置项以外还有很多可配置的，比如让Git显示颜色

1
git config --global color.ui true
　　这样配置以后文件名就会被标上颜色

忽略特殊文件名
　　有些时候某些文件必须存放在工作区内，举个例子，比如java编译后的class文件，python的pyc文件，这些文件在上传远程服务器时都是不需要的，那么如何忽略某些文件呢？

　　在Git工作区的根目录下创建一个特殊的.gitignore文件，然后把要忽略的文件名填进去，Git就会自动忽略这些文件。

1
2
3
4
[root@daxin-vpn myfirstrepo]# cat .gitignore
# Linux:
readme.txt2
[root@daxin-vpn myfirstrepo]#
　　提交的时候就不会检查readme.txt2文件了，如果我们要强制提交readme.txt2，需要使用-f参数（强制添加）

1
$ git add -f readme.txt2
　　或者你想确认下，自己的文件是否被.gitignore过滤掉

1
git check-ignore -v readme.txt2
　　附：python的gitignore文件

 Python的gitignore文件
更多的过滤列表可以参考：https://github.com/github/gitignore

配置别名
　　向checkout，status这种常用的命令，一来单词比较长，二来还容易打错，所以如果用co表示checkout，用st表示status就好了，Git给我们提供了命令别名的方式。

1
2
$ git config --global alias.st status
$ git config --global alias.co checkout
　　而对于回滚文件的 git reset HEAD file可以有如下简写

1
2
3
$ git config --global alias.unchange "reset HEAD"
$ git unchange readme.txt
# 实际上执行的就是 git reset HEAD readme.txt
　　还有人把lg配置成

1
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
PS：自己可以尝试一下，真的是好看的一比啊，哈哈。

　　由于config是保存在git的用户相关配置文件中的，所以如果想要删除某些alias，可以进入在.gitconfig或者.git/config下删除alias部分的设定，具体在那个文件，取决与你是否使用了--global等参数。