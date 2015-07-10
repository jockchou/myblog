使用过MySQL的同学，刚开始接触最多的莫过于MyISAM表引擎了，这种引擎的数据库会分别创建三个文件：表结构、表索引、表数据空间。我们可以将某个数据库目录直接迁移到其他数据库也可以正常工作。然而当你使用InnoDB的时候，一切都变了。

InnoDB 默认会将所有的数据库InnoDB引擎的表数据存储在一个共享空间中：ibdata1，这样就感觉不爽，增删数据库的时候，ibdata1文件不会自动收缩，单个数据库的备份也将成为问题。通常只能将数据使用mysqldump 导出，然后再导入解决这个问题。

在MySQL的配置文件[mysqld]部分，增加innodb_file_per_table参数。

可以修改InnoDB为独立表空间模式，每个数据库的每个表都会生成一个数据空间。

独立表空间：

优点：

1． 每个表都有自已独立的表空间。
2． 每个表的数据和索引都会存在自已的表空间中。
3． 可以实现单表在不同的数据库中移动。
4． 空间可以回收（除drop table操作处，表空不能自已回收）

a) Drop table操作自动回收表空间，如果对于统计分析或是日值表，删除大量数据后可以通过:alter table TableName engine=innodb;回缩不用的空间。

b) 对于使innodb-plugin的Innodb使用turncate table也会使空间收缩。

c) 对于使用独立表空间的表，不管怎么删除，表空间的碎片不会太严重的影响性能，而且还有机会处理。

缺点：

单表增加过大，如超过100个G。

结论：

共享表空间在Insert操作上少有优势。其它都没独立表空间表现好。当启用独立表空间时，请合理调整一 下：innodb_open_files 。

InnoDB Hot Backup（冷备）的表空间cp不会面对很多无用的copy了。而且利用innodb hot backup及表空间的管理命令可以实现单现移动。

1.innodb_file_per_table设置.开启方法：
在my.cnf中[mysqld]下设置
innodb_file_per_table=1

2.查看是否开启：
mysql> show variables like ‘%per_table%’;

3.关闭独享表空间
innodb_file_per_table=0关闭独立的表空间
mysql> show variables like ‘%per_table%’