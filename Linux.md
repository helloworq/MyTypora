# Linux



![640](E:\DistCode\TyporaLoad\Linux.assets\640.jpg)

- **bin  存放二进制可执行文件(ls,cat,mkdir等)**
- boot  存放用于系统引导时使用的各种文件
- dev  用于存放设备文件
- **etc   存放系统配置文件**
- home  存放所有用户文件的根目录
- lib   存放跟文件系统中的程序运行所需要的共享库及内核模块
- mnt  系统管理员安装临时文件系统的安装点
- **opt   额外安装的可选应用程序包所放置的位置**
- proc  虚拟文件系统，存放当前内存的映射
- **root  超级用户目录**
- sbin  存放二进制可执行文件，只有root才能访问
- tmp  用于存放各种临时文件
- usr   用于存放系统应用程序，比较重要的目录/usr/local 本地管理员软件安装目录
- var   用于存放运行时需要改变数据的文件



## Linux用户和权限管理

Linux是一个**多用户的系统**，我们可以多个用户同时登陆Linux

### 账户概念

Linux中的账户包括:

- **用户账户**

- - **普通用户账户**：在系统上的任务是进行普通工作
  - **超级用户账户**（或管理员账户）：在系统上的任务是对普通用户和整个系统进行管理。

- **组账户**(组是用户的集合)

- - 标准组：标准组可以容纳多个用户
  - 私有组：私有组中只有用户自己

当一个用户同**属于多个组**时，将这些组分为

- **主组**（初始组）：用户登录系统时的组。
- **附加组**：登录后可切换的其他组

**上面也说了**，账户的实质上就是用户在系统上的标识，这些标识是用**文件保存**起来的：

- 用户名和 UID 被保存在`/etc/passwd`文件中，文件权限 `(-rw-r--r--)`
- 组和GID 被保存在 `/etc/group`文件中，文件权限`(-r--------)`
- 用户口令(密码)被保存在 `/etc/shadow`文件中  ，文件权限`(-rw-r--r-- )`
- 组口令被保存在 `/etc/gshadow`文件中 ，文件权限 `(-r--------)`

### 管理linux用户

**用户管理**：                                             **组管理**：

`useradd `                                              `groupadd`

`usermod `                                              `groupmod`

`userdel `                                              `groupdel`

**批量管理用户**：

- 成批添加/更新一组账户：`newusers`
- 成批更新用户的口令：`chpasswd`

**组成员管理**：

- 向标准组中添加用户

- - `gpasswd -a <用户账号名> <组账号名>`
  - `usermod -G <组账号名> <用户账号名>`

- 从标准组中删除用户

- - `gpasswd -d <用户账号名> <组账号名>`

**口令维护**(禁用、恢复和删除用户口令)：

- **设置用户口令**：

- - `passwd [<用户账号名>]`

- 禁用用户账户口令

- - `passwd -l <用户账号名>`

- 查看用户账户口令状态

- - `passwd -S <用户账号名>`

- 恢复用户账户口令

- - `passwd -u <用户账号名>`

- 清除用户账户口令

- - `passwd -d <用户账号名>`

**用户相关的命令**：

- `id`：显示用户当前的uid、gid和用户所属的组列表

- `groups`：显示指定用户所属的组列表

- `whoami`：显示当前用户的名称

- `w/who`：显示登录用户及相关信息

- `newgrp`：用于转换用户的当前组到指定的组账号，用户必须属于该组才可以正确执行该命令

  

### 权限管理

![641](E:\DistCode\TyporaLoad\Linux.assets\641.jpg)







```
可以使用 man [命令] 来查看各个命令的使用文档，如 ：man cp。

处理目录的常用命令
    ls: 列出目录及文件名
    cd：切换目录
    pwd：显示目前的目录
    mkdir：创建一个新的目录
    rmdir：删除一个空的目录
    cp: 复制文件或目录
    rm: 移除文件或目录
    mv: 移动文件与目录，或修改文件与目录的名称
```

## 处理目录的常用命令

### ls (列出目录)

选项与参数：
        -a ：全部的文件，连同隐藏文件( 开头为 . 的文件) 一起列出来(常用)
        -d ：仅列出目录本身，而不是列出目录内的文件数据(常用)
        -l ：长数据串列出，包含文件的属性与权限等等数据；(常用)
        将目录下的所有文件列出来(含属性与隐藏档)
        [root@www ~]# ls -al ~      可叠加使用

### cd (切换目录)  

 changedirectory



### pwd (显示目前所在的目录)   

Print Working Directory

选项与参数：
    -P ：显示出确实的路径，而非使用连结 (link) 路径。 



### mkdir (创建新目录)

选项与参数：
    -m ：配置文件的权限喔！直接配置，不需要看默认权限 (umask) 的脸色～
    -p ：帮助你直接将所需要的目录(包含上一级目录)创建起来



###  rmdir (删除空目录)

选项与参数：
    -p ：连同上一级『空的』目录也一起删除 
    

### cp (复制文件或目录)

语法:
    [root@www ~]# cp [-adfilprsu] 来源档(source) 目标档(destination)
    [root@www ~]# cp [options] source1 source2 source3 .... directory
选项与参数：
    -a：相当於 -pdr 的意思，至於 pdr 请参考下列说明；(常用)
    -d：若来源档为连结档的属性(link file)，则复制连结档属性而非文件本身；
     -f：为强制(force)的意思若目标文件已经存在且无法开启，则移除后再尝试一次
     -i：若目标档(destination)已经存在时，在覆盖时会先询问动作的进行(常用)
     -l：进行硬式连结(hard link)的连结档创建，而非复制文件本身；
    -p：连同文件的属性一起复制过去，而非使用默认属性(备份常用)；
     -r：递归持续复制，用於目录的复制行为；(常用)
    -s：复制成为符号连结档 (symbolic link)，亦即『捷径』文件；
    -u：若 destination 比 source 旧才升级 destination ！



### rm (移除文件或目录)

语法：
    rm [-fir] 文件或目录
选项与参数：
    -f ：就是 force 的意思，忽略不存在的文件，不会出现警告信息；
    -i ：互动模式，在删除前会询问使用者是否动作
    -r ：递归删除啊！最常用在目录的删除了！这是非常危险的选项！！！   



### mv (移动文件与目录，或修改名称)

语法：
    [root@www ~]# mv [-fiu] source destination
    [root@www ~]# mv [options] source1 source2 source3 .... directory
选项与参数：
    -f ：force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖；
    -i ：若目标文件 (destination) 已经存在时，就会询问是否覆盖！
    -u ：若目标文件已经存在，且 source 比较新，才会升级 (update)
	mv file dir 如果dir不存在文件将被命名成dir



## 文本操作

* cat test.txt                   //查看文本

  ```
  参数说明：
  -n 或 --number：由 1 开始对所有输出的行数编号。
  -b 或 --number-nonblank：和 -n 相似，只不过对于空白行不编号。
  -s 或 --squeeze-blank：当遇到有连续两行以上的空白行，就代换为一行的空白行。
  -v 或 --show-nonprinting：使用 ^ 和 M- 符号，除了 LFD 和 TAB 之外。
  -E 或 --show-ends : 在每行结束处显示 $。
  -T 或 --show-tabs: 将 TAB 字符显示为 ^I。
  -A, --show-all：等价于 -vET。
  -e：等价于"-vE"选项；
  -t：等价于"-vT"选项；
  
  cat txtfile > newfile //将txtfile内容复制到newfile，newfile不存在将创建
  
  cat -n service.log | grep 13888888888  //查找目标文件中符合条件的数据
  
  tail -f test.txt                //实时跟踪文本变化，可用来查看项目实时日志
  
  vim test.txt                  //编辑文本
  ```

  

  ## 查看进程和端口

  - ps -ef    //简略

  - ps aux  //较详细

    ```
    ps -ef | grep java               //查看名为java的进程
    kill -9 processId                //杀掉指定进程
    netstat -lntup                   //查看当前所有tcp/udp端口信息
    lsof -i:prot                     //查看指定port端口信息
    ```

## 存储

```
free            //查看内存

df              //查看磁盘
```





## vi/vim



## 阿里云linux服务器

dasffdroot Aly(dist)*ddafafessda

已安装jdk，tomcat

已手动开放3306,8080端口

开始学习Linux系统以及vim操作





返回页面的controller的映射名需要和页面名一致



### 工具安装

一般情况下在没有用到高级一点的工具比如redis，zk之类话最简单的只需要安装jdk，tomcat和mysql就能跑一个springboot项目了。

参考链接:https://zhuanlan.zhihu.com/p/88836928?utm_source=wechat_session&utm_medium=social&utm_oi=757958042888212480

下载地址：

jdk：https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html

mysql：https://link.zhihu.com/?target=https%3A//dev.mysql.com/downloads/mysql/5.6.html%23downloads

tomcat：https://link.zhihu.com/?target=https%3A//tomcat.apache.org/download-80.cgi

下载使需要的账户:

```
账号：liwei@xiaostudy.com

密码：OracleTest1234
```

1.安装jdk

下载好之后使用xftp传到云服务器上，选择一个合适的目录开始安装

```
tar -zxvf jdk-8u231-linux-x64.tar.gz
编辑配置文件
vim /etc/profile 
在配置文件后添加下面的内容(我的是直接安装在root目录，所以路径直接写root)
export JAVA_HOME="/root/jdk1.8.0_231"
export PATH="$JAVA_HOME/bin:$PATH"
刷新配置文件
source /etc/profile
查看是否安装成功
java -version
########################################################### ##############
这里会遇到问题，/etc/profile里的配置可能只会在当前终端中生效，如果遇到了这个问题可以把jdk的配置放在~/.bashrc此文件里再执行 source ~/.bashrc
#########################################################################
```



2.安装MySQL

```
下载这个配置文件在linux上sudo打开  https://dev.mysql.com/downloads/repo/apt/
步骤如下
sudo dpkg -i mysql-apt-config_0.8.10-1_all.deb
//安装软件并进行配置，在这里配置我都选择了8.0
sudo apt-get update
//更新软件列表，apt-get update到底做了什么可以参考这篇博客(https://www.cnblogs.com/fenglongyu/p/8654991.html)
sudo apt-get install mysql-server
//设置mysql密码并选择加密方式，这里我选择了第一个
//mysql是一个数据库，mysql-server是一个数据库的服务程序

######################################################################
遇到的问题，在linux虚拟机里配置的是阿里云apt仓库，里面貌似只有5.7版本，使用这个配置文件也不生效，还是只安装5.7的版本，遇到这个问题只能自己去下载完整安装包手动安装
######################################################################

linux下mysql的命令：
service mysql status   //查看mysql状态
service mysql restart  //重启mysql

mysql远程连接配置
如果想myuser使用mypassword从任何主机连接到mysql服务器的话。
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;

FLUSH PRIVILEGES;
```



3.tomcat直接解压启动就行

```
ubuntu下启动tomcat时实时打印日志的命令:  ./startup.sh && tail -f ../logs/catalina.out
```

4.配置完成之后需要登录阿里云控制台打开对应端口