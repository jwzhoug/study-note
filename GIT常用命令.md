# 1. 配置信息

## 1.1 用户信息

`git config  `命令用于设置/查看 配置信息，那么也包括用户信息

### 1.1.1 查看用户信息

* `git config --global user.name` :    //获取当前登录的用户
* `git config --global user.email` :   //获取当前登录用户的邮箱

### 1.1.2 设置/登录用户

* `git config --global user.name` 'uesename' :    //获取当前登录的用户
* `git config --global user.email`'email' :   //获取当前登录用户的邮箱

## 1.2 查看所有配置信息

* `git config --list`

# 2.  获取命令使用帮助

## 2.1 查看所有命令

* `git help`
* `git --help`

## 2.2 查看特定命令

比如查看config相关的命令：`git help config`

# 3. git 仓库

关于git仓库有两种方式

## 3.1 对现有的目录进行 git 管理，叫做初始化仓库

进入需要管理的目录中，并且输入：

`git init`

这个命令就会在你的目录中创建一个.git的子目录，这个子目录含有你初始化的 Git 仓库中所有的必须文件，这些文件是 Git 仓库的骨干。 但是，在这个时候，我们仅仅是做了一个初始化的操作，你的目录里的文件还没有被跟踪。

如何跟踪这些文件我会在之后的小结中说明

## 3.2 克隆已存在的仓库

如果你想克隆一份已存在的仓库，那就可以使用：`git clone [url]`

* 比如克隆 github上的项目`git clone https://github.com/xxxx/xxxx`
* 你也可以在克隆的同时自定义本地仓库的名臣 目`git clone https://github.com/xxxx/xxxx consumName`

这里需要说明的是，git支持多种传输协议，不光是这里例子中的https协议

## 3.3 本地仓库中的文件状态

在3.1中我们有提到目录中的我文件还没有跟踪的，相对应的自然有跟踪，所以仓库中的每一个文件的状态不外乎这两种了：

* 已跟踪 ：被纳入版本控制的文件，初次使用 3.2 中的方法克隆的仓库中文件都属于已跟踪文件，并处于未修改状态
* 未跟踪 

已跟踪文件在一段时间之后可能处于：

* 未修改
* 已修改
* 已放入暂存区

自上次提交后做了修改的文件，Git 将它们标记为已修改文件。 我们逐步将这些修改过的文件放入暂存区，然后提交所有暂存了的修改，如此反复。 

### 3.3.1 检查当前文件状态

`git status`命令，在克隆仓库后立即使用此命令，会看到类似这样的输出： 

```
$ git status
On branch master
nothing to commit, working directory clean
```

这说明你现在的工作目录相当干净。换句话说，所有已跟踪文件在上次提交后都未被更改过。 此外，上面的信息还表明，当前目录下没有出现任何处于未跟踪状态的新文件，否则 Git 会在这里列出来。 最后，该命令还显示了当前所在分支，并告诉你这个分支同远程服务器上对应的分支没有偏离。 现在，分支名是 “master”,这是默认的分支名。 

让我们在项目下创建一个新的 redeme.md 文件。 如果之前并不存在这个文件，使用 `git status` 命令，你将看到一个新的未跟踪文件： 

 ```
On branch master
Your branch is up to date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        "redeme.md"

nothing added to commit but untracked files present (use "git add" to track)

 ```

#### 跟踪新文件

**使用  `git add [file/path]` 命令跟踪新的文件**

**`git add` 命令使用文件或目录的路径作为参数；如果参数是目录的路径，该命令将递归地跟踪该目录下的所有文件。** 

比如：上文中的  redeme.md 文件

输入：`git add redeme.md`，然后输入  `git status`可以看到新的信息

    On branch master
    
    Your branch is up to date with 'origin/master'.
    
    Changes to be committed:
    
      (use "git reset HEAD <file>..." to unstage)
        new file:   "redeme.md"

只要在 `Changes to be committed` 这行下面的，就说明是已暂存状态。 如果此时提交，那么该文件此时此刻的版本将被留存在历史记录中。 

#### 暂存已修改文件

现在我们来修改一个已被跟踪的文件。 如果你修改了一个名为 `redeme.md` 的已被跟踪的文件，然后运行 `git status` 命令，会看到下面内容：

```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   "redeme.md"

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   "redeme.md"
```

文件 `redeme.md` 出现在 `Changes not staged for commit` 这行下面，说明已跟踪文件的内容发生了变化，但还没有放到暂存区。 要暂存这次更新，需要运行 `git add` 命令。 这是个多功能命令：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等。 将这个命令理解为“添加内容到下一次提交中”而不是“将一个文件添加到项目中”要更加合适。 现在让我们运行 `git add` 将"CONTRIBUTING.md"放到暂存区，然后再看看 `git status` 的输出：

```
$ git add redeme.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
```

现在两个文件都已暂存，下次提交时就会一并记录到仓库。 假设此时，你想要在 `CONTRIBUTING.md` 里再加条注释， 重新编辑存盘后，准备好提交。 不过且慢，再运行 `git status` 看看：