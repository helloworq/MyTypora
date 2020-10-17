# Oracle数据库学习

## oracle转pg

### 参考链接

ora2pg官网文档链接:http://ora2pg.darold.net/documentation.html#Oracle-schema-to-export

参考博客链接:https://www.cnblogs.com/lottu/p/9114959.html

基本的操作如上述文档，博客所示，但是里面用到了一个命令：make，Windows下使用这个命令需要安装这个工具;MinGW

安装参考链接：https://www.cnblogs.com/TonyJia/p/13212110.html

如官网所言所有使用到的perl模块工具都可以在CPAN(http://search.cpan.org/) 上找到，工具需要的模块有：perl，DBD :: Oracle，DBI，ora2pg(https://sourceforge.net/projects/ora2pg/)，oracle客户端(需配置oracle环境变量)

### 安装Perl,安装MinGW,安装Oracle

### 安装ora2pg

```
===========Windows========
cd至解压目录
perl Makefile.PL
dmake && dmake install    ||     make && make install
```

### 安装DBD-Oracle

```
===========Windows========
cd至解压目录,确保已配置oracle环境变量
perl Makefile.PL
dmake && dmake install    ||     make && make install
```

### 安装DBI

```
===========Windows========
cd至解压目录
perl Makefile.PL
dmake && dmake install    ||     make && make install
```

### 导出数据

```
--> cd到ora2pg的解压目录
--> 编辑conf配置文件配置需要转换的数据库链接
--> 其中的TYPE字段控制操作的类型
--> 执行命令 ora2pg -c ora2pg_table.conf
--> 目录下将生成对应的sql文件

TYPE可选操作有(摘自官网):

        - TABLE: Extract all tables with indexes, primary keys, unique keys,
          foreign keys and check constraints.
        - VIEW: Extract only views.
        - GRANT: Extract roles converted to Pg groups, users and grants on all
          objects.
        - SEQUENCE: Extract all sequence and their last position.
        - TABLESPACE: Extract storage spaces for tables and indexes (Pg >= v8).
        - TRIGGER: Extract triggers defined following actions.
        - FUNCTION: Extract functions.
        - PROCEDURE: Extract procedures.
        - PACKAGE: Extract packages and package bodies.
        - INSERT: Extract data as INSERT statement.
        - COPY: Extract data as COPY statement.
        - PARTITION: Extract range and list Oracle partitions with subpartitions.
        - TYPE: Extract user defined Oracle type.
        - FDW: Export Oracle tables as foreign table for oracle_fdw.
        - MVIEW: Export materialized view.
        - QUERY: Try to automatically convert Oracle SQL queries.
        - KETTLE: Generate XML ktr template files to be used by Kettle.
        - DBLINK: Generate oracle foreign data wrapper server to use as dblink.
        - SYNONYM: Export Oracle's synonyms as views on other schema's objects.
        - DIRECTORY: Export Oracle's directories as external_file extension objects.
        - LOAD: Dispatch a list of queries over multiple PostgreSQl connections.
        - TEST: perform a diff between Oracle and PostgreSQL database.
        - TEST_VIEW: perform a count on both side of rows returned by views
```



### 迁移完之后遇到的问题。

pg对大小写敏感，所以实体类的字段必须全小写，驼峰命名不允许，例如xxId在数据库中会被识别成XX_id，造成自增加字段以及无法查询到数据的问题

数据导出的时候最好是分的细一点，比如表结构和数据可以分别导出来，视图和函数，存储过程也分别导出来。

视图里的部分函数导出来的时候可能未转换成pg的函数，此时需重新导出或者手动更换。

从oracle转成pg的数据导入测试库之后，如果要迁移到正式数据库最好还是使用原始的导出数据，直接使用测试数据库里转好的数据再导出很可能有很多未知的问题，亲测在Navicat和Datagrip里原生的导出数据功能很多问题。

本文档依据记忆写成不一定完全正确，遇到问题可参考官网以及博客。