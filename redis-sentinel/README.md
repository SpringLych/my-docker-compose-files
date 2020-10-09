# 哨兵模式Docker-compose搭建

Redis 集群方案的哨兵模式。使用一主两从三哨兵<br />


| 角色 | ip | 端口 |
| --- | --- | --- |
| 主节点 master | 127.0.0.1 | 6391 |
| 从节点1 redis-sentinel-slave1 | 127.0.0.1 | 6392 |
| 从节点2 redis-sentinel-slave2 | 127.0.0.1 | 6393 |
| 哨兵1 redis-sentinel1 | 127.0.0.1 | 16394 |
| 哨兵2 redis-sentinel2 | 127.0.0.1 | 16395 |
| 哨兵3 redis-sentinel3 | 127.0.0.1 | 16396 |



# 一主两从

主节点 master conf 配置文件
```nginx
port 6391
daemonize no
protected-mode no
timeout 300
databases 16
dbfilename dump-master.db
save 900 1 
save 300 10
save 60 10000 
rdbcompression no
appendonly yes
appendfsync everysec
# masterauth 123456
# requirepass 123456

```
从节点1 slave1 conf配置文件
```nginx
port 6392
daemonize no
protected-mode no
timeout 300
databases 16
dbfilename dump-slave1.db
save 900 1 
save 300 10
save 60 10000 
rdbcompression no
appendonly yes
appendfsync everysec
# masterauth 123456
# requirepass 123456
# 从节点开启复制模式
# slaveof 10.2.103.90 6391

# 对外宣布自己端口和IP，不然哨兵检测不到
replica-announce-ip 10.2.103.90
replica-announce-port  6392
```
从节点2配置文件跟从节点1一样，只有端口号不同。我测试在配置文件中配置slaveof从节点在同步时报错：Error condition on socket for SYNC: Operation now in progress<br />`replica-announce-ip`和 `replica-announce-port` 一定要配置，不然在哨兵中使用 `sentinel slaves mymaster` 检测不到从节点<br />在 docker-compose 配置 slaveof 成功<br />
docker-compse
```yaml
version: "3"
# 哨兵模式-主从
services:
  redis-sentinel-master1:
    image: redis
    container_name: redis-sentinel-master1
    ports:
      - "6391:6391"
    volumes:
      - C:\worktool\dockerData\redis-sentinel\config\master.conf:/usr/local/etc/redis/redis.conf
      - C:\worktool\dockerData\redis-sentinel\data\master:/data
    command: redis-server /usr/local/etc/redis/redis.conf
  
  redis-sentinel-slave1:
    image: redis
    container_name: redis-sentinel-slave1
    ports:
      - "6392:6392"
    volumes:
      - C:\worktool\dockerData\redis-sentinel\config\slave1.conf:/usr/local/etc/redis/redis.conf
      - C:\worktool\dockerData\redis-sentinel\data\slave1:/data
    # 添加--salveof，从节点开启主从复制；添加--replica-announce-ip,对外宣布自己的ip和端口
    # command: redis-server /usr/local/etc/redis/redis.conf --slaveof redis-sentinel-master1 6391 --replica-announce-ip 从节点ip  --replica-announce-port  6392
    command: redis-server /usr/local/etc/redis/redis.conf --slaveof redis-sentinel-master1 6391

  
  redis-sentinel-slave2:
    image: redis
    container_name: redis-sentinel-slave2
    ports:
      - "6393:6393"
    volumes:
      - C:\worktool\dockerData\redis-sentinel\config\slave2.conf:/usr/local/etc/redis/redis.conf
      - C:\worktool\dockerData\redis-sentinel\data\slave2:/data
    # 添加--salveof，从节点开启主从复制；添加--replica-announce-ip,对外宣布自己的ip和端口
    # command: redis-server /usr/local/etc/redis/redis.conf --slaveof redis-sentinel-master1 6391 --replica-announce-ip 从节点ip  --replica-announce-port  6393
    command: redis-server /usr/local/etc/redis/redis.conf  --slaveof redis-sentinel-master1 6391

```


# 哨兵

<br />sentinel配置文件
```nginx
# 哨兵sentinel实例运行的端口，默认26379  
port 16394
# 哨兵sentinel的工作目录
# dir ./

# 哨兵sentinel监控的redis主节点的 
## ip：master ip地址
## port：master端口
## master-name：可以自己命名的主节点名字（只能由字母A-z、数字0-9 、这三个字符".-_"组成。）
## quorum：最小投票数
# sentinel monitor <master-name> <ip> <redis-port> <quorum>  
sentinel monitor mymaster 10.2.103.90 6391 2

# 当在Redis实例中开启了requirepass <foobared>，所有连接Redis实例的客户端都要提供密码。
# sentinel auth-pass <master-name> <password>  
# sentinel auth-pass mymaster 123456  

# 指定主节点应答哨兵sentinel的最大时间间隔，超过这个时间，哨兵主观上认为主节点下线，默认30秒  
# sentinel down-after-milliseconds <master-name> <milliseconds>
sentinel down-after-milliseconds mymaster 30000  

# 指定了在发生failover主备切换时，最多可以有多少个slave同时对新的master进行同步。这个数字越小，完成failover所需的时间就越长；反之，但是如果这个数字越大，就意味着越多的slave因为replication而不可用。可以通过将这个值设为1，来保证每次只有一个slave，处于不能处理命令请求的状态。
# sentinel parallel-syncs <master-name> <numslaves>
sentinel parallel-syncs mymaster 1  

# 故障转移的超时时间failover-timeout，默认三分钟，可以用在以下这些方面：
## 1. 同一个sentinel对同一个master两次failover之间的间隔时间。  
## 2. 当一个slave从一个错误的master那里同步数据时开始，直到slave被纠正为从正确的master那里同步数据时结束。  
## 3. 当想要取消一个正在进行的failover时所需要的时间。
## 4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来同步数据了
# sentinel failover-timeout <master-name> <milliseconds>  
sentinel failover-timeout mymaster 180000

# 当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本。一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。
# 对于脚本的运行结果有以下规则：  
## 1. 若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10。
## 2. 若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。  
## 3. 如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
# sentinel notification-script <master-name> <script-path>  
# sentinel notification-script mymaster /var/redis/notify.sh

# 这个脚本应该是通用的，能被多次调用，不是针对性的。
# sentinel client-reconfig-script <master-name> <script-path>
# sentinel client-reconfig-script mymaster /var/redis/reconfig.sh

sentinel deny-scripts-reconfig yes

```
哨兵2和3配置文件类似，只有端口号不同<br />
<br />docker-compose<br />需要注意，command那里的命令是**redis-sentinel**，这样才能启动哨兵模式。如果是 redis-serve 会报错 redis sentinel directive while not in sentinel mode
```yaml
version: "3"
# 哨兵模式-哨兵
services:
  redis-sentinel1:
    image: redis
    container_name: redis-sentinel1
    ports:
      - "16394:16394"
    volumes:
      - C:\worktool\dockerData\redis-sentinel\config\sentinel1.conf:/usr/local/etc/redis/redis.conf
      - C:\worktool\dockerData\redis-sentinel\data\sentinel1:/data
    command: redis-sentinel /usr/local/etc/redis/redis.conf
  
  redis-sentinel2:
    image: redis
    container_name: redis-sentinel2
    ports:
      - "16395:16395"
    volumes:
      - C:\worktool\dockerData\redis-sentinel\config\sentinel2.conf:/usr/local/etc/redis/redis.conf
      - C:\worktool\dockerData\redis-sentinel\data\sentinel2:/data
    # command: redis-server /usr/local/etc/redis/redis.conf --slaveof redis-master 6391
    command: redis-sentinel /usr/local/etc/redis/redis.conf

  
  redis-sentinel3:
    image: redis
    container_name: redis-sentinel3
    ports:
      - "16396:16396"
    volumes:
      - C:\worktool\dockerData\redis-sentinel\config\sentinel3.conf:/usr/local/etc/redis/redis.conf
      - C:\worktool\dockerData\redis-sentinel\data\sentinel3:/data
    # command: redis-server /usr/local/etc/redis/redis.conf --slaveof redis-master1 6391
    command: redis-sentinel /usr/local/etc/redis/redis.conf
```


# 启动测试与故障恢复
## 启动测试
先进入 `sentinel-master` 启动主从节点，后进入 `sentinel` 启动哨兵，<br />
<br />确认两个 slave 容器已连接
```bash
# 进入容器
docker exec -it redis-sentinel-master1 bash

# 连接master
> redis-cli -c -h 10.2.103.90 -p 6391
# 查看信息
10.2.103.90:6391> info replication
# Replication

```
可以看到两个 slave 连接成功
> # Replication
> role:master
> connected_slaves:2
> slave0:ip=192.168.160.3,port=6392,state=online,offset=1744338,lag=0
> slave1:ip=192.168.160.2,port=6393,state=online,offset=1744338,lag=1


<br />连接到 sentinel 容器，测试与 master 的连接
```bash
redis-cli -c -h 10.2.103.90 -p 16394
# 查看master信息
sentinel master mymaster
```
出现以下信息则连接成功
> 1) "name"
>  2) "mymaster"
>  3) "ip"
>  4) "10.2.103.90"
>  5) "port"
>  6) "6391"
>  7) "runid"
>  8) "bccf748024a11b9944f55c768710341808a98af8"
> 还检测到连接到master的两个slave和另外两个sentinel
> 31) "num-slaves"
> 32) "2"
> 33) "num-other-sentinels"
> 34) "2"


<br />测试与从节点的连接
```bash
sentinel slaves mymaster
```


> 1)  1) "name"
>     2) "10.2.103.90:6393"
>     7) "runid"
>     8) "3409a19d95319382761b353386b1144c3679ceba"
> 2)  1) "name"
>     2) "10.2.103.90:6392"
>     7) "runid"
>     8) "4549b36bc1fb2f4382eb5097e6b00ebcd62bd98d"


<br />以上测试完成说明哨兵模式部署成功。<br />

## 故障恢复
停止原来的master容器，`docker stop redis-sentinel-master1`，然后查看任意sentinel容器的日志 `docker logs redis-sentinel1`：
> 1:X 30 Sep 2020 06:19:42.486 # +sdown master mymaster 10.2.103.90 6391
> 1:X 30 Sep 2020 06:19:42.554 # +new-epoch 1
> 1:X 30 Sep 2020 06:19:42.557 # +vote-for-leader 37ab506d039b398616a60f963e3d29c5929a6812 1

> 1:X 30 Sep 2020 06:19:43.580 # +odown master mymaster 10.2.103.90 6391 #quorum 3/2

> 1:X 30 Sep 2020 06:19:43.580 # Next failover delay: I will not start a failover before Wed Sep 30 06:25:43 2020

> 1:X 30 Sep 2020 06:19:43.806 # +config-update-from sentinel 37ab506d039b398616a60f963e3d29c5929a6812 192.168.32.5 16396 @ mymaster 10.2.103.90 6391

> 1:X 30 Sep 2020 06:19:43.806 # +switch-master mymaster 10.2.103.90 6391 10.2.103.90 6393
> 1:X 30 Sep 2020 06:19:43.806 * +slave slave 10.2.103.90:6392 10.2.103.90 6392 @ mymaster 10.2.103.90 6393

> 1:X 30 Sep 2020 06:19:43.806 * +slave slave 10.2.103.90:6391 10.2.103.90 6391 @ mymaster 10.2.103.90 6393


<br />master 节点进入主观下线状态
> +sdown master mymaster 10.2.103.90 6391

哨兵检测到 master 出现故障，sentinel 进入新纪元，从 0 到 1
> +new-epoch 1

三个节点协商 master 状态，判断是否需要客观下线
> +vote-for-leader 37ab506d039b398616a60f963e3d29c5929a6812 1

超过 `quorum` 个数的 `Sentinel` 节点认为 **主节点** 出现故障,master 进入客观下线状态
> +odown master mymaster 10.2.103.90 6391 #quorum 3/2

`Sentinal` 进行 **自动故障切换**，协商选定端口为 6393 的从节点作为新的 **主节点**
> +switch-master mymaster 10.2.103.90 6391 10.2.103.90 6393

原来的 6391 和 6392 成为 6393 的从节点
> +slave slave 10.2.103.90:6392 10.2.103.90 6392 @ mymaster 10.2.103.90 6393
> +slave slave 10.2.103.90:6391 10.2.103.90 6391 @ mymaster 10.2.103.90 6393

至此，sentinel 检测到故障并自动进行修复。
# 注意事项
由于 Docker 容器的原因，配置文件和编写 compose 文件需要特别注意 IP 地址，最开始在 sentinel 的 conf 文件中配置监视 master 的 ip 是127.0.0.1，创建 sentinel 容器后，配置文件被加载到容器内部，内部的ip 就不是127.0.0.1，因此 sentinel 连接不到 master。redis-cli 连接sentine 后，使用 `sentinel master mymaster` 查看不到 master 信息，runid为空。<br />另一个要注意的就是进入 master 容器后，使用 `redis-cli  -c -h 10.2.103.90 -p 16394`也要注意 IP。<br />
<br />未尝试使用 Redis 客户端连接，听说可能会出现问题
# 参考

- [深入剖析Redis系列(二) - Redis哨兵模式与高可用集群](https://juejin.im/post/6844903663362637832#heading-12)
- [docker-compose搭建redis集群之哨兵模式](https://cloud.tencent.com/developer/article/1678729)
- [docker-compose搭建redis哨兵集群](https://www.cnblogs.com/JulianHuang/p/12650721.html)
