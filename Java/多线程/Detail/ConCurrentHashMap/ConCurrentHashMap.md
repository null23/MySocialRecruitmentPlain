### 

### get

    为什么get操作不需要加锁？
        1.因为CHM的静态内部类Node，其中的成员变量val和next是由volatile来修饰的。如果线程B修改了Node元素的值val，以及Node元素的next指针，是可以立即对线程A可见的。

    对保存CHM节点(链表头节点/红黑树根节点)的数组加上volatile修饰有用么？
    transient volatile Node<K,V>[] table; 就是这个东西，保存了链表头节点和红黑树根节点
        1.不管用，在这个前边加volatile修饰，是来保证扩容的size的可见性的，并不可以保证里边元素的可见性
        