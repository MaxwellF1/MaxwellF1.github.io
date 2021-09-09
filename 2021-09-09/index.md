# WSL相关操作


前一段在新闻上看到了许多WSL2的新特性，于是就加入了Windows的开发者计划更新了一下dev版本的Win11，也学着玩玩WSL。因为感觉除了性能可能受影响，新版本的WSL2支持的GPU访问和WSLg对图形界面的支持基本上满足了我对linux的额外需求。

## 问题

1. ### 位置移动

   WSL默认装在C盘里，再在里面装各种工具后会导致空间占用很大，影响C盘空间。于是想通过某种方式将其移动到另一块比较空的磁盘中。网上查到可以直接通过wsl指令来进行压缩、移动、解压的过程(例如wsl --export和wsl --import进行打包并自定义目录安装)，然而不知道是否我操作出了问题，没有成功实现。后来查到了[DDoSolitary/LxRunOffline: A full-featured utility for managing Windows Subsystem for Linux (WSL) (github.com)](https://github.com/DDoSolitary/LxRunOffline)这款软件，可以很方便的通过一些指令来对WSL进行管理，它可以安装任意发行版到任意目录，还可以对已经安装的WSL进行转移、备份、设置默认用户和修改环境变量等操作。安装以及使用过程参考了[LxRunOffline 使用教程 - WSL 自定义安装、备份 - P3TERX ZONE](https://p3terx.com/archives/manage-wsl-with-lxrunoffline.html)上的说明(安装后记得将路径添加到系统环境变量中)。

   以下命令均在以管理员权限运行的Windows Powershell下执行。

   查看系统中已经安装的WSL：

   ```shell
   lxrunoffline l
   ```

   将WSL镜像移动到指定目录：

   ```shell
   lxrunoffline m -n <WSL名称> -d <路径>
   ```

   查看路径，进行确认：

   ```shell
   lxrunoffline -di -n <WSL名称>
   ```

   然而初次执行移动命令的时候有报错：

   ```shell
   [ERROR] Couldn't set the case sensitive attribute of the directory "......
   Reason: Indicates that the directory trying to be deleted is not empty.
   ```

   看起来是没有默认设为大小写敏感，于是尝试在Powershell中使用fsutil指令尝试enable大小写敏感，然而手动执行的时候仍然会报“文件夹不为空”的错误。后来参考[迁移WSL时的报错：0x80073d21 此应用的发布者不允许将其移动到其他位置-dtcms模板网 (dtmao.cc)](https://www.dtmao.cc/news_show_4481932.shtml)的解决办法，换了[另一个版本的LxRunOffline](https://ddosolitary-builds.sourceforge.io/LxRunOffline/LxRunOffline-v3.5.0-11-gfdab71a-msvc.zip),成功解决了报错问题。移动之后就是把C盘中对应文件夹下的镜像文件ext4.vhdx移动到了指定目录下，然而移动完成后发现无法打开移动后的WSL子系统，后来通过重启电脑解决了这个问题。

2. 


