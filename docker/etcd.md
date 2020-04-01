etcd是一个开源的、分布式的键值对数据存储系统，因为etcd支持对key的版本记录和txn操作和client对key的watch，所以一般用做共享配置、服务的注册和发现。







简单从以下几个方面说一下redis为啥在微服务中不能取代 etcd：
1、redis 没有版本的概念，历史版本数据在大规模微服务中非常有必要，对于状态回滚和故障排查，甚至定锅都很重要
2、redis 的注册和发现目前只能通过 pub 和 sub 来实现，这两个命令完全不能满足生产环境的要求，具体原因可以 gg 或看源码实现
3、etcd 在 2.+版本时，watch 到数据官方文档均建议再 get 一次，因为会存在数据延迟，3.+版本不再需要，可想 redis 的 pub 和 sub 能否达到此种低延迟的要求
4、楼主看到的微服务架构应该都是将 etcd 直接暴露给 client 和 server 的，etcd 的性能摆在那，能够承受多少的 c/s 直连呢，更好的做法应该是对 etcd 做一层保护，当然这种做法会损失一些功能
5、redis 和 etcd 的集群实现方案是不一致的，etcd 采用的是 raft 协议，一主多从，只能写主，底层采用 boltdb 作为 k/v 存储，直接落盘
6、redis 的持久化方案有 aof 和 rdb，这两种方案在宕机的时候都或多或少的会丢失数据

总结，redis从来没有想过抢 etcd 在服务注册和发现的饭碗，目前的架构来说也抢不动，在缓存方面目前在性能和功能也无出其右； etcd 只关注在服务注册与发现方面，非要当做 k/v 存储来用（丢弃 watch 特性而言）也可以用，性能也不错，但只能说你选错对象了
