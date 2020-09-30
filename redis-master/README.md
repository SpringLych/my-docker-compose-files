# 主从复制模式docker-compose搭建
Redis高可用至主从复制模式

从节点开启主从复制有三种方式
1. 在conf文件中配置`slaveof masterip masterport`
1. redis-server 启动命令后加上 --slaveof
1. Redis 服务器启动后，直接通过客户端执行命令 slaveof 则该 Redis 实例成为从节点
# 配置文件


## conf
master conf


```nginx
port 6381
protected-mode no
daemonize no
appendonly yes
###RDB持久化配置###
# 900s内至少一次写操作则执行bgsave进行RDB持久化
save 900 1 
save 300 10
save 60 10000 
rdbcompression no
appendonly yes
appendfsync everysec
```
slave conf，只需要在master的基础上添加一部分


```nginx
port 6388
protected-mode no
daemonize no
appendonly yes
###RDB持久化配置###
save 900 1 # 900s内至少一次写操作则执行bgsave进行RDB持久化
save 300 10
save 60 10000 
rdbcompression no
appendonly yes 
appendfsync everysec
# master的ip，port
replicaof 127.0.0.1 6381 
# masterauth pass # master密码
# 如果slave无法与master同步，设置成slave不可读，方便监控脚本发现问题
replica-serve-stale-data no 
# 从节点开启主从复制方式1：conf配置 master ip port
# slaveof 127.0.0.1 6381

```


## docker-comopse


```yaml
version: "3"
# 主从复制模式
services:
  redis-master1:
    image: redis
    container_name: redis-master-1
    ports:
      - "6381:6381"
    volumes:
      - C:\worktool\dockerData\redis-master\6381\conf\master-1.conf:/usr/local/etc/redis/redis.conf
      - C:\worktool\dockerData\redis-master\6381\data:/data
    command: redis-server /usr/local/etc/redis/redis.conf
  
  redis-slave-1:
    image: redis
    container_name: redis-slave-1
    ports:
      - "6388:6388"
    volumes:
      - C:\worktool\dockerData\redis-master\6388\conf\slave-1.conf:/usr/local/etc/redis/redis.conf
      - C:\worktool\dockerData\redis-master\6388\data:/data
    # 开启方式2：添加--salveof，从节点开启主从复制
    command: redis-server /usr/local/etc/redis/redis.conf --slaveof redis-master-1 6381
  
  redis-slave-2:
    image: redis
    container_name: redis-slave-2
    ports:
      - "6389:6389"
    volumes:
      - C:\worktool\dockerData\redis-master\6389\conf\slave-2.conf:/usr/local/etc/redis/redis.conf
      - C:\worktool\dockerData\redis-master\6389\data:/data
    # 添加--salveof，从节点开启主从复制
    command: redis-server /usr/local/etc/redis/redis.conf --slaveof redis-master-1 6381

```
# 验证
开启成功后，使用客户端登录主节点，添加一个键值对，之后连接从节点，能够在从节点中看到添加的信息
# 参考

- [Redis 主从复制详细解读](https://learnku.com/articles/36375)
- [使用docker 搭建redis的主从复制](https://cloud.tencent.com/developer/article/1693904)
- [深入剖析Redis系列(一) - Redis入门简介与主从搭建](https://juejin.im/post/6844903661223542792#heading-0)
