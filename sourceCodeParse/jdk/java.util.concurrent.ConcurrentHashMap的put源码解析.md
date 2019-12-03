## put方法源码解析
### put方法入口

```java
    /* 
     *put方法入口
     */
    public V put(K key, V value) {
        return putVal(key, value, false);
    }
```
### 具体实现putVal方法
```java
/** 具体pubVal方法实现，如果已存在，put方法则直接替换，返回老值 */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // key 或 value 为 null 则抛出空指针异常
    if (key == null || value == null) throw new NullPointerException();

    // 根据key计算hash值：高低16为异或，结果和0x7fffffff位与，详细看小结:hashCode方法
    int hash = spread(key.hashCode());

    // 树化当前值，当达到8,，则树化，当小于6，则退化为链表
    int binCount = 0;

    // 遍历table,保证可见性，字段声明为volatile
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;

        // tab为空 或 tab长度为0,则初始化表
        if (tab == null || (n = tab.length) == 0)
            // 详细看小结：initTable方法
            tab = initTable();

            // 对应位置节点目前为null，则cas将值放到节点位置
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                            new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        // hash是MOVED(正在移动)，立刻协助并发移动
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);

            // 都不满足上述特殊条件，则进入正常流程
        else {
            V oldVal = null;
            // 头结点加锁（偏向锁-->轻量锁-->重量级锁）
            synchronized (f) {
                // 若等于头结点，则说明已经加入
                if (tabAt(tab, i) == f) {
                    // hash值大于0
                    if (fh >= 0) {
                        binCount = 1;
                        // 遍历结点
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;

                            // hash值相等，key相等，覆盖value值，终止循环
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                    (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }


                            Node<K,V> pred = e;
                            // 若下个结点为null,则将当前key,value赋值给下一个结点，终止循环
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                            value, null);
                                break;
                            }
                        }
                    }
                    // 树化结构，新增结点到红黑树中
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        // 红黑树：新增结点
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                        value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    // 到达树化阈值8，进行树化
                    treeifyBin(tab, i);

                    // 老值不为空，则直接返回老值
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}

```
### hashCode方法

```C++
//Object获取hashCode
intptr_t ObjectSynchronizer::FastHashCode (Thread * Self, oop obj) {
  // 省略部分源码... 
  ObjectMonitor* monitor = NULL;
  markOop temp, test;
  intptr_t hash;
  markOop mark = ReadStableMark (obj);
  // object should remain ineligible for biased locking
  assert (!mark->has_bias_pattern(), "invariant") ;

    // 1. 无锁
  if (mark->is_neutral()) {
    hash = mark->hash();              // this is a normal header
    if (hash) {                       // if it has hash, just return it
      return hash;
    }
    hash = get_next_hash(Self, obj);  // allocate a new hash code
    temp = mark->copy_set_hash(hash); // merge the hash code into header
    // use (machine word version) atomic operation to install the hash
    test = (markOop) Atomic::cmpxchg_ptr(temp, obj->mark_addr(), mark);
    if (test == mark) {
      return hash;
    }
    // 2. 拥有锁
  } else if (mark->has_monitor()) {
    monitor = mark->monitor();
    temp = monitor->header();
    assert (temp->is_neutral(), "invariant") ;
    hash = temp->hash();
    if (hash) {
      return hash;
    }
    // 3. 自己的锁
  } else if (Self->is_lock_owned((address)mark->locker())) {
    temp = mark->displaced_mark_helper(); // this is a lightweight monitor owned
    assert (temp->is_neutral(), "invariant") ;
    hash = temp->hash();              // by current thread, check if the displaced
    if (hash) {                       // header contains hash code
      return hash;
    }
  }

    // 4. 膨胀
  monitor = ObjectSynchronizer::inflate(Self, obj);
  mark = monitor->header();
  assert (mark->is_neutral(), "invariant") ;
  hash = mark->hash();
  if (hash == 0) {
    hash = get_next_hash(Self, obj);
    temp = mark->copy_set_hash(hash); // merge hash code into header
    assert (temp->is_neutral(), "invariant") ;
    test = (markOop) Atomic::cmpxchg_ptr(temp, monitor, mark);
    if (test != mark) {
      hash = test->hash();
      assert (test->is_neutral(), "invariant") ;
      assert (hash != 0, "Trivial unexpected object/monitor header usage.");
    }
  }
  return hash;
}
```


```java
// key值一般使用String类，String重写了Object的hashCode方法
public int hashCode() {
    // 默认为0
    int h = hash; 
    // 
    if (h == 0 && value.length > 0) {
        char val[] = value;
        // s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
        for (int i = 0; i < value.length; i++) {
            // 为什么选择31作为乘子：https://segmentfault.com/a/1190000010799123
            // 理由一：31 * h = (h << 5) - h
            // 理由二：分布较为均匀，冲突率低（可以拿英文词典验证）
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

### initTable方法
```java
/**
* CAS使用场景，利用volatile的sizeCtl作为互斥手段：如果发现竞争性的初始化，就spin在那里，等待条件恢复； *   否则利用CAS设置排他标志。如果成功则进行初始化；否则重试
*/
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    // 未初始化，则进行初始化
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
            // 底层线程调度策略:sched_yield
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```




