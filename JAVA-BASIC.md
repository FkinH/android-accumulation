## 线程状态及切换

https://www.jianshu.com/p/dbbcceb6bc2a

## HashMap

### Hash冲突

``` 
public V put(K key, V value) {  
    if (key == null)  
        return putForNullKey(value);  
    int hash = hash(key.hashCode());  
    int i = indexFor(hash, table.length);  
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {  
        Object k;  
        //判断当前确定的索引位置是否存在相同hashcode和相同key的元素，如果存在相同的hashcode和相同的key的元素，那么新值覆盖原来的旧值，并返回旧值。  
        //如果存在相同的hashcode，那么他们确定的索引位置就相同，这时判断他们的key是否相同，如果不相同，这时就是产生了hash冲突。  
        //Hash冲突后，那么HashMap的单个bucket里存储的不是一个 Entry，而是一个 Entry 链。  
        //系统只能必须按顺序遍历每个 Entry，直到找到想搜索的 Entry 为止——如果恰好要搜索的 Entry 位于该 Entry 链的最末端（该 Entry 是最早放入该 bucket 中），  
        //那系统必须循环到最后才能找到该元素。  
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {  
            V oldValue = e.value;  
            e.value = value;  
            return oldValue;  
        }  
    }  
    modCount++;  
    addEntry(hash, key, value, i);  
    return null;  
}  
``` 

### Map增长

```
void addEntry(int hash, K key, V value, int bucketIndex) {  
    Entry<K,V> e = table[bucketIndex];  
    table[bucketIndex] = new Entry<K,V>(hash, key, value, e);  
    if (size++ >= threshold)  
        resize(2 * table.length);  
```

### 负载因子
创建 HashMap 时，有一个默认的负载因子（load factor），其默认值为 0.75，
增大负载因子可以减少 Hash 表（就是那个 Entry 数组）所占用的内存空间，但会增加查询数据（get/put）的时间开销;
减小负载因子会提高数据查询的性能，但会增加 Hash 表所占用的内存空间;
