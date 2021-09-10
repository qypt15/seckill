# seckill 秒杀系统
## 环境
* php5.6 + phpredis扩展
* redis服务
* apache2
* mysql：table 商品表(goods) + 订单表（order）

## 工作流程
1. 基于goods表中的库存，创建redis商品库存队列
2. 客户端访问秒杀API
3. **先从redis的商品库存队列中查询剩余库存**
4. redis队列中有剩余，则在mysql中创建订单，去库存，抢购成功
5. redis队列中没有剩余，则提示库存不足，抢购失败


seckill 秒杀系统

环境：
1. php5.6 + phpredis扩展
2. redis服务
3. apache2
4. mysql：table 商品表(goods) + 订单表（order）

实现功能：
1. 基于redis队列，防止高并发的超卖
2. 基于mysql的事务加排它锁，防止高并发的超卖

基于redis队列工作流程：
1. 管理员根据goods表中的库存，创建redis商品库存队列
2. 客户端访问秒杀API
3. web服务器先从redis的商品库存队列中查询剩余库存
4. redis队列中有剩余，则在mysql中创建订单，去库存，抢购成功
5. redis队列中没有剩余，则提示库存不足，抢购失败

基于mysql事务和排它锁工作流程：
1：开启事务
2：查询库存，并显示的设置写锁（排他锁）：SELECT * FROM goods WHERE id = 1 FOR UPDATE
3：生成订单
4：去库存，隐示的设置写锁（排他锁）：UPDATE goods SET counts = counts – 1 WHERE id = 1
5：commit，释放锁
注意：第二步不可以设置共享锁，不然有可能会造成死锁。

压测工具：
apache自带ab测试工具 ./ab -n1000 -c100 http://host/buy_mysql.php
apache自带ab测试工具 ./ab -n1000 -c100 http://host/buy_redis.php
apache自带ab测试工具 ./ab -n1000 -c100 http://host/buy_transaction.php

如果您想下载到本地运行：
1. 修改 ./Seckill/Model/Model.php的mysql数据库链接信息
2. 修改 ./Seckill/Redis/QRedis.php的redis数据库链接信息
3. 访问：http://host/index.php
4. 访问规则：http://host/index.php?app=app&c=order&a=orderList&gid=1
