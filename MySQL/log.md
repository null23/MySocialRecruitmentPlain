# redo buffer
   存储于内存

# redo log ： 保证事务的持久性
   物理日志，可以恢复事务提交修改的页操作，记录的是页的物理修改操作
   存储于磁盘，是持久的
   
   Force Log At Commit
    在事务提交前，必须先持久化redo log到磁盘
    
   1.当客户端执行每条SQL（更新语句）时，redo log会被首先写入log buffer；
   2.当客户端执行COMMIT命令时，log buffer中的内容会被视情况刷新到磁盘。
   3.在事务提交前，只要将Redo Log持久化即可，不需要将数据持久化。
   4.当系统崩溃时，虽然数据没有持久化，但是Redo Log已经持久化。
   5.当Mysql失效重启进行恢复时重新执行redo log记录的SQL进行数据恢复。
   6.redo log在磁盘上作为一个独立的文件存在，即Innodb的log文件。

# undo buffer

# undo log ：MVCC 以及 事务回滚（事务的原子性）
   逻辑日志，记录的是每一行的逻辑操作
   
   为了满足事务的原子性，在操作任何数据之前，首先将数据备份到Undo Log.然后再进行数据的修改。
   如果出现了错误或者用户执行了ROLLBACK语句，系统可以利用Undo Log中的备份将数据恢复到事务开始之前的状态。
   undo log用于数据的回滚操作。   
   具体操作是 copy事务前的数据行到undo buffer，在适合的时间(事务提交前)把undo buffer中的内容刷新到磁盘。
   undo log记录了数据变更历史如，删除前备份 insert 插入undo log中原行记录再在删除行记录上打删除标记，
   修改前备份 insert插入undo log中原行记录再修改更新行数据打标事务id&回滚指针（指向undo log中原行记录）。
   通过undo log可以实现事务回滚，并且可以根据undo log回溯到某个特定的版本的数据，实现MVCC。  
   与redo log不同的是，磁盘上不存在单独的undo log文件，所有的undo log均存放在主ibd数据文件中（表空间）。

# bin log