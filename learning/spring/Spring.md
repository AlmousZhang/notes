## Spring IOC

## Spring AOP

## Spring事务

### @Transactional

#### 事务的四大特性(ACID)
1. Atomicity(原子性)：原子性确认动作要么全部完成，要么完全不起作用
2. Consisteny(一致性)：事务在完成是，所有的数据都必须保持一致状态
3. Isolation(隔离性)： 并发的事务之间无影响，在一个事务内部的操作不会对其他事务产生影响，通过事务的隔离级别来指定事务的隔离性
4. Durability(持久性)：一旦事务完成，数据库的改变必须是持久化的

#### 隔离性
1. Read uncommitted 脏读：一个事务读取到了另外一个事务未提交的数据
2. Read committed 不可重复读:一个事务两次读同一行数据，两次读取的数据不一样（对应的是修改）
3. Repeatable read 幻读：一个事务的两次查询，第二次查询比第一次查询多了一个数据行
4. Serializable 序列化

	实现细节：[事务的隔离级别](https://juejin.im/post/5c9b1b7df265da60e21c0b57)

