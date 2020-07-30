# Linux

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

## vi/vim