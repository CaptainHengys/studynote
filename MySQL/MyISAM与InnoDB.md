#### MyISAM与InnoDB

- **MyISAM与InnoDB区别**

  - 在5.5之前,MyISAM是默认的数据库引擎,后来改成了InnoDB

    虽然MyISAM提供了大量特性,包括全文索引,空间函数等,但MyISAM不支持事务和行级锁,而且最大的缺陷是**崩溃后无法安全恢复**

    大多数时候我们使用的都是 InnoDB 存储引擎，但是在某些情况下使用 MyISAM 也是合适的比如读密集的情况下。（如果你不介意 MyISAM 崩溃恢复问题的话）。

- **为什么MYISAM读效率高,InnoDB写效率高?**

  - https://www.zhihu.com/question/22146769
  - 读:MYISAM默认把索引读入内存(索引和数据分开),直接在内存中操作;InonoDB则是IO操作
  - 写:MyISAM,表级锁,InnoDB是行锁
  - **不存在这么简单的对比,要不然直接多读的表用MYISAM,多写的表用InnoDB好了**
    - 如果缓存数据在内存中,速度不会有大的差异,此时获取数据的速度更决定于物理硬件的限制；
    - 如果访问并发高,表锁的表现一般比行锁差,因为要处理并发冲突

- **MYISAM和InnoDB的对比**

  - 是否支持行级锁:MYISAM只有表级锁,而InnoDB支持行级锁和表级锁,默认为行级锁
  - 是否支持外键:MYISAM不支持,INNODB支持
  - 是否支持MVCC:MYISAM不支持,InnoDB支持,,应对高并发事务,MVCC比单纯的加锁更高效
  - 是否支持事务和崩溃后的安全回复:MyISAM强调性能,每次查询具有原子性,他的执行速度比InnoDB更快(其实并不快,尤其是InnoDB用到了聚簇索引),但是不提供事务支持,但是InnoDB提供事务支持,外部键等高级数据库功能,具有事务(commit),回滚(rollback)和崩溃修复能力,和事务安全型表?