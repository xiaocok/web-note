

# lua脚本性能优化

## EVAL、EVALSHA命令
Redis从2.6.0版本开始提供了eval命令，通过内置的Lua解释器，可以让用户执行一段Lua脚本并返回数据。因为Redis单线程模型的特点，可以保证多个命令的原子性(因为最近的项目才想到用Lua)，详细的使用方法请移步官方文档。

## 脚本性能
1. Redis保证了脚本执行的原子性，所以在当前脚本没执行完之前，别的命令和脚本都是等待状态，所以一定要控制好脚本中的内容，防止出现需要消耗大量时间的内容(逻辑相对简单)。

## 带宽优化
1. 为了避免每次执行都重复的将Lua脚本内容发送，Redis提供了evalsha命令，只需要将Lua脚本内容的SHA1校验和发送即可(evalsha 6b1bf486c81ceb7edf3c093f4c48582e38c0e791 0)。
2. Lua脚本中的变量(动态数据)请使用KEYS和ARGV获取，如果把变量放在脚本中，必然会导致每次的脚本内容都不同(SHA1)，Redis缓存大量无用或者一次性的脚本内容。


## Redis Lua Scripts的好处
第一，减少了网络的RTT消耗。

第二，原子化的封装，在redis里lua脚本的执行触发原子的，不可被中断的。

不可被中断？ 如果发生阻塞命令怎么办？ redis lua的wiki里写明了是不允许调用那些堵塞方法的，比如sub订阅，brpop这类的。

## Redis+Lua的使用注意事项

1. Redis 的操作为什么是的原子性的?
    1. 因为redis是单线程的！Redis的API是原子性的操作

2. Redis + Lua 形式为什么是原子性的？
    1. Redis从2.6.0版本开始提供了eval命令，通过内置的Lua解释器，可以让用户执行一段Lua脚本并返回数据。因为Redis单线程模型的特点，可以保证多个命令的原子性;  Redis的API是原子性的操作 eval是redis的一个Api

3. Redis集群+Lua 有什么要注意的地方
    1. Redis cluster对多key操作有限，要求命令中所有的key都属于一个slot，才可以被执行
    2. 如何将key放到同一个slot中呢：  
    3. 你需要将把key中的一部分使用{}包起来，redis将通过{}中间的内容作为计算slot的key，类似key1{mykey}、key2{mykey}这样的都会存放到同一个slot中




* [EVAL、EVALSHA命令](https://blog.csdn.net/weixin_34043301/article/details/88731765)
* [对比redis lua和modules模块的性能损耗](http://xiaorui.cc/archives/5377)
* [Redis+Lua的使用注意事项](https://shirenchuang.blog.csdn.net/article/details/88660886)
