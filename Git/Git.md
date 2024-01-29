- [Git 操作流程](#git-操作流程)
  - [git remote](#git-remote)
  - [git config](#git-config)
- [Git 常用命令](#git-常用命令)
  - [本地常用的一些操作](#本地常用的一些操作)
    - [git clone](#git-clone)
    - [git add 暂存状态](#git-add-暂存状态)
    - [git commit  提交状态](#git-commit--提交状态)
    - [git push 推送状态](#git-push-推送状态)
    - [常用流程](#常用流程)
  - [rebase \& merge](#rebase--merge)
    - [rebase](#rebase)
      - [rebase时发生了什么](#rebase时发生了什么)
    - [merge](#merge)
  - [合并 commit 记录](#合并-commit-记录)
    - [reset](#reset)
    - [rebase 整理commit记录](#rebase-整理commit记录)
  - [git cherry-pick](#git-cherry-pick)
  - [git stash 暂存](#git-stash-暂存)
  - [代码撤销](#代码撤销)
    - [已修改，未暂存add](#已修改未暂存add)
    - [已add, 未commit](#已add-未commit)
    - [已commit 未push](#已commit-未push)
    - [已推送](#已推送)
    - [git revert](#git-revert)
- [Git 工作流](#git-工作流)
  - [GitFlow 工作流](#gitflow-工作流)
  - [Git Fork 工作流](#git-fork-工作流)

# Git 操作流程

Git的本质是一个文件系统，其工作目录中的所有文件的历史版本以及提交记录(Commit)都是以文件对象的方式保存在.git目录中的。

*代码提交和同步*

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403121709436.png)

*代码撤销*

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403121709719.png)

## git remote

```bash
git remote -v：显示当前仓库关联的所有远程仓库的详细信息，包括远程仓库的名称和 URL。
git remote add <remote_name> <remote_url>：将一个新的远程仓库添加到本地仓库，指定远程仓库的名称和 URL。
git remote rename <old_name> <new_name>：重命名已存在的远程仓库的名称。
git remote remove <remote_name>：从本地仓库中移除指定的远程仓库。
git remote set-url <remote_name> <new_url>：更改指定远程仓库的 URL。
```

## git config

```bash
$ git config --list

$ git config --global user.name "<name>"
$ git config --global user.email "<email address>"


```

# Git 常用命令

## 本地常用的一些操作

### git clone

将本机的公钥放到gitlab

```bash
git clone ssh://...
```

### git add 暂存状态

```bash
git status
git add --all # 当前项目下的所有更改   git add -a
git add .  # 当前目录下的所有更改
git add xx/xx.py xx/xx2.py  # 添加某几个文件
```

### git commit  提交状态

```bash
git commit -m"<这里写commit的描述>"   #提交的是暂存区的
```

### git push 推送状态

```bash
git push -u origin <branch-name> # 将本地创建的分支推送到远程，第一次需要建立映射，后面直接push即可
git push # 之后再推送就不用指明应该推送的远程分支了
git branch # 可以查看本地仓库的分支
git branch -a # 可以查看本地仓库和本地远程仓库(远程仓库的本地镜像)的所有分支
```

### 常用流程

```bash
git status
git add -a
git status
git commit -m 'xxx'
git checkout 需要合并到的分支，本地没有就fetch
git checkout 需要提交的分支
git rebase 需要合并到的分支
去vscode解决冲突 下面几个步骤

# rebase的时候，修改冲突后的提交不是使用commit命令，而是执行rebase命令指定 --continue选项。若要取消rebase，指定 --abort选项。
$ git add myfile.txt
$ git rebase --continue

git push origin xxbranch    #不建立映射，每次都这样提交

# 合并到master分支
$ git checkout master
Switched to branch 'master'
$ git merge 需要提交的分支
```

## rebase & merge

### rebase

`rebase` 的执行过程是首先找到这两个分支（即当前分支 `Feature`、 `rebase` 操作的目标基底分支 `Master`） 的最近共同祖先提交 A，然后对比当前分支相对于该祖先提交的历次提交（D 和 E），**提取相应的修改并存为临时文件**，然后将当前分支指向**目标基底 ****`Master`**** 所指向的提交 C**, 最后以此作为新的基端将之前另存为临时文件的**修改依序应用。**

理解成将 `Feature` 分支的基础从提交 A 改成了提交 C，看起来就像是从提交 C 创建了该分支，并提交了 D 和 E。但实际上这只是「看起来」，在内部 Git 复制了提交 D 和 E 的内容，创建新的提交 D' 和 E' 并将其应用到特定基础上（A→B→C）。尽管新的 `Feature` 分支和之前看起来是一样的，但它是由全新的提交组成的。

- 有效解决冲突;  想在 `feature` 分支集成 `master` 的最新更改
- 将多路径合为一条路径；完全线性
- 比marge少一次merge提交

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403121721547.png)

#### rebase时发生了什么

- 首先，git 会把 feature 分支里面的每个 commit 取消掉；
- 其次，把上面的操作临时保存成 patch 文件，存在 .git/rebase 目录下；
- 然后，把 feature 分支更新到最新的 master 分支；
- 最后，把上面保存的 patch 文件应用到 feature 分支上

### merge

合并 bugfix分支到master分支时，如果master分支的状态没有被更改过，那么这个合并是非常简单的。 bugfix分支的历史记录包含master分支所有的历史记录，所以通过把master分支的位置移动到bugfix的最新分支上，Git 就会合并。这样的合并被称为fast-forward（快进）合并。

但是，master分支的历史记录有可能在bugfix分支分叉出去后有新的更新。这种情况下，要把master分支的修改内容和bugfix分支的修改内容汇合起来。因此，合并两个修改会生成一个提交。这时，master分支的HEAD会移动到该提交上。

使用 `merge`

1. 切换到 `feature` 分支： `git checkout feature`
2. 合并 `master` 分支的更新： `git merge master`
3. 新增一个提交 F： `git add . && git commit -m "commit F"` 
4. 切回 `master` 分支并执行快进合并： `git chekcout master && git merge feature`

```bash
* 6fa5484 (HEAD -> master, feature) commit F
*   875906b Merge branch 'master' into feature
|\  
| | 5b05585 commit E
| | f5b0fc0 commit D
* * d017dff commit C
* * 9df916f commit B
|/  
* cb932a6 commit A
```

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403121723885.png)

合并过程

在 `master` 合并了 `feature` 分支，现在我们回溯一下合并的过程：

此时 `master` 正指向提交 C，Git 首先找到两个分支最近的唯一共同祖先提交 A，然后分别对 A、C、E 提交的文件快照进行对比，我们下文称呼它们为 A、C、E 文件。接下来 Git 将逐「行」对三个文件的内容进行比较，如果三个文件中有两个文件该行的内容一致，则丢弃 A 文件中该行的内容，保留与 A 文件中不同的内容放到结果文件中。

具体来说，假如 A、C 内容一致，说明这是在 E 中更改的内容，需要保留该更改；A、E 内容一致同理；假如 C、E 内容一致，说明 C 和 E都相对于 A 做了同样的更改，同样需要保留。除此之外的内容差异仅剩两种情况：如果 A、C、E的内容都一致，说明什么都没有发生；**如果该行在 A、C、E的内容都不一致，说明发生了冲突，需要我们手动合并选择需要保留的内容。**

## 合并 commit 记录

### reset

```bash
git log #查看要合并的commit.记住最早的commit号。
git reset commit_number #回退到此commit号。因为没有使用--hard,所以内容都保存在工作区。
git add . #将回退的的内容再次添加到暂存区
git commit -m "comment" #再次提交。
git log #使用此命令查看，发现合并完成。
#这种方式主要利用了git reset回退时并不会删除工作区的内容。因此可以一次性重新提交。
```

### rebase 整理commit记录

```bash
commit 22ad2912ff010751ae35f3963f9b5aa03c8c79c2 (HEAD -> feature002)
Author: xxx <xxx@xxx.com>
Date:   Mon Jan 11 01:00:16 2021 +0800

    fix feature002

commit 5a70aa941faade4571040044bce9234833c3f096
Author: xxx <xxx@xxx.com>
Date:   Mon Jan 11 00:54:28 2021 +0800

    fix feature002

commit 70740239cfcb6bdcda4999f9a00767aed35d0ff1
Author: xxx <xxx@xxx.com>
Date:   Mon Jan 11 00:54:12 2021 +0800

    dev feature002


此时我们想将后面两次bug修复的commit合并为一
git rebase -i 70740239cfcb #以指定的commit为基准，通过不同的指令操作后面的commit

# 此时会进入交互模式，在交互模式中将最后一次commit前面的指令从pick修改为s，保存退出即可(此时会进入到下一个交互中，是用来编写合并后的commit信息的)


commit 055e66d782c5a7e9fcb2131bc2bd12fd017708dc (HEAD -> feature002)
Author: xxx <xxx@xxx.com>
Date:   Mon Jan 11 00:54:28 2021 +0800

    fix feature002

    fix feature002

commit 70740239cfcb6bdcda4999f9a00767aed35d0ff1
Author: xxx <xxx@xxx.com>
Date:   Mon Jan 11 00:54:12 2021 +0800

    dev feature002
```

## git cherry-pick

git cherry-pick命令的作用，就是将指定的提交（commit）应用于其他分支。

```bash
a - b - c - d   Master

  \

    e - f - g Feature

# 切换到 master 分支
$ git checkout master

# Cherry pick 操作
$ git cherry-pick f

a - b - c - d - f   Master

  \

    e - f - g Feature


# 转移多个提交
git cherry-pick <HashA> <HashB>
```

## git stash 暂存

```bash
git stash #工作区和暂存区的都被stash
git stash save "test-cmd-stash" #相比git stash 可以通过此增加记录
git stash list   #查看现有stash
git stash drop stash@{0} #删除指定的stash
git stash show stash@{0} #查看stash的更改内容
git pull #更新分支
git stash apply #可以通过名字指定使用哪个stash，默认使用最近的stash（即stash@{0}）
git stash pop stash@{1} #应用后删除了stash
```

## 代码撤销

### 已修改，未暂存add

```bash
git diff     #列出所有修改
git diff xx/xx.py xx/xx2.py # 列出某(几)个文件的修改

 #git checkout # 撤销项目下所有的修改
$ git checkout . # 撤销当前文件夹下所有的修改
$ git checkout xx/xx.py xx/xx2.py # 撤销某几个文件的修改
$ git clean -f # untracked状态，撤销新增的文件
$ git clean -df # untracked状态，撤销新增的文件和文件夹
```

### 已add, 未commit

```bash
$ git diff --cached # 这个命令显示暂存区和本地仓库的差异

$ git reset # 暂存区的修改恢复到工作区
$ git reset --soft # 与git reset等价，回到已修改状态，修改的内容仍然在工作区中
$ git reset --hard # 回到未修改状态，清空暂存区和工作区
```

### 已commit 未push

```bash
$ git diff <branch-name1> <branch-name2> # 比较2个分支之间的差异
$ git diff master origin/master # 查看本地仓库与本地远程仓库的差异

$ git reset --hard origin/master # 回退与本地远程仓库一致
$ git reset --hard HEAD^ # 回退到本地仓库上一个版本
$ git reset --hard <hash code> # 回退到任意版本
$ git reset --soft / git reset # 回退且回到已修改状态，修改仍保留在工作区中。
```

### 已推送

只能强制覆盖

### git revert

在 Git 中，撤销提交可以使用 git revert 命令。这个命令会创建一个新的提交来撤销之前的提交，而原来的提交仍然存在。这个操作的原理是，Git 会生成一个新的提交，在这个新的提交中，会包含对先前提交所做更改的相反操作，从而达到撤销先前提交的效果。因此，通过 git revert 创建的新提交相当于是对历史进行了修正，而不是直接修改历史。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403121744562.png)


# Git 工作流

## GitFlow 工作流

系统的线上版本即master不能随意修改，Gitflow工作流通过为功能开发develop、发布准备release和维护hotfix设立了独立的分支，每一次修改都会合并到开发分支中，当功能开发完成之后，经过发布准备release之后，并到主版本master中

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403121750145.png)

**由于当前并行分支较多，之前说的关于分支管理的细化，以v2.4.0版本为例：**

1. 开发阶段：

    拉取 develop/v2.4.0 分支，本地开发分支名规则为 featrue/v2.4.0/xxx ；开发期间mr均合入develop分支；

2. 冒烟阶段：

    冒烟阶段一般自提测日前1-2开始，所有需求侧代码均已合并，在联调环境开始执行冒烟用例；

    此时自master拉取release/v2.4.0 分支，develop/v2.4.0 分支合并至release分支；

    自冒烟阶段开始，问题修复分支均以 bugfix/v2.4.0/xxx 分支命名，合并入 release 分支；

    以release分支提测至qa环境；

3. 测试阶段：

    以release分支为主，问题修复均以 bugfix/v2.4.0/xxx 分支命名，合并入 release 分支；

4. 上线阶段：

    已上线的release分支，不再提交任何代码；

    若有hotfix需要，则拉取 release/v2.4.0-hotfix-x 分支作为hotfix主线分支；

5. 版本稳定：

    版本基本稳定后（暂定上线2-3天后），将 release 分支并入 master;

其他：

若测试期间存在延迟交付或者插入需求，拉取feature分支，先合入develop分支，在联调环境验证完毕后再合入release分支，并更新至qa环境正式提测；

## Git Fork 工作流

每个开发者将具体版本比如V1.0 fork到自己的仓库或者拉取本地

建立自己的分支，例如V1.0-wlx

每次修改修改之后提交，都是提交到自己的分支

然后拿那边有权限的将几个人的合并

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403121751884.png)

