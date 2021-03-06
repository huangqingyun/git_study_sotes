# Git总结

## Git基本配置

### 配置 user 信息（Git使用的最小配置）

- 配置 user.name

   ```shell
   git config --global user.name 'your_name'
   ```

- 配置 user.email

   ```shell
   git config --global user.email 'your_email@domain.com'
   ```

### config 的三个作用域

- 只对配置的当前仓库有效

   ```shell
   git config --local <配置项> <配置值>
   git config --local --add <配置项> <配置值>
   ```

- 对当前登录用户的所有仓库有效

   ```shell
   git config --global <配置项> <配置值>
    git config --global --add <配置项> <配置值>
   ```

- 系统所有登录的用户有效

   ```shell
   git config --system <配置项> <配置值>
   git config --system --add <配置项> <配置值>
   ```

### 显示配置信息，加 --list

```shell
git config --local --list
git config --global --list
git config --system --list
```

## Git 本地仓库版本管理

### 创建 Git 本地仓库

1. 把已有的项目纳入Git管理

   ```shell
   cd 项目所在的文件夹
   git init
   ```

2. 新建的项目,直接用Git管理

   ```shell
   cd 任意文件夹
   git init your_prject # 会在当前目录下创建和项目名称同名的文件夹
   cd your_prject
   ```

3. 创建第一个仓库，并配置local信息

   ```shell
   cd 任意文件夹
   git init your_prject # 会在当前目录下创建和项目名称同名的文件夹
   cd your_prject
   git config --local user.name 'huang'
   git config --local user.email 'qyhuang@jiliason.com'
   git config --local --list
   ```

### Git 原理

#### Git 工作区域

Git本地有三个工作区域：工作目录（Working Directory）、暂存区(Stage/Index)、资源库(Repository)。

- Workspace：工作区，就是你平时存放项目代码的地方。
- Index/Stage：暂存区，用于临时存放你的改动。一般存放在 ".git目录下" 下的index文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）。
- Repository：版本仓库区，就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中HEAD指向最新放入仓库的版本。

![git结构图](./image/Git工作结构示意图.png)

- Directory：使用Git管理的一个目录，也就是一个仓库，包含我们的工作空间和Git的管理空间。
- .git：存放Git管理信息的目录，初始化仓库的时候自动创建。
- Stash：隐藏，是一个工作状态保存栈，用于保存/恢复WorkSpace中的临时状态。

#### Git 工作流程

##### 三棵树

Git 是将管理三棵不同**树**的思维框架作为内容管理器。**树**在这里的实际意思是 “文件的集合”，而不是指特定的数据结构。 （在某些情况下**index/stage**看起来并不像一棵树，不过我们现在的目的是用简单的方式思考它。）

Git 作为一个系统，是以它的一般操作来管理并操纵这三棵**树**的：
| 树                | 用途                                 |
| ----------------- | ------------------------------------ |
| HEAD              | 上一次提交的快照，下一次提交的父结点 |
| Index             | 预期的下一次提交的快照               |
| Working Directory | 沙盒                                 |

- `HEAD`: 是当前分支引用的指针，它总是指向该分支上的最后一次提交。 这表示 `HEAD` 将是下一次提交的父结点。 通常，理解 `HEAD` 的最简方式，就是将它看做 该分支上的最后一次commit 的快照。
- `Index`： 是你的 预期的下一次提交。 我们也会将这个概念引用为 Git 的**暂存区**，这就是当你运行 `git commit` 时 Git 看起来的样子。Git 将上一次检出到**工作目录**中的所有文件填充到**暂存区**，它们看起来就像最初被检出时的样子。 之后你会将其中一些文件替换为新版本，接着通过 `git commit` 将它们转换为**树**来用作新的提交。
- `Working Directory`:工作目录（通常也叫 工作区）。 另外两棵**树**以一种高效但并不直观的方式，将它们的内容存储在 `.git` 文件夹中。 工作目录会将它们解包为实际的文件以便编辑。 你可以把工作目录当做**沙盒**。在你将修改提交到**暂存区**并记录到历史之前，可以随意更改。
  
经典的 Git 工作流程是通过操纵这三个区域来以更加连续的状态记录项目快照的。
![工作流程](image/workflow.png)

##### 可视化工作流程

1. 进入到一个新目录，其中有一个文件。 我们称其为该文件的 v1 版本，将它标记为蓝色。 现在运行 `git init`，这会创建一个 Git 仓库，其中的 `HEAD` 引用指向未创建的 `master` 分支。
   ![init](image/git_init.png)

2. 要提交这个文件，所以用 `git add` 来获取工作目录中的内容，并将其复制到索引中。
   ![add](image/git_add.png)

3. 接着运行 `git commit`，它会取得索引中的内容并将它保存为一个永久的快照， 然后创建一个指向该快照的提交对象，最后更新 master 来指向本次提交。
   ![commit](image/commit.png)
   此时如果我们运行 `git status`，会发现没有任何改动，因为现在三棵树完全相同。

4. 现在我们想要对文件进行修改然后提交它。 我们将会经历同样的过程；首先在工作目录中修改文件。 我们称其为该文件的 v2 版本，并将它标记为红色。
   ![edit file](image/edit_file.png)
   如果现在运行 git status，我们会看到文件显示在 “Changes not staged for commit” 下面并被标记为红色，因为该条目在索引与工作目录之间存在不同。

5. 接着我们运行 git add 来将它暂存到索引中。
   ![add edit](image/add_edit.png)
   此时，由于索引和 `HEAD` 不同，若运行 `git status` 的话就会看到 “Changes to be committed” 下的该文件变为绿色 ——也就是说，现在预期的下一次提交与上一次提交不同。

6. 最后，我们运行 `git commit` 来完成提交。
   ![commit edit](image/commit_edit.png)
   现在运行 `git status` 会没有输出，因为三棵树又变得相同了。

`git checkout [branch]`或`git clone` 的过程也类似。 当`git checkout branch`一个分支时，它会修改 `HEAD` 指向新的分支引用，将**索引** 填充为该次提交的快照， 然后将 **索引** 的内容复制到**工作目录**中。

##### Git 工作流程中的文件状态

- **Untracked**:未跟踪, 此文件在文件夹中, 但并没有加入到git库, 不参与版本控制. 通过git add 状态变为Staged.
- **Unmodify**:文件已经入库, 未修改, 即版本库中的文件快照内容与文件夹中完全一致. 这种类型的文件有两种去处, 如果它被修改, 而变为Modified.如果使用git rm移出版本库, 则成为Untracked文件
- **Modified**: 文件已修改, 仅仅是修改, 并没有进行其他的操作. 这个文件也有两个去处, 通过git add可进入暂存staged状态, 使用git checkout 则丢弃修改过,返回到unmodify状态, 这个git checkout即从库中取出文件, 覆盖当前修改。
- **Staged**: 暂存状态. 执行git commit则将修改同步到库中, 这时库中的文件和本地文件又变为一致, 文件为Unmodify状态. 执行git reset HEAD filename取消暂存,文件状态为Modified。

![文件状态](image/git_file_status.png)

![文件状态转换](image/git_chanage_file_status.png)

#### 通操作认识Git工作区、工作方式以及文件状态变化

```shell
git status # 查看工作区和暂存区的文件状态
cp <path> <file name> #拷贝文件到本地或GUI方式新建文件
git status #查看上一步创建或拷贝的文件状态。
git add <file name> #将文件添加到暂存区或者使用 git add -u 将工作区的所有跟踪的文件提交到暂存区。
git status
git commit -m 'message' #将暂存区的所有文件的更改提交到版本库，或者使用git commit -m 'message' [file name...]指定一个或多个文件提交
git status

```

### 文件重命名

1. 手动更改

   ```shell
   mv <old file name> <new file name> #或者使用GUI方式修改
   git add <new file name> # 将修改后的文件添加到暂存区
   git rm <old file name> #将暂存区的旧文件删除。
   ```

2. Git方式

   ```shell
   git mv <old file name> <new file name> #一步到位
   ```

### 删除文件

#### 删除未提交文件

1. 手动更改

   ```shell
   rm <file name>
   git rm <file name> #将暂存区的旧文件删除。
   ```

2. Git方式

 ```shell
 git rm <file name>
 git rm log/\*.log # 该命令删除log目录下的所有以 `.log` 结尾的文件
 git rm \*~ # 删除所有以`~`结尾的文件
 git rm --cached <file name> # 从暂存区删除文件。
                            # 比如：当你忘记添加 .gitignore 文件，不小心把一个很大
                            # 的日志文件或一堆 .a 这样的编译生成文件添加到暂存区时，
                            # 这一做法尤其有用。
```

#### 删除历史提交文件(移除对象)

`git clone` 的特性，会下载整个项目的历史，包括每一个文件的每一个版本。 如果所有的东西都是源代码那么这很好，因为 Git 被高度优化来有效地存储这种数据。 然而，如果某个人在之前向项目添加了一个大小特别大的文件，即使你将这个文件从项目中移除了，每次克隆还是都要强制的下载这个大文件。 之所以会产生这个问题，是因为这个文件在历史中是存在的，它会永远在那里。

为解决上面的问题，你必须修改或移除一个大文件引用最早的树对象开始重写每一次提交。 如果你在导入仓库（操作需要 `push -f`）后，在任何人开始基于这些提交工作前执行这个操作，那么将不会有任何问题——否则， 你必须通知所有的贡献者他们需要将他们的成果变基到你的新提交上。

>**警告：这个操作对提交历史的修改是破坏性的。**

```shell
# 查看文件大小
git gc # 查看数据库占用空间
git count-objects -v # 快速查看空间占用信息
git verify-pack -v .git/objects/pack/pack-*.idx   | sort -k 3 -n   | tail -3 # 查看文件或哪些文件占用了多少的空间，git verify-pack -v显示的为SHA-1值、对象类型、对象大小、打包大小、包文件的偏移位置。

# 查找该文件相关的对象
git rev-list --objects --all | grep <目标对象的SHA-1值> # 列出所有提交的 SHA-1、数据对象的 SHA-1 和与它们相关联的文件路径。

# 查看哪些提交对改文件产生改动
git log --oneline --branches -- <file name> # 查看哪些提交对这个文件产生改动

# 重写历史提交信息
git filter-branch index-filter 'git rm --ignore-unmatch --cached <filename>' --<SHA-1值>^.. # 重写 SHA-1值 提交之后的所有提交来从 Git 历史中完全移除这个文件。--index-filter 选项：只是修改在暂存区或索引中的文件。git rm --cached 命令来移除文件；--ignore-unmatch 选项告诉命令：如果尝试删除的模式不存在时，不提示错误。使用 filter-branch 选项来重写自 <SHA-1值> 提交以来的历史，也就是这个问题产生的地方开始。

# 移除 引用日志和你在 .git/refs/original 通过 filter-branch 选项添加的新引用中还存有对这个文件的引用。
rm -Rf .git/refs/original
rm -Rf .git/logs/ 

#文件还在你的松散对象中，并没有消失；但是它不会在推送或接下来的克隆中出现，这才是最重要的。想要删除它，可以通过有 --expire 选项的 git prune 命令来完全地移除那个对象：
git prune --expire now
```

### 查看commit版本的演变历史

#### 命令行工具查看

多屏显示控制方式：

- 空格：向下翻页
- b: 向上翻页
- q: 退出

```shell
git log # 查看当前分支，所在版本的演变历史
git log --oneline #简洁模式查看
git log -n2 #查看最近的2次版本演变历史
git log -n4 --oneline #简洁模式查看最近的4次版本演变历史
git branch -v # 查看分支。或者使用 git branch -av查看所有分支
git branch -vv # 查看所有远程和本地分支，及远程跟踪分支详情。
git log --all #查看所有分支的当前所在版本演变历史。
git log --all --graph # 图形化方式的查看所有分支的版本演变历史。
git show [sha1值|分支名] # 查看commit 详细。
git reflog # 查看当前分支，所有版本演变历史。HEAD@{移动到当前版本需要多少步}
# git log --前缀的参数可以组合使用
git --help --web <command name> #使用网页方式查看命令详细帮助
```

#### 图形界面工具查看

```shell
gitk #打开图形界面
# Git GUI
```

### 回滚和撤销

#### `reset`命令

`reset`以一种简单可预见的方式直接操纵**三棵树**，实现重置的功能（包括版本回滚和撤销commit，以及文件）。 它做了三步基本操作:

1. `reset`移动`HEAD`的**指向**。
2. 更新索引（--mixed）
3. 更新工作目录（--hard）

#### 回滚和撤销commit

##### 演示可视化`reset`三步基本操作

假设我们修改 file.txt 文件并三次提交它。 历史看起来是这样的：
![reset start](image/reset-start.png)

###### 第 1 步：移动 HEAD（--soft）

`reset` 做的第一件事是移动`HEAD`的**指向**,或者可以理解为同时移动 HEAD 和它所指向的 branch。如下示意图
![git reset](./image/reset.webp)
这与改变`HEAD`自身不同（`checkout`所做的，即改变自身）；`reset`移动`HEAD`指向的**分支**。 这意味着如果 `HEAD` 设置为 `master` 分支（例如，你正在`master`分支上），运行`git reset 9e5e6a4`将会使`master`指向`9e5e6a4`。
![reset soft](image/reset-soft.png)
无论你调用了何种形式的带有一个提交的 reset，它首先都会尝试这样做。 如果指定 `--soft` 选项，它将仅仅完成这步。

现在看一眼上图，理解一下发生的事情：它本质上是**撤销**了上一次 `git commit` 命令。 当你在运行 `git commit` 时，Git 会创建一个新的提交，并移动 **`HEAD` 所指向的分支**来使其指向该提交。 当你将它 `reset` 回 `HEAD~`（`HEAD` 的父结点）时，其实就是把该分支移动回原来的位置，而不会改变索引和工作目录。

###### 第 2 步：更新索引（--mixed）

注意，如果你现在运行 `git status` 的话，就会看到新的 `HEAD` 和以绿色标出的它和索引之间的区别。

接下来，`reset` 会用 `HEAD` 指向的当前快照的内容来**更新索引**。
![reset mixed](image/reset-mixed.png)
如果指定 `--mixed` 选项，`reset` 将会在这步完成时停止。 这也是**默认行为**，所以如果没有指定任何选项（在本例中只是 git reset HEAD~），这就是命令将会停止的地方。

现在再看一眼上图，理解一下发生的事情：它依然会撤销一上次 提交，但还会 取消暂存 所有的东西。 于是，我们回滚到了所有 `git add` 和 `git commit` 的命令执行之前。

###### 第 3 步：更新工作目录（--hard）

`reset` 要做的的第三件事情就是让**工作目录**看起来像**索引**。 如果使用 `--hard` 选项，它将会继续这一步。
![reset hard](image/reset-hard.png)
现在让我们回想一下刚才发生的事情。 你撤销了最后的提交、git add 和 git commit 命令 以及 工作目录中的所有工作。

**必须注意**：

`--hard` 标记是`reset` 命令唯一的**危险用法**，它也是 Git 会真正地销毁数据的仅有的几个操作之一。 其他任何形式的 reset 调用都可以轻松撤消，但是 `--hard` 选项不能，因为它**强制覆盖了工作目录中的文件**。 在这种特殊情况下，我们的 Git 数据库中的一个提交内还留有该文件的 v3 版本， 我们可以通过 `reflog` 来找回它。但是若该文件还未提交，Git 仍会覆盖它从而导致无法恢复。

**销毁的数据**：

- `--hard` 标记是`reset`销毁的数据是所有未提交的数据（包括未添加到暂存区的文件修改和暂存区的所有数据）。
- reset（包括三个选项）后，已提交的历史数据（**处于detached HEAD头指针分离状态**）可通过`reflog` 来找回它，但是`reflog`的有效期默认为90天。过期或者手动删除`reflog`将找不回已提交的历史数据。详情可查看`git reflog`命令。

###### `reset`命令三个选项区别

- `--soft`：只移动`HEAD`的**指向**。
  
  所以效果看起来就缓存区和工作目录都不会被改变，只是原节点和Reset节点之间的所有**差异都会放到暂存区**中。
  >**常用操作**：
  >
  >1. 合并提交：开发一个功能，阶段性地频繁提交。可以考虑使用`reset --soft`来合并之前的`commit`，让 commit演进线路更为清晰。

- --mixed（默认）：移动`HEAD`的**指向**同时，会用 `HEAD` 指向的当前快照的内容来**更新索引**。所以效果看起来就是缓存区和你指定的提交同步，但工作目录不受影响。原节点和Reset节点之间的所有**差异都会放到工作目录**中。
  
  >**常用操作**：
  >
  >1. 清空暂存区：有时添加了不想要的或错误的文件和修改内容到暂存区，这时可使用`git reset Head`清空暂存区。
  >2. 重置（撤销）本次commit：commit后，知道这次commit了错误或不想commit的数据。这时可使用`git reset Head~`或`git reset Head^`

- --hard：
  
  移动`HEAD`的**指向**，会用 `HEAD` 指向的当前快照的内容来**更新索引**,再更新工作目录，让**工作目录**看起来像**索引**。所以效果看起来缓存区和工作目录都**同步到你指定的提交**。

  >**常用操作**：
  >
  >1. 放弃本次所有未提交的本地编辑内容:
  可以执行 `git reset -hard HEAD` 来强制清理掉当前的所有更改。
  >2. 回滚版本和撤销某节点后的所有commit:有时某节点到原节点之间的commit提交都是错了，或者是某一版本之后出现未知Bug,导致无法使用。
  >
  >```shell
  >   git reset --hard <标识commit的哈希值> # 回滚到指定commit
  >   git reset --hard HEAD^ # 注：一个^表示回滚一个commit，n 个表示回滚n个commit
  >   git reset --hard HEAD~n # 注：表示回滚n个commit
  >   ```

#### 回滚和撤销文件

上一章节讲述了 `reset` 基本形式的行为，我们还可以给它提供一个作用文件路径。 若指定了一个路径，`reset` 将会跳过第 1 步，并且将它的**作用范围限定**为指定的文件或文件集合。这样做是因为 `HEAD` 只是一个指针，你无法让它同时指向两个commit中各自的一部分。 不过**索引**和**工作目录**可以部分更新，所以会继续进行第 2、3 步。

假如Git的状态为：
![path](image/reset_path_init_status.png)
我们运行 `git reset file.txt` （这其实是 `git reset --mixed HEAD file.txt` 的简写形式，因为你既没有指定一个提交的 `SHA-1` 或`branch`，也没有指定 --soft 或 --hard），它会：

1. 移动 HEAD 分支的指向 （已跳过）
2. 让**索引/暂存区**看起来像 HEAD （到此处停止）

所以它本质上只是将 file.txt 从 HEAD 复制到索引中。
![回滚文件](image/reset-path1.png)

它还有 **取消暂存文件** 的实际效果。 如果我们查看该命令的示意图，然后再想想 git add 所做的事，就会发现它们正好相反。

![回滚文件](image/reset-path2.png)

这就是为什么 git status 命令的输出会建议运行此命令来取消暂存一个文件。

我们可以不让Git从 `HEAD` 拉取数据，而是通过具体指定一个`commit`来拉取该文件的对应版本。 我们只需运行类似于 `git reset [commit 哈希值] [文件相对路径]` 的命令即可。

![拉取文件的指定版本](image/reset-path3.png)

它其实做了同样的事情，也就是把**索引/暂存区**中的文件恢复到 v1 版本，运行 `git add` 添加它， 然后再将它恢复到 v3 版本（只是不用真的过一遍这些步骤）。 如果我们现在运行 `git commit`，它就会记录一条“将该文件恢复到 v1 版本”的更改， 尽管我们并未在工作目录中真正地再次拥有它。

### 分离头指针（detached HEAD）

本质：正工作在一个没有分支的状态下。

风险：分离头指针状态下的所有提交，如果最后没有和分支进行挂钩，在切换到其他分支后，可通过`reflog` 来找回它，默认有效期90天。之后会被Git当垃圾清理掉。

好处：可以基于某个commit进行修改，提交。

### Git 存储机制

#### Git 对象

1. `blob` 对象（保存着文件快照）
2. `tree` 对象 （记录着目录结构和 blob 对象索引）以及一个
3. `commit` 对象（包含着指向前述tree对象的指针和所有commit信息）

#### Git 对象之间的关系

1. 一个commit对象对应一个tree对象
2. tree可以嵌套，文件夹在Git 里也是tree.
3. 一个tree可保护多个blob,嵌套多个tree.
4. commit对象及其树结构
   ![提交对象及其树结构1](image/commit-and-tree.png)
   ![提交对象及其树结构2](image/commit-tree-blob.png)

5. commit对象及其父对象
   ![提交对象及其父对象](image/commits-and-parents.png)

>Note:Git 是根据文件内容产生blob对象，因此文件内容相同那么对象就相同。

### 分支（branch)

#### 简介

分支意味着你可以把你的工作从开发主线上分离开来，以免影响开发主线。
>分支在实际使用中可以理解为独立的工作空间。

Git 的分支，其实本质上仅仅是指向`commit对象`的**可变指针**。 Git 的默认分支名字是 master。 在多次提交操作之后，你其实已经有一个指向最后那个提交对象的 master 分支。 master 分支会在每次提交时自动向前移动。
>Git 的 master 分支并不是一个特殊分支。 它就跟其它分支完全没有区别。 之所以几乎每一个仓库都有 master 分支，是因为 git init 命令默认创建它，并且大多数人都懒得去改动它。

#### 分支管理

##### 分支创建

Git 创建新分支，只是为你创建了一个可以移动的新的指针。 比如，创建一个 testing 分支， 你需要使用 git branch 命令：

```shell
git branch testing
```

这会在当前所在的commit对象上创建一个指针。

![两个指向相同提交历史的分支](image/two-branches.png)

在 Git 中，`HEAD`是一个特殊指针，它指向当前所在的本地分支（译注：将 `HEAD` 想象为当前分支的别名）。 在本例中，你仍然在 `master` 分支上。 因为 `git branch` 命令仅仅创建 一个新分支，并**不会自动切换**到新分支中去。

![HEAD 指向当前所在的分支](image/head-to-master.png)

使用 git log 命令查看各个分支当前所指的对象。 提供这一功能的参数是 --decorate。

```shell
$ git log --oneline --decorate
f30ab (HEAD -> master, testing) add feature #32 - ability to add new formats to the central interface
34ac2 Fixed bug #1328 - stack overflow under certain conditions
98ca9 The initial commit of my project
```

正如你所见，当前 master 和 testing 分支均指向校验和以 f30ab 开头的提交对象。

**创建分支的命令总结**：

```shell
git branch [new branch name] #在当前所在的commit对象上创建一个分支
git checkout -b [new branch name]#在当前所在的commit对象上创建一个分支，并切换到创建的新分支
git branch [new branch name] [commit]# 创建基于commit上的分支
git checkout -b [new branch name] [branch|commit] # 创建基于分支或提交的分支，并切换到创建的新分支
```

##### 分支查看

```shell
git branch # 不加任何参数运行它，会得到当前所有分支的一个列表
git branch -v # 查看每一个分支的最后一次提交
git branch --merged # --merged 与 --no-merged 这两个有用的选项可以过滤这个列表中已经合并或尚未合并到当前分支的分支。
git branch -a # 列出远程跟踪分支和本地分支。
```

注意 master 分支前的 * 字符：它代表现在checkout的那一个分支（也就是说，当前 HEAD 指针所指向的分支）。

##### 分支重名名

```shell
git branch (-m | -M) [oldbranch] <newbranch> # -M是强制移动或重命名。-m 移动分支或重命名
```

##### 分支切换

###### `checkout`命令

`checkout` ：和 `reset` 一样，也操纵三棵树。实现切换的功能， 它做了三步基本操作:

1. `checkout`只移动`HEAD`
2. 更新索引
3. 更新工作目录

和`reset`命令的区别：**变更暂存区使用reset命令，变更工作区使用checkout命令。**

1. `reset` 会移动 `HEAD` 分支的**指向**，而 `checkout` 只会移动 `HEAD` **自身**来指向另一个分支。
2. `checkout` 对工作目录是安全的，它会通过检查来确保不会将已更改的文件弄丢。 其实它还更聪明一些。它会在工作目录中先试着简单合并一下，这样所有**还未修改过的**文件都会被更新。 而 `reset --hard` 则会不做检查就全面地替换所有东西。
![reset and checkout](image/reset-checkout.png)

>运行 `checkout` 的另一种方式就是指定一个文件路径，这会像 `reset` 一样不会移动 `HEAD`。 它就像 `git reset [branch] file` 那样用该次提交中的那个文件来更新索引，但是它也会覆盖工作目录中对应的文件。 它就像是 `git reset --hard [branch] file`（如果 `reset` 允许你这样运行的话）， 这样对工作目录并不安全，它也不会移动 `HEAD`。

要切换到一个已存在的分支，你需要使用 git checkout 命令。我们现在切换到新创建的 testing 分支去：

```shell
git checkout testing
```

这样 HEAD 就指向 testing 分支了。
![HEAD 指向当前所在的分支](iamge/../image/head-to-testing.png)

那么，这样的实现方式会给我们带来什么好处呢？ 现在不妨再提交一次：

```shell
vim test.rb
git commit -a -m 'made a change' # 同git commit -am 'made a change' 一样
```

![HEAD 分支随着提交操作自动向前移动](image/advance-testing.png)

如图所示，你的 `testing` 分支向前移动了，但是 `master` 分支却没有，它仍然指向运行 `git checkout` 时所指的对象。 这就有意思了，现在我们切换回 `master` 分支看看：

```shell
git checkout master
```

![checkout时 HEAD 随之移动](image/checkout-master.png)

这条命令做了两件事。 一是使 `HEAD` 指回 `master` 分支，二是将工作目录恢复成 `master` 分支所指向的快照内容。 也就是说，你现在做修改的话，项目将始于一个较旧的版本。 本质上来讲，这就是忽略`testing`分支所做的修改，以便于向另一个方向进行开发。

>**分支切换会改变你工作目录中的文件**
>
>在切换分支时，一定要注意你工作目录里的文件会被改变。 如果是切换到一个较旧的分支，你的工作目录会恢复到该分支最后一次提交时的样子。 如果`Git`不能干净利落地完成这个任务，它将禁止切换分支。

##### 整合分支

**Git 中整合来自不同分支的修改主要有两种方法：merge 以及 rebase。**

###### 合并分支

`git merge`命令合并分支。
例如：合并 iss53 分支到 master 分支

```shell
git checkout master # 切换到master
git merge iss53 # 将iss53合并入master分支。
```

![典型合并中所用到的三个快照](image/basic-merging-1.png)

![合并提交](image/basic-merging-2.png)

修改已经合并进来了，就不再需要 iss53 分支了。 现在你可以在任务追踪系统中关闭此项任务，并删除这个分支。

####### 遇到冲突时的分支合并

两个不同的分支中，对同一个文件的同一个部分进行了不同的修改，Git 就没法处理，哪些是需要保留的。Git 会暂停下来，等待你去解决合并产生的冲突。

任何因包含合并冲突而有待解决的文件，都会以**未合并状态标识**出来。 Git 会在有冲突的文件中加入标准的冲突解决标记，这样你可以打开这些包含冲突的文件然后手动解决冲突。 出现冲突的文件会包含一些特殊区段，看起来像下面这个样子：

```shell

<<<<<<< HEAD:index.html # (当前更改)
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
 please contact us at support@github.com
</div>
>>>>>>> iss53:index.html (传入的更改)
```

这表示 HEAD 所指示的版本（也就是你的 master 分支所在的位置，因为你在运行 merge 命令的时候已经检出到了这个分支）在这个区段的上半部分（======= 的上半部分），而 iss53 分支所指示的版本在 ======= 的下半部分。

**解决冲突步骤**：

1. 为了解决冲突，你必须选择使用由 ======= 分割的两部分中的一个，或者你也可以自行合并这些内容。
2. 运行 git status 来确认所有的合并冲突都已被解决
3. 确定之前有冲突的的文件都git add 到暂存区。
4. git commit 来完成合并提交。

###### 分支变基

![rebase](image/basic-rebase-1.png)

变基（rebase）:如上图，提取在 C4 中引入的补丁和修改，然后在 C3 的基础上应用一次的操作。

使用 `rebase` 命令将`commit`到某一分支上的所有修改都移至另一分支上，就好像“重新播放”一样。

例：检出 experiment 分支，然后将它变基到 master 分支上

```shell
git checkout experiment
git rebase master

git rebase <basebranch> <topicbranch>#直接将主题分支 （topicbranch）变基到目标分支（即 basebranch）上。 这样做能省去你先切换到 topicbranch 分支，再对其执行变基命令的多个步骤。
```

它的原理是首先找到这两个分支（即当前分支 `experiment`、变基操作的目标基底分支 `master`） 的最近共同祖先 `C2`，然后对比当前分支相对于该祖先的历次提交，提取相应的修改并存为临时文件， 然后将当前分支指向目标基底 `C3`, 最后以此将之前另存为临时文件的修改依序应用。`C4`则成为头指针分离状态的`commit`。会被`Git`的垃圾回收机制，清理掉。

![将 C4 中的修改变基到 C3 上](image/basic-rebase-3.png)

现在`master`分支和`experiment`分支可以进行`fast forwoard`合并。

```shell
git checkout master
git merge experiment
```

![master 分支的快进合并](image/basic-rebase-4.png)

>这两种整合方法的最终结果没有任何区别，但是变基使得提交历史更加整洁。 你在查看一个经过变基的分支的历史记录时会发现，尽管实际的开发工作是并行的， 但它们看上去就像是串行的一样，提交历史是一条直线没有分叉。
>
>一般我们这样做的目的是为了确保在向远程分支推送时能保持提交历史的整洁——例如向某个其他人维护的项目贡献代码时。 在这种情况下，你首先在自己的分支里进行开发，当开发完成时你需要先将你的代码变基到 origin/master 上，然后再向主项目提交修改。 这样的话，该项目的维护者就不再需要进行整合工作，只需要快进合并便可。
>
>请注意，无论是通过变基，还是通过三方合并，整合的最终结果所指向的快照始终是一样的，只不过提交历史不同罢了。 变基是将一系列提交按照原有次序依次应用到另一分支上，而合并是把最终结果合在一起。

**解决冲突**：

- 方式一：解决冲突
   1. 为了解决冲突，你必须选择使用由 ======= 分割的两部分中的一个，或者你也可以自行合并这些内容。
   2. 运行 git status 来确认所有的合并冲突都已被解决
   3. 确定之前有冲突的的文件都git add 到暂存区。
   4. git rebase --continue 来继续解决冲突。
- 方式二：
   忽略冲突（即放弃rebase 分支的修改，直接使用其他分支。)直接执行`git rebase --skip`。如果所有冲突都忽略了，就相当于把`rebase`分支直接指向目标分支。

   ```shell
   git rebase --skip
   ```

**`rebase`其他操作**：

![从一个主题分支里再分出一个主题分支的提交历史](image/interesting-rebase-1.png)

假设你希望将 `client` 中的修改合并到主分支并发布，但暂时并不想合并 `server` 中的修改， 因为它们还需要经过更全面的测试。这时，你就可以使用 `git rebase` 命令的 `--onto` 选项， 选中在 `client` 分支里但不在 `server` 分支里的修改（即 `C8` 和 `C9`），将它们在 `master` 分支上重放：

```shell
git rebase --onto master server client
```

以上命令的意思是：“取出 client 分支，找出它从 server 分支分歧之后的补丁， 然后把这些补丁在 master 分支上重放一遍，让 client 看起来像直接基于 master 修改一样”。

![截取主题分支上的另一个主题分支，然后变基到其他分支](image/interesting-rebase-2.png)

**变基风险：**
>**遵守一条准则：如果提交存在于你的仓库（即本地仓库）之外，而别人可能基于这些提交进行开发，那么不要执行变基。**

变基操作的实质是丢弃一些现有的提交，然后相应地新建一些内容一样但实际上不同的提交。 如果你已经将提交推送至某个仓库，而其他人也已经从该仓库拉取提交并进行了后续工作，此时，如果你用 git rebase 命令重新整理了提交并再次推送，你的同伴因此将不得不再次将他们手头的工作与你的提交进行整合，如果接下来你还要拉取并整合他们修改过的提交，事情就会变得一团糟。

##### 分支删除

```shell
git branch -d iss53 #删除已经合并的分支
git branch -D iss53 #强制删除为合并的分支
```

### 临时保存操作现场

#### 保存操作现场的场景

Git的Commit操作规范

- 建议：功能未开发完成前，不要commit
- 必须：在没有commit前，不要checkout分支。（例外：不同分支在处在同一个commit阶段，在commit前可以任意切换）

如果当前一个功能没有开发完成，就要切换分到比较紧急（修复Bug等）的其他功能分支进行开发。则需要保存当前开发的现场状态。等紧急的完成后在切换回来恢复现场状态继续开发。

- 保存现场（git stash）
- 切换分支
- 恢复现场（git stash apply 或 git stash pop）

#### 保存操作现场的相关命令

##### 保存现场

```shell
git stash
```

##### 还原现场

```shell
git stash pop # 默认还原最近一次，并删除
git stash apply # 默认还原最近一次，不删除保存的现场状态
git stash apply stash@{n}  #还原指定的现场。 n为记录最近保存现场的下标。最近的为0，依次递增1.
```

##### 查看现场

```shell
git stash list # 查看保存的现场列表
```

##### 删除现场

```shell
git stash drop  stash@{n}  #删除指定的现场。 n为记录最近保存现场的下标。最近的为0，依次递增1.
```

### 整理commit 的 message

#### 修改最新的commit的message

  ```shell
  git commit amend
  ```

#### 修改历史的commit的message
  
  ```shell
  git rebase -i [要修改的commit的父commit id] # 交互时，选择'reword'选项，进行编辑commit message操作。即将'pick'修改为'reword'
  ```

#### 连续多个commit的message整理成一个

```shell
  git rebase -i [要修改的commit的父commit id] # 交互时，选择'reword'选项，进行编辑commit message操作。即将'pick'修改为'reword'
```

#### 间隔的几个commit的message整理成一个

```shell
  git rebase -i [要修改间隔的commit的最早commit的父commit id] # 交互时，选择'squash'选项，进行合并commit操作。将间隔的commit中选择最早commit为基础commit，然后把其余的commit的操作选项'pick'修改为'squash'，并连续的放到基础commit之后。确定后，即可把其他的commit并入基础commit中。
```

如果rebase的父commit没有，则选择最早commit，在进行交互时，选择最早的commit的操作为'pick'选项。

### 挑选commit

常见使用场景：

- 发现已提交的修改，写到错误的branch上，需要将这些commit挑拣到正确的分支上。
- 在已有的分支上，需要功能或修改在其他分支已经修改好并提交，这时只需要将其他分支已经完成的commit挑拣到自己所在的分支。

使用cherry-pick命令，挑选commit。挑拣是复制操作，复制commit内容，在目标分支上生成一个新的commit。

```shell
git cherry-pick [目标 commit id]# 将目标commit挑拣到当前分支，并将当前分支的HEAD指向移到目标commit。
```

### 比较文件差异

1. 比较暂存区和HEAD所含文件差异

   ```shell
   git diff --cached
   ```

2. 比较工作区和暂存区

   ```shell
   git diff # 查看暂存前后的变化
   git diff -- [<filepath>...]# 比较一个或多个文件差异
   ```

### 恢复文件

1. 暂存区恢复成HEAD一样

   ```shell
   git reset HEAD #恢复全部
   git reset HEAD -- [<filepath>...] #恢复一个或多个文件
   ```

2. 工作区恢复成暂存区一样

   ```shell
   git checkout -- [<filepath>...] #恢复一个或多个文件
   ```

### Git 的备份

#### 常用协议

| 常用协议       | 语法格式                                                                                                            | 说明     |
| -------------- | ------------------------------------------------------------------------------------------------------------------- | -------- |
| 本地协议（1）  | `/path/to/repo.git`                                                                                                 | 哑协议   |
| 本地协议（2）  | `file:///path/to/repo.git`                                                                                          | 智能协议 |
| http/https协议 | <div>`http://git-server.com:port/path/to/repo.git`</div> <div>`https://git-server.com:port/path/to/repo.git`</div> | 智能协议 |
| ssh协议        | `user@git-server.com:path/to/repo.git`                                                                             | 智能协议 |

>协议中的path 使用Git Bash 中的显示的路径。
>哑协议和智能协议的区别：
>直观区别：哑协议传输进度不可见，智能协议传输进度可见
>
>传输速度：智能协议比哑协议快。

#### 本地备份

##### Clone操作方式

1. 在本地创建一个备份文件夹
2. 直接克隆一个要备份的项目到本地仓库

```shell

# 克隆一个没有工作区的裸仓库。window 的路径：E:\folder\.git 转成git 路径为：/e/folder/.git例如：git clone --bare /e/hqy/学习/Git/.git testya.git或者使用智能协议git clone --bare file:///e/hqy/学习/Git/.git testya.git

git remote clone --bare [本地路径] [备份的仓库名字]
```

##### Push操作方式

1. 在本地创建一个备份文件夹
2. 在项目工作目录操作：

```shell
git remote -v # 查看远程仓库和本地备份仓库列表
# git remote add localRep file:///e/GitbackupTest/localRep.git  添加本地仓库
# git remote add remoteRep https://git-server.com:port/path/to/repo.git  添加远程仓库
# 添加远程仓库，
git remote add [备份仓库名] [对应的协议地址]
# 将数据推送到仓库
git push [备份仓库名]

```

### Git 标签

标签适用于项目，和具体`branch`无关。

```shell
git tag # 查看标签
git show [tag name]#查看标签详细信息。
git tag [tag name] # 在当前HEAD指向的commit上打标签。
git tag -a [tag name] -m [tag message] # 在当前HEAD指向的commit上打标签，并附加标签信息。
git tag -d [tag name] # 删除指定的标签
git tag -l [tag name] # 模糊查询 列：git tag -l 'v2.*' 查询所有以'v2.'开头的所有标签。
```

### Git 责任人

```shell
git blame [file] # 查看该文件的所有修改提交的作者。
```

## Git 远程仓库版本管理

远程仓库交互图：
![工作区关系图](image/work_space.png)

### 创建远程仓库（以GitHub为例）

1. ssh协议创建

```shell
ssh user@host #使用ssh连接主机账号或GitHub帐号
git init --bare /path/to/repo.git #在/path/to/初始化一个裸(bare)仓库repo.git
```

1. Web 服务创建
![创建远程仓库](image/creater_remote_1.png)

![创建远程仓库](image/creater_remote_2.png)

### 添加并设置远程库地址别名

```shell
git remote -v #查看当前所有远程地址别名
git remote add [别名] [远程地址]#设置添加远程仓库别名到本地配置
```

### 克隆（Clone）远程仓库到本地

```shell
git clone [远程仓库地址或别名] # 克隆远程仓库到本地
```

clone命令做了三个操作：

1. 完整的把远程库下载到本地
2. 添加并设置`origin`为**远程地址仓库地址**别名
3. 初始化本地库
4. 创建本地的`master`分支，并设置为跟踪远程的`origin/master`分支的**跟踪分支**

>跟踪分支:基于**远程跟踪分支**`checkout`一个本地分支，会自动创建所谓的“跟踪分支”（它跟踪的分支叫做“upstream branch”）。 跟踪分支是与远程分支有直接关系的本地分支。

### 删除远程库

1. 进入要仓库

2. 点击仓库 `Settings` 选项

![进入设置](image/repository_del_1.png)
3. 点击`Delete this repository`按钮进行删除

![进入设置](image/repository_del_2.png)
![进入设置](image/repository_del_3.png)

### 本地和远程仓库关联操作

#### 查看远程仓库

```shell
git remote show origin # 命令列出了当你在特定的分支上执行 git push 会自动地推送到哪一个远程分支。 它也同样地列出了哪些远程分支不在你的本地，哪些远程分支已经从服务器上移除了， 还有当你执行 git pull 时哪些本地分支可以与它跟踪的远程分支自动合并。
```

#### 引用规范(refspec)

格式: `[+] <src>:<dst>`

由一个**可选的** + 号和紧随其后的 `<src>:<dst>` 组成， 其中 `<src>` 代表`source`(来源)； `<dst>`代表`destination`(目的地) 。 + 号告诉`Git`即使在不能Fast Forward（快进）的情况下也要（强制）更新引用。

##### `fetch`引用规范

我们从添加一个远程仓库说起，例：

```shell
git remote add origin https://github.com/schacon/simplegit-progit
```

运行上述命令会在仓库里`.git/config`文件中添加一节，并配置指定远程仓库名称为`origin`、`URL`和拉取操作的**refspec(引用规范)**:

```shell
[remote "origin"]
   url = https://github.com/schacon/simplegit-progit
   fetch = +refs/heads/*:refs/remotes/origin/*
```

格式由一个**可选的** + 号和紧随其后的 `<src>:<dst>` 组成， 其中 `<src>` 代表`source`(来源)，即远程版本库中的引用； `<dst>`代表`destination`(目的地) 是本地跟踪的远程引用的位置。 + 号告诉`Git`即使在不能Fast Forward（快进）的情况下也要（强制）更新引用。

默认情况下，引用规范由`git remote add origin`命令自动生成，`Git`获取服务器中`refs/heads/`下面的所有引用，并将它写入到本地的`refs/remotes/origin/`中。 所以，如果服务器上有一个`master`分支，你可以在本地通过下面任意一种方式来访问该分支上的提交记录：

```shell
git log origin/master
git log remotes/origin/master
git log refs/remotes/origin/master
```

上面的三个命令作用相同，因为`Git`会把它们都扩展成 `refs/remotes/origin/master`。

想让 Git 每次只拉取远程的 master 分支，而不是所有分支， 可以把（引用规范的）获取那一行修改为只引用该分支：

```shell
# 修改`.git/config`文件。
fetch = +refs/heads/master:refs/remote/origin/master
```

针对该远程版本库的`git fetch`操作的默认引用规范。即，使用`git fetch`命令默认拉取远程仓库的`master`分支。如果希望被执行一次的操作，我们也可以在命令行指定引用规范。例，要将远程的 master 分支拉到本地的 origin/mymaster 分支，可以运行：

```shell
# 修改`.git/config`文件。
git fetch origin master:refs/remote/origin/mymaster
```

也可以指定多个引用规范。 在命令行中，你可以按照如下的方式拉取多个分支：

```shell
git fetch origin master:refs/remotes/origin/mymaster topic:refs/remotes/origin/topic
```

这个例子中，对`master`分支的拉取操作被拒绝，因为它不是一个可以Fast Forward(快进)的引用。 我们可以通过在引用规范之前指定`+`号来覆盖该规则。

在配置文件中指定多个用于`fetch`操作的引用规范。 如果想在每次从`origin`远程仓库`fetch`时都包括`master`和`experiment`分支，添加如下两行：

```shell
[remote "origin"]
   url = https://github.com/schacon/simplegit-progit
   fetch = +refs/heads/master:refs/remotes/origin/master
   fetch = +refs/heads/experiment:refs/remotes/origin/experiment
```

 假设你有一个 QA 团队，他们推送了一系列分支，同时你只想要获取 master 和 QA 团队的所有分支而不关心其他任何分支，那么可以使用如下配置：

```shell
[remote "origin"]
   url = https://github.com/schacon/simplegit-progit
   fetch = +refs/heads/master:refs/remotes/origin/master
   fetch = +refs/heads/qa/*:refs/remotes/origin/qa/*
```

如果项目的工作流很复杂，有`QA`团队推送分支、开发人员推送分支、集成团队推送并且在远程分支上展开协作，你就可以像这样（在本地）为这些分支创建各自的命名空间，非常方便。

##### `push`引用规范

上面这样从远程版本库`fetch`已在命名空间中的引用当然很棒，但`QA`团队最初应该如何将他们的分支放入远程的`qa/`命名空间呢？ 我们可以通过引用规范`push`来完成这个任务。

如果 QA 团队想把他们的 master 分支推送到远程服务器的 qa/master 分支上，可以运行：

```shell
git push origin master:refs/heads/qa/master
```

如果他们希望`Git`每次运行`git push origin`时都像上面这样推送，可以在他们的配置文件中添加一条 push 值：

```shell
[remote "origin"]
   url = https://github.com/schacon/simplegit-progit
   fetch = +refs/heads/*:refs/remotes/origin/*
   push = refs/heads/master:refs/heads/qa/master
```

正如刚才所指出的，这会让`git push origin`默认把本地`master`分支推送到远程 qa/master 分支。

##### 删除引用规范

你还可以借助类似下面的命令通过引用规范从**远程服务器**上删除引用：

```shell
git push origin :topic
```

因为引用规范（的格式）是`<src>:<dst>`，所以上述命令把`<src>`留空，意味着把远程版本库的`topic`分支定义为空值，也就是删除它。

或者你可以使用更新的语法（自 Git v1.7.0 以后可用）：

```shell
git push origin --delete topic
```

#### 查看所有的本地与远程的分支

```shell
git branch -av
git branch -vv
```

#### 本地、远程、远程跟踪分支

![### 本地仓库和远程仓库分支](image/remote_local.png)

本地和远程同步过程：

1. 使用push命令，push命令将检查远程跟踪分支的最后一次提交和远程分支的最后一次提交sha1值比较。
2. 如果一致，则跳过此步。push命令将拒绝执行，并报拒绝信息。提示远程版本比本地新。使用fetch 命令拉取远程新版修改，同步到远程跟踪分支。
3. 将本地分支（跟踪分支）merger到远程跟踪分支。解决冲突，进行合并。
4. 将本地的最新修改同步到远程分支。

##### 远程分支

远程引用是对远程仓库的引用（指针），包括分支、标签等等。 你可以通过`git ls-remote <remote>`来显式地获得远程引用的完整列表， 或者通过`git remote show <remote>`获得远程分支的更多信息。然而，一个更常见的做法是利用**远程跟踪分支**。

##### 远程跟踪分支

远程跟踪分支是远程分支状态的引用。它们是你无法移动的本地引用。一旦你进行了网络通信， `Git`就会为你移动它们以精确反映远程仓库的状态。请将它们看做书签，这样可以提醒你该分支在远程仓库中的位置就是你最后一次连接到它们的位置。

它们以`<remote>/<branch>`的形式命名。 例如，如果你想要看你最后一次与远程仓库`origin`通信时`master`分支的状态，你可以查看`origin/master`分支。 你与同事合作解决一个问题并且他们推送了一个`iss53`分支，你可能有自己的本地`iss53`分支， 然而在服务器上的分支会以`origin/iss53`来表示。

这可能有一点儿难以理解，让我们来看一个例子。 假设你的网络里有一个在`git.ourcompany.com`的`Git`服务器。 如果你从这里克隆，`Git`的`clone`命令会为你自动将其命名为`origin`，拉取它的所有数据， 创建一个指向它的`master`分支的指针，并且在本地将其命名为`origin/master`。 `Git`也会给你一个与`origin`的`master`分支在指向同一个地方的本地`master`分支，这样你就有工作的基础。

![克隆之后的远程仓库与本地仓库](image/remote-branches-1.png)

如果你在本地的`master`分支做了一些工作，在同一段时间内有其他人推送提交到`git.ourcompany.com`并且更新了它的`master`分支，这就是说你们的提交历史已走向不同的方向。 即便这样，只要你保持不与`origin`服务器连接（并拉取数据），你的`origin/master`指针就不会移动。

![本地与远程的工作可以分叉](image/remote-branches-2.png)

如果要与给定的远程仓库同步数据，运行`git fetch <remote>`命令（在本例中为`git fetch origin`）。 这个命令查找 “origin” 是哪一个服务器（在本例中，它是`git.ourcompany.com`）， 从中抓取本地没有的数据，并且更新本地数据库，移动`origin/master`指针到更新之后的位置。

![git fetch 更新你的远程跟踪分支](image/remote-branches-3.png)

为了演示有多个远程仓库与远程分支的情况，我们假定你有另一个内部`Git`服务器，仅服务于你的某个敏捷开发团队。 这个服务器位于`git.team1.ourcompany.com`。 你可以运行`git remote add`命令添加一个新的远程仓库引用到当前的项目，将这个远程仓库命名为`teamone`，将其作为完整`URL`的别名或缩写。

![添加另一个仓库](image/remote-branches-4.png)

现在，可以运行`git fetch teamone`来抓取远程仓库`teamone`有而本地没有的数据。 因为那台服务器上现有的数据是`origin`远程仓库上的一个子集， 所以`Git`并不会抓取数据而是会设置远程跟踪分支`teamone/master`指向`teamone`的`master`分支。

![远程跟踪分支 teamone/master](image/remote-branches-5.png)

###### `push`

当你想要公开分享一个分支时，需要将其推送到有写入权限的远程仓库上。本地的分支并不会自动与远程仓库同步——你必须显式地推送想要分享的分支。这样，你就可以把不愿意分享的内容放到私人分支上，而将需要和别人协作的内容推送到公开分支。

如果希望和别人一起在名为`serverfix`的分支上工作，你可以像推送第一个分支那样推送它。 运行`git push <remote> <branch>`.
这里有些工作被简化了。`Git`自动将`serverfix`分支名字展开为 `refs/heads/serverfix:refs/heads/serverfix`，那意味着，“推送本地的`serverfix`分支来更新远程仓库上的`serverfix`分支。” 我们将会详细学习`Git`内部原理的 `refs/heads/`部分， 但是现在可以先把它放在儿。你也可以运行`git push origin serverfix:serverfix`， 它会做同样的事——也就是说“推送本地的`serverfix`分支，将其作为远程仓库的`serverfix` 分支” 可以通过这种格式来推送本地分支到一个命名不相同的远程分支。 如果并不想让远程仓库上的分支叫做 `serverfix`，可以运行`git push origin serverfix:awesomebranch`来将本地的`serverfix`分支推送到远程仓库上的`awesomebranch`分支。

下一次其他协作者从服务器上抓取数据时，他们会在本地生成一个远程分支`origin/serverfix`，指向服务器的`serverfix`分支的引用。

要特别注意的一点是当抓取到新的远程跟踪分支时，本地不会自动生成一份可编辑的副本（拷贝）。换一句话说，这种情况下，不会有一个新的`serverfix`分支——只有一个不可以修改的`origin/serverfix`指针。

可以运行`git merge origin/serverfix`将这些工作合并到当前所在的分支。 如果想要在自己的`serverfix`分支上工作，可以将其建立在远程跟踪分支之上：

```shell
git checkout -b serverfix origin/serverfix
```

这会给你一个用于工作的本地分支，并且起点位于 origin/serverfix。

###### 跟踪分支

从一个**远程跟踪分支**`checkout`一个**本地分支**会自动创建所谓的“跟踪分支”（它跟踪的分支叫做`upstream branch`（上游分支）”，即**远程跟踪分支**）。 跟踪分支是与远程分支有直接关系的本地分支。 如果在一个跟踪分支上输入`git pull`，`Git`能自动地识别去哪个服务器上抓取、合并到哪个分支。

当克隆一个仓库时，它通常会自动地创建一个跟踪`origin/master`的`master`分支。 然而，如果你愿意的话可以设置其他的跟踪分支，或是一个在其他远程仓库上的跟踪分支，又或者不跟踪`master`分支。 最简单的实例就是像之前看到的那样，运行`git checkout -b <branch> <remote>/<branch>`。 这是一个十分常用的操作所以 `Git`提供了`--track`快捷方式：

```shell
git checkout --track origin/serverfix
```

由于这个操作太常用了，该捷径本身还有一个捷径。如果您尝试`checkout`的分支名称(a)不存在，且仅与一个远程仓库上的(b)名称完全匹配，那么`Git`就会为你创建一个跟踪分支：

```shell
git checkout serverfix
```

如果想要将本地分支与远程分支设置为不同的名字，你可以轻松地使用上一个命令增加一个不同名字的本地分支：

```shell
git checkout -b sf origin/serverfix # 命令原型：git checkout -b <local branch name> <remote branch name>
```

现在，本地分支`sf`会自动从`origin/serverfix`拉取。

设置已有的本地分支跟踪一个刚刚拉取下来的远程分支，或者想要修改正在跟踪的上游分支，你可以在任意时间使用`-u`或`--set-upstream-to`选项运行`git branch`来显式地设置。

```shell
git branch -u origin/serverfix
```

如果想要查看设置的所有**跟踪分支**，可以使用`git branch`的`-vv`选项。 这会将所有的本地分支列出来并且包含更多的信息，如每一个分支正在跟踪哪个远程分支与本地分支是否是领先、落后或是都有。

```shell
git branch -vv
  iss53     7e424c3 [origin/iss53: ahead 2] forgot the brackets
  master    1ae2a45 [origin/master] deploying index fix
* serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] this should do it
  testing   5ea463a trying something new
```

这里可以看到`iss53`分支正在跟踪`origin/iss53`并且'`ahead`'是`2`，意味着本地有两个提交还没有推送到服务器上。也能看到`master`分支正在跟踪`origin/master`分支并且是最新的。 接下来可以看到`serverfix`分支正在跟踪`teamone`服务器上的`server-fix-good`分支并且领先`3`落后`1`， 意味着服务器上有一次提交还没有合并入同时本地有三次提交还没有推送。 最后看到`testing`分支并没有跟踪任何远程分支。

需要重点注意的一点是这些数字的值来自于你从每个服务器上**最后**一次`fetch`的数据。 这个命令并没有连接服务器，它只会告诉你关于本地缓存的服务器数据。 如果想要统计最新的领先与落后数字，需要在运行此命令前抓取所有的远程仓库。 可以像这样做：

```shell
git fetch --all; git branch -vv
```

###### `fetch`

当`git fetch`命令从服务器上抓取本地没有的数据时，它并不会修改工作目录中的内容。 它只会获取数据然后让你自己合并。 然而，有一个命令叫作`git pull`在大多数情况下它的含义是一个`git fetch` 紧接着一个`git merge`命令。 如果有一个像之前章节中演示的设置好的跟踪分支，不管它是显式地设置还是通过`clone`或 `checkout`命令为你创建的，`git pull`都会查找当前分支所跟踪的服务器与分支， 从服务器上抓取数据然后尝试合并入那个远程分支。

由于`git pull`的魔法经常令人困惑所以通常单独显式地使用`fetch`与`merge`命令会更好一些。

#### `Push`到远程常用命令

```shell
git push origin master # Git 自动将 master 分支名字展开为 refs/heads/master:refs/heads/master。将本地的master分支推送到origin远程仓库的master分支。如果master不存在，则会被新建。
git push origin :master # 等同于 git push origin --delete master
git push origin # 将当前分支推送到origin远程仓库的对应分支。如果当前分支只有一个追踪分支，那么仓库名都可以省略。即：git push
git push -u origin master # -u 为--set-upstream 缩写。设置的当前分支跟踪一个远程master分支。
git push --all origin # 将所有本地跟踪分支都推送到origin远程仓库。
git push origin HEAD # 将当前分支推到远程上相同名称的分支。
git push origin HEAD:master #将当前分支推到“origin”远程仓库中“master”分支。
git push origin master:refs/heads/experimental #通过复制当前主分支，在origin存储库中创建experimental的分支。当本地名称和远程名称不同时，只需要在远程存储库中创建新的分支或标记;否则，使用ref名称本身。
```

#### `fetch`到本地常用命令

配置默认的分支：

```shell
[branch "master"]
   remote = origin
   merge = refs/heads/master
```

```shell
git fetch --all # 抓取所有的远程仓库的分支、标签等
git fetch origin # 从远程refs/heads/命名空间复制所有分支，并将它们存储到本地的refs/remotes/origin/命名空间中，除非使用branch.<name>.fetch选项来指定非默认的refspec
git fetch origin master # Git 自动将 master 分支名字展开为 refs/heads/master:refs/heads/master。将origin远程仓库的master分支。拉取到本地的master分支。
git pull # 等于 fetch + merger，即：拉取远程分支，合并本地和远程分支。
         # 完整命令是：git pull origin [本地分支]：[远程分支]相当于git pull + git checkout -b [本地分支名] [远程分支名]
git checkout -b [本地分支名] [远程跟踪分支名]# 创建一个基于远程跟踪分支的本地分支，并切换到该分支。
git checkout -b [本地分支名] --track [远程跟踪分支名] #创建一个追踪分支，跟踪远程跟踪分支。切换到该分支。
git checkout --track [远程跟踪分支名] #为git checkout -b [本地分支名] [远程跟踪分支名]的快捷方式
git checkout [远程分支名] #为git checkout -b [本地分支名] [远程跟踪分支名]的快捷方式，有条件限制：该分支名称不存在，且仅与一个远程仓库上的分支名称完全匹配
git branch -u [远程跟踪分支名] #为已有的本地分支设置远程跟踪分支。
```

### 删除远程仓库分支

```shell
git push origin :[分支名]# git push origin [本地分支]:[远程分支名]命令，作用是将本地的远程分支和本地分支同步，所以将本地分支置为 空，即可删除远程分支。
git push origin --delete [分支名]
git remote prune origin --dry-run # 查看不需要的远程跟踪分支，即本地有远程追踪分支，但远程分支已经删除了。
git remote prune origin #清理无效的远程追踪分支。
```

### 远程标签

#### 推送标签

```shell
git push origin [tag name ...] # 推送远程标签到远程仓库
# 完整写法：git push origin [要推送的tag]:[目标tag]
git push origin --tags # 推送本地所有标签到远程仓库。
```

#### 拉取远程标签

```shell
git pull # 拉取所有分支和标签
git fetch origin tag [tag name] # 只拉取指定标签
```

#### 删除远程标签

```shell
git push origin :[目标tag] # 删除目标tag
```

### 多人协作冲突解决（同一分支）

#### 基本步骤

1. `git push` 操作报冲突，即：远程分支和本地分支处于non fast forwoard。
2. `git pull` 拉取远程分支到本地，尝试自动将远程分支合并到本地分支。此步骤也可分开操作，先`git fetch`拉取远程分支，手动合并（等于第3步骤）。
3. 无法自动合并，需要人工解决（基本需要和冲突的提交人进行沟通解决）。
4. 完成合并后，远程分支和本地分支处于fast forwoard。然后`git push`到远程仓库。

#### 冲突产生的原因

1. 不同人修改不同文件

2. 不同人修改同一文件的不同区域

3. 不同人修改同一文件的同一区域

4. 同时修改了文件名和文件内容

5. 把同一文件改成了不同的文件名

#### 禁止操作

1. 禁止向集成分支执行 `git push -f`操作。
2. 禁止向集成分支执行变更历史操作。（即将远程分支拉取到本地做rebase操作）

### 团队协作开发的工作流程

#### 集中式工作流

