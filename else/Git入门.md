# Git入门

git是一个分布式版本控制系统，中文文档地址：[https://git-scm.com/book/zh/v2/](https://git-scm.com/book/zh/v2/)

1. 初次运行配置：

   ```
   // 设置姓名、邮箱
   git config --global user.name
   git config --global user.email
   // 查看设置的信息
   git config --list
   ```

2. 初始化：

   1. 项目刚刚建立，还没有使用版本控制时，可以通过git命令将该项目加入到版本控制中。在使用Maven进行管理的项目中，有用的文件基本上就是src目录和pom.xml文件，除此之外的文件一般忽略。在git中配置忽略文件很简单，在项目的顶层目录中或具体的某个目录中添加一个名为`.gitignore`的文件即可；在Windows上创建时，由于只有文件后缀，没有文件名，需要使用cmd或Git Bash，输入`echo"> .gitignore`，然后进行输入，结束时输入`"`即可。

      常用的`.gitignore`文件内容：

      ```
      # Maven #
      target/
      
      # IDEA #
      .idea/
      
      *.iml
      
      # Eclipse #
      .settings/
      .classpath
      .project
      ```

      在Git Bash中输入`git init`，就能完成初始化；

   2. 克隆本地仓库：如果要操作的仓库已经使用Git进行版本控制了，那么便可以使用git clone 命令将项目克隆到本地；

3. 本地操作：使用git对项目进行版本控制时，基本是大量重复的本地操作。最常见的就是添加文件到版本控制，提交到仓库，查看状态，查看历史记录等。

   ```
   # 查看状态
   git status
   # 将文件添加到版本控制
   git add --all
   # 提交
   git commit -m "提交"
   # 查看提交记录
   git log
   ```

4. 远程操作：

   ```
   # 查看所有远程源
   git remote -v
   # 查看远程源别名
   git remote
   # 添加远程源
   git remote add [别名] [url]
   # 删除远程源
   git remote remove [别名]
   
   # 远程源有两种链接方式：一种https，一种ssh
   # 修改链接方式：
   git remote set-url [本地分支] [url] 
   
   或者在.git/conf文件中修改：修改对应分支的url为指定url即可
   [core]
   	repositoryformatversion = 0
   	filemode = false
   	bare = false
   	logallrefupdates = true
   	symlinks = false
   	ignorecase = true
   [remote "origin"]
   	url = git@github.com:ZhSMM/new.git
   	fetch = +refs/heads/*:refs/remotes/origin/*
   [branch "master"]
   	remote = origin
   	merge = refs/heads/master
   
   # 将本地仓库与远程仓库同步
   git pull origin master
   
   # github：可以fork官方仓库到自己的仓库
   # 然后git clone [url] 下来
   # 给clone下来的仓库添加一个上游的仓库
   git remote add upstream [url]
   # 同步
   git push upstream master
   # 比如合并分支
   git fetch upstream
   git merge upstream/master
   # 提交
   git push origin master
   
# 分支
   git tag
# 切换分支
   git checkout [分支名]
   ```
   
   
   
   