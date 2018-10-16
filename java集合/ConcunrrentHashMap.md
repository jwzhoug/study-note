### 注意 

这个类和hashTable一样它不支持null 作为key/value

### 原理

利用 ==`CAS` +` synchronized`== 来保证并发更新的安全  

底层使用==数组+链表+红黑树==来实现  （这一点和hashMap 相同）

### 静态成员变量
* `private static final int MAXIMUM_CAPACITY = 1 << 30; ` hash表的最大容量

* `private static final int DEFAULT_CAPACITY = 16;`初始化时候使用的 默认初始容量
* `private static final int DEFAULT_CONCURRENCY_LEVEL = 16;`初始化时候默认的并发级别
* `private static final float LOAD_FACTOR = 0.75f;` 负载因子
* `static final int TREEIFY_THRESHOLD = 8;` 树化条件之一,作为链表转红黑树的一个阀值（链表长度大于8）
* `static final int MIN_TREEIFY_CAPACITY = 64;` 树化的另一个条件,作为链表转红黑树的一个阀值（要树化需要 当前hash表的数组长度 >=64 ）
* `static final int NCPU = Runtime.getRuntime().availableProcessors();` Number of CPUS, to place bounds on some sizings  这事源码的注释，首先我们知道这个值是获取当前服务器的cpu 数量的，根据注释我们知道这事作为一个边界使用的，作为某些值的边界（我还不是很清楚）

* 通过不同hash 值定义不同的节点类型

  * `static final int MOVED     = -1; // hash for forwarding nodes`

    代表ForwardingNode节点的hash值的定义。它是一个用于连接两个table的节点类。它包含一个nextTable指针，用于指向下一张表。而且这个节点的key value next指针全部为null，它的hash值为-1. 这里面定义的find的方法是从nextTable里进行查询节点，而不是以自身为头节点进行查找。 

  ```java
  static final class ForwardingNode<K,V> extends Node<K,V> {
          final Node<K,V>[] nextTable;
          ForwardingNode(Node<K,V>[] tab) {
              super(MOVED, null, null, null);
              this.nextTable = tab;
          }
  
          Node<K,V> find(int h, Object k) {
              // loop to avoid arbitrarily deep recursion on forwarding nodes
              outer: for (Node<K,V>[] tab = nextTable;;) {
                  Node<K,V> e; int n;
                  if (k == null || tab == null || (n = tab.length) == 0 ||
                      (e = tabAt(tab, (n - 1) & h)) == null)
                      return null;
                  for (;;) {
                      int eh; K ek;
                      if ((eh = e.hash) == h &&
                          ((ek = e.key) == k || (ek != null && k.equals(ek))))
                          return e;
                      if (eh < 0) {
                          if (e instanceof ForwardingNode) {
                              tab = ((ForwardingNode<K,V>)e).nextTable;
                              continue outer;
                          }
                          else
                              return e.find(h, k);
                      }
                      if ((e = e.next) == null)
                          return null;
                  }
              }
          }
      }
  ```

  

  * `static final int TREEBIN   = -2; // hash for roots of trees`

    这个代表的是这是一个树的根节点

  * `static final int RESERVED  = -3; // hash for transient reservations`

    代表位置保持器节点

* `static final int HASH_BITS = 0x7fffffff; // 用于辅助计算key 的hash值（内部hash算法的辅助变量）`

* 这些是不知道作用的字段

```java
/**
 * The largest possible (non-power of two) array size.
 * Needed by toArray and related methods.
 */
static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;


/**
 * The bin count threshold for untreeifying a (split) bin during a
 * resize operation. Should be less than TREEIFY_THRESHOLD, and at
 * most 6 to mesh with shrinkage detection under removal.
 */
static final int UNTREEIFY_THRESHOLD = 6;

/**
 * Minimum number of rebinnings per transfer step. Ranges are
 * subdivided to allow multiple resizer threads.  This value
 * serves as a lower bound to avoid resizers encountering
 * excessive memory contention.  The value should be at least
 * DEFAULT_CAPACITY.
 */
private static final int MIN_TRANSFER_STRIDE = 16;

/**
 * The number of bits used for generation stamp in sizeCtl.
 * Must be at least 6 for 32bit arrays.
 */
private static int RESIZE_STAMP_BITS = 16;

/**
 * The maximum number of threads that can help resize.
 * Must fit in 32 - RESIZE_STAMP_BITS bits.
 */
private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;

/**
 * The bit shift for recording size stamp in sizeCtl.
 * 用来记录 标记（?）的位移的bit位
 */
private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;


```
### 重要成员变量

* `transient volatile Node<K,V>[] table ` : 和hashMap 盛装Node元素的数组一样，默认为null，初始化发生在第一次插入操作，默认大小为16的数组，它的大小是2的整数次幂 扩容时大小总是2的幂次方 这些地方都和hashMap 是一样的

* `private transient volatile Node<K,V>[] nextTable`：默认为null，扩容时新生成的数组，其大小为原数组的两倍。 他的作用是可以在并发时根据判断，让别的线程辅助正在扩容的线程进行扩容

* `private transient volatile int sizeCt` ：是一个控制标识符，在不同的地方有不同用途，而且它的取值不同，也代表不同的含义。;
  * 负数代表正在进行初始化或扩容操作
  * -1代表正在初始化
  * -N 表示有N-1个线程正在进行扩容操作
  * 正数或0代表hash表还没有被初始化，这个数值表示初始化或下一次是否需要扩容的扩容阀值（后面可以看到，它的值始终是当前ConcurrentHashMap容量的0.75倍，刚好就是扩容阀值）

* `private transient volatile long baseCount;`基本计数器，用于没有线程竞争时候记录节点总量

* `private transient KeySetView<K,V> keySet;` 记录key 的set集合容器

* `private transient ValuesView<K,V> values;`记录所有value的集合容器

* 暂时不知道作用的

  ```java
  /**
   * The next table index (plus one) to split while resizing.
   */
  private transient volatile int transferIndex;
  
  /**
   * Spinlock (locked via CAS) used when resizing and/or creating CounterCells.
   */
  private transient volatile int cellsBusy;
  
  /**
   * Table of counter cells. When non-null, size is a power of 2.
   */
  private transient volatile CounterCell[] counterCells;
  
  ```

  

### 重要成员类

* **Node**：保存key，value及key的hash值的数据结构。 和hashMap 基本差不多，但是不同之处在于下面变量，其中value和next都用volatile修饰，保证并发的可见性。这是为了提供并发时候的可见性。它不允许调用setValue方法直接改变Node的value域，增加了find方法辅助map.get()方法。 

  ```java
  static class Node<K,V> implements Map.Entry<K,V> {
          final int hash;
          final K key;
          volatile V val;
          volatile Node<K,V> next;
  
          Node(int hash, K key, V val, Node<K,V> next) {
              this.hash = hash;
              this.key = key;
              this.val = val;
              this.next = next;
          }
  
          public final K getKey()       { return key; }
          public final V getValue()     { return val; }
          public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
          public final String toString(){ return key + "=" + val; }
          public final V setValue(V value) {
              throw new UnsupportedOperationException();
          }
  
          public final boolean equals(Object o) {
              Object k, v, u; Map.Entry<?,?> e;
              return ((o instanceof Map.Entry) &&
                      (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&
                      (v = e.getValue()) != null &&
                      (k == key || k.equals(key)) &&
                      (v == (u = val) || v.equals(u)));
          }
  
          /**
           * Virtualized support for map.get(); overridden in subclasses.
           */
          Node<K,V> find(int h, Object k) {
              Node<K,V> e = this;
              if (k != null) {
                  do {
                      K ek;
                      if (e.hash == h &&
                          ((ek = e.key) == k || (ek != null && k.equals(ek))))
                          return e;
                  } while ((e = e.next) != null);
              }
              return null;
          }
      }
  ```

* ####  **TreeNode** 

  树节点类，另外一个核心的数据结构。当链表长度过长的时候，会转换为TreeNode。但是与HashMap不相同的是，它并不是直接转换为红黑树，而是把这些结点包装成TreeNode放在TreeBin对象中，由TreeBin完成对红黑树的包装。而且TreeNode在ConcurrentHashMap集成自Node类，而并非HashMap中的集成自LinkedHashMap.Entry<K,V>类，也就是说TreeNode带有next指针，这样做的目的是方便基于TreeBin的访问。 

  ```java
  static final class TreeNode<K,V> extends Node<K,V> {
      TreeNode<K,V> parent;  // red-black tree links
      TreeNode<K,V> left;
      TreeNode<K,V> right;
      TreeNode<K,V> prev;    // needed to unlink next upon deletion
      boolean red;
  
      TreeNode(int hash, K key, V val, Node<K,V> next,
               TreeNode<K,V> parent) {
          super(hash, key, val, next);
          this.parent = parent;
      }
  
      Node<K,V> find(int h, Object k) {
          return findTreeNode(h, k, null);
      }
  
      /**
       * Returns the TreeNode (or null if not found) for the given key
       * starting at given root.
       */
      final TreeNode<K,V> findTreeNode(int h, Object k, Class<?> kc) {
          if (k != null) {
              TreeNode<K,V> p = this;
              do  {
                  int ph, dir; K pk; TreeNode<K,V> q;
                  TreeNode<K,V> pl = p.left, pr = p.right;
                  if ((ph = p.hash) > h)
                      p = pl;
                  else if (ph < h)
                      p = pr;
                  else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
                      return p;
                  else if (pl == null)
                      p = pr;
                  else if (pr == null)
                      p = pl;
                  else if ((kc != null ||
                            (kc = comparableClassFor(k)) != null) &&
                           (dir = compareComparables(kc, k, pk)) != 0)
                      p = (dir < 0) ? pl : pr;
                  else if ((q = pr.findTreeNode(h, k, kc)) != null)
                      return q;
                  else
                      p = pl;
              } while (p != null);
          }
          return null;
      }
  }
  ```
* **TreeBin**

这个类并不负责包装用户的key、value信息，而是包装的很多TreeNode节点。它代替了TreeNode的根节点，也就是说在实际的ConcurrentHashMap“数组”中，存放的是TreeBin对象，而不是TreeNode对象，这是与HashMap的区别。另外这个类还带有了读写锁。和juc 中的读写锁相似，读锁是共享实现，写锁是独占实现

```java
static final class TreeBin<K,V> extends Node<K,V> {
        TreeNode<K,V> root;
        volatile TreeNode<K,V> first;
        volatile Thread waiter;
        volatile int lockState;
        // values for lockState
        static final int WRITER = 1; // set while holding write lock 持有写锁的一个状态 
        static final int WAITER = 2; // set when waiting for write lock 等待获取写锁的状态
        static final int READER = 4; // increment value for setting read lock 共享式读锁的状态增量
        /**
         * Creates bin with initial set of nodes headed by b.
         */
        TreeBin(TreeNode<K,V> b) {
            super(TREEBIN, null, null, null);
            this.first = b;
            TreeNode<K,V> r = null;
            for (TreeNode<K,V> x = b, next; x != null; x = next) {
                next = (TreeNode<K,V>)x.next;
                x.left = x.right = null;
                if (r == null) {
                    x.parent = null;
                    x.red = false;
                    r = x;
                }
                else {
                    K k = x.key;
                    int h = x.hash;
                    Class<?> kc = null;
                    for (TreeNode<K,V> p = r;;) {
                        int dir, ph;
                        K pk = p.key;
                        if ((ph = p.hash) > h)
                            dir = -1;
                        else if (ph < h)
                            dir = 1;
                        else if ((kc == null &&
                                  (kc = comparableClassFor(k)) == null) ||
                                 (dir = compareComparables(kc, k, pk)) == 0)
                            dir = tieBreakOrder(k, pk);
                            TreeNode<K,V> xp = p;
                        if ((p = (dir <= 0) ? p.left : p.right) == null) {
                            x.parent = xp;
                            if (dir <= 0)
                                xp.left = x;
                            else
                                xp.right = x;
                            r = balanceInsertion(r, x);
                            break;
                        }
                    }
                }
            }
            this.root = r;
            assert checkInvariants(root);
        }

        /**
         * Acquires write lock for tree restructuring.
         */
        private final void lockRoot() {
            if (!U.compareAndSwapInt(this, LOCKSTATE, 0, WRITER))
                contendedLock(); // offload to separate method
        }

        /**
         * Releases write lock for tree restructuring.
         */
        private final void unlockRoot() {
            lockState = 0;
        }

        /**
         * Possibly blocks awaiting root lock.
         */
        private final void contendedLock() {
            boolean waiting = false;
            for (int s;;) {
                if (((s = lockState) & ~WAITER) == 0) {
                    if (U.compareAndSwapInt(this, LOCKSTATE, s, WRITER)) {
                        if (waiting)
                            waiter = null;
                        return;
                    }
                }
                else if ((s & WAITER) == 0) {
                    if (U.compareAndSwapInt(this, LOCKSTATE, s, s | WAITER)) {
                        waiting = true;
                        waiter = Thread.currentThread();
                    }
                }
                else if (waiting)
                    LockSupport.park(this);
            }
        }
```



这里仅贴出它的构造方法。可以看到在构造TreeBin节点时，仅仅指定了它的hash值为TREEBIN常量，这也就是个标识为树的根节点。同时也看到我们熟悉的红黑树构造方法

### 实例初始化
实例化ConcurrentHashMap时倘若声明了table的容量，在初始化时会根据参数调整table大小，==确保table的大小总是2的幂次方==。默认的table大小为16. 
上面介绍成员变量也介绍过，table 这个容器的初始化是延迟到了第一次put 的时候执行
![1535687738304](E:\文档\study-note\ConcunrrentHashMap.assets\1535687738304.png)
```java
private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; // table 变量的方法内临时变量
    	int sc;// sizeCtl 变量的方法内临时变量
        while ((tab = table) == null || tab.length == 0) {
            // 如果sizeCtl<0 代表别的线程正在进行初始化或扩容操作，这一点我在介绍成员变量时候有做介绍
            // 既然有别的线程在初始化 那就应该让出时间片
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // lost initialization race; just spin
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                //调用U.compareAndSwapInt(this, SIZECTL, sc, -1) 修改 sizeCtl 状态成功
                try {
                    if ((tab = table) == null || tab.length == 0) {
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;// 不知道这里为什么是这个值 可以去看上面的成员变量里面对 sizeCtl 的说明
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
                        sc = n - (n >>> 2);// 0.75*capacity 这个值搞好就是  是否扩容的阀值
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

### Put解析

#### 定位索引

```jaava
int index = (n - 1) & hash  // n为bucket的个数
```

####  获取table对应的索引元素

```java
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
        return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
    }123
```

采用`Unsafe.getObjectVolatie()`来获取，而不是直接用`table[index]`的原因跟ConcurrentHashMap的弱一致性有关。在java内存模型中，我们已经知道每个线程都有一个工作内存，里面存储着table的副本，虽然table是volatile修饰的，但不能保证线程每次都拿到table中的最新元素，Unsafe.getObjectVolatile可以直接获取指定内存的数据，保证了每次拿到数据都是最新的。

#### put方法

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());// 和hashMap 一样的
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();// 初始化
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {// 如果hash表数组部分这个位置为空
            // 使用cas修改这个值
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)// 当前Map在扩容，先协助扩容，在更新值。
            tab = helpTransfer(tab, f);
        else {// 如果发生hash 冲突 走下面的逻辑
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {//判断是否是链表头节点
                    if (fh >= 0) {// 是链表的节点（头）
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                // 如果key相同
                                oldVal = e.val;// 作用是返回旧值
                                if (!onlyIfAbsent)
                                    e.val = value;//替换掉旧值
                                break;
                            }
                            Node<K,V> pred = e;//记录前一个节点
                            if ((e = e.next) == null) {// 判断是否需要做插入
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {// 红黑树的逻辑
                        Node<K,V> p;
                        binCount = 2;
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
                    treeifyBin(tab, i);// 树化的逻辑	
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);//修改数量值 扩容检查的逻辑也在里面
    return null;
}
```

#### treefyBin方法

 如果新增节点之后，所在的链表的元素个数大于等于8，则会调用`treeifyBin`把链表转换为红黑树。在转换结构时，若tab的长度小于`MIN_TREEIFY_CAPACITY`，默认值为64，则会触发`transfer`将数组长度扩大到原来的两倍，重新调整节点位置。（只有当`tab.length >= 64, ConcurrentHashMap`才会使用红黑树。） 

```java
private final void treeifyBin(Node<K,V>[] tab, int index) {
    Node<K,V> b; int n, sc;
    if (tab != null) {
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)//若tab的长度小于MIN_TREEIFY_CAPACITY，默认值为64，则会触发transfer将数组长度扩大到原来的两倍
            tryPresize(n << 1);
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            synchronized (b) {
                if (tabAt(tab, index) == b) {
                    TreeNode<K,V> hd = null, tl = null;
                    //把所有Node节点包装（组装成链）成TreeNode放进去 下面的 TreeBin对象中
                    for (Node<K,V> e = b; e != null; e = e.next) {
                        TreeNode<K,V> p =
                            new TreeNode<K,V>(e.hash, e.key, e.val,
                                              null, null);
                        if ((p.prev = tl) == null)
                            hd = p;
                        else
                            tl.next = p;
                        tl = p;
                    }
                    //构造了一个TreeBin对象   
                    setTabAt(tab, index, new TreeBin<K,V>(hd));//转换红黑树的逻辑在TreeBin的构造方法中
                }
            }
        }
    }
}
```

 

#### addCount方法

新增节点后，`addCount`统计tab中的节点个数大于阈值（sizeCtl），会触发`transfer`，重新调整节点位置。 

如果这时候有别的线程在做扩容操作，这里会去做一个辅助扩容

```java
private final void addCount(long x, int check) {
        CounterCell[] as; long b, s;
        if ((as = counterCells) != null ||
            !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
            CounterCell a; long v; int m;
            boolean uncontended = true;
            if (as == null || (m = as.length - 1) < 0 ||
                (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                !(uncontended =
                  U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
                fullAddCount(x, uncontended);
                return;
            }
            if (check <= 1)
                return;
            s = sumCount();
        }
        if (check >= 0) {
            Node<K,V>[] tab, nt; int n, sc;
            // tab中的节点个数大于阈值（sizeCtl），会触发transfer，重新调整节点位置。
            while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                   (n = tab.length) < MAXIMUM_CAPACITY) {
                int rs = resizeStamp(n);
                if (sc < 0) {// 如果有别的线程在扩容或则初始化
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)// // 其他线程在初始化，break；
                        break;
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))// 其他线程正在扩容，协助扩容
                        transfer(tab, nt);
                }
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);// 仅当前线程在扩容
                s = sumCount();
            }
        }
    }
```

#### transfer方法

当table的元素数量达到容量阈值sizeCtl，需要对table进行扩容：  

- 构建一个nextTable，大小为table两倍  
- 把table的数据复制到nextTable中。  

在扩容过程中，依然支持并发更新操作；也支持并发插入。  

```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride;
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range
        if (nextTab == null) {            // initiating
            try {
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];  // 构建一个nextTable，大小为table两倍
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            transferIndex = n;
        }
        int nextn = nextTab.length;
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        boolean advance = true;
        boolean finishing = false; // to ensure sweep before committing nextTab
        //通过for自循环处理每个槽位中的链表元素，默认advace为真，通过CAS设置transferIndex属性值，并初始化i和bound值，i指当前处理的槽位序号，bound指需要处理的槽位边界，先处理槽位15的节点；
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            while (advance) { // 遍历table中的每一个节点 
                int nextIndex, nextBound;
                if (--i >= bound || finishing)
                    advance = false;
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                if (finishing) {//如果所有的节点都已经完成复制工作  就把nextTable赋值给table 清空临时对象nextTable 
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);  //扩容阈值设置为原来容量的1.5倍  依然相当于现在容量的0.75倍
                    return;
                }
                // 利用CAS方法更新这个扩容阈值，在这里面sizectl值减一，说明新加入一个线程参与到扩容操作
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            //如果遍历到的节点为空 则放入ForwardingNode指针 
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            //如果遍历到ForwardingNode节点  说明这个点已经被处理过了 直接跳过  这里是控制并发扩容的核心 
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            else {
                synchronized (f) {
                    if (tabAt(tab, i) == f) {  
                        Node<K,V> ln, hn;
                        if (fh >= 0) {  // 链表节点
                            int runBit = fh & n;  // resize后的元素要么在原地，要么移动n位（n为原capacity），详解见：https://huanglei.rocks/coding/194.html#4%20resize()%E7%9A%84%E5%AE%9E%E7%8E%B0
                            Node<K,V> lastRun = f;
                            //以下的部分在完成的工作是构造两个链表  一个是原链表  另一个是原链表的反序排列
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            //在nextTable的i位置上插入一个链表 
                            setTabAt(nextTab, i, ln);
                            //在nextTable的i+n的位置上插入另一个链表
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            //设置advance为true 返回到上面的while循环中 就可以执行i--操作

                            advance = true;
                        }
                        //对TreeBin对象进行处理  与上面的过程类似 
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            //构造正序和反序两个链表 
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            // （1）如果lo链表的元素个数小于等于UNTREEIFY_THRESHOLD，默认为6，则通过untreeify方法把树节点链表转化成普通节点链表；（2）否则判断hi链表中的元素个数是否等于0：如果等于0，表示lo链表中包含了所有原始节点，则设置原始红黑树给ln，否则根据lo链表重新构造红黑树。
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd); // tab[i]已经处理完了
                            advance = true;
                        }
                    }
                }
            }
        }
    }
```

