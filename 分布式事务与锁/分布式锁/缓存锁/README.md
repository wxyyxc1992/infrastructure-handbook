# 基于缓存实现分布式锁

基于缓存实现分布式锁的方式，非常适合解决这种场景下的问题。所谓基于缓存，也就是说 把数据存放在计算机内存中，不需要写入磁盘，减少了 IO 读写。接下来，我以 Redis 为例 与你展开这部分内容。Redis 通常可以使用 setnx(key, value) 函数来实现分布式锁。key 和 value 就是基于缓存 的分布式锁的两个属性，其中 key 表示锁 id，value = currentTime + timeOut，表示当 前时间 + 超时时间。也就是说，某个进程获得 key 这把锁后，如果在 value 的时间内未释 放锁，系统就会主动释放锁。

setnx 函数的返回值有 0 和 1:

- 返回 1，说明该服务器获得锁，setnx 将 key 对应的 value 设置为当前时间 + 锁的有效 时间。
- 返回 0，说明其他服务器已经获得了锁，进程不能进入临界区。该服务器可以不断尝试 setnx 操作，以获得锁。

总结来说,Redis 通过队列来维持进程访问共享资源的先后顺序。Redis 锁主要基于 setnx 函数实现分布式锁,当进程通过 setnx<key,value> 函数返回 1 时,表示已经获得锁。排在后面的进程只能等待前面的进程主动释放锁,或者等到时间超时才能获得锁。相对于基于数据库实现分布式锁的方案来说,基于缓存实现的分布式锁的优势表现在以下几个方面:

- 性能更好。数据被存放在内存,而不是磁盘,避免了频繁的 10 操作。
- 很多缓存可以跨集群部署,避免了单点故障问题。
- 很多缓存服务都提供了可以用来实现分布式锁的方法,比如 Redis 的 setnx 方法等。
- 可以直接设置超时时间来控制锁的释放,因为这些缓存服务器一般支持自动删除过期数据。

这个方案的不足是，通过超时时间来控制锁的失效时间，并不是十分靠谱，因为一个进程执行时间可能比较长，或受系统进程做内存回收等影响，导致时间超时，从而不正确地释放了
锁。为了解决基于缓存实现的分布式锁的这些问题，我们再来看看基于 ZooKeeper 实现的分布式锁吧。

![基于 ZooKeeper 的分布式锁](https://pic.imgdb.cn/item/6058644d8322e6675c8890b1.jpg)

可以看到,使用 ZooKeeper 可以完美解决设计分布式锁时遇到的各种问题,比如单点故障、不可重入、死锁等问题。虽然 ZooKeeper 实现的分布式锁,几乎能涵盖所有分布式锁的特性,且易于实现,但需要频繁地添加和删除节点,所以性能不如基于缓存实现的分布式锁。
