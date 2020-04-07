# git命令

分支操作：

1. `git branch [分支名]`：创建分支（分支只是简单的指向某个提交记录，不会造成储存或内存开销）；
2. `git checkout [分支名]`：切换分支；
3. `git checkout -b [分支名]`：创建新分支并切换到该分支；
4. `git branch -f master HEAD~3`：强制修改分支位置；

合并分支：

1. `git merge [其他分支]`：在当前分支上使用，以当前分支节点和其他分支为父节点合并产生新节点；
2. `git rebase [其他分支]`：Rebase 实际上就是在当前分支取出一系列的提交记录，“复制”它们，然后在其他分支逐个的放下去；

HEAD：HEAD 总是指向当前分支上最近一次提交记录。大多数修改提交树的 Git 命令都是从改变 HEAD 的指向开始的；

- 查看HEAD指向：`cat .git/HEAD`
- 如果HEAD指向的是一个引用，则可以使用：`git symbolic-ref HEAD`

查看提交记录的哈希值：`git log`

相对引用：

- ^：向上移动一个提交记录；
- `~<num>`：向上移动num个提交记录；比如：`git checkout HEAD~5`

- `^<num>`：左右节点的选择，用于merge合并的节点

撤销变更：

- `git reset HEAD~`：（对远程分支无效）回退几个提交记录来实现撤销改动；
- `git revert HEAD~`：用于修改远程（相当于把回退之前的节点复制成下一个节点）；

将一些提交复制到当前所在位置（HEAD）下面：

- `git cherry-pick <提交号>`：提交号是哈希值，可以为多个；

不知道哈希值的时候，可以使用`git rebase -i `，Git 会打开一个 UI 界面并列出将要被复制到目标分支的备选提交记录，它还会显示每个提交记录的哈希值和提交说明，提交说明有助于你理解这个提交进行了哪些更改。

- `git tag [版本号] [节点]`：标签
- `git describe`：在多次提交中找到方向；

远程操作：

- `git clone [url]`：从远程仓库下载
- 远程分支命名：`<remote name>/<branch name>`

`git pull [url]`：等效于以下两个操作，先fetch再merge合并；

`git pull --rebase`：等效于git与rebase的缩写；

- `git fetch`：`git fetch` 实际上将本地仓库中的远程分支更新成了远程仓库相应分支最新的状态；
  - 从远程仓库下载本地仓库中缺失的提交记录；
  - 更新远程分支指针(如 `origin/master`)；
  - `git fetch` 并不会改变你本地仓库的状态。它不会更新你的 `master` 分支，也不会修改你磁盘上的文件；
- 合并远程操作：
  - `git cherry-pick origin/master`
  - `git rebase origin/master`
  - `git merge origin/master`

`git push`：负责将**你的**变更上传到指定的远程仓库，并在远程仓库上合并你的新提交记录；

- `git push` ：不带任何参数时的行为与 Git 的一个名为 `push.default` 的配置有关。

- 如果在push之前远程库有更改，会push失败：
  - git fetch
  - git rebase origin/master
  - git push