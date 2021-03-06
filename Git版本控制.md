Date:2013-08-29
Tags: Linux

##版本控制系统的演进

1、复制项目目录，备注修改时间（可能会混淆工作目录，一旦丢了文件错了数据就无法恢复）

2、本地版本控制系统，用简单的数据库，来记录文件的历次更新差异，如rcs记录着对应文件修订前后的内容变化。（不同系统上的开发者无法协同工作）

3、集中化的版本控制系统，Centralized Version Control Systems，简称CVCS。诸如CVS,Subversion以及Perforce等，都有一个单一的集中管理的服务器，保存所有文件的修订版本，协同工作人员通过客户端连接服务器来取出新的文件或者提交更新。（服务器有故障时，所有人都无法协同工作提交更新。备份不够及时，磁盘有故障时，有个丢失整个项目的历史记录）

4、分布式版本控制系统，Distributed Version Control Systems，简称DVCS。客户端并不只提取最新版本的文件快照，而是将代码仓库完整的镜像下来。这样，任何一处协同工作用的服务器发生单点故障，事后都可以用任意一个镜像出来的本地仓库恢复。

文件修订后保存的演进：和工作文件一起另外保存 -- 从工作文件中分离出来单独保存在本地 -- 保存在管理服务器 -- 同时保存在管理服务器与本地 

##系统目标

Linux内核维护工作（提交补丁与保存归档）事务繁琐 -- 启用分布式版本控制系统

系统目标：

- 速度；
- 简单的设计；
- 对非线性开发模式的强力支持（允许上千个并行开发的分支）；
- 完全分布式；
- 有能力高效管理类似Linux内核一样的超大规模项目（速度和数据量）。

##git init/clone有无(--bare)的区别

git init，这时候你的工作区也在这里。

bare的意思，是克隆出一个赤裸的仓库。这个仓库没有.git文件，它本身就是GIT_DIR，主要是用作分享版本库。开发者使用bare repository可以向其他人分享版本库，以便于实时分享代码更新和团队协作 。

“工作目录”是通过使用“git init“或“git clone”创建的本地项目拷贝。我们可以在工作目录下面修改和测试代码。通过测试后我们可以使用“git add“和”git commit“命令本地提交修改，然后使用“git push”命令向远程 bare repository库提交更新，通常bare repository指定其他服务器，其他开发者将可以及时看到你的更新。当我们想去更新本地工作目录的时候，我们可以使用“git pull”命令去接受其他开发者提交的更新。

##Git思想及基本工作原理

直接记录快照，而非比较差异：只关心文件数据的整体是否发生变化，而不关心文件内容的具体差异。Git更像是把变化的文件作快照后，记录在一个微型的文件系统中。每次提交更新时，它会总览一遍所有文件的指纹信息并对文件作一快照，然后保存一个指向这次快照的索引。为提高性能，若文件没有变化，Git不会再次保存，而只对上次保存的快照作一链接。

几乎所有操作都是本地执行。

时刻保持数据完整性：在保存到 Git 之前，所有数据都要进行内容的校验和（checksum）计算，并将此结果作为数据的唯一标识和索引。换句话说，不可能在你修改了文件或目录之后，Git 一无所知。

多数操作仅添加数据。

文件的三种状态:对于任何一个文件，在Git内都只有三种状态：已提交（committed），已修改（modified）和已暂存（staged）。已提交表示该文件已经被安全地保存在本地数据库中了；已修改表示修改了某个文件，但还没有提交保存；已暂存表示把已修改的文件放在下次提交时要保存的清单中。

由此我们看到 Git 管理项目时，文件流转的三个工作区域：Git 的工作目录，暂存区域，以及本地仓库。

所谓的暂存区域只不过是个简单的文件，一般都放在 Git 目录中。有时候人们会把这个文件叫做索引文件，不过标准说法还是叫暂存区域。

基本的 Git 工作流程如下：

在工作目录中修改某些文件。
对修改后的文件进行快照，然后保存到暂存区域。
提交更新，将保存在暂存区域的文件快照永久转储到 Git 目录中。

##初次运行Git前的配置

/etc/gitconfig 文件：系统中对所有用户都普遍适用的配置。若使用 git config 时用 --system 选项，读写的就是这个文件。

~/.gitconfig 文件：用户目录下的配置文件只适用于该用户。若使用 git config 时用 --global 选项，读写的就是这个文件。

当前项目的 git 目录中的配置文件（也就是工作目录中的 .git/config 文件）：这里的配置仅仅针对当前项目有效。每一个级别的配置都会覆盖上层的相同配置，所以 .git/config 里的配置会覆盖 /etc/gitconfig 中的同名变量。

##用户信息
配置个人的用户名和邮件地址。每次 Git 提交时都会引用这两条信息，会随更新内容一起被永久纳入历史记录：

    git config --global user.name "John Doe"
    git config --global user.email johndoe@example.com

如果用了 --global 选项，那么更改的配置文件就是位于你用户主目录下的那个，以后你所有的项目都会默认使用这里配置的用户信息。如果要在某个特定的项目中使用其他名字或者电邮，只要去掉 --global 选项重新配置即可，新的设定保存在当前项目的 .git/config 文件里。

##文本编辑器
接下来要设置的是默认使用的文本编辑器。Git 需要你输入一些额外消息的时候，会自动调用一个外部文本编辑器给你用。默认会使用操作系统指定的默认编辑器，一般可能会是 Vi 或者 Vim。如果你有其他偏好，比如 Emacs 的话，可以重新设置：

    git config --global core.editor emacs

##差异分析工具

还有一个比较常用的是，在解决合并冲突时使用哪种差异分析工具。比如要改用 vimdiff 的话：

    git config --global merge.tool vimdiff

Git 可以理解 kdiff3，tkdiff，meld，xxdiff，emerge，vimdiff，gvimdiff，ecmerge，和 opendiff 等合并工具的输出信息。当然，你也可以指定使用自己开发的工具。

##查看配置信息

要检查已有的配置信息，可以使用 git config --list 命令。

也可以直接查阅某个环境变量的设定，只要把特定的名字跟在后面即可，像这样：

    git config user.name

##获取帮助

想了解 Git 的各式工具该怎么用，可以阅读它们的使用帮助，方法有三：

    git help <verb>
    git <verb> --help
    man git-<verb>

比如，要学习 config 命令可以怎么用，运行：

    git help config

##在工作目录中初始化新仓库

"git init"仅仅是按照既有的结构框架初始化好了里边所有的文件和目录，但我们还没有开始跟踪管理项目中的任何一个文件。

如果当前目录下有几个文件想要纳入版本控制，需要先用 git add 命令告诉 Git 开始对这些文件进行跟踪，然后提交。

Git 支持许多数据传输协议。之前的例子使用的是 git:// 协议，不过你也可以用 http(s):// 或者 user@server:/path.git 表示的 SSH 传输协议。

git add 命令是个多功能命令，根据目标文件的状态不同，此命令的效果也不同：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等）。

git commit提交的是git add已暂存的版本，如果文件修改后没有执行git add，是不会被提交的。

##忽略某些文件

    # 此为注释 – 将被 Git 忽略
    # 忽略所有 .a 结尾的文件
    *.a
    # 但 lib.a 除外
    !lib.a
    # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
    /TODO
    # 忽略 build/ 目录下的所有文件
    build/
    # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
    doc/*.txt

##查看已暂存和未暂存的更新

git status 的显示比较简单，仅仅是列出了修改过的文件，如果要查看具体修改了什么地方，可以用 git diff 命令。

git diff 命令比较的是工作目录中当前文件和暂存区域快照之间的差异，也就是修改之后还没有暂存起来的变化内容。

若要看已经暂存起来的文件和上次提交时的快照之间的差异，可以用 git diff --cached 命令。（Git 1.6.1 及更高版本还允许使用 git diff --staged，效果是相同的，但更好记些。)

请注意，单单 git diff 不过是显示还没有暂存起来的改动，而不是这次工作和上次提交之间的差异。所以有时候你一下子暂存了所有更新过的文件后，运行 git diff 后却什么也没有，就是这个原因。

##提交更新

git commit:启动文本编辑器，输入本次提交的说明。默认的提交信息包含最后一次运行git status的输出，放在注释行里。

git commit -v:可以将修改差异的每一行都包含到注释中来。

退出编辑器时，git会丢掉注释行，将说明内容和本次更新提交到仓库。

##跳过使用暂存区域

    git commit -m -a

##移除文件

1、从已跟踪文件清单中（暂存区域）移除，并连带从工作目录中删除指定文件：git rm

2、单独只执行命令rm，提交时仍旧需要执行git rm

3、修改后未git add放入暂存，删除会出现以下提示：
    
    git rm test
    error: 'test' has local modifications
    (use --cached to keep the file, or -f to force removal)

4、修改后已git add放入暂存，删除会出现以下提示：
    
    git rm test
    error: 'test' has changes staged in the index
    (use --cached to keep the file, or -f to force removal)

5、如果删除之前修改过并且已经放到暂存区域的话，必须要用强制删除选项 -f，以防误删除文件后丢失修改的内容。

6、如果想把文件从Git仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。换句话说，仅是从跟踪清单中删除。用--cached选项即可。

    git rm --cached test.txt

7、glob模式，git有自己的文件模式扩展匹配方式，操作时会以递归方式匹配：
    
    #删除所有/log目录下扩展名为.log的文件
    git rm log/\*.log

    #递归删除当前目录及其子目录所有~结尾的文件
    git rm \*~


##用git archive来打包发布软件

1）git archive -o foo.zip — 把当前最新的版本打包成foo.zip
2) git archive -o foo.zip 4d84321a388fa3e55cfee38d61f5fd4e21741bf8 — 把指定版本打包成foo.zip

##git代码库回滚

指的是将代码库某分支退回到以前的某个commit id.

【本地代码库回滚】：

git reset --hard commit-id :回滚到commit-id，讲commit-id之后提交的commit都去除

git reset --hard HEAD~3：将最近3次的提交回滚

git reset --hard：将最近一次提交回滚

 

【远程代码库回滚】：

 应用场景：自动部署系统发布后发现问题，需要回滚到某一个commit，再重新发布。

 原理：先将本地分支退回到某个commit，删除远程分支，再重新push本地分支(master分支不能删除？)。

 操作步骤：

     git checkout the_branch

     git pull

     git branch the_branch_backup //备份一下这个分支当前的情况

     git reset --hard the_commit_id //把the_branch本地回滚到the_commit_id

     git push origin :the_branch //删除远程 the_branch

     git push origin the_branch //用回滚后的本地分支重新建立远程分支

     git push origin :the_branch_backup //如果前面都成功了，删除这个备份分支

区别：git-reset --soft


##忽略已经跟踪的文件

在.gitignore文件中添加该文件；如果该文件之前已经被git跟踪了，这样修改没有用，还需要在git中删除这个文件的跟踪记录，使用命令：
    
    git rm --cached ***
    
这样就从git的跟踪记录中删除了这个文件的跟踪记录。配合之前在.gitignore加的那行配置，以后修改*** 这个文件，git就不会有提示了。  
