# ZAB
ZAB 协议既不是强一致性，也不是弱一致性，而是处于两者之间的单调一致性（顺序一致性）。它依靠事务 ID 和版本号，保证了数据的更新和读取是有序的

zk的写是同步到master，master会把写请求同步给follower，但是有一个过半机制。一半的follower提交commit成功，master就提交了。
假设此时客户端访问的是另一半的某个节点，就是拿到的还是历史数据。。zk只保证顺序一致性吧我记得。。

也是最终一致性，保证已提交的最终能传播到所有节点。

非事务请求是不会同步到master，读操作没有必要，写操作才算事务请求，感觉就像redis单线程执行写一样，保证那个事务id单调递增。
然后zab 选举的时候 就能用这个 zxid 做选举的最高因素  来保证提交出去的事务能最终传播到所有节点。