# MySQL高并发时的问题

## 1.&nbsp; MySQL默认最大连接数有限（为100）

解决办法：

1）修改MySQL配置文件，增大最大连接数；

2）使用数据库连接池技术。

编写一个类实现javax.sql.DataSource接口，创建数据库连接池时，使用一个List将Connection添加进其中。
使用时用list.remove()取出；关闭数据库连接池则list.add()重新放回去。

**通常不需要自己实现数据库连接池，可以使用开源包dbcp或者c3p0以及Tomcat自带的Tomcat jdbc pool（这个性能似乎好些）。 **

数据库连接池常用参数：

InitSize：10

MaxSize：20

MinSize：3

## 2.&nbsp; 数据一致性问题

通常在秒杀等场合的处理逻辑是：

先select出该商品的库存量，如果库存量 > 0，则执行update操作。

首先这两个操作必须放在一个事务里，但事务只是数据一致性的必要条件，并非充要条件，必须配合数据库的锁机制完成。

解决办法：

1）先update再select。由于update会自动给行加锁（排它锁），若select出该商品库存为0，则说明update之前库存为1，可以秒杀。commit这个事务；
若select出该商品库存为负数，则说明已经被别的事务秒杀了，rollback。

2）利用数据库的锁机制。

悲观锁：for update排它锁或者lock in share mode都可以。

乐观锁：为表增加version字段

**不推荐在数据库层面加锁，推荐使用缓存机制的锁**
