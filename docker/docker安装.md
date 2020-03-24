# docker安装

在Windows上安装docker：

1. 使用 `docker-for-mac` 或者 `docker-for-windows` (仅`Windows10`专业版支持)客户端；
2. 使用[Docker Toolbox](https://get.daocloud.io/toolbox/)（需要下载[boot2docker](https://github.com/boot2docker/boot2docker/releases)镜像）

使用Docker Toolbox安装的细节：

1. 安装之前确认CPU虚拟化已启动；

2. 安装后会安装一个VirtualBox虚拟机，一个Kitematic，这是GUI管理Docker的工具，还有在命令行下用到的docker-machine和docker命令了；

3. 基本使用：

   ```powershell
   // 查看当前docker虚拟机状态
   >docker-machine ls 
   
   // 创建一个叫default的docker虚拟机
   >docker-machine create --driver=virtualbox default 
   
   // 获得虚拟机的环境变量
   >docker-machine env default
   
   // 将当前的PowerShell和虚拟机里面的Docker Linux建立连接，以便在
   // PowerShell中使用docker命令
   docker-machine env default | Invoke-Expression
   ```

4. 默认情况下，docker-machine创建的虚拟机文件，是保存在C盘的C:\Users\用户名\.docker\machine\machines\default 目录下的，如果下载和使用的镜像过多，那么必然导致该文件夹膨胀过大，如果C盘比较吃紧，那么我们就得考虑把该虚拟机移到另一个盘上。

   1. 使用docker-machine stop default停掉Docker的虚拟机；
   2. 打开VirtualBox，选择“管理”菜单下的“虚拟介质管理”，我们可以看到Docker虚拟机用的虚拟硬盘的文件disk；
   3. 选中“disk”，然后点击菜单中的“复制”命令，根据向导，把当前的disk复制到另一个盘上面去；
   4. 回到VirtualBox主界面，右键“default”这个虚拟机，选择“设置”命令，在弹出的窗口中选择“存储”选项；
   5. 把disk从“控制器SATA”中删除，然后重新添加我们刚才复制到另外一个磁盘上的那个文件；
   6. .确定，回到PowerShell，我们使用docker-machine start default就可以启动新地址的Docker虚拟机了。确保新磁盘的虚拟机没有问题。就可以把C盘那个disk文件删除了；
   7. 注意：不要在Window中直接去复制粘贴disk文件，这样会在步骤5的时候报错的，报错的内容如下，所以一定要在VirtualBox中去复制！

5. 镜像加速：DaoCloud、阿里云的镜像、网易的蜂巢，以DaoCloud为例：

   1. 注册账号

   2. [https://www.daocloud.io/mirror](https://www.daocloud.io/mirror)得到加速地址

   3. 在powershell中执行：

      ```powershell
      docker-machine ssh default 
      sudo sed -i "s|EXTRA_ARGS='|EXTRA_ARGS='--registry-mirror=加速地址 |g" /var/lib/boot2docker/profile 
      exit 
      docker-machine restart default
      ```

   4. 重启Docker