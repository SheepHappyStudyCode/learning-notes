# JDK 源码解读

## ConcurrentHashMap

### 插入元素

插入核心方法：putVal

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // 不允许 null 值和 null 键
    if (key == null || value == null) throw new NullPointerException();
    
    // 计算 key 的哈希值，spread 函数用于增加哈希分散性
    int hash = spread(key.hashCode());
    
    // 记录一个桶中的节点数，用于判断是否需要转成树
    int binCount = 0;
    
    // 无限循环，直到插入成功
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        
        // 如果表未初始化，先初始化表
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
            
        // 如果目标桶为空，直接用 CAS 操作插入新节点
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // tabAt 是原子操作，获取指定位置的节点
            // (n - 1) & hash 计算桶索引
            // CAS 无锁方式尝试插入节点
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
                break;  // 插入成功则退出循环
        }
        
        // 如果检测到正在扩容（MOVED 是一个标记值）
        else if ((fh = f.hash) == MOVED)
            // 协助扩容，帮助其他线程完成扩容操作
            tab = helpTransfer(tab, f);
            
        // 正常情况，桶中已有节点
        else {
            V oldVal = null;
            // 锁定首节点，保证线程安全
            synchronized (f) {
                // 再次检查，确保首节点没有被其他线程修改
                if (tabAt(tab, i) == f) {
                    // fh >= 0 表示是普通链表节点
                    if (fh >= 0) {
                        binCount = 1;
                        // 遍历链表
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            // 如果找到相同的 key
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                // 根据 onlyIfAbsent 决定是否更新值
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            // 到达链表末尾，添加新节点
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    // 如果是红黑树节点
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        // 在树中插入或更新节点
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            
            // 插入完成后的处理
            if (binCount != 0) {
                // 如果链表长度超过阈值，转换为红黑树
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                // 如果是更新操作，返回旧值
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    
    // 增加计数，如果需要会触发扩容
    addCount(1L, binCount);
    return null;  // 插入操作返回 null
}
```

扩容决策方法：addCount

```java
private final void addCount(long x, int check) {
    // as: counterCells数组引用
    // b: 当前baseCount值
    // s: 计算后的新计数值或汇总计数
    CounterCell[] as; long b, s;
    
    // 尝试用两种方式更新计数：
    // 1. 如果counterCells数组已初始化，使用分散计数方式
    // 2. 否则尝试直接CAS更新baseCount，如果失败也转向分散计数
    if ((as = counterCells) != null ||
        !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
        
        // a: 当前线程的CounterCell
        // v: 当前CounterCell的值
        // m: counterCells数组的掩码(长度-1)
        // uncontended: 标记是否没有竞争冲突
        CounterCell a; long v; int m;
        boolean uncontended = true;
        
        // 以下任一条件为true时，进入fullAddCount处理:
        // - counterCells数组为null或空
        // - 当前线程对应的cell为null
        // - 尝试CAS更新cell失败(有竞争)
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
            !(uncontended =
              U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
            // 完整的计数更新逻辑(处理初始化和竞争)
            fullAddCount(x, uncontended);
            return;
        }
        
        // 如果check小于等于1，不需要检查扩容
        // 这是一个优化路径，表示当前桶的冲突很少
        if (check <= 1)
            return;
            
        // 计算map的总元素数量
        s = sumCount();
    }
    
    // 如果check>=0，检查是否需要扩容
    // (check是从putVal传入的binCount值)
    if (check >= 0) {
        // tab: 当前table引用
        // nt: nextTable引用(扩容时的新表)
        // n: 当前表长度
        // sc: sizeCtl的当前值
        Node<K,V>[] tab, nt; int n, sc;
        
        // 扩容条件:
        // 1. 元素数量s超过阈值sizeCtl
        // 2. 表已初始化
        // 3. 表长度小于最大容量
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {
            
            // 根据当前表大小生成扩容标记
            // 这个标记包含表大小的信息，确保扩容操作的正确性
            int rs = resizeStamp(n);
            
            // 如果sizeCtl<0，说明已经有线程在执行扩容
            if (sc < 0) {
                // 检查是否可以参与当前的扩容工作:
                // - 确认是同一轮扩容(通过rs比较)
                // - 确认扩容未完成
                // - 确认nextTable有效
                // - 确认还有工作要做
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;  // 不满足条件则退出循环
                    
                // 尝试增加扩容线程计数
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    // 参与扩容，传入已有的nextTable
                    transfer(tab, nt);
            }
            // 没有线程在扩容，尝试作为第一个线程启动扩容
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                // 启动扩容，此时nextTable为null，会在transfer中创建
                transfer(tab, null);
                
            // 重新计算元素总数，用于下一轮检查
            // 因为扩容后阈值变化，需要重新检查是否还需要继续扩容
            s = sumCount();
        }
    }
}
```

