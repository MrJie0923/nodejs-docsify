### 场景一：小张创建项目并提交到远程Git仓库

https://www.cnblogs.com/supiaopiao/p/11093371.html

创建好项目，选择VCS - > Import into Version Control -> Create Git Repository

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191359187-1860171399.png)

接下来指定本地仓库的位置，按个人习惯指定即可，例如这里选择了项目源代码同目录

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191424484-1177933528.png)

点击OK后创建完成本地仓库，注意，这里仅仅是本地的。下面把项目源码添加到本地仓库。

下图是Git与提交有关的三个命令对应的操作，Add命令是把文件从IDE的工作目录添加到本地仓库的stage区，Commit命令把stage区的暂存文件提交到当前分支的仓库，并清空stage区。Push命令把本地仓库的提交同步到远程仓库。

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191442000-2001925393.png)

 介绍一下版本库的概念：

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190627093634940-1332392815.png)

 

IDEA中对操作做了一定的简化，Commit和Push可以在一步中完成。

具体操作，在项目上点击右键，选择Git菜单

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191510877-1645103499.png)

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191609594-1214648522.png)

 因为是第一次提交，Push前需要指定远程仓库的地址。如下图，点击Define remote后，在弹出的窗口中输入远程仓库地址。
![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191621844-720986537.png)

 

### 场景二：小袁从远程Git仓库上获取项目源码

即克隆项目，操作如下：

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191634758-777282851.png)
输入小张Push时填写的远程仓库地址

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191647161-2057886327.png)

接下来按向导操作，即可把项目从远程仓库克隆到本地仓库和IDE工作区。

### 场景三：小袁修改了部分源码，提交到远程仓库

这个操作和首次提交的流程基本一致，分别是 Add -> Commit -> Push。请参考场景一

### 场景四：小张从远程仓库获取小袁的提交

获取更新有两个命令：Fetch和Pull，Fetch是从远程仓库下载文件到本地的origin/master，然后可以手动对比修改决定是否合并到本地的master库。Pull则是直接下载并合并。如果各成员在工作中都执行修改前先更新的规范，则可以直接使用Pull方式以简化操作。

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191701650-522350775.png)

 

### 场景五：小袁接受了一个新功能的任务，创建了一个分支并在分支上开发

建分支也是一个常用的操作，例如临时修改bug、开发不确定是否加入的功能等，都可以创建一个分支，再等待合适的时机合并到主干。

创建流程如下：
![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191718678-1479339413.png)
选择New Branch并输入一个分支的名称
![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191729860-828336031.png)

 创建完成后注意IDEA的右下角，如下图，Git: wangpangzi_branch表示已经自动切换到wangpangzi_branch分支，当前工作在这个分支上。
点击后弹出一个小窗口，在Local Branches中有其他可用的本地分支选项，点击后选择Checkout即可切换当前工作的分支。
![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191743072-1997442321.png)

 如下图，点击Checkout

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191756279-210972420.png)

注意，这里创建的分支仅仅在本地仓库，如果想让组长小张获取到这个分支，还需要提交到远程仓库。

### 场景六：小袁把分支提交到远程Git仓库

切换到新建的分支，使用Push功能

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191815183-1189668277.png)

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191829029-116085391.png)

### 场景七：小张获取小袁提交的分支

使用Pull功能打开更新窗口，点击Remote栏后面的刷新按钮，会在Branches to merge栏中刷新出新的分支。这里并不想做合并，所以不要选中任何分支，直接点击Pull按钮完成操作。

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191840106-351394877.png)

 更新后，再点击右下角，可以看到在Remote Branches区已经有了新的分支，点击后在弹出的子菜单中选择Checkout as new local branch，在本地仓库中创建该分支。完成后在Local Branches区也会出现该分支的选项，可以按上面的方法，点击后选择Checkout切换。
![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191904733-308447494.png)

 

### 场景八：小张把分支合并到主干

新功能开发完成，体验很好，项目组决定把该功能合并到主干上。

切换到master分支，选择Merge Changes

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191917611-1806455291.png)

 选择要合并的分支，点击Merge完成

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626191930507-1439850753.png)

### 场景九：master代码同步到test分支

假设有master主干和test分支，现在要把master上的代码同步到test分支上：
首先切换到test分支（git checkout test），将test分支代码先提交Git上保存备份，
方案一：和代码（git merge master），IDEA中点击右下角，点击master，再点击Merge

![img](https://img2018.cnblogs.com/blog/1188380/201907/1188380-20190731092833706-2116517775.png)

方案二：分支上代码先保存或提交，再直接pull

![img](https://img2018.cnblogs.com/blog/1188380/201907/1188380-20190731093337870-1008240337.png)

选中上面的test和下面的master，之后就会把master上的代码同步到test分支了（图中分支名称不一致，按照你自己的分支来操作就ok）

 ![img](https://img2018.cnblogs.com/blog/1188380/201907/1188380-20190731093258361-265155769.png)

 

### 版本冲突的解决办法

**1.提交commit冲突：**

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626193548852-39714681.png)

 

如上图所示，当提交代码时发生冲突，此时操作步骤如下

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626193624952-1981095504.png)

 

 

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626193640860-1259076716.png)

在这里我们希望将本地冲突代码和服务端的代码都保留，此时在上图中点击中间的两个红色按钮后即可将本地和服务端的冲突代码全部保留，在中间一栏result中可以预览到冲突解决后的代码情况，冲突处理完成后如下图

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626193707530-638934208.png)

在上图merge对话框中点击右下角的apply时，一般会在idea的右下角出现如下的Push rejected提示对话框

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626193728469-1283724535.png)

 

上图对话框提示我们，在push的时候发生了失败，此时要注意，将本地代码更新提交到服务端，需要经历两个步骤：commit和push，commit相当于提交到了本地库，而push则是将本地库提交更新到服务端。根据上图提示push rejected，我们可以发现冲突已经在本地库commit时解决了，我们只需push一次即可，因此后续操作只需要push一次即可，如下：

在冲突的文件上，右键

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626193745133-2031336526.png)

push完成后即可将本地库的文件提交到服务端。

**2.更新update冲突**

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626193817845-51568870.png)

 

如上图所示，点击更新按钮时发现冲突，提示用户如下所示的merge页面上进行冲突的处理，比如此处我们希望将本地修改和远端服务器的修改均保留，因此按照如下图所示的红色1和2操作，合并到中间的result一栏中。

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626193830562-524034758.png)

 冲突解决后，如下所示：

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626193845138-1707946768.png)

至此，更新冲突就已解决完成。

## Git命令全家福

![img](https://img2018.cnblogs.com/blog/1188380/201906/1188380-20190626193905690-1543978978.png)