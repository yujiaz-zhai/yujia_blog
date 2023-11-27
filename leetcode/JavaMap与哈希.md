# Map与哈希

### java中的Map使用方法详解：

#### （一）`Map`：

以键值对形式来保存数据   ---key   ---value；键(key)值(value)来保存数据，其中值(value)可以重复，但键(key)必须是唯一，相同就覆盖；也可以为空，但最多只能有一个key为空

`Map`仅仅是一个接口，它的主要实现类有HashMap(去重)、LinkedHashMap、TreeMap(排序)。 指的都是对key 的操作。

`Map`与`Entry`的关系：`Map` 是 Java 集合框架的一部分，它定义了一个对象到对象的映射，意味着它可以存储键值对。`Entry` 是 `Map` 接口的一个内部接口（`Map.Entry`），它表示 `Map` 中的一个单独的键值对。用于实际存储，表示和操作 `Map` 中的这些键值对。在实现 `Map` 接口的类（如 `HashMap`, `TreeMap` 等）中，`Entry` 通常用作存储单位，用于实际存储每个键值对。在 Java 8 及以后的版本中，`HashMap` 使用 `Node` 类来代替之前版本的 `Entry` 类。`Node` 是 `Map.Entry` 接口的一个实现，用于存储每个键值对。

**简要源码：**

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey() { return key; }
    public final V getValue() { return value; }
    public final String toString() { return key + "=" + value; }
    // 其他实现...
}

```

**`Map`通用方法：**

- **`put(key, value)`**: Associates the specified value with the specified key in this map. If the map previously contained a mapping for the key, the old value is replaced.
- **`get(key)`**: Returns the value to which the specified key is mapped, or `null` if this map contains no mapping for the key.
- **`int size()`**: Returns the number of key-value mappings in this map.
- **`remove(key)`**: Removes the mapping for the specified key from this map if present. Returns the previous value associated with key, or null if there was no mapping for key. (A null return can also indicate that the map previously associated null with key.)
- **`containsKey(key)`**: Returns `true` if this map contains a mapping for the specified key. Other, `false`.
- **`containsValue(value)`**: Returns `true` if this map maps one or more keys to the specified value.

#### （二）`HashMap`:

​	1、使用`keySet`遍历`HashMap`:

- **`Set<K> keySet()`**: Returns a [`Set`](dfile:///Users/yujia/Library/Application Support/Dash/DocSets/Java_SE8/Java.docset/Contents/Resources/Documents/java/util/Set.html) view of the keys contained in this map. The set is backed by the map, so changes to the map are reflected in the set, and vice-versa. If the map is modified while an iteration over the set is in progress (except through the iterator's own `remove` operation), the results of the iteration are undefined. The set supports element removal, which removes the corresponding mapping from the map, via the `Iterator.remove`, `Set.remove`,`removeAll`, `retainAll`, and `clear` operations. It does not support the `add` or `addAll` operations.

示例代码：

```java
Set<K> keySet = map.keySet();
for(K key: keySet) {
	System.out.println(map.get(key));
}
```

​	2、使用`map.values()`遍历`HashMap`

- **`map.values()`**: Returns a [`Collection`](dfile:///Users/yujia/Library/Application Support/Dash/DocSets/Java_SE8/Java.docset/Contents/Resources/Documents/java/util/Collection.html) view of the values contained in this map. The collection is backed by the map, so changes to the map are reflected in the collection, and vice-versa. If the map is modified while an iteration over the collection is in progress (except through the iterator's own `remove` operation), the results of the iteration are undefined. The collection supports element removal, which removes the corresponding mapping from the map, via the `Iterator.remove`, `Collection.remove`, `removeAll`, `retainAll` and `clear` operations. It does not support the `add` or `addAll` operations.

示例代码：

```java
for(K value: map.values()) {
	System.out.println(value);
}
```

​	3、使用`entrySet`遍历`HashMap`

- **`Set<Map.Entry<K,V>> entrySet()`**: Returns a [`Set`](dfile:///Users/yujia/Library/Application Support/Dash/DocSets/Java_SE8/Java.docset/Contents/Resources/Documents/java/util/Set.html) view of the mappings contained in this map. The set is backed by the map, so changes to the map are reflected in the set, and vice-versa. If the map is modified while an iteration over the set is in progress (except through the iterator's own `remove` operation, or through the `setValue` operation on a map entry returned by the iterator) the results of the iteration are undefined. The set supports element removal, which removes the corresponding mapping from the map, via the `Iterator.remove`, `Set.remove`, `removeAll`, `retainAll` and `clear`operations. It does not support the `add` or `addAll` operations.

示例代码：

```java
Set<Map.Entry<K, V>> entrySet = map.entrySet();
for (Map.Entry<K, V> entry: entrySet) {
	System.out.println(entry.getKey() + "==>" + entry.getValue());
}
```

或者使用`Iterator`:

```java
Iterator<Map.Entry<K, V>> iterator = map.entrySet().iterator();
while(iterator.hasNext()){
	Map.Entry<K, V> next = iterator.next();
	System.out.println(next.getKey() + "==>" + next.getValue());
}
```

​	4、其他常用方法：

- **`getOrDefault(Object key,V defaultValue)`**:Returns the value to which the specified key is mapped, or `defaultValue` if this map contains no mapping for the key.
- **`boolean isEmpty()`**: Returns true if this map contains no key-value mappings.
- **`replace(K, V)`**: Replaces the entry for the specified key only if it is currently mapped to some value.

#### （三）`LinkedHashMap`:

`LinkedHashMap` 是 Java 中 `Map` 接口的一个具体实现，它基于哈希表和链表实现。与 `HashMap` 类似，`LinkedHashMap` 也存储键值对，但它在内部维护了一个双向链表，用以维护插入顺序或者访问顺序。这使得 `LinkedHashMap` 在某些场景下比 `HashMap` 更为合适。以下是 `LinkedHashMap` 的一些关键特性：

​	1、保持插入顺序

默认情况下，`LinkedHashMap` 保留了键值对的插入顺序。当你迭代 `LinkedHashMap` 时，键值对将按照它们被插入到映射中的顺序返回，这是它与 `HashMap` 的主要区别（`HashMap` 不保证任何顺序）。

​	2、可选的访问顺序模式

通过在构造器中指定一个特殊的布尔值（`accessOrder`），`LinkedHashMap` 可以配置成按访问顺序（访问指插入或查询）来维护键值对。在这种模式下，最近读取或写入的元素会被移动到链表的末尾。

​	3、性能

`LinkedHashMap` 的性能在大多数情况下与 `HashMap` 相似。对于插入和查询操作，它提供了恒定时间的性能（在哈希函数良好的情况下）。

​	4、用途

由于 `LinkedHashMap` 可以维护元素的插入或访问顺序，它特别适用于构建基于 LRU (最近最少使用) 缓存策略的缓存系统。

#### （四）`TreeMap`

`TreeMap` 是 Java 集合框架中的一部分，是 `Map` 接口的一个实现。它基于红黑树（一种自平衡的二叉查找树）实现，这使得它在维护键值对的同时能保持键的排序，默认为升序排列。

​	1、实现`TreeMap`降序：

```java
Map<K, V> map = new TreeMap<>(new Comparator<K>() {
	@Override
	public int compare(K o1, K o2) {
		return o2.compareTo(o1);
	}
});
```

​	2、常用特殊方法：

- **`firstKey() & lastKey()`**: Returns the first (lowest) key currently in this map & Returns the last (highest) key currently in this map.(分别获取映射中的最小键和最大键)
- **`higherKey(K key) & lowerKey(K key)`**: Returns the least key strictly greater than the given key, or `null` if there is no such key. & Returns the greatest key strictly less than the given key, or `null` if there is no such key.(分别获取给定键的下一个更高的键和下一个更低的键)
- **`floorKey(K Key) & ceilingKey(K Key)`**:Returns the greatest key less than or equal to the given key, or `null` if there is no such key. & Returns the least key greater than or equal to the given key, or `null` if there is no such key.(分别获取小于等于给定键的最大键和大于等于给定键的最小键)
- **`subMap(K fromKey, K toKey)`**: Returns a view of the portion of this map whose keys range from `fromKey`, inclusive, to `toKey`, exclusive. (If `fromKey`and `toKey` are equal, the returned map is empty.) The returned map is backed by this map, so changes in the returned map are reflected in this map, and vice-versa. The returned map supports all optional map operations that this map supports.(获取一个键范围从 `fromKey` 到 `toKey` 的子映射)
- **`headMap(K toKey) & tailMap(K fromKey)`**: Returns a view of the portion of this map whose keys are strictly less than `toKey`. & Returns a view of the portion of this map whose keys are greater than or equal to `fromKey`.(获取键小于`toKey` 的子映射和获取键大于`fromKey` 的子映射)