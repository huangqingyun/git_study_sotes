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
   git config --local [配置项] [配置值]
   ```

- 对当前登录用户的所有仓库有效

   ```shell
   git config --global [配置项] [配置值]
   ```

- 系统所有登录的用户有效

   ```shell
   git config --system [配置项] [配置值]
   ```

### 显示配置信息，加 --list

```shell
git config --local --list
git config --global --list
git config --system --list
```

## Git 本地版本管理

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

#### Git 工作方式

```sequence
participant workspace as W
participant index/stage as S
participant Repository as R
W->S: git add [file]
S->R:git commit -m "提交信息"
```

#### Git 管理的中的文件状态

- **Untracked**:未跟踪, 此文件在文件夹中, 但并没有加入到git库, 不参与版本控制. 通过git add 状态变为Staged.
- **Unmodify**:文件已经入库, 未修改, 即版本库中的文件快照内容与文件夹中完全一致. 这种类型的文件有两种去处, 如果它被修改, 而变为Modified.如果使用git rm移出版本库, 则成为Untracked文件
- **Modified**: 文件已修改, 仅仅是修改, 并没有进行其他的操作. 这个文件也有两个去处, 通过git add可进入暂存staged状态, 使用git checkout 则丢弃修改过,返回到unmodify状态, 这个git checkout即从库中取出文件, 覆盖当前修改。
- **Staged**: 暂存状态. 执行git commit则将修改同步到库中, 这时库中的文件和本地文件又变为一致, 文件为Unmodify状态. 执行git reset HEAD filename取消暂存,文件状态为Modified。

![文件状态](image/git_file_status.png)

![文件状态转换](image/git_chanage_file_status.png)

#### 通操作认识Git工作区、工作方式以及文件状态变化

```shell
git status # 查看工作区和暂存区的文件状态
cp [path] [file name] #拷贝文件到本地或GUI方式新建文件
git status #查看上一步创建或拷贝的文件状态。
git add [file name] #将文件添加到暂存区或者使用 git add -u 将工作区的所有跟踪的文件提交到暂存区。
git status
git commit -m 'message' #将暂存区的所有文件的更改提交到版本库，或者使用git commit -m 'message' [file name...]指定一个或多个文件提交
git status

```

#### 文件重命名

1. 手动更改

   ```shell
   mv [old file name] [new file name] #或者使用GUI方式修改
   git add [new file name] # 将修改后的文件添加到暂存区
   git rm [old file name] #将暂存区的旧文件删除。
   ```

2. Git方式

   ```shell
   git mv [old file name] [new file name] #一步到位
   ```

#### 查看历史版本

##### 命令行工具查看

多屏显示控制方式：

- 空格：向下翻页
- b: 向上翻页
- q: 退出

```shell
git log # 查看当前分支所有版本演变历史
git log --oneline #简洁模式查看
git log -n2 #查看最近的2次版本演变历史
git log -n4 --oneline #简洁模式查看最近的4次版本演变历史
git branch -v # 查看分支。或者使用 git branch -av查看所有分支
git log --all #查看所有分支的版本演变历史。
git log --all --graph # 图形化方式的查看所有分支的版本演变历史。

# git log --前缀的参数可以组合使用
git --help --web [command name] #使用网页方式查看命令详细帮助
```

##### 图形界面工具查看

```shell
gitk #打开图形界面
```

#### 分支

分支可以理解为独立的工作空间。
写文档可以根据章节创建分支，进行编辑
项目可以根据不同的功能创建分支，进行开发
修改文档里的公司logo,可根据不同的公司创建不同分支。