# Java Collections & Streams - Complete Guide

## Table of Contents
1. [Java Collections Framework](#java-collections-framework)
2. [Streams API](#streams-api)
3. [DSA Practical Tricks](#dsa-practical-tricks)
4. [LLD Practical Patterns](#lld-practical-patterns)

---

## Java Collections Framework

### 1. List Interface

#### ArrayList
- **Use Case**: Fast random access, frequent reads
- **Time Complexity**: O(1) get, O(n) add/remove from middle

```java
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(0, 10); // Insert at index
list.get(0); // Random access
list.remove(Integer.valueOf(10)); // Remove by value
list.remove(0); // Remove by index

// DSA Trick: Pre-size for performance
List<Integer> preSized = new ArrayList<>(1000);

// Convert array to list
Integer[] arr = {1, 2, 3};
List<Integer> fromArray = new ArrayList<>(Arrays.asList(arr));
```

#### LinkedList
- **Use Case**: Frequent insertions/deletions at ends, queue/deque operations
- **Time Complexity**: O(n) get, O(1) add/remove at ends

```java
LinkedList<String> linkedList = new LinkedList<>();
linkedList.addFirst("first");
linkedList.addLast("last");
linkedList.removeFirst();
linkedList.removeLast();

// DSA Trick: Use as Queue/Deque
Queue<String> queue = new LinkedList<>();
Deque<String> deque = new LinkedList<>();
```

#### Vector (Thread-Safe but Slower)
```java
Vector<Integer> vector = new Vector<>();
// Better alternative: Collections.synchronizedList()
List<Integer> syncList = Collections.synchronizedList(new ArrayList<>());
```

### 2. Set Interface

#### HashSet
- **Use Case**: Fast lookup, no duplicates, no order
- **Time Complexity**: O(1) average for add/remove/contains

```java
Set<String> hashSet = new HashSet<>();
hashSet.add("apple");
hashSet.add("banana");
hashSet.contains("apple"); // true

// DSA Trick: Remove duplicates from array
int[] nums = {1, 2, 2, 3, 3, 4};
Set<Integer> unique = new HashSet<>();
for (int num : nums) unique.add(num);

// DSA Trick: Pre-size to avoid rehashing
Set<Integer> preSized = new HashSet<>((int)(expectedSize / 0.75 + 1));
```

#### LinkedHashSet
- **Use Case**: Maintains insertion order
- **Time Complexity**: O(1) operations with predictable iteration order

```java
Set<String> linkedHashSet = new LinkedHashSet<>();
linkedHashSet.add("z");
linkedHashSet.add("a");
linkedHashSet.add("m");
// Iteration order: z, a, m

// DSA Trick: LRU Cache key tracking
```

#### TreeSet
- **Use Case**: Sorted set, range queries
- **Time Complexity**: O(log n) operations

```java
TreeSet<Integer> treeSet = new TreeSet<>();
treeSet.add(5);
treeSet.add(1);
treeSet.add(3);

// DSA Tricks
int first = treeSet.first(); // 1
int last = treeSet.last(); // 5
int ceiling = treeSet.ceiling(2); // 3 (>= 2)
int floor = treeSet.floor(4); // 3 (<= 4)
int higher = treeSet.higher(3); // 5 (> 3)
int lower = treeSet.lower(3); // 1 (< 3)

SortedSet<Integer> headSet = treeSet.headSet(3); // [1]
SortedSet<Integer> tailSet = treeSet.tailSet(3); // [3, 5]
SortedSet<Integer> subSet = treeSet.subSet(1, 5); // [1, 3]

// Custom comparator
TreeSet<String> descending = new TreeSet<>(Collections.reverseOrder());
```

### 3. Queue Interface

#### PriorityQueue (Min-Heap by default)
- **Use Case**: Top K elements, scheduling
- **Time Complexity**: O(log n) add/remove, O(1) peek

```java
// Min Heap
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.offer(5);
minHeap.offer(1);
minHeap.offer(3);
int min = minHeap.peek(); // 1
int removed = minHeap.poll(); // 1

// Max Heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
maxHeap.offer(5);
maxHeap.offer(1);
int max = maxHeap.peek(); // 5

// DSA Trick: Custom comparator for objects
class Task {
    String name;
    int priority;
    Task(String n, int p) { name = n; priority = p; }
}

PriorityQueue<Task> taskQueue = new PriorityQueue<>(
    (a, b) -> Integer.compare(a.priority, b.priority)
);

// DSA Trick: Kth largest element
PriorityQueue<Integer> kLargest = new PriorityQueue<>();
int k = 3;
for (int num : nums) {
    kLargest.offer(num);
    if (kLargest.size() > k) {
        kLargest.poll(); // Remove smallest
    }
}
int kthLargest = kLargest.peek();
```

#### ArrayDeque
- **Use Case**: Stack/Queue operations, faster than Stack/LinkedList
- **Time Complexity**: O(1) operations at both ends

```java
Deque<Integer> deque = new ArrayDeque<>();

// Use as Stack
deque.push(1);
deque.push(2);
int top = deque.pop(); // 2

// Use as Queue
deque.offer(1);
deque.offer(2);
int first = deque.poll(); // 1

// DSA Trick: Sliding window maximum
Deque<Integer> window = new ArrayDeque<>();
```

### 4. Map Interface

#### HashMap
- **Use Case**: Fast key-value lookup
- **Time Complexity**: O(1) average get/put

```java
Map<String, Integer> map = new HashMap<>();
map.put("one", 1);
map.put("two", 2);
int value = map.get("one");
boolean hasKey = map.containsKey("one");
boolean hasValue = map.containsValue(1);

// DSA Tricks
map.putIfAbsent("three", 3);
map.getOrDefault("four", 0);
map.computeIfAbsent("five", k -> k.length());
map.merge("one", 1, Integer::sum); // Increment

// Iterate through map
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}

// DSA Trick: Frequency map
String str = "hello";
Map<Character, Integer> freq = new HashMap<>();
for (char c : str.toCharArray()) {
    freq.put(c, freq.getOrDefault(c, 0) + 1);
}
```

#### LinkedHashMap
- **Use Case**: Maintains insertion/access order
- **LLD Trick**: LRU Cache implementation

```java
// Insertion order
Map<String, Integer> insertionOrder = new LinkedHashMap<>();

// Access order (LRU Cache)
Map<String, Integer> lruCache = new LinkedHashMap<>(16, 0.75f, true) {
    @Override
    protected boolean removeEldestEntry(Map.Entry<String, Integer> eldest) {
        return size() > 100; // Max cache size
    }
};

lruCache.put("a", 1);
lruCache.put("b", 2);
lruCache.get("a"); // "a" becomes most recently used
```

#### TreeMap
- **Use Case**: Sorted map, range queries
- **Time Complexity**: O(log n) operations

```java
TreeMap<Integer, String> treeMap = new TreeMap<>();
treeMap.put(3, "three");
treeMap.put(1, "one");
treeMap.put(2, "two");

// DSA Tricks
Map.Entry<Integer, String> firstEntry = treeMap.firstEntry();
Map.Entry<Integer, String> lastEntry = treeMap.lastEntry();
Integer ceilingKey = treeMap.ceilingKey(2);
Integer floorKey = treeMap.floorKey(2);

SortedMap<Integer, String> headMap = treeMap.headMap(2);
SortedMap<Integer, String> tailMap = treeMap.tailMap(2);
SortedMap<Integer, String> subMap = treeMap.subMap(1, 3);

// DSA Trick: Range sum queries
TreeMap<Integer, Integer> prefixSum = new TreeMap<>();
```

#### ConcurrentHashMap (Thread-Safe)
- **LLD Use Case**: Multi-threaded applications

```java
ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
concurrentMap.put("key", 1);

// Atomic operations
concurrentMap.putIfAbsent("key", 2);
concurrentMap.computeIfPresent("key", (k, v) -> v + 1);
```

---

## Streams API

### Creating Streams

```java
// From Collection
List<Integer> list = Arrays.asList(1, 2, 3);
Stream<Integer> stream1 = list.stream();

// From Array
int[] arr = {1, 2, 3};
IntStream stream2 = Arrays.stream(arr);

// From values
Stream<String> stream3 = Stream.of("a", "b", "c");

// Infinite streams
Stream<Integer> infinite = Stream.iterate(0, n -> n + 1);
Stream<Double> random = Stream.generate(Math::random);

// Range
IntStream range = IntStream.range(0, 10); // 0 to 9
IntStream rangeClosed = IntStream.rangeClosed(1, 10); // 1 to 10
```

### Intermediate Operations (Lazy)

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Filter
List<Integer> evens = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

// Map
List<Integer> squared = numbers.stream()
    .map(n -> n * n)
    .collect(Collectors.toList());

// FlatMap (flatten nested structures)
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4)
);
List<Integer> flattened = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());

// Distinct
List<Integer> unique = Arrays.asList(1, 2, 2, 3, 3)
    .stream()
    .distinct()
    .collect(Collectors.toList());

// Sorted
List<Integer> sorted = numbers.stream()
    .sorted()
    .collect(Collectors.toList());

List<Integer> descending = numbers.stream()
    .sorted(Collections.reverseOrder())
    .collect(Collectors.toList());

// Limit & Skip
List<Integer> first5 = numbers.stream()
    .limit(5)
    .collect(Collectors.toList());

List<Integer> skip5 = numbers.stream()
    .skip(5)
    .collect(Collectors.toList());

// Peek (debugging)
numbers.stream()
    .peek(n -> System.out.println("Before: " + n))
    .map(n -> n * 2)
    .peek(n -> System.out.println("After: " + n))
    .collect(Collectors.toList());
```

### Terminal Operations (Eager)

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Collect
List<Integer> list = numbers.stream().collect(Collectors.toList());
Set<Integer> set = numbers.stream().collect(Collectors.toSet());

// ForEach
numbers.stream().forEach(System.out::println);

// Count
long count = numbers.stream().count();

// Reduce
int sum = numbers.stream().reduce(0, Integer::sum);
int product = numbers.stream().reduce(1, (a, b) -> a * b);
Optional<Integer> max = numbers.stream().reduce(Integer::max);

// Min/Max
Optional<Integer> min = numbers.stream().min(Integer::compareTo);
Optional<Integer> max2 = numbers.stream().max(Integer::compareTo);

// AnyMatch, AllMatch, NoneMatch
boolean hasEven = numbers.stream().anyMatch(n -> n % 2 == 0);
boolean allPositive = numbers.stream().allMatch(n -> n > 0);
boolean noneNegative = numbers.stream().noneMatch(n -> n < 0);

// FindFirst, FindAny
Optional<Integer> first = numbers.stream().findFirst();
Optional<Integer> any = numbers.stream().findAny();

// ToArray
Integer[] array = numbers.stream().toArray(Integer[]::new);
```

### Advanced Collectors

```java
List<String> words = Arrays.asList("apple", "banana", "cherry", "date");

// Joining
String joined = words.stream().collect(Collectors.joining(", "));
// Result: "apple, banana, cherry, date"

// Grouping By
Map<Integer, List<String>> byLength = words.stream()
    .collect(Collectors.groupingBy(String::length));
// Result: {5=[apple], 6=[banana, cherry], 4=[date]}

// Partition By
Map<Boolean, List<String>> partitioned = words.stream()
    .collect(Collectors.partitioningBy(w -> w.length() > 5));
// Result: {false=[apple, date], true=[banana, cherry]}

// Counting
Map<Integer, Long> lengthCount = words.stream()
    .collect(Collectors.groupingBy(String::length, Collectors.counting()));
// Result: {5=1, 6=2, 4=1}

// Summing
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5);
int total = nums.stream().collect(Collectors.summingInt(Integer::intValue));

// Averaging
double avg = nums.stream().collect(Collectors.averagingInt(Integer::intValue));

// Statistics
IntSummaryStatistics stats = nums.stream()
    .collect(Collectors.summarizingInt(Integer::intValue));
System.out.println("Max: " + stats.getMax());
System.out.println("Min: " + stats.getMin());
System.out.println("Avg: " + stats.getAverage());

// Mapping
Map<Integer, Set<String>> lengthToWords = words.stream()
    .collect(Collectors.groupingBy(
        String::length,
        Collectors.mapping(String::toUpperCase, Collectors.toSet())
    ));

// ToMap
Map<String, Integer> wordToLength = words.stream()
    .collect(Collectors.toMap(
        w -> w,
        String::length,
        (existing, replacement) -> existing // Handle duplicates
    ));
```

### Parallel Streams

```java
List<Integer> numbers = IntStream.rangeClosed(1, 1000000)
    .boxed()
    .collect(Collectors.toList());

// Parallel processing
long sum = numbers.parallelStream()
    .mapToLong(Integer::longValue)
    .sum();

// Be careful with parallel streams:
// - Overhead for small datasets
// - Thread-safety issues
// - Order may not be preserved
```

---

## DSA Practical Tricks

### 1. Two Pointer Technique

```java
// Remove duplicates from sorted array
public int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    int i = 0;
    for (int j = 1; j < nums.length; j++) {
        if (nums[j] != nums[i]) {
            i++;
            nums[i] = nums[j];
        }
    }
    return i + 1;
}

// Container with most water
public int maxArea(int[] height) {
    int left = 0, right = height.length - 1;
    int maxArea = 0;
    while (left < right) {
        int area = Math.min(height[left], height[right]) * (right - left);
        maxArea = Math.max(maxArea, area);
        if (height[left] < height[right]) {
            left++;
        } else {
            right--;
        }
    }
    return maxArea;
}
```

### 2. Sliding Window

```java
// Maximum sum subarray of size k
public int maxSum(int[] arr, int k) {
    int maxSum = 0, windowSum = 0;

    // First window
    for (int i = 0; i < k; i++) {
        windowSum += arr[i];
    }
    maxSum = windowSum;

    // Slide window
    for (int i = k; i < arr.length; i++) {
        windowSum += arr[i] - arr[i - k];
        maxSum = Math.max(maxSum, windowSum);
    }
    return maxSum;
}

// Longest substring without repeating characters
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> map = new HashMap<>();
    int maxLen = 0, left = 0;

    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (map.containsKey(c)) {
            left = Math.max(left, map.get(c) + 1);
        }
        map.put(c, right);
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

### 3. HashMap Tricks

```java
// Two sum
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[] {map.get(complement), i};
        }
        map.put(nums[i], i);
    }
    return new int[] {};
}

// Subarray sum equals k
public int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> map = new HashMap<>();
    map.put(0, 1);
    int sum = 0, count = 0;

    for (int num : nums) {
        sum += num;
        if (map.containsKey(sum - k)) {
            count += map.get(sum - k);
        }
        map.put(sum, map.getOrDefault(sum, 0) + 1);
    }
    return count;
}
```

### 4. Stack Tricks

```java
// Valid parentheses
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> map = Map.of(')', '(', '}', '{', ']', '[');

    for (char c : s.toCharArray()) {
        if (map.containsKey(c)) {
            if (stack.isEmpty() || stack.pop() != map.get(c)) {
                return false;
            }
        } else {
            stack.push(c);
        }
    }
    return stack.isEmpty();
}

// Daily temperatures (next greater element)
public int[] dailyTemperatures(int[] temperatures) {
    int n = temperatures.length;
    int[] result = new int[n];
    Deque<Integer> stack = new ArrayDeque<>();

    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && temperatures[i] > temperatures[stack.peek()]) {
            int idx = stack.pop();
            result[idx] = i - idx;
        }
        stack.push(i);
    }
    return result;
}
```

### 5. Priority Queue Tricks

```java
// Kth largest element
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    for (int num : nums) {
        minHeap.offer(num);
        if (minHeap.size() > k) {
            minHeap.poll();
        }
    }
    return minHeap.peek();
}

// Merge k sorted lists
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>(
        (a, b) -> Integer.compare(a.val, b.val)
    );

    for (ListNode list : lists) {
        if (list != null) {
            pq.offer(list);
        }
    }

    ListNode dummy = new ListNode(0);
    ListNode current = dummy;

    while (!pq.isEmpty()) {
        ListNode node = pq.poll();
        current.next = node;
        current = current.next;
        if (node.next != null) {
            pq.offer(node.next);
        }
    }
    return dummy.next;
}
```

### 6. TreeSet/TreeMap Tricks

```java
// Contains nearby duplicate within k distance
public boolean containsNearbyDuplicate(int[] nums, int k) {
    Set<Integer> set = new HashSet<>();
    for (int i = 0; i < nums.length; i++) {
        if (set.contains(nums[i])) return true;
        set.add(nums[i]);
        if (set.size() > k) {
            set.remove(nums[i - k]);
        }
    }
    return false;
}

// Ceiling and floor using TreeSet
public int[] findRange(int[] nums, int target) {
    TreeSet<Integer> set = new TreeSet<>();
    for (int num : nums) set.add(num);

    Integer floor = set.floor(target);
    Integer ceiling = set.ceiling(target);

    return new int[] {
        floor != null ? floor : -1,
        ceiling != null ? ceiling : -1
    };
}
```

---

## LLD Practical Patterns

### 1. LRU Cache

```java
class LRUCache {
    private final int capacity;
    private final Map<Integer, Integer> cache;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.cache = new LinkedHashMap<>(capacity, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
                return size() > capacity;
            }
        };
    }

    public int get(int key) {
        return cache.getOrDefault(key, -1);
    }

    public void put(int key, int value) {
        cache.put(key, value);
    }
}
```

### 2. Thread-Safe Counter

```java
class Counter {
    private final Map<String, AtomicInteger> counts = new ConcurrentHashMap<>();

    public void increment(String key) {
        counts.computeIfAbsent(key, k -> new AtomicInteger(0)).incrementAndGet();
    }

    public int get(String key) {
        return counts.getOrDefault(key, new AtomicInteger(0)).get();
    }
}
```

### 3. Event Bus Pattern

```java
class EventBus {
    private final Map<String, List<Consumer<Object>>> listeners = new ConcurrentHashMap<>();

    public void subscribe(String event, Consumer<Object> listener) {
        listeners.computeIfAbsent(event, k -> new CopyOnWriteArrayList<>()).add(listener);
    }

    public void publish(String event, Object data) {
        listeners.getOrDefault(event, Collections.emptyList())
                 .forEach(listener -> listener.accept(data));
    }
}
```

### 4. Object Pool Pattern

```java
class ObjectPool<T> {
    private final Queue<T> pool = new ConcurrentLinkedQueue<>();
    private final Supplier<T> factory;
    private final int maxSize;

    public ObjectPool(Supplier<T> factory, int maxSize) {
        this.factory = factory;
        this.maxSize = maxSize;
    }

    public T acquire() {
        T obj = pool.poll();
        return obj != null ? obj : factory.get();
    }

    public void release(T obj) {
        if (pool.size() < maxSize) {
            pool.offer(obj);
        }
    }
}
```

### 5. Rate Limiter (Sliding Window)

```java
class RateLimiter {
    private final Map<String, Deque<Long>> requestTimestamps = new ConcurrentHashMap<>();
    private final int maxRequests;
    private final long windowMs;

    public RateLimiter(int maxRequests, long windowMs) {
        this.maxRequests = maxRequests;
        this.windowMs = windowMs;
    }

    public boolean allowRequest(String userId) {
        long now = System.currentTimeMillis();
        Deque<Long> timestamps = requestTimestamps.computeIfAbsent(
            userId, k -> new LinkedList<>()
        );

        // Remove old timestamps
        while (!timestamps.isEmpty() && now - timestamps.peekFirst() > windowMs) {
            timestamps.pollFirst();
        }

        if (timestamps.size() < maxRequests) {
            timestamps.offerLast(now);
            return true;
        }
        return false;
    }
}
```

### 6. Task Scheduler with Priority

```java
class TaskScheduler {
    private final PriorityQueue<Task> taskQueue = new PriorityQueue<>(
        Comparator.comparing(Task::getPriority).reversed()
                  .thenComparing(Task::getTimestamp)
    );

    static class Task {
        String id;
        int priority;
        long timestamp;
        Runnable action;

        Task(String id, int priority, Runnable action) {
            this.id = id;
            this.priority = priority;
            this.timestamp = System.currentTimeMillis();
            this.action = action;
        }

        int getPriority() { return priority; }
        long getTimestamp() { return timestamp; }
    }

    public void schedule(String id, int priority, Runnable action) {
        taskQueue.offer(new Task(id, priority, action));
    }

    public void executeBatch(int count) {
        for (int i = 0; i < count && !taskQueue.isEmpty(); i++) {
            Task task = taskQueue.poll();
            task.action.run();
        }
    }
}
```

### 7. Caching with TTL (Time To Live)

```java
class TTLCache<K, V> {
    private final Map<K, CacheEntry<V>> cache = new ConcurrentHashMap<>();
    private final long ttlMs;

    static class CacheEntry<V> {
        V value;
        long expiryTime;

        CacheEntry(V value, long expiryTime) {
            this.value = value;
            this.expiryTime = expiryTime;
        }
    }

    public TTLCache(long ttlMs) {
        this.ttlMs = ttlMs;
    }

    public void put(K key, V value) {
        long expiryTime = System.currentTimeMillis() + ttlMs;
        cache.put(key, new CacheEntry<>(value, expiryTime));
    }

    public V get(K key) {
        CacheEntry<V> entry = cache.get(key);
        if (entry == null) return null;

        if (System.currentTimeMillis() > entry.expiryTime) {
            cache.remove(key);
            return null;
        }
        return entry.value;
    }

    public void cleanup() {
        long now = System.currentTimeMillis();
        cache.entrySet().removeIf(e -> now > e.getValue().expiryTime);
    }
}
```

---

## Performance Tips

### 1. Collection Initialization
```java
// Pre-size collections when you know the size
List<Integer> list = new ArrayList<>(1000);
Map<String, Integer> map = new HashMap<>(1000);
Set<String> set = new HashSet<>((int)(expectedSize / 0.75 + 1));
```

### 2. Avoid Autoboxing in Loops
```java
// Bad - autoboxing overhead
List<Integer> list = new ArrayList<>();
for (int i = 0; i < 1000; i++) {
    list.add(i); // Boxing happens here
}

// Good - use primitive streams
int[] array = IntStream.range(0, 1000).toArray();
```

### 3. Use EnumSet/EnumMap for Enums
```java
enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }

// Much faster than HashSet<Day>
Set<Day> days = EnumSet.of(Day.MON, Day.FRI);

// Much faster than HashMap<Day, String>
Map<Day, String> schedule = new EnumMap<>(Day.class);
```

### 4. StringBuilder for String Concatenation
```java
// Bad
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i; // Creates new String each time
}

// Good
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

### 5. Immutable Collections (Java 9+)
```java
List<String> immutableList = List.of("a", "b", "c");
Set<Integer> immutableSet = Set.of(1, 2, 3);
Map<String, Integer> immutableMap = Map.of("a", 1, "b", 2);

// For larger collections
List<String> copy = List.copyOf(mutableList);
```

---

## Common Patterns Summary

| Pattern | Collection | Use Case |
|---------|-----------|----------|
| Frequency Counter | HashMap | Count occurrences |
| Sliding Window | HashMap/HashSet | Substring problems |
| Two Pointers | Array/List | Sorted array operations |
| Stack | Deque | Parentheses, monotonic stack |
| Queue | LinkedList/ArrayDeque | BFS, level order |
| Priority Queue | PriorityQueue | K-th element, scheduling |
| TreeSet/TreeMap | TreeSet/TreeMap | Range queries, sorted data |
| LRU Cache | LinkedHashMap | Access-order tracking |
| Graph | HashMap<Node, List<Node>> | Adjacency list |

---

This guide covers the most essential collections, streams, and practical patterns used in DSA and LLD. Keep this as a reference for interviews and real-world development!
