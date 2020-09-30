注意：如果要是用自定义的Redis.conf，拉取最新版Redis镜像后仅仅在volumes中指定配置是不会读取配置的，一定要有command命令，其中路径表示Redis容器中的路径。

Redis.conf内容在[Redis-redis.conf](https://github.com/redis/redis/blob/unstable/redis.conf)