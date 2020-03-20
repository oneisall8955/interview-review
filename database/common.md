# 数据库

## 什么是锁？为什么用锁？MySQL锁机制


## MySQL分页查询是用什么？有什么性能问题？Oracle分页查询？
MySQL分页查询：

    select t.* from table t limit n,m;  -- n
    
Oracle分页查询：

    select t.*, ROWNUM from table t where ROWNUM < N

