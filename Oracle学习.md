# Oracle数据库学习

## oracle转pg(ora2pg工具使用)

参考链接:https://www.cnblogs.com/lottu/p/9114959.html

基本的操作入上述博客所示，但是里面用到了一个命令：make，Windows下使用这个命令需要安装这个工具;MinGW

安装链接：https://www.cnblogs.com/TonyJia/p/13212110.html

![091612071297_0Oracle迁移至PostgreSQL工具之Ora2Pg-lottu-博客园_1](E:\DistCode\TyporaLoad\Oracle学习.assets\091612071297_0Oracle迁移至PostgreSQL工具之Ora2Pg-lottu-博客园_1.Jpeg)





![091612071297_0Oracle迁移至PostgreSQL工具之Ora2Pg-lottu-博客园_2](E:\DistCode\TyporaLoad\Oracle学习.assets\091612071297_0Oracle迁移至PostgreSQL工具之Ora2Pg-lottu-博客园_2.Jpeg)





![091612071297_0Oracle迁移至PostgreSQL工具之Ora2Pg-lottu-博客园_3](E:\DistCode\TyporaLoad\Oracle学习.assets\091612071297_0Oracle迁移至PostgreSQL工具之Ora2Pg-lottu-博客园_3.Jpeg)





![091612071297_0Oracle迁移至PostgreSQL工具之Ora2Pg-lottu-博客园_4](E:\DistCode\TyporaLoad\Oracle学习.assets\091612071297_0Oracle迁移至PostgreSQL工具之Ora2Pg-lottu-博客园_4.Jpeg)







![091612071297_0Oracle迁移至PostgreSQL工具之Ora2Pg-lottu-博客园_5](E:\DistCode\TyporaLoad\Oracle学习.assets\091612071297_0Oracle迁移至PostgreSQL工具之Ora2Pg-lottu-博客园_5.Jpeg)





![091612071297_0Oracle迁移至PostgreSQL工具之Ora2Pg-lottu-博客园_6](E:\DistCode\TyporaLoad\Oracle学习.assets\091612071297_0Oracle迁移至PostgreSQL工具之Ora2Pg-lottu-博客园_6.Jpeg)

迁移完之后遇到的问题。

pg对大小写敏感，所以实体类的字段必须全小写，驼峰命名不允许，例如xxId在数据库中会被识别成XX_id，造成自增加字段以及无法查询到数据的问题