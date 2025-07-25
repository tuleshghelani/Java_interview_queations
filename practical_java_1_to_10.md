# Practical Java Interview Questions

## 1. Implement a Thread-Safe Singleton
Create a thread-safe Singleton class in Java that follows the best practices for lazy initialization. Your implementation should:

1. Ensure that only one instance of the class can exist
2. Be thread-safe without using unnecessary synchronization
3. Handle serialization correctly
4. Prevent creation of additional instances via reflection

Example solution:

```java
public class ThreadSafeSingleton implements Serializable {
    private static final long serialVersionUID = 1L;
    private static volatile ThreadSafeSingleton instance;
    
    // Private constructor to prevent instantiation
    private ThreadSafeSingleton() {
        // Protect against reflection attacks
        if (instance != null) {
            throw new IllegalStateException("Singleton already initialized");
        }
    }
    
    // Double-checked locking pattern
    public static ThreadSafeSingleton getInstance() {
        if (instance == null) {
            synchronized (ThreadSafeSingleton.class) {
                if (instance == null) {
                    instance = new ThreadSafeSingleton();
                }
            }
        }
        return instance;
    }
    
    // Protect against serialization
    protected Object readResolve() {
        return getInstance();
    }
}
```


## 2. Implement a Custom ArrayList
Implement a simplified version of ArrayList that can store integers. Your implementation should include:

1. A constructor that initializes with a default capacity
2. Methods to add elements (with automatic resizing when needed)
3. Methods to get elements by index
4. A method to remove elements by index
5. A method to get the current size

Example solution:

```java
public class CustomArrayList {
    private int[] elements;
    private int size;
    private static final int DEFAULT_CAPACITY = 10;
    
    public CustomArrayList() {
        elements = new int[DEFAULT_CAPACITY];
        size = 0;
    }
    
    public void add(int element) {
        if (size == elements.length) {
            // Resize array if needed
            int[] newElements = new int[elements.length * 2];
            System.arraycopy(elements, 0, newElements, 0, elements.length);
            elements = newElements;
        }
        elements[size++] = element;
    }
    
    public int get(int index) {
        if (index < 0 || index >= size) {
            throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);
        }
        return elements[index];
    }
    
    public void remove(int index) {
        if (index < 0 || index >= size) {
            throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);
        }
        // Shift elements to remove the element at the specified index
        System.arraycopy(elements, index + 1, elements, index, size - index - 1);
        size--;
    }
    
    public int size() {
        return size;
    }
}
```

## 3. Implement a Producer-Consumer Pattern
Implement a producer-consumer pattern using Java's built-in concurrency utilities. Your solution should:

1. Create a bounded buffer shared between producers and consumers
2. Implement producer threads that add items to the buffer
3. Implement consumer threads that remove items from the buffer
4. Ensure thread safety and prevent race conditions
5. Handle the case when the buffer is full (producers wait) or empty (consumers wait)

Example solution:

```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

public class ProducerConsumerExample {
    private static final int BUFFER_SIZE = 5;
    
    public static void main(String[] args) {
        // Shared buffer
        BlockingQueue<Integer> buffer = new LinkedBlockingQueue<>(BUFFER_SIZE);
        
        // Create producer and consumer threads
        Thread producerThread = new Thread(new Producer(buffer));
        Thread consumerThread = new Thread(new Consumer(buffer));
        
        // Start threads
        producerThread.start();
        consumerThread.start();
    }
    
    static class Producer implements Runnable {
        private final BlockingQueue<Integer> buffer;
        
        Producer(BlockingQueue<Integer> buffer) {
            this.buffer = buffer;
        }
        
        @Override
        public void run() {
            try {
                for (int i = 0; i < 10; i++) {
                    System.out.println("Producing: " + i);
                    // put() will block if the buffer is full
                    buffer.put(i);
                    Thread.sleep(100); // Simulate work
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    
    static class Consumer implements Runnable {
        private final BlockingQueue<Integer> buffer;
        
        Consumer(BlockingQueue<Integer> buffer) {
            this.buffer = buffer;
        }
        
        @Override
        public void run() {
            try {
                while (true) {
                    // take() will block if the buffer is empty
                    int value = buffer.take();
                    System.out.println("Consuming: " + value);
                    Thread.sleep(200); // Simulate work
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

## 4. Implement a Binary Search Tree
Implement a Binary Search Tree (BST) in Java with the following operations:

1. Insert a value
2. Search for a value
3. Delete a value
4. Perform in-order traversal
5. Find the minimum and maximum values

Example solution:

```java
public class BinarySearchTree {
    private Node root;
    
    private static class Node {
        int value;
        Node left;
        Node right;
        
        Node(int value) {
            this.value = value;
            this.left = null;
            this.right = null;
        }
    }
    
    public void insert(int value) {
        root = insertRec(root, value);
    }
    
    private Node insertRec(Node root, int value) {
        if (root == null) {
            root = new Node(value);
            return root;
        }
        
        if (value < root.value) {
            root.left = insertRec(root.left, value);
        } else if (value > root.value) {
            root.right = insertRec(root.right, value);
        }
        
        return root;
    }
    
    public boolean search(int value) {
        return searchRec(root, value);
    }
    
    private boolean searchRec(Node root, int value) {
        if (root == null) {
            return false;
        }
        
        if (root.value == value) {
            return true;
        }
        
        if (value < root.value) {
            return searchRec(root.left, value);
        } else {
            return searchRec(root.right, value);
        }
    }
    
    public void delete(int value) {
        root = deleteRec(root, value);
    }
    
    private Node deleteRec(Node root, int value) {
        if (root == null) {
            return null;
        }
        
        if (value < root.value) {
            root.left = deleteRec(root.left, value);
        } else if (value > root.value) {
            root.right = deleteRec(root.right, value);
        } else {
            // Node with only one child or no child
            if (root.left == null) {
                return root.right;
            } else if (root.right == null) {
                return root.left;
            }
            
            // Node with two children
            root.value = minValue(root.right);
            
            // Delete the inorder successor
            root.right = deleteRec(root.right, root.value);
        }
        
        return root;
    }
    
    private int minValue(Node root) {
        int minValue = root.value;
        while (root.left != null) {
            minValue = root.left.value;
            root = root.left;
        }
        return minValue;
    }
    
    public void inOrderTraversal() {
        inOrderRec(root);
        System.out.println();
    }
    
    private void inOrderRec(Node root) {
        if (root != null) {
            inOrderRec(root.left);
            System.out.print(root.value + " ");
            inOrderRec(root.right);
        }
    }
    
    public int findMin() {
        if (root == null) {
            throw new IllegalStateException("Tree is empty");
        }
        
        Node current = root;
        while (current.left != null) {
            current = current.left;
        }
        
        return current.value;
    }
    
    public int findMax() {
        if (root == null) {
            throw new IllegalStateException("Tree is empty");
        }
        
        Node current = root;
        while (current.right != null) {
            current = current.right;
        }
        
        return current.value;
    }
}
```

## 5. Implement a Custom Cache with LRU Eviction Policy
Implement a simple Least Recently Used (LRU) cache in Java with the following features:

1. Fixed capacity
2. O(1) time complexity for get and put operations
3. Automatic eviction of least recently used items when capacity is reached
4. Thread safety (optional bonus)

Example solution:

```java
import java.util.HashMap;
import java.util.Map;

public class LRUCache<K, V> {
    private final int capacity;
    private final Map<K, Node<K, V>> cache;
    private Node<K, V> head;
    private Node<K, V> tail;
    
    private static class Node<K, V> {
        K key;
        V value;
        Node<K, V> prev;
        Node<K, V> next;
        
        Node(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.cache = new HashMap<>();
        this.head = null;
        this.tail = null;
    }
    
    public V get(K key) {
        Node<K, V> node = cache.get(key);
        if (node == null) {
            return null;
        }
        
        // Move accessed node to the front (most recently used)
        moveToFront(node);
        return node.value;
    }
    
    public void put(K key, V value) {
        Node<K, V> existingNode = cache.get(key);
        
        if (existingNode != null) {
            // Update existing node
            existingNode.value = value;
            moveToFront(existingNode);
            return;
        }
        
        // Create new node
        Node<K, V> newNode = new Node<>(key, value);
        
        // Check if cache is at capacity
        if (cache.size() >= capacity) {
            // Remove least recently used item (tail)
            cache.remove(tail.key);
            removeTail();
        }
        
        // Add new node to front
        addToFront(newNode);
        cache.put(key, newNode);
    }
    
    private void moveToFront(Node<K, V> node) {
        if (node == head) {
            return; // Already at front
        }
        
        // Remove from current position
        if (node.prev != null) {
            node.prev.next = node.next;
        }
        
        if (node.next != null) {
            node.next.prev = node.prev;
        }
        
        if (node == tail) {
            tail = node.prev;
        }
        
        // Add to front
        node.next = head;
        node.prev = null;
        
        if (head != null) {
            head.prev = node;
        }
        
        head = node;
        
        if (tail == null) {
            tail = node;
        }
    }
    
    private void addToFront(Node<K, V> node) {
        node.next = head;
        node.prev = null;
        
        if (head != null) {
            head.prev = node;
        }
        
        head = node;
        
        if (tail == null) {
            tail = node;
        }
    }
    
    private void removeTail() {
        if (tail == null) {
            return;
        }
        
        if (tail.prev != null) {
            tail.prev.next = null;
            tail = tail.prev;
        } else {
            // Only one node in the list
            head = null;
            tail = null;
        }
    }
    
    public int size() {
        return cache.size();
    }
}
```

## 6. Comparing Objects and Their Use in HashMap/HashSet

```java
class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && name.equals(person.name);
    }

    @Override
    public int hashCode() {
        return name.hashCode() * 31 + age;
    }

    @Override
    public String toString() {
        return name + ", " + age;
    }
}

public class CompareDemo {
    public static void main(String[] args) {
        Person p1 = new Person("Alice", 30);
        Person p2 = new Person("Alice", 30);
        Person p3 = new Person("Bob", 25);

        // Direct object comparison
        System.out.println("p1.equals(p2): " + p1.equals(p2)); // true
        System.out.println("p1 == p2: " + (p1 == p2)); // false

        // HashSet demonstration
        java.util.HashSet<Person> set = new java.util.HashSet<>();
        set.add(p1);
        set.add(p2);
        set.add(p3);
        System.out.println("HashSet: " + set); // Only two unique persons

        // HashMap demonstration
        java.util.HashMap<Person, String> map = new java.util.HashMap<>();
        map.put(p1, "First");
        map.put(p2, "Second"); // Overwrites value for same logical key
        map.put(p3, "Third");
        System.out.println("HashMap: " + map);
    }
}
```

## 7. Advanced Stream Operations in Java

### 7.1 Understanding the `reduce()` Operation

The `reduce()` operation is a terminal operation that combines stream elements into a single result using a binary operation. It's one of the most powerful yet complex operations in the Stream API.

**Theory:**
- `reduce()` has three forms:
  1. `Optional<T> reduce(BinaryOperator<T> accumulator)`
  2. `T reduce(T identity, BinaryOperator<T> accumulator)`
  3. `<U> U reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator<U> combiner)`
- The accumulator function must be associative to ensure correct results in parallel processing
- The identity element must be an identity for the accumulator function
- The combiner is used in parallel streams to combine results from different threads

**Hard Example:**

```java
import java.util.Arrays;
import java.util.List;
import java.util.Optional;
import java.util.function.BinaryOperator;

public class ReduceExample {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee("Alice", "Engineering", 85000),
            new Employee("Bob", "Engineering", 75000),
            new Employee("Charlie", "Marketing", 65000),
            new Employee("Diana", "Marketing", 70000),
            new Employee("Eve", "Engineering", 90000)
        );
        
        // Challenge 1: Find the highest-paid employee in each department
        // This is complex because we need to maintain a map during reduction
        
        // Solution using the third form of reduce
        var departmentHighestPaid = employees.stream()
            .reduce(
                new java.util.HashMap<String, Employee>(),  // identity: empty map
                (map, emp) -> {  // accumulator
                    map.compute(emp.getDepartment(), (dept, currentHighest) -> 
                        currentHighest == null || emp.getSalary() > currentHighest.getSalary() ? emp : currentHighest
                    );
                    return map;
                },
                (map1, map2) -> {  // combiner for parallel streams
                    map2.forEach((dept, emp) -> 
                        map1.merge(dept, emp, (e1, e2) -> e1.getSalary() > e2.getSalary() ? e1 : e2)
                    );
                    return map1;
                }
            );
        
        System.out.println("Highest paid by department: " + departmentHighestPaid);
        
        // Challenge 2: Calculate factorial using reduce
        // Shows how reduce can replace recursive algorithms
        int n = 5;
        int factorial = java.util.stream.IntStream.rangeClosed(1, n)
            .reduce(1, (a, b) -> a * b);
        System.out.println(n + "! = " + factorial);
        
        // Challenge 3: Custom reduction with complex logic
        // Find the employee with the longest name, or if tied, the highest salary
        Optional<Employee> result = employees.stream()
            .reduce((e1, e2) -> {
                int nameCompare = Integer.compare(e1.getName().length(), e2.getName().length());
                if (nameCompare != 0) return nameCompare > 0 ? e1 : e2;
                return e1.getSalary() > e2.getSalary() ? e1 : e2;
            });
        
        result.ifPresent(e -> System.out.println("Employee with longest name or highest salary: " + e));
    }
    
    static class Employee {
        private final String name;
        private final String department;
        private final double salary;
        
        public Employee(String name, String department, double salary) {
            this.name = name;
            this.department = department;
            this.salary = salary;
        }
        
        public String getName() { return name; }
        public String getDepartment() { return department; }
        public double getSalary() { return salary; }
        
        @Override
        public String toString() {
            return name + " (" + department + ", $" + salary + ")";
        }
    }
}
```

### 7.2 Mastering the `toArray()` Operation

**Theory:**
- `toArray()` is a terminal operation that converts a stream into an array
- Two variants:
  1. `Object[] toArray()` - returns an Object array
  2. `<A> A[] toArray(IntFunction<A[]> generator)` - returns a typed array using the provided generator
- The generator function is crucial for type safety and performance
- For primitive streams, specialized methods like `toArray()` in IntStream return primitive arrays directly

**Hard Example:**

```java
import java.util.Arrays;
import java.util.List;
import java.util.function.IntFunction;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class ToArrayExample {
    public static void main(String[] args) {
        // Challenge 1: Convert a stream of custom objects to a typed array efficiently
        List<DataPoint> dataPoints = Arrays.asList(
            new DataPoint(1, 5.6),
            new DataPoint(2, 7.8),
            new DataPoint(3, 9.1)
        );
        
        // Using method reference for the generator function
        DataPoint[] array1 = dataPoints.stream().toArray(DataPoint[]::new);
        System.out.println("Array using method reference: " + Arrays.toString(array1));
        
        // Using lambda for the generator function
        DataPoint[] array2 = dataPoints.stream().toArray(size -> new DataPoint[size]);
        System.out.println("Array using lambda: " + Arrays.toString(array2));
        
        // Challenge 2: Convert a filtered stream to an array with exact size
        // This demonstrates the importance of the generator function
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // First, count the elements that will pass the filter
        long evenCount = numbers.stream().filter(n -> n % 2 == 0).count();
        
        // Then create a stream again with the same filter and convert to array with exact size
        Integer[] evenArray = numbers.stream()
            .filter(n -> n % 2 == 0)
            .toArray(size -> {
                System.out.println("Generator called with size: " + size);
                // size will be the count of elements that passed the filter
                assert size == evenCount;
                return new Integer[size];
            });
        
        System.out.println("Even numbers array: " + Arrays.toString(evenArray));
        
        // Challenge 3: Working with primitive streams and arrays
        int[] primitiveArray = numbers.stream()
            .mapToInt(Integer::intValue)  // Convert to IntStream
            .filter(n -> n % 2 == 1)      // Keep odd numbers
            .toArray();                   // Direct conversion to int[]
        
        System.out.println("Primitive array of odd numbers: " + Arrays.toString(primitiveArray));
        
        // Challenge 4: Creating a 2D array from a stream of streams
        Stream<Stream<Integer>> streamOfStreams = Stream.of(
            Stream.of(1, 2, 3),
            Stream.of(4, 5),
            Stream.of(6, 7, 8, 9)
        );
        
        // Convert to a 2D array - this is tricky and requires careful handling
        Integer[][] matrix = streamOfStreams
            .map(stream -> stream.toArray(Integer[]::new))
            .toArray(Integer[][]::new);
        
        System.out.println("2D array from stream of streams:");
        for (Integer[] row : matrix) {
            System.out.println(Arrays.toString(row));
        }
    }
    
    static class DataPoint {
        private final int id;
        private final double value;
        
        public DataPoint(int id, double value) {
            this.id = id;
            this.value = value;
        }
        
        @Override
        public String toString() {
            return "Point(" + id + ", " + value + ")";
        }
    }
}
```

### 7.3 Advanced `map()` Operations

**Theory:**
- `map()` is an intermediate operation that transforms each element in a stream using a function
- It's a one-to-one transformation where each input element produces exactly one output element
- Specialized variants exist for primitive streams: `mapToInt()`, `mapToLong()`, `mapToDouble()`
- The mapping function must be stateless and non-interfering for parallel stream processing
- Unlike `flatMap()`, it cannot change the number of elements in the stream

**Hard Example:**

```java
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.function.Function;
import java.util.stream.Collectors;

public class MapExample {
    public static void main(String[] args) {
        List<Transaction> transactions = Arrays.asList(
            new Transaction("T1001", "Alice", 150.0, "PURCHASE"),
            new Transaction("T1002", "Bob", 50.0, "REFUND"),
            new Transaction("T1003", "Charlie", 300.0, "PURCHASE"),
            new Transaction("T1004", "Alice", 200.0, "PURCHASE"),
            new Transaction("T1005", "Bob", 75.0, "PURCHASE")
        );
        
        // Challenge 1: Complex mapping with conditional logic
        // Map transactions to different objects based on type
        List<TransactionSummary> summaries = transactions.stream()
            .map(t -> {
                if ("PURCHASE".equals(t.getType())) {
                    return new PurchaseSummary(t.getId(), t.getCustomer(), t.getAmount(), 0.05 * t.getAmount());
                } else if ("REFUND".equals(t.getType())) {
                    return new RefundSummary(t.getId(), t.getCustomer(), t.getAmount(), "Refund processed");
                } else {
                    return new TransactionSummary(t.getId(), t.getCustomer(), t.getAmount());
                }
            })
            .collect(Collectors.toList());
        
        System.out.println("Transaction summaries:");
        summaries.forEach(System.out::println);
        
        // Challenge 2: Mapping with stateful transformations (generally avoided but shown for educational purposes)
        // Add a sequential number to each transaction by customer
        Map<String, AtomicInteger> customerCounters = new java.util.HashMap<>();
        
        List<String> numberedTransactions = transactions.stream()
            .map(t -> {
                // This is stateful and would be problematic in parallel streams
                AtomicInteger counter = customerCounters.computeIfAbsent(t.getCustomer(), k -> new AtomicInteger(0));
                int num = counter.incrementAndGet();
                return String.format("%s - Transaction #%d: $%.2f", t.getCustomer(), num, t.getAmount());
            })
            .collect(Collectors.toList());
        
        System.out.println("\nNumbered transactions by customer:");
        numberedTransactions.forEach(System.out::println);
        
        // Challenge 3: Mapping with exception handling
        List<String> transactionIds = Arrays.asList("T1001", "T1002", "INVALID", "T1004", "T1005");
        
        // Create a map for quick lookup
        Map<String, Transaction> transactionMap = transactions.stream()
            .collect(Collectors.toMap(Transaction::getId, Function.identity()));
        
        // Map IDs to transactions with exception handling
        List<Transaction> retrievedTransactions = transactionIds.stream()
            .map(id -> {
                try {
                    Transaction t = transactionMap.get(id);
                    if (t == null) throw new IllegalArgumentException("Transaction not found: " + id);
                    return t;
                } catch (Exception e) {
                    System.err.println("Error processing ID " + id + ": " + e.getMessage());
                    return new Transaction(id, "UNKNOWN", 0.0, "ERROR");
                }
            })
            .collect(Collectors.toList());
        
        System.out.println("\nRetrieved transactions with error handling:");
        retrievedTransactions.forEach(System.out::println);
    }
    
    static class Transaction {
        private final String id;
        private final String customer;
        private final double amount;
        private final String type;
        
        public Transaction(String id, String customer, double amount, String type) {
            this.id = id;
            this.customer = customer;
            this.amount = amount;
            this.type = type;
        }
        
        public String getId() { return id; }
        public String getCustomer() { return customer; }
        public double getAmount() { return amount; }
        public String getType() { return type; }
        
        @Override
        public String toString() {
            return "Transaction{" + id + ", " + customer + ", $" + amount + ", " + type + "}";
        }
    }
    
    static class TransactionSummary {
        protected final String id;
        protected final String customer;
        protected final double amount;
        
        public TransactionSummary(String id, String customer, double amount) {
            this.id = id;
            this.customer = customer;
            this.amount = amount;
        }
        
        @Override
        public String toString() {
            return "Summary{" + id + ", " + customer + ", $" + amount + "}";
        }
    }
    
    static class PurchaseSummary extends TransactionSummary {
        private final double rewardPoints;
        
        public PurchaseSummary(String id, String customer, double amount, double rewardPoints) {
            super(id, customer, amount);
            this.rewardPoints = rewardPoints;
        }
        
        @Override
        public String toString() {
            return "Purchase{" + id + ", " + customer + ", $" + amount + ", points: " + rewardPoints + "}";
        }
    }
    
    static class RefundSummary extends TransactionSummary {
        private final String notes;
        
        public RefundSummary(String id, String customer, double amount, String notes) {
            super(id, customer, amount);
            this.notes = notes;
        }
        
        @Override
        public String toString() {
            return "Refund{" + id + ", " + customer + ", $" + amount + ", notes: " + notes + "}";
        }
    }
}
```

### 7.4 Understanding `flatMap()` Operations

**Theory:**
- `flatMap()` is an intermediate operation that transforms each element into a stream and then flattens these streams into a single stream
- It's a one-to-many transformation where each input element can produce zero, one, or many output elements
- Specialized variants exist for primitive streams: `flatMapToInt()`, `flatMapToLong()`, `flatMapToDouble()`
- Particularly useful for working with nested collections or when you need to both transform and flatten
- The key difference from `map()` is that `flatMap()` can change the number of elements in the resulting stream

**Hard Example:**

```java
import java.util.*;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class FlatMapExample {
    public static void main(String[] args) {
        // Challenge 1: Process a complex hierarchical structure
        List<Department> company = Arrays.asList(
            new Department("Engineering", Arrays.asList(
                new Team("Frontend", Arrays.asList(
                    new Employee("Alice", Arrays.asList("JavaScript", "React", "CSS")),
                    new Employee("Bob", Arrays.asList("TypeScript", "Angular"))
                )),
                new Team("Backend", Arrays.asList(
                    new Employee("Charlie", Arrays.asList("Java", "Spring", "Hibernate")),
                    new Employee("Diana", Arrays.asList("Python", "Django", "PostgreSQL"))
                ))
            )),
            new Department("Marketing", Arrays.asList(
                new Team("Digital", Arrays.asList(
                    new Employee("Eve", Arrays.asList("SEO", "Content Strategy", "Analytics")),
                    new Employee("Frank", Arrays.asList("Social Media", "Email Marketing"))
                ))
            ))
        );
        
        // Find all unique skills across the entire company
        Set<String> allSkills = company.stream()
            .flatMap(dept -> dept.getTeams().stream())                // Stream<Team>
            .flatMap(team -> team.getEmployees().stream())            // Stream<Employee>
            .flatMap(emp -> emp.getSkills().stream())                // Stream<String>
            .collect(Collectors.toSet());
        
        System.out.println("All skills in the company: " + allSkills);
        
        // Challenge 2: Find employees who have a specific skill
        String targetSkill = "Java";
        List<String> employeesWithSkill = company.stream()
            .flatMap(dept -> dept.getTeams().stream())
            .flatMap(team -> team.getEmployees().stream())
            .filter(emp -> emp.getSkills().contains(targetSkill))
            .map(Employee::getName)
            .collect(Collectors.toList());
        
        System.out.println("\nEmployees with " + targetSkill + " skill: " + employeesWithSkill);
        
        // Challenge 3: Create a map of skills to employees who have that skill
        Map<String, List<String>> skillToEmployees = company.stream()
            .flatMap(dept -> dept.getTeams().stream())
            .flatMap(team -> team.getEmployees().stream())
            .flatMap(emp -> emp.getSkills().stream()
                .map(skill -> new AbstractMap.SimpleEntry<>(skill, emp.getName())))
            .collect(Collectors.groupingBy(
                Map.Entry::getKey,
                Collectors.mapping(Map.Entry::getValue, Collectors.toList())
            ));
        
        System.out.println("\nSkill to employees mapping:");
        skillToEmployees.forEach((skill, emps) -> 
            System.out.println(skill + ": " + emps));
        
        // Challenge 4: Find departments that have employees with all of a set of required skills
        Set<String> requiredSkills = new HashSet<>(Arrays.asList("Java", "Spring"));
        
        List<String> departmentsWithRequiredSkills = company.stream()
            .filter(dept -> {
                // For each department, collect all skills across all employees
                Set<String> deptSkills = dept.getTeams().stream()
                    .flatMap(team -> team.getEmployees().stream())
                    .flatMap(emp -> emp.getSkills().stream())
                    .collect(Collectors.toSet());
                
                // Check if department has all required skills
                return deptSkills.containsAll(requiredSkills);
            })
            .map(Department::getName)
            .collect(Collectors.toList());
        
        System.out.println("\nDepartments with all required skills " + requiredSkills + ": " + departmentsWithRequiredSkills);
    }
    
    static class Department {
        private final String name;
        private final List<Team> teams;
        
        public Department(String name, List<Team> teams) {
            this.name = name;
            this.teams = teams;
        }
        
        public String getName() { return name; }
        public List<Team> getTeams() { return teams; }
    }
    
    static class Team {
        private final String name;
        private final List<Employee> employees;
        
        public Team(String name, List<Employee> employees) {
            this.name = name;
            this.employees = employees;
        }
        
        public String getName() { return name; }
        public List<Employee> getEmployees() { return employees; }
    }
    
    static class Employee {
        private final String name;
        private final List<String> skills;
        
        public Employee(String name, List<String> skills) {
            this.name = name;
            this.skills = skills;
        }
        
        public String getName() { return name; }
        public List<String> getSkills() { return skills; }
    }
}
```

### 7.5 Mastering `sorted()` Operations

**Theory:**
- `sorted()` is an intermediate operation that sorts the elements of a stream
- Two variants:
  1. `sorted()` - uses natural ordering (elements must implement Comparable)
  2. `sorted(Comparator<? super T> comparator)` - uses the provided comparator
- Sorting is a stateful operation that may require processing the entire stream before producing any results
- For parallel streams, sorting involves merging pre-sorted segments, which can be expensive
- The operation is stable (equal elements maintain their relative order)

**Hard Example:**

```java
import java.time.LocalDate;
import java.util.*;
import java.util.stream.Collectors;

public class SortedExample {
    public static void main(String[] args) {
        List<Project> projects = Arrays.asList(
            new Project("Website Redesign", 3, LocalDate.of(2023, 6, 15), Status.IN_PROGRESS, 75000),
            new Project("Mobile App Development", 5, LocalDate.of(2023, 3, 10), Status.IN_PROGRESS, 120000),
            new Project("Database Migration", 2, LocalDate.of(2023, 8, 1), Status.NOT_STARTED, 45000),
            new Project("Cloud Infrastructure", 4, LocalDate.of(2023, 1, 20), Status.COMPLETED, 95000),
            new Project("Security Audit", 1, LocalDate.of(2023, 7, 5), Status.NOT_STARTED, 30000),
            new Project("AI Integration", 4, LocalDate.of(2023, 4, 12), Status.IN_PROGRESS, 110000),
            new Project("Data Analytics Platform", 3, LocalDate.of(2023, 2, 28), Status.COMPLETED, 85000)
        );
        
        // Challenge 1: Complex multi-level sorting
        // Sort projects by status (NOT_STARTED first, then IN_PROGRESS, then COMPLETED),
        // then by priority (highest first), then by budget (highest first)
        List<Project> sortedProjects = projects.stream()
            .sorted(Comparator
                .comparing(Project::getStatus, (s1, s2) -> {
                    // Custom order for status
                    Map<Status, Integer> statusOrder = Map.of(
                        Status.NOT_STARTED, 1,
                        Status.IN_PROGRESS, 2,
                        Status.COMPLETED, 3
                    );
                    return Integer.compare(statusOrder.get(s1), statusOrder.get(s2));
                })
                .thenComparing(Project::getPriority, Comparator.reverseOrder())
                .thenComparing(Project::getBudget, Comparator.reverseOrder())
            )
            .collect(Collectors.toList());
        
        System.out.println("Projects sorted by status, priority, and budget:");
        sortedProjects.forEach(System.out::println);
        
        // Challenge 2: Sorting with a dynamic comparator based on user preference
        String sortField = "startDate"; // This could come from user input
        boolean ascending = false;      // This could come from user input
        
        Comparator<Project> dynamicComparator;
        switch (sortField) {
            case "name":
                dynamicComparator = Comparator.comparing(Project::getName);
                break;
            case "priority":
                dynamicComparator = Comparator.comparing(Project::getPriority);
                break;
            case "startDate":
                dynamicComparator = Comparator.comparing(Project::getStartDate);
                break;
            case "status":
                dynamicComparator = Comparator.comparing(Project::getStatus);
                break;
            case "budget":
                dynamicComparator = Comparator.comparing(Project::getBudget);
                break;
            default:
                dynamicComparator = Comparator.comparing(Project::getName);
        }
        
        if (!ascending) {
            dynamicComparator = dynamicComparator.reversed();
        }
        
        List<Project> dynamicallySorted = projects.stream()
            .sorted(dynamicComparator)
            .collect(Collectors.toList());
        
        System.out.println("\nProjects sorted by " + sortField + " in " + 
                          (ascending ? "ascending" : "descending") + " order:");
        dynamicallySorted.forEach(System.out::println);
        
        // Challenge 3: Sorting with null values
        List<Project> projectsWithNulls = new ArrayList<>(projects);
        projectsWithNulls.add(new Project("Legacy System Maintenance", null, LocalDate.of(2023, 5, 15), Status.IN_PROGRESS, 60000));
        projectsWithNulls.add(new Project("Documentation", 2, null, Status.NOT_STARTED, 25000));
        
        // Sort handling null values (nulls last)
        List<Project> sortedWithNulls = projectsWithNulls.stream()
            .sorted(Comparator
                .comparing(Project::getPriority, Comparator.nullsLast(Comparator.naturalOrder()))
                .thenComparing(Project::getStartDate, Comparator.nullsLast(Comparator.naturalOrder()))
            )
            .collect(Collectors.toList());
        
        System.out.println("\nProjects sorted with null handling:");
        sortedWithNulls.forEach(System.out::println);
    }
    
    enum Status { NOT_STARTED, IN_PROGRESS, COMPLETED }
    
    static class Project {
        private final String name;
        private final Integer priority; // 1 (lowest) to 5 (highest), can be null
        private final LocalDate startDate; // can be null
        private final Status status;
        private final double budget;
        
        public Project(String name, Integer priority, LocalDate startDate, Status status, double budget) {
            this.name = name;
            this.priority = priority;
            this.startDate = startDate;
            this.status = status;
            this.budget = budget;
        }
        
        public String getName() { return name; }
        public Integer getPriority() { return priority; }
        public LocalDate getStartDate() { return startDate; }
        public Status getStatus() { return status; }
        public double getBudget() { return budget; }
        
        @Override
        public String toString() {
            return String.format("Project{%s, priority=%s, startDate=%s, status=%s, budget=$%.2f}",
                               name, priority, startDate, status, budget);
        }
    }
}
```

### 7.6 Advanced `joining()` Operations

**Theory:**
- `joining()` is a collector that concatenates the elements of a stream into a single String
- Three variants:
  1. `joining()` - simple concatenation
  2. `joining(CharSequence delimiter)` - concatenation with a delimiter
  3. `joining(CharSequence delimiter, CharSequence prefix, CharSequence suffix)` - with delimiter, prefix, and suffix
- Works only with streams of CharSequence types (String, StringBuilder, etc.)
- For other types, you need to map them to strings first
- Internally uses a StringBuilder for efficient string concatenation

**Hard Example:**

```java
import java.util.*;
import java.util.stream.Collectors;

public class JoiningExample {
    public static void main(String[] args) {
        List<Document> documents = Arrays.asList(
            new Document("DOC-001", "Annual Report", "Finance", Arrays.asList(
                new Section("Executive Summary", 1, "This report outlines..."),
                new Section("Financial Results", 2, "The company achieved..."),
                new Section("Future Outlook", 3, "We expect growth in...")
            )),
            new Document("DOC-002", "Product Specification", "Engineering", Arrays.asList(
                new Section("Overview", 1, "This product is designed to..."),
                new Section("Technical Details", 2, "The system architecture...")
            )),
            new Document("DOC-003", "Marketing Plan", "Marketing", Arrays.asList(
                new Section("Target Audience", 1, "Our primary demographic is..."),
                new Section("Strategy", 2, "We will focus on..."),
                new Section("Budget", 3, "The allocated budget is..."),
                new Section("Timeline", 4, "The campaign will run from...")
            ))
        );
        
        // Challenge 1: Generate a table of contents for each document
        Map<String, String> tableOfContents = documents.stream()
            .collect(Collectors.toMap(
                Document::getTitle,
                doc -> doc.getSections().stream()
                    .map(section -> String.format("%d. %s", section.getNumber(), section.getTitle()))
                    .collect(Collectors.joining("\n", "Table of Contents:\n", "\n\nEnd of Contents"))
            ));
        
        System.out.println("Table of Contents for each document:");
        tableOfContents.forEach((title, toc) -> {
            System.out.println("\n" + title + ":\n" + toc);
        });
        
        // Challenge 2: Create a complex hierarchical representation
        String documentHierarchy = documents.stream()
            .map(doc -> {
                String sectionContent = doc.getSections().stream()
                    .map(section -> String.format("    %d. %s\n       %s", 
                                               section.getNumber(), 
                                               section.getTitle(), 
                                               section.getContent()))
                    .collect(Collectors.joining("\n", "\n", ""));
                
                return String.format("%s - %s (%s)%s", 
                                   doc.getId(), 
                                   doc.getTitle(), 
                                   doc.getDepartment(), 
                                   sectionContent);
            })
            .collect(Collectors.joining("\n\n", "Document Repository:\n\n", "\n\nEnd of Repository"));
        
        System.out.println("\n" + documentHierarchy);
        
        // Challenge 3: Generate CSV data with proper escaping
        String csv = documents.stream()
            .flatMap(doc -> doc.getSections().stream()
                .map(section -> new String[]{
                    doc.getId(),
                    doc.getTitle(),
                    doc.getDepartment(),
                    String.valueOf(section.getNumber()),
                    section.getTitle(),
                    section.getContent()
                }))
            .map(JoiningExample::escapeAndJoinCSV)
            .collect(Collectors.joining("\n", "DocID,DocTitle,Department,SectionNumber,SectionTitle,Content\n", ""));
        
        System.out.println("\nCSV Export:\n" + csv);
    }
    
    // Helper method to properly escape and join CSV fields
    private static String escapeAndJoinCSV(String[] fields) {
        return Arrays.stream(fields)
            .map(field -> {
                // Escape quotes and wrap in quotes if needed
                if (field.contains(",") || field.contains("\"") || field.contains("\n")) {
                    return "\"" + field.replace("\"", "\"\"") + "\"";
                }
                return field;
            })
            .collect(Collectors.joining(","));
    }
    
    static class Document {
        private final String id;
        private final String title;
        private final String department;
        private final List<Section> sections;
        
        public Document(String id, String title, String department, List<Section> sections) {
            this.id = id;
            this.title = title;
            this.department = department;
            this.sections = sections;
        }
        
        public String getId() { return id; }
        public String getTitle() { return title; }
        public String getDepartment() { return department; }
        public List<Section> getSections() { return sections; }
    }
    
    static class Section {
        private final String title;
        private final int number;
        private final String content;
        
        public Section(String title, int number, String content) {
            this.title = title;
            this.number = number;
            this.content = content;
        }
        
        public String getTitle() { return title; }
        public int getNumber() { return number; }
        public String getContent() { return content; }
    }
}
```

### 7.7 Advanced Collector Functions

**Theory:**
Collectors are a powerful feature of the Stream API that allow for complex reduction operations. They transform stream elements into different forms of collections or single values. Here we'll explore the most powerful collector functions in depth:

1. **`groupingBy()`**
   - Groups elements into a Map according to a classification function
   - Three overloaded forms:
     - `groupingBy(Function classifier)`
     - `groupingBy(Function classifier, Collector downstream)`
     - `groupingBy(Function classifier, Supplier mapFactory, Collector downstream)`
   - Allows multi-level grouping when combined with other collectors
   - Common use cases: categorizing data, creating hierarchical structures

2. **`partitioningBy()`**
   - Special case of groupingBy that partitions elements into true/false groups
   - Two overloaded forms:
     - `partitioningBy(Predicate predicate)`
     - `partitioningBy(Predicate predicate, Collector downstream)`
   - More efficient than groupingBy for boolean classification
   - Always returns a Map with exactly two keys: true and false

3. **`counting()`**
   - Counts the number of elements
   - Often used as a downstream collector with groupingBy or partitioningBy
   - Returns a Long value

4. **`summarizingInt/Long/Double()`**
   - Produces statistics like count, sum, min, max, average in a single pass
   - Returns IntSummaryStatistics, LongSummaryStatistics, or DoubleSummaryStatistics
   - Efficient for computing multiple statistics simultaneously

5. **`averagingInt/Long/Double()`**
   - Calculates the arithmetic mean of numeric values
   - Returns a Double value

6. **`mapping()`**
   - Adapts a collector to operate on the results of applying a mapping function
   - Signature: `mapping(Function mapper, Collector downstream)`
   - Allows transformation before collection

7. **`filtering()`** (Java 9+)
   - Adapts a collector to operate only on elements that match a predicate
   - Signature: `filtering(Predicate predicate, Collector downstream)`
   - Allows filtering within a collector operation

8. **`flatMapping()`** (Java 9+)
   - Similar to mapping but flattens nested collections
   - Signature: `flatMapping(Function<? super T, ? extends Stream<? extends U>> mapper, Collector downstream)`
   - Useful for handling nested collections in collectors

9. **`collectingAndThen()`**
   - Performs an additional finishing transformation after collection
   - Signature: `collectingAndThen(Collector downstream, Function finisher)`
   - Useful for post-processing collection results

10. **`toMap()`** / **`toConcurrentMap()`**
    - Creates a Map from stream elements
    - Multiple overloaded forms to handle key/value mapping and merge functions
    - toConcurrentMap creates a thread-safe ConcurrentHashMap

11. **`reducing()`**
    - Performs a reduction operation
    - Three overloaded forms similar to Stream.reduce()
    - Useful as a downstream collector

12. **`teeing()`** (Java 12+)
    - Forwards input to two collectors and merges their results
    - Signature: `teeing(Collector downstream1, Collector downstream2, BiFunction merger)`
    - Allows processing the same data in two different ways simultaneously

13. **`groupingByConcurrent()`**
    - Concurrent version of groupingBy that returns a ConcurrentMap
    - Better performance in parallel streams
    - Same overloaded forms as groupingBy

**Hard Examples:**

```java
import java.util.*;
import java.util.function.Function;
import java.util.stream.Collector;
import java.util.stream.Collectors;
import java.util.stream.Stream;
import java.time.LocalDate;
import java.time.Month;
import java.time.Year;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;

public class AdvancedCollectorsExample {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee(1, "Alice", "Engineering", "Senior Developer", 120000, LocalDate.of(2018, Month.MARCH, 15), "New York", Arrays.asList("Java", "Spring", "Kubernetes")),
            new Employee(2, "Bob", "Engineering", "Developer", 85000, LocalDate.of(2020, Month.JULY, 1), "San Francisco", Arrays.asList("JavaScript", "React", "Node.js")),
            new Employee(3, "Charlie", "Marketing", "Marketing Manager", 95000, LocalDate.of(2019, Month.JANUARY, 10), "Chicago", Arrays.asList("SEO", "Content Strategy", "Analytics")),
            new Employee(4, "Diana", "Sales", "Sales Representative", 75000, LocalDate.of(2021, Month.APRIL, 5), "New York", Arrays.asList("CRM", "Negotiation", "Presentation")),
            new Employee(5, "Eva", "Engineering", "Lead Architect", 150000, LocalDate.of(2017, Month.OCTOBER, 20), "San Francisco", Arrays.asList("Java", "AWS", "Microservices", "System Design")),
            new Employee(6, "Frank", "HR", "HR Manager", 90000, LocalDate.of(2019, Month.MAY, 15), "Chicago", Arrays.asList("Recruitment", "Employee Relations", "Compliance")),
            new Employee(7, "Grace", "Engineering", "Developer", 82000, LocalDate.of(2020, Month.AUGUST, 10), "New York", Arrays.asList("Python", "Django", "Machine Learning")),
            new Employee(8, "Henry", "Sales", "Sales Director", 130000, LocalDate.of(2018, Month.FEBRUARY, 25), "Chicago", Arrays.asList("Strategy", "Team Management", "Client Relations")),
            new Employee(9, "Ivy", "Marketing", "Marketing Specialist", 70000, LocalDate.of(2021, Month.MARCH, 1), "San Francisco", Arrays.asList("Social Media", "Content Creation", "Campaign Management")),
            new Employee(10, "Jack", "Engineering", "QA Engineer", 88000, LocalDate.of(2019, Month.NOVEMBER, 12), "New York", Arrays.asList("Testing", "Automation", "Quality Assurance")),
            new Employee(11, "Karen", "Product", "Product Manager", 115000, LocalDate.of(2018, Month.JUNE, 5), "San Francisco", Arrays.asList("Product Strategy", "Roadmap Planning", "User Research")),
            new Employee(12, "Leo", "Engineering", "DevOps Engineer", 105000, LocalDate.of(2019, Month.AUGUST, 15), "Chicago", Arrays.asList("Docker", "Jenkins", "CI/CD", "Kubernetes"))
        );
        
        System.out.println("========== ADVANCED COLLECTOR EXAMPLES ==========\n");
        
        // 1. GROUPING BY EXAMPLES
        System.out.println("1. GROUPING BY EXAMPLES\n");
        
        // Simple groupingBy
        Map<String, List<Employee>> byDepartment = employees.stream()
            .collect(Collectors.groupingBy(Employee::getDepartment));
            
        System.out.println("1.1 Simple groupingBy by department:");
        byDepartment.forEach((dept, emps) -> {
            System.out.println("\n" + dept + " Department: " + emps.size() + " employees");
            emps.forEach(e -> System.out.println("  - " + e.getName() + ", " + e.getTitle()));
        });
        
        // Multi-level groupingBy
        Map<String, Map<String, List<Employee>>> byLocationAndDepartment = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getLocation,
                Collectors.groupingBy(Employee::getDepartment)
            ));
            
        System.out.println("\n1.2 Multi-level groupingBy by location and department:");
        byLocationAndDepartment.forEach((location, deptMap) -> {
            System.out.println("\n" + location + ":");
            deptMap.forEach((dept, emps) -> {
                System.out.println("  " + dept + " Department: " + emps.size() + " employees");
                emps.forEach(e -> System.out.println("    - " + e.getName() + ", " + e.getTitle()));
            });
        });
        
        // GroupingBy with custom downstream collector
        Map<String, DoubleSummaryStatistics> salaryStatsByDepartment = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.summarizingDouble(Employee::getSalary)
            ));
            
        System.out.println("\n1.3 GroupingBy with summarizingDouble downstream:");
        salaryStatsByDepartment.forEach((dept, stats) -> {
            System.out.println(dept + " Department:");
            System.out.println("  Count: " + stats.getCount());
            System.out.println("  Sum: $" + String.format("%.2f", stats.getSum()));
            System.out.println("  Average: $" + String.format("%.2f", stats.getAverage()));
            System.out.println("  Min: $" + String.format("%.2f", stats.getMin()));
            System.out.println("  Max: $" + String.format("%.2f", stats.getMax()));
        });
        
        // 2. PARTITIONING BY EXAMPLES
        System.out.println("\n2. PARTITIONING BY EXAMPLES\n");
        
        // Simple partitioningBy
        Map<Boolean, List<Employee>> partitionedBySalary = employees.stream()
            .collect(Collectors.partitioningBy(e -> e.getSalary() >= 100000));
            
        System.out.println("2.1 Simple partitioningBy by salary threshold (≥$100k):");
        System.out.println("\nHigh earners (≥$100k): " + partitionedBySalary.get(true).size() + " employees");
        partitionedBySalary.get(true).forEach(e -> 
            System.out.println("  - " + e.getName() + ", " + e.getTitle() + ", $" + e.getSalary()));
            
        System.out.println("\nOther employees (<$100k): " + partitionedBySalary.get(false).size() + " employees");
        partitionedBySalary.get(false).forEach(e -> 
            System.out.println("  - " + e.getName() + ", " + e.getTitle() + ", $" + e.getSalary()));
        
        // PartitioningBy with downstream collector
        Map<Boolean, Map<String, List<Employee>>> seniorityByDepartment = employees.stream()
            .collect(Collectors.partitioningBy(
                e -> e.getHireDate().isBefore(LocalDate.of(2020, 1, 1)),
                Collectors.groupingBy(Employee::getDepartment)
            ));
            
        System.out.println("\n2.2 PartitioningBy with groupingBy downstream:");
        System.out.println("\nSenior employees (hired before 2020) by department:");
        seniorityByDepartment.get(true).forEach((dept, emps) -> {
            System.out.println("  " + dept + ": " + emps.size() + " employees");
            emps.forEach(e -> System.out.println("    - " + e.getName() + ", hired " + e.getHireDate()));
        });
        
        System.out.println("\nJunior employees (hired 2020 or later) by department:");
        seniorityByDepartment.get(false).forEach((dept, emps) -> {
            System.out.println("  " + dept + ": " + emps.size() + " employees");
            emps.forEach(e -> System.out.println("    - " + e.getName() + ", hired " + e.getHireDate()));
        });
        
        // 3. COUNTING EXAMPLES
        System.out.println("\n3. COUNTING EXAMPLES\n");
        
        // Simple counting
        long totalEmployees = employees.stream().collect(Collectors.counting());
        System.out.println("3.1 Simple counting: " + totalEmployees + " total employees");
        
        // Counting with groupingBy
        Map<String, Long> employeeCountByLocation = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getLocation,
                Collectors.counting()
            ));
            
        System.out.println("\n3.2 Counting with groupingBy by location:");
        employeeCountByLocation.forEach((location, count) -> 
            System.out.println("  " + location + ": " + count + " employees"));
        
        // 4. SUMMARIZING EXAMPLES
        System.out.println("\n4. SUMMARIZING EXAMPLES\n");
        
        // SummarizingDouble
        DoubleSummaryStatistics overallSalaryStats = employees.stream()
            .collect(Collectors.summarizingDouble(Employee::getSalary));
            
        System.out.println("4.1 SummarizingDouble for salaries:");
        System.out.println("  Count: " + overallSalaryStats.getCount());
        System.out.println("  Sum: $" + String.format("%.2f", overallSalaryStats.getSum()));
        System.out.println("  Average: $" + String.format("%.2f", overallSalaryStats.getAverage()));
        System.out.println("  Min: $" + String.format("%.2f", overallSalaryStats.getMin()));
        System.out.println("  Max: $" + String.format("%.2f", overallSalaryStats.getMax()));
        
        // SummarizingInt with mapping
        Map<String, IntSummaryStatistics> skillCountByDepartment = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.summarizingInt(e -> e.getSkills().size())
            ));
            
        System.out.println("\n4.2 SummarizingInt for skill counts by department:");
        skillCountByDepartment.forEach((dept, stats) -> {
            System.out.println("  " + dept + " Department:");
            System.out.println("    Average skills per employee: " + String.format("%.1f", stats.getAverage()));
            System.out.println("    Range: " + stats.getMin() + "-" + stats.getMax() + " skills");
        });
        
        // 5. AVERAGING EXAMPLES
        System.out.println("\n5. AVERAGING EXAMPLES\n");
        
        // Simple averaging
        double averageSalary = employees.stream()
            .collect(Collectors.averagingDouble(Employee::getSalary));
            
        System.out.println("5.1 Simple averagingDouble: Average salary is $" + 
            String.format("%.2f", averageSalary));
        
        // Averaging with groupingBy
        Map<Integer, Double> avgSalaryByYearHired = employees.stream()
            .collect(Collectors.groupingBy(
                e -> e.getHireDate().getYear(),
                Collectors.averagingDouble(Employee::getSalary)
            ));
            
        System.out.println("\n5.2 AveragingDouble with groupingBy by hire year:");
        avgSalaryByYearHired.entrySet().stream()
            .sorted(Map.Entry.comparingByKey())
            .forEach(entry -> System.out.println("  " + entry.getKey() + ": $" + 
                String.format("%.2f", entry.getValue())));
        
        // 6. MAPPING EXAMPLES
        System.out.println("\n6. MAPPING EXAMPLES\n");
        
        // Mapping with joining
        Map<String, String> employeeNamesByDept = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.mapping(
                    Employee::getName,
                    Collectors.joining(", ")
                )
            ));
            
        System.out.println("6.1 Mapping with joining by department:");
        employeeNamesByDept.forEach((dept, names) -> 
            System.out.println("  " + dept + ": " + names));
        
        // Mapping with complex transformation
        Map<String, Set<String>> skillsByDepartment = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.mapping(
                    e -> e.getSkills(),
                    Collectors.flatMapping(List::stream, Collectors.toSet())
                )
            ));
            
        System.out.println("\n6.2 Mapping with flatMapping to get unique skills by department:");
        skillsByDepartment.forEach((dept, skills) -> {
            System.out.println("  " + dept + " Department skills: " + skills.size() + " unique skills");
            System.out.println("    " + String.join(", ", skills));
        });
        
        // 7. FILTERING EXAMPLES (Java 9+)
        System.out.println("\n7. FILTERING EXAMPLES\n");
        
        // Filtering with groupingBy
        Map<String, List<Employee>> highEarnersByDepartment = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.filtering(
                    e -> e.getSalary() > 100000,
                    Collectors.toList()
                )
            ));
            
        System.out.println("7.1 Filtering with groupingBy to find high earners by department:");
        highEarnersByDepartment.forEach((dept, emps) -> {
            if (!emps.isEmpty()) {
                System.out.println("  " + dept + " Department high earners (>$100k): " + emps.size());
                emps.forEach(e -> System.out.println("    - " + e.getName() + ", " + e.getTitle() + ", $" + e.getSalary()));
            } else {
                System.out.println("  " + dept + " Department: No employees earning >$100k");
            }
        });
        
        // 8. FLATMAPPING EXAMPLES (Java 9+)
        System.out.println("\n8. FLATMAPPING EXAMPLES\n");
        
        // FlatMapping to get all skills
        Map<String, Long> skillFrequencyByLocation = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getLocation,
                Collectors.flatMapping(
                    e -> e.getSkills().stream(),
                    Collectors.groupingBy(
                        Function.identity(),
                        Collectors.counting()
                    )
                )
            ));
            
        System.out.println("8.1 FlatMapping to get skill frequency by location:");
        skillFrequencyByLocation.forEach((location, skillMap) -> {
            System.out.println("  " + location + " top skills:");
            skillMap.entrySet().stream()
                .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
                .limit(5)
                .forEach(entry -> System.out.println("    - " + entry.getKey() + ": " + entry.getValue()));
        });
        
        // 9. COLLECTINGANDTHEN EXAMPLES
        System.out.println("\n9. COLLECTINGANDTHEN EXAMPLES\n");
        
        // CollectingAndThen to find highest paid employee
        Employee highestPaid = employees.stream()
            .collect(Collectors.collectingAndThen(
                Collectors.maxBy(Comparator.comparing(Employee::getSalary)),
                opt -> opt.orElseThrow(() -> new NoSuchElementException("No employees found"))
            ));
            
        System.out.println("9.1 CollectingAndThen to find highest paid employee:");
        System.out.println("  Highest paid: " + highestPaid.getName() + ", " + 
            highestPaid.getTitle() + ", $" + highestPaid.getSalary());
        
        // CollectingAndThen with groupingBy to find department with most employees
        Map.Entry<String, Integer> largestDepartment = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.collectingAndThen(
                    Collectors.counting(),
                    Long::intValue
                )
            ))
            .entrySet().stream()
            .max(Map.Entry.comparingByValue())
            .orElseThrow(() -> new NoSuchElementException("No departments found"));
            
        System.out.println("\n9.2 CollectingAndThen to find department with most employees:");
        System.out.println("  Largest department: " + largestDepartment.getKey() + 
            " with " + largestDepartment.getValue() + " employees");
        
        // 10. TOMAP EXAMPLES
        System.out.println("\n10. TOMAP EXAMPLES\n");
        
        // Simple toMap
        try {
            Map<String, Double> nameSalaryMap = employees.stream()
                .collect(Collectors.toMap(
                    Employee::getName,
                    Employee::getSalary
                ));
                
            System.out.println("10.1 Simple toMap (name -> salary):");
            nameSalaryMap.entrySet().stream()
                .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
                .limit(5)
                .forEach(entry -> System.out.println("  " + entry.getKey() + ": $" + entry.getValue()));
        } catch (IllegalStateException e) {
            System.out.println("Error: " + e.getMessage());
        }
        
        // ToMap with merge function
        Map<String, Double> departmentTotalSalary = employees.stream()
            .collect(Collectors.toMap(
                Employee::getDepartment,
                Employee::getSalary,
                Double::sum  // Merge function to handle duplicate keys
            ));
            
        System.out.println("\n10.2 ToMap with merge function (department -> total salary):");
        departmentTotalSalary.forEach((dept, total) -> 
            System.out.println("  " + dept + ": $" + String.format("%.2f", total)));
        
        // ToMap with complex key and value mapping
        Map<Year, Map<String, Long>> hiresByYearAndDepartment = employees.stream()
            .collect(Collectors.groupingBy(
                e -> Year.of(e.getHireDate().getYear()),
                Collectors.groupingBy(
                    Employee::getDepartment,
                    Collectors.counting()
                )
            ));
            
        System.out.println("\n10.3 Complex mapping with nested groupingBy (year -> department -> count):");
        hiresByYearAndDepartment.entrySet().stream()
            .sorted(Map.Entry.comparingByKey())
            .forEach(yearEntry -> {
                System.out.println("  " + yearEntry.getKey() + " hires:");
                yearEntry.getValue().forEach((dept, count) -> 
                    System.out.println("    " + dept + ": " + count));
            });
        
        // 11. TOCONCURRENTMAP EXAMPLES
        System.out.println("\n11. TOCONCURRENTMAP EXAMPLES\n");
        
        // Simple toConcurrentMap
        ConcurrentMap<Integer, String> idToNameMap = employees.stream()
            .collect(Collectors.toConcurrentMap(
                Employee::getId,
                Employee::getName
            ));
            
        System.out.println("11.1 Simple toConcurrentMap (id -> name):");
        idToNameMap.entrySet().stream()
            .sorted(Map.Entry.comparingByKey())
            .limit(5)
            .forEach(entry -> System.out.println("  " + entry.getKey() + ": " + entry.getValue()));
        
        // 12. REDUCING EXAMPLES
        System.out.println("\n12. REDUCING EXAMPLES\n");
        
        // Simple reducing
        double totalSalary = employees.stream()
            .collect(Collectors.reducing(
                0.0,
                Employee::getSalary,
                Double::sum
            ));
            
        System.out.println("12.1 Simple reducing to calculate total salary: $" + 
            String.format("%.2f", totalSalary));
        
        // Reducing with groupingBy
        Map<String, Optional<Employee>> highestPaidByDepartment = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.reducing(
                    BinaryOperator.maxBy(Comparator.comparing(Employee::getSalary))
                )
            ));
            
        System.out.println("\n12.2 Reducing with groupingBy to find highest paid by department:");
        highestPaidByDepartment.forEach((dept, empOpt) -> {
            empOpt.ifPresent(e -> System.out.println("  " + dept + ": " + 
                e.getName() + ", " + e.getTitle() + ", $" + e.getSalary()));
        });
        
        // 13. TEEING EXAMPLES (Java 12+)
        System.out.println("\n13. TEEING EXAMPLES\n");
        
        // Teeing to calculate statistics
        Map<String, Object> salaryStats = employees.stream()
            .collect(Collectors.teeing(
                Collectors.averagingDouble(Employee::getSalary),
                Collectors.counting(),
                (avg, count) -> {
                    Map<String, Object> stats = new HashMap<>();
                    stats.put("average", avg);
                    stats.put("count", count);
                    stats.put("estimatedTotal", avg * count);
                    return stats;
                }
            ));
            
        System.out.println("13.1 Teeing to calculate multiple statistics in one pass:");
        System.out.println("  Employee count: " + salaryStats.get("count"));
        System.out.println("  Average salary: $" + String.format("%.2f", (Double)salaryStats.get("average")));
        System.out.println("  Estimated total: $" + String.format("%.2f", (Double)salaryStats.get("estimatedTotal")));
        
        // Teeing for min and max
        Map<String, Employee> salaryExtremes = employees.stream()
            .collect(Collectors.teeing(
                Collectors.minBy(Comparator.comparing(Employee::getSalary)),
                Collectors.maxBy(Comparator.comparing(Employee::getSalary)),
                (min, max) -> {
                    Map<String, Employee> result = new HashMap<>();
                    min.ifPresent(e -> result.put("min", e));
                    max.ifPresent(e -> result.put("max", e));
                    return result;
                }
            ));
            
        System.out.println("\n13.2 Teeing to find salary extremes in one pass:");
        Employee minSalary = salaryExtremes.get("min");
        Employee maxSalary = salaryExtremes.get("max");
        System.out.println("  Lowest paid: " + minSalary.getName() + ", " + 
            minSalary.getTitle() + ", $" + minSalary.getSalary());
        System.out.println("  Highest paid: " + maxSalary.getName() + ", " + 
            maxSalary.getTitle() + ", $" + maxSalary.getSalary());
        
        // 14. GROUPINGBYCONCURRENT EXAMPLES
        System.out.println("\n14. GROUPINGBYCONCURRENT EXAMPLES\n");
        
        // Simple groupingByConcurrent
        ConcurrentMap<String, List<Employee>> concurrentDeptMap = employees.parallelStream()
            .collect(Collectors.groupingByConcurrent(Employee::getDepartment));
            
        System.out.println("14.1 Simple groupingByConcurrent by department:");
        concurrentDeptMap.forEach((dept, emps) -> 
            System.out.println("  " + dept + ": " + emps.size() + " employees"));
        
        // GroupingByConcurrent with downstream collector
        ConcurrentMap<String, ConcurrentMap<Integer, List<Employee>>> concurrentLocationYearMap = 
            employees.parallelStream()
                .collect(Collectors.groupingByConcurrent(
                    Employee::getLocation,
                    Collectors.groupingByConcurrent(
                        e -> e.getHireDate().getYear()
                    )
                ));
                
        System.out.println("\n14.2 GroupingByConcurrent with downstream groupingByConcurrent:");
        concurrentLocationYearMap.forEach((location, yearMap) -> {
            System.out.println("  " + location + ":");
            yearMap.entrySet().stream()
                .sorted(Map.Entry.comparingByKey())
                .forEach(entry -> 
                    System.out.println("    " + entry.getKey() + ": " + entry.getValue().size() + " employees"));
        });
        
        // REAL-WORLD COMPLEX EXAMPLE
        System.out.println("\n15. REAL-WORLD COMPLEX EXAMPLE\n");
        
        // Complex analysis combining multiple collectors
        // Goal: Generate a comprehensive department report with multiple metrics
        Map<String, DepartmentReport> departmentReports = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.collectingAndThen(
                    Collectors.toList(),
                    deptEmployees -> {
                        // Calculate salary statistics
                        DoubleSummaryStatistics salaryStats = deptEmployees.stream()
                            .collect(Collectors.summarizingDouble(Employee::getSalary));
                        
                        // Find highest paid employee
                        Employee highestPaid = deptEmployees.stream()
                            .max(Comparator.comparing(Employee::getSalary))
                            .orElse(null);
                        
                        // Group by hire year
                        Map<Integer, Long> hiresByYear = deptEmployees.stream()
                            .collect(Collectors.groupingBy(
                                e -> e.getHireDate().getYear(),
                                Collectors.counting()
                            ));
                        
                        // Get all unique skills in department
                        Set<String> departmentSkills = deptEmployees.stream()
                            .flatMap(e -> e.getSkills().stream())
                            .collect(Collectors.toSet());
                        
                        // Get skill frequency
                        Map<String, Long> skillFrequency = deptEmployees.stream()
                            .flatMap(e -> e.getSkills().stream())
                            .collect(Collectors.groupingBy(
                                Function.identity(),
                                Collectors.counting()
                            ));
                        
                        // Find top skills
                        List<String> topSkills = skillFrequency.entrySet().stream()
                            .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
                            .limit(3)
                            .map(Map.Entry::getKey)
                            .collect(Collectors.toList());
                        
                        // Calculate seniority distribution
                        Map<String, Long> seniorityDistribution = deptEmployees.stream()
                            .collect(Collectors.groupingBy(
                                e -> {
                                    LocalDate now = LocalDate.now();
                                    int yearsEmployed = now.getYear() - e.getHireDate().getYear();
                                    if (yearsEmployed < 2) return "Junior (0-1 years)";
                                    else if (yearsEmployed < 5) return "Mid-level (2-4 years)";
                                    else return "Senior (5+ years)";
                                },
                                Collectors.counting()
                            ));
                        
                        // Create the report
                        return new DepartmentReport(
                            deptEmployees.size(),
                            salaryStats,
                            highestPaid,
                            hiresByYear,
                            departmentSkills,
                            topSkills,
                            seniorityDistribution
                        );
                    }
                )
            ));
        
        // Print the comprehensive report
        System.out.println("Comprehensive Department Analysis Report:\n");
        departmentReports.forEach((dept, report) -> {
            System.out.println("=== " + dept + " Department Report ===");
            System.out.println("Employee Count: " + report.getEmployeeCount());
            
            System.out.println("\nSalary Statistics:");
            DoubleSummaryStatistics stats = report.getSalaryStats();
            System.out.println("  Average: $" + String.format("%.2f", stats.getAverage()));
            System.out.println("  Total Budget: $" + String.format("%.2f", stats.getSum()));
            System.out.println("  Range: $" + String.format("%.2f", stats.getMin()) + 
                " - $" + String.format("%.2f", stats.getMax()));
            
            Employee topEarner = report.getHighestPaidEmployee();
            System.out.println("\nHighest Paid Employee:");
            System.out.println("  " + topEarner.getName() + ", " + topEarner.getTitle() + 
                ", $" + topEarner.getSalary());
            
            System.out.println("\nHiring History:");
            report.getHiresByYear().entrySet().stream()
                .sorted(Map.Entry.comparingByKey())
                .forEach(entry -> System.out.println("  " + entry.getKey() + ": " + entry.getValue() + " hires"));
            
            System.out.println("\nSkill Coverage: " + report.getDepartmentSkills().size() + " unique skills");
            System.out.println("Top Skills: " + String.join(", ", report.getTopSkills()));
            
            System.out.println("\nSeniority Distribution:");
            report.getSeniorityDistribution().forEach((level, count) -> 
                System.out.println("  " + level + ": " + count + " employees"));
            
            System.out.println();
        });
    }
    
    static class DepartmentReport {
        private final int employeeCount;
        private final DoubleSummaryStatistics salaryStats;
        private final Employee highestPaidEmployee;
        private final Map<Integer, Long> hiresByYear;
        private final Set<String> departmentSkills;
        private final List<String> topSkills;
        private final Map<String, Long> seniorityDistribution;
        
        public DepartmentReport(int employeeCount, DoubleSummaryStatistics salaryStats,
                              Employee highestPaidEmployee, Map<Integer, Long> hiresByYear,
                              Set<String> departmentSkills, List<String> topSkills,
                              Map<String, Long> seniorityDistribution) {
            this.employeeCount = employeeCount;
            this.salaryStats = salaryStats;
            this.highestPaidEmployee = highestPaidEmployee;
            this.hiresByYear = hiresByYear;
            this.departmentSkills = departmentSkills;
            this.topSkills = topSkills;
            this.seniorityDistribution = seniorityDistribution;
        }
        
        public int getEmployeeCount() { return employeeCount; }
        public DoubleSummaryStatistics getSalaryStats() { return salaryStats; }
        public Employee getHighestPaidEmployee() { return highestPaidEmployee; }
        public Map<Integer, Long> getHiresByYear() { return hiresByYear; }
        public Set<String> getDepartmentSkills() { return departmentSkills; }
        public List<String> getTopSkills() { return topSkills; }
        public Map<String, Long> getSeniorityDistribution() { return seniorityDistribution; }
    }
    
    static class Employee {
        private final int id;
        private final String name;
        private final String department;
        private final String title;
        private final double salary;
        private final LocalDate hireDate;
        private final String location;
        private final List<String> skills;
        
        public Employee(int id, String name, String department, String title, 
                       double salary, LocalDate hireDate, String location, List<String> skills) {
            this.id = id;
            this.name = name;
            this.department = department;
            this.title = title;
            this.salary = salary;
            this.hireDate = hireDate;
            this.location = location;
            this.skills = skills;
        }
        
        public int getId() { return id; }
        public String getName() { return name; }
        public String getDepartment() { return department; }
        public String getTitle() { return title; }
        public double getSalary() { return salary; }
        public LocalDate getHireDate() { return hireDate; }
        public String getLocation() { return location; }
        public List<String> getSkills() { return skills; }
        
        @Override
        public String toString() {
            return "Employee{" +
                   "id=" + id +
                   ", name='" + name + '\'' +
                   ", department='" + department + '\'' +
                   ", title='" + title + '\'' +
                   ", salary=" + salary +
                   ", hireDate=" + hireDate +
                   ", location='" + location + '\'' +
                   ", skills=" + skills +
                   '}';
        }
    }
}
```

## 8. Java Concurrency Patterns and Best Practices

Java provides robust support for concurrent programming. This section covers essential concurrency patterns and best practices that are commonly asked in interviews.

### 8.1 Implementing Thread Pools with ExecutorService

**Theory:**
- `ExecutorService` is a high-level replacement for working with threads directly
- Provides thread pooling, scheduling, and management capabilities
- Several factory methods in `Executors` class for creating different types of thread pools:
  1. `newFixedThreadPool(n)` - Creates a pool with a fixed number of threads
  2. `newCachedThreadPool()` - Creates an expandable pool that reuses threads
  3. `newSingleThreadExecutor()` - Creates a single-threaded executor
  4. `newScheduledThreadPool(n)` - Creates a pool that can schedule tasks
- Proper shutdown is essential to prevent resource leaks

**Hard Example:**

```java
import java.util.concurrent.*;
import java.util.List;
import java.util.ArrayList;
import java.util.Random;

public class ThreadPoolExample {
    public static void main(String[] args) {
        // Create a fixed thread pool with 3 threads
        ExecutorService executor = Executors.newFixedThreadPool(3);
        
        // Create a list to hold Future objects for results
        List<Future<Integer>> resultList = new ArrayList<>();
        
        // Random number generator
        Random random = new Random();
        
        // Submit 10 tasks
        for (int i = 0; i < 10; i++) {
            final int taskId = i;
            // Submit a Callable task that returns a result
            Future<Integer> result = executor.submit(() -> {
                // Simulate work
                int processingTime = random.nextInt(3000) + 1000; // 1-4 seconds
                System.out.println("Task #" + taskId + " started by thread " + 
                                   Thread.currentThread().getName());
                try {
                    Thread.sleep(processingTime);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return -1;
                }
                System.out.println("Task #" + taskId + " completed in " + processingTime + "ms");
                return processingTime;
            });
            
            resultList.add(result);
        }
        
        // Process the results
        int totalProcessingTime = 0;
        for (int i = 0; i < resultList.size(); i++) {
            try {
                // get() blocks until the task completes
                int processingTime = resultList.get(i).get();
                System.out.println("Task #" + i + " returned: " + processingTime);
                totalProcessingTime += processingTime;
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        }
        
        System.out.println("Total processing time: " + totalProcessingTime + "ms");
        System.out.println("Average processing time: " + (totalProcessingTime / 10) + "ms");
        
        // Demonstrate proper shutdown
        try {
            System.out.println("Attempting to shutdown executor");
            executor.shutdown();
            executor.awaitTermination(5, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            System.err.println("Tasks interrupted");
        } finally {
            if (!executor.isTerminated()) {
                System.err.println("Cancelling non-finished tasks");
                executor.shutdownNow();
            }
            System.out.println("Executor shutdown finished");
        }
    }
}
```

### 8.2 Implementing a Thread-Safe Counter

**Theory:**
- Thread safety ensures correct behavior in concurrent environments
- Several approaches to achieve thread safety:
  1. Synchronization (synchronized keyword)
  2. Atomic classes (java.util.concurrent.atomic package)
  3. Locks (java.util.concurrent.locks package)
  4. Thread-local variables
  5. Immutable objects
- Each approach has different performance characteristics and use cases

**Hard Example:**

```java
import java.util.concurrent.atomic.AtomicLong;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadSafeCounterExample {
    public static void main(String[] args) throws InterruptedException {
        // Number of threads and increments per thread
        final int THREADS = 10;
        final int INCREMENTS_PER_THREAD = 100_000;
        
        // Create different counter implementations
        SynchronizedCounter syncCounter = new SynchronizedCounter();
        LockBasedCounter lockCounter = new LockBasedCounter();
        AtomicCounter atomicCounter = new AtomicCounter();
        UnsafeCounter unsafeCounter = new UnsafeCounter();
        
        // Test each counter implementation
        System.out.println("Testing counter implementations with " + THREADS + 
                           " threads and " + INCREMENTS_PER_THREAD + " increments per thread");
        
        testCounter("Synchronized Counter", syncCounter, THREADS, INCREMENTS_PER_THREAD);
        testCounter("Lock-Based Counter", lockCounter, THREADS, INCREMENTS_PER_THREAD);
        testCounter("Atomic Counter", atomicCounter, THREADS, INCREMENTS_PER_THREAD);
        testCounter("Unsafe Counter", unsafeCounter, THREADS, INCREMENTS_PER_THREAD);
    }
    
    private static void testCounter(String name, Counter counter, int threads, int incrementsPerThread) 
            throws InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(threads);
        CountDownLatch latch = new CountDownLatch(threads);
        
        long startTime = System.nanoTime();
        
        for (int i = 0; i < threads; i++) {
            executor.execute(() -> {
                for (int j = 0; j < incrementsPerThread; j++) {
                    counter.increment();
                }
                latch.countDown();
            });
        }
        
        latch.await(); // Wait for all threads to finish
        long endTime = System.nanoTime();
        
        executor.shutdown();
        
        long expectedCount = (long) threads * incrementsPerThread;
        long actualCount = counter.getValue();
        boolean isCorrect = expectedCount == actualCount;
        
        System.out.printf("%s: Count = %,d (expected %,d), Correct = %s, Time = %.3f ms%n", 
                          name, actualCount, expectedCount, isCorrect, 
                          (endTime - startTime) / 1_000_000.0);
    }
    
    // Counter interface
    interface Counter {
        void increment();
        long getValue();
    }
    
    // Implementation 1: Synchronized method
    static class SynchronizedCounter implements Counter {
        private long count = 0;
        
        @Override
        public synchronized void increment() {
            count++;
        }
        
        @Override
        public synchronized long getValue() {
            return count;
        }
    }
    
    // Implementation 2: Explicit lock
    static class LockBasedCounter implements Counter {
        private long count = 0;
        private final Lock lock = new ReentrantLock();
        
        @Override
        public void increment() {
            lock.lock();
            try {
                count++;
            } finally {
                lock.unlock();
            }
        }
        
        @Override
        public long getValue() {
            lock.lock();
            try {
                return count;
            } finally {
                lock.unlock();
            }
        }
    }
    
    // Implementation 3: Atomic variable
    static class AtomicCounter implements Counter {
        private final AtomicLong count = new AtomicLong(0);
        
        @Override
        public void increment() {
            count.incrementAndGet();
        }
        
        @Override
        public long getValue() {
            return count.get();
        }
    }
    
    // Implementation 4: Unsafe (non-thread-safe)
    static class UnsafeCounter implements Counter {
        private long count = 0;
        
        @Override
        public void increment() {
            count++; // Not thread-safe!
        }
        
        @Override
        public long getValue() {
            return count;
        }
    }
}
```
### 8.3 Implementing a Thread-Safe Cache with ReadWriteLock

**Theory:**
- `ReadWriteLock` provides separate locks for read and write operations
- Multiple threads can read simultaneously, but writes are exclusive
- Improves performance in read-heavy scenarios
- `ReentrantReadWriteLock` is the standard implementation
- Can be configured with fairness policy to prevent starvation

**Hard Example:**

```java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;
import java.util.List;
import java.util.ArrayList;
import java.util.Random;

public class ReadWriteLockCacheExample {
    public static void main(String[] args) throws Exception {
        // Create a cache with expensive computation
        ReadWriteLockCache<Integer, String> cache = new ReadWriteLockCache<>(
            key -> {
                // Simulate expensive computation
                Thread.sleep(500);
                return "Value for key " + key;
            }
        );
        
        // Create a thread pool
        ExecutorService executor = Executors.newFixedThreadPool(10);
        
        // Create a random number generator
        Random random = new Random();
        
        // Submit tasks that read from and write to the cache
        List<Future<String>> futures = new ArrayList<>();
        
        for (int i = 0; i < 100; i++) {
            final int taskId = i;
            futures.add(executor.submit(() -> {
                int key = random.nextInt(10); // Use a small range to ensure cache hits
                boolean isWrite = random.nextDouble() < 0.1; // 10% writes, 90% reads
                
                if (isWrite) {
                    String newValue = "Updated value for key " + key + " by task " + taskId;
                    cache.put(key, newValue);
                    return "Task " + taskId + " wrote: " + newValue;
                } else {
                    long startTime = System.nanoTime();
                    String value = cache.get(key);
                    long endTime = System.nanoTime();
                    return String.format("Task %d read: %s (took %.2f ms)", 
                                        taskId, value, (endTime - startTime) / 1_000_000.0);
                }
            }));
        }
        
        // Process results
        for (Future<String> future : futures) {
            System.out.println(future.get());
        }
        
        // Shutdown the executor
        executor.shutdown();
        executor.awaitTermination(10, TimeUnit.SECONDS);
        
        // Print cache statistics
        System.out.println("\nCache statistics:");
        System.out.println("Cache size: " + cache.size());
        System.out.println("Cache hits: " + cache.getHits());
        System.out.println("Cache misses: " + cache.getMisses());
        System.out.println("Hit ratio: " + String.format("%.2f%%", cache.getHitRatio() * 100));
    }
    
    static class ReadWriteLockCache<K, V> {
        private final Map<K, V> cache = new HashMap<>();
        private final ReadWriteLock lock = new ReentrantReadWriteLock();
        private final Computable<K, V> computable;
        
        // Statistics
        private int hits = 0;
        private int misses = 0;
        
        public ReadWriteLockCache(Computable<K, V> computable) {
            this.computable = computable;
        }
        
        public V get(K key) throws Exception {
            // First try to read from cache without computing
            lock.readLock().lock();
            try {
                V value = cache.get(key);
                if (value != null) {
                    hits++;
                    return value;
                }
            } finally {
                lock.readLock().unlock();
            }
            
            // Value not in cache, acquire write lock and compute
            lock.writeLock().lock();
            try {
                // Check again in case another thread computed the value
                // while we were waiting for the write lock
                V value = cache.get(key);
                if (value == null) {
                    misses++;
                    value = computable.compute(key);
                    cache.put(key, value);
                } else {
                    hits++;
                }
                return value;
            } finally {
                lock.writeLock().unlock();
            }
        }
        
        public void put(K key, V value) {
            lock.writeLock().lock();
            try {
                cache.put(key, value);
            } finally {
                lock.writeLock().unlock();
            }
        }
        
        public int size() {
            lock.readLock().lock();
            try {
                return cache.size();
            } finally {
                lock.readLock().unlock();
            }
        }
        
        public int getHits() {
            return hits;
        }
        
        public int getMisses() {
            return misses;
        }
        
        public double getHitRatio() {
            int total = hits + misses;
            return total == 0 ? 0 : (double) hits / total;
        }
    }
    
    interface Computable<K, V> {
        V compute(K key) throws Exception;
    }
}
```

### 8.4 Implementing a CompletableFuture Pipeline

**Theory:**
- `CompletableFuture` was introduced in Java 8 for asynchronous programming
- Provides a rich API for composing, combining, and handling asynchronous operations
- Key features:
  1. Chaining operations with `thenApply`, `thenAccept`, `thenRun`
  2. Combining futures with `thenCombine`, `allOf`, `anyOf`
  3. Error handling with `exceptionally`, `handle`
  4. Controlling execution with `supplyAsync`, `runAsync`
- Non-blocking and more flexible than traditional `Future`

**Hard Example:**

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadLocalRandom;
import java.util.concurrent.TimeUnit;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class CompletableFutureExample {
    public static void main(String[] args) {
        // Create a custom executor service
        ExecutorService executor = Executors.newFixedThreadPool(4);
        
        try {
            // Example 1: Basic CompletableFuture with supplyAsync
            System.out.println("\nExample 1: Basic CompletableFuture");
            CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
                simulateWork(500);
                return "Result from the future";
            }, executor);
            
            // Block and get the result
            System.out.println(future1.get());
            
            // Example 2: Chaining operations
            System.out.println("\nExample 2: Chaining operations");
            CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
                simulateWork(500);
                return "Step 1 complete";
            }, executor).thenApply(result -> {
                simulateWork(300);
                return result + " -> Step 2 complete";
            }).thenApply(result -> {
                simulateWork(200);
                return result + " -> Step 3 complete";
            });
            
            System.out.println(future2.get());
            
            // Example 3: Error handling
            System.out.println("\nExample 3: Error handling");
            CompletableFuture<String> future3 = CompletableFuture.supplyAsync(() -> {
                if (ThreadLocalRandom.current().nextBoolean()) {
                    throw new RuntimeException("Simulated error");
                }
                return "Success";
            }, executor).exceptionally(ex -> {
                System.out.println("Exception caught: " + ex.getMessage());
                return "Recovery value";
            });
            
            System.out.println(future3.get());
            
            // Example 4: Combining futures with thenCombine
            System.out.println("\nExample 4: Combining futures");
            CompletableFuture<String> future4a = CompletableFuture.supplyAsync(() -> {
                simulateWork(500);
                return "Result from future 4a";
            }, executor);
            
            CompletableFuture<String> future4b = CompletableFuture.supplyAsync(() -> {
                simulateWork(700);
                return "Result from future 4b";
            }, executor);
            
            CompletableFuture<String> combinedFuture = future4a.thenCombine(future4b, 
                (result1, result2) -> result1 + " + " + result2);
            
            System.out.println(combinedFuture.get());
            
            // Example 5: allOf - wait for all futures to complete
            System.out.println("\nExample 5: allOf");
            List<CompletableFuture<String>> futures = IntStream.range(0, 5)
                .mapToObj(i -> CompletableFuture.supplyAsync(() -> {
                    int delay = ThreadLocalRandom.current().nextInt(500, 1500);
                    simulateWork(delay);
                    return "Result " + i + " (delayed " + delay + "ms)";
                }, executor))
                .collect(Collectors.toList());
            
            // Create a combined future that completes when all individual futures complete
            CompletableFuture<Void> allFutures = CompletableFuture.allOf(
                futures.toArray(new CompletableFuture[0]));
            
            // Wait for all futures to complete
            allFutures.get();
            
            // Get all results
            List<String> results = futures.stream()
                .map(CompletableFuture::join) // join() is similar to get() but doesn't throw checked exceptions
                .collect(Collectors.toList());
            
            System.out.println("All results: " + results);
            
            // Example 6: anyOf - wait for any future to complete
            System.out.println("\nExample 6: anyOf");
            CompletableFuture<Object> anyFuture = CompletableFuture.anyOf(
                CompletableFuture.supplyAsync(() -> {
                    simulateWork(500);
                    return "Fast task completed";
                }, executor),
                CompletableFuture.supplyAsync(() -> {
                    simulateWork(1000);
                    return "Medium task completed";
                }, executor),
                CompletableFuture.supplyAsync(() -> {
                    simulateWork(1500);
                    return "Slow task completed";
                }, executor)
            );
            
            System.out.println("First completed: " + anyFuture.get());
            
            // Example 7: Real-world scenario - parallel API calls with timeout
            System.out.println("\nExample 7: Parallel API calls with timeout");
            simulateParallelApiCalls(executor);
            
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // Shutdown the executor
            executor.shutdown();
            try {
                if (!executor.awaitTermination(5, TimeUnit.SECONDS)) {
                    executor.shutdownNow();
                }
            } catch (InterruptedException e) {
                executor.shutdownNow();
            }
        }
    }
    
    private static void simulateWork(int milliseconds) {
        try {
            Thread.sleep(milliseconds);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    private static void simulateParallelApiCalls(ExecutorService executor) throws Exception {
        // Simulate multiple API calls with different response times
        CompletableFuture<String> userInfoFuture = CompletableFuture.supplyAsync(() -> {
            simulateWork(300); // Fast API
            return "User: John Doe, ID: 12345";
        }, executor);
        
        CompletableFuture<String> orderHistoryFuture = CompletableFuture.supplyAsync(() -> {
            simulateWork(700); // Medium API
            return "Orders: 5 items in the last month";
        }, executor);
        
        CompletableFuture<String> recommendationsFuture = CompletableFuture.supplyAsync(() -> {
            simulateWork(1200); // Slow API
            return "Recommendations: 3 products based on history";
        }, executor);
        
        // Add timeout to each future
        CompletableFuture<String> userInfoWithTimeout = addTimeout(userInfoFuture, 500, 
            "User info timed out", executor);
        CompletableFuture<String> orderHistoryWithTimeout = addTimeout(orderHistoryFuture, 500, 
            "Order history timed out", executor);
        CompletableFuture<String> recommendationsWithTimeout = addTimeout(recommendationsFuture, 500, 
            "Recommendations timed out", executor);
        
        // Combine all results
        CompletableFuture<Void> allFutures = CompletableFuture.allOf(
            userInfoWithTimeout, orderHistoryWithTimeout, recommendationsWithTimeout);
        
        // Wait for all to complete
        allFutures.get();
        
        // Print results
        System.out.println(userInfoWithTimeout.get());
        System.out.println(orderHistoryWithTimeout.get());
        System.out.println(recommendationsWithTimeout.get());
    }
    
    private static <T> CompletableFuture<T> addTimeout(CompletableFuture<T> future, 
                                                     long timeoutMillis, 
                                                     T timeoutValue, 
                                                     ExecutorService executor) {
        CompletableFuture<T> timeout = new CompletableFuture<>();
        executor.schedule(() -> timeout.complete(timeoutValue), 
                         timeoutMillis, TimeUnit.MILLISECONDS);
        return CompletableFuture.anyOf(future, timeout)
            .thenApply(result -> (T) result);
    }
}
```

### 8.5 Implementing Thread-Safe Singleton Patterns

**Theory:**
- Singleton pattern ensures a class has only one instance and provides a global point of access to it
- Thread safety is critical for singletons in concurrent environments
- Common thread-safe singleton implementations:
  1. Double-checked locking (with volatile)
  2. Initialization-on-demand holder idiom
  3. Enum-based singleton
  4. Early initialization (eager loading)
- Each approach has different characteristics regarding:
  - Lazy initialization
  - Thread safety
  - Serialization safety
  - Performance

**Hard Example:**

```java
import java.io.*;

// 1. Double-Checked Locking Singleton
class DCLSingleton implements Serializable {
    private static final long serialVersionUID = 1L;
    
    // The 'volatile' keyword ensures that multiple threads handle the instance variable correctly
    private static volatile DCLSingleton instance;
    
    // Private constructor prevents instantiation from other classes
    private DCLSingleton() {
        // Protect against reflection attacks
        if (instance != null) {
            throw new IllegalStateException("Singleton already initialized");
        }
        // Initialization code
        System.out.println("DCLSingleton initialized");
    }
    
    public static DCLSingleton getInstance() {
        // Local variable improves performance by reducing volatile reads
        DCLSingleton result = instance;
        
        // First check (no synchronization)
        if (result == null) {
            // Synchronize only when instance might be null
            synchronized (DCLSingleton.class) {
                result = instance;
                // Second check (with synchronization)
                if (result == null) {
                    instance = result = new DCLSingleton();
                }
            }
        }
        return result;
    }
    
    // Protect against serialization
    protected Object readResolve() {
        return getInstance();
    }
}

// 2. Initialization-on-demand holder idiom
class HolderSingleton implements Serializable {
    private static final long serialVersionUID = 1L;
    
    // Private constructor
    private HolderSingleton() {
        System.out.println("HolderSingleton initialized");
    }
    
    // Static holder class - not loaded until first access
    private static class SingletonHolder {
        // JVM guarantees this will be thread-safe
        private static final HolderSingleton INSTANCE = new HolderSingleton();
    }
    
    public static HolderSingleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
    
    // Protect against serialization
    protected Object readResolve() {
        return getInstance();
    }
}

// 3. Enum Singleton - the most concise and safest approach
enum EnumSingleton {
    INSTANCE;
    
    // Constructor is automatically private in enums
    EnumSingleton() {
        System.out.println("EnumSingleton initialized");
    }
    
    public void doSomething() {
        System.out.println("EnumSingleton doing something");
    }
}

// 4. Early Initialization (Eager Loading)
class EagerSingleton implements Serializable {
    private static final long serialVersionUID = 1L;
    
    // Instance is created when the class is loaded
    private static final EagerSingleton INSTANCE = new EagerSingleton();
    
    private EagerSingleton() {
        System.out.println("EagerSingleton initialized");
    }
    
    public static EagerSingleton getInstance() {
        return INSTANCE;
    }
    
    // Protect against serialization
    protected Object readResolve() {
        return getInstance();
    }
}

// Performance and thread-safety test
public class SingletonPatternExample {
    public static void main(String[] args) throws Exception {
        // Test thread safety with multiple threads trying to get the singleton instance
        System.out.println("\n=== Testing Thread Safety ===");
        testThreadSafety();
        
        // Test serialization safety
        System.out.println("\n=== Testing Serialization Safety ===");
        testSerialization();
        
        // Test performance
        System.out.println("\n=== Testing Performance ===");
        testPerformance();
    }
    
    private static void testThreadSafety() {
        // Create multiple threads that try to get the singleton instance
        Runnable dcl = () -> {
            DCLSingleton singleton = DCLSingleton.getInstance();
            System.out.println(Thread.currentThread().getName() + ": " + singleton.hashCode());
        };
        
        Runnable holder = () -> {
            HolderSingleton singleton = HolderSingleton.getInstance();
            System.out.println(Thread.currentThread().getName() + ": " + singleton.hashCode());
        };
        
        Runnable enumS = () -> {
            EnumSingleton singleton = EnumSingleton.INSTANCE;
            System.out.println(Thread.currentThread().getName() + ": " + singleton.hashCode());
        };
        
        Runnable eager = () -> {
            EagerSingleton singleton = EagerSingleton.getInstance();
            System.out.println(Thread.currentThread().getName() + ": " + singleton.hashCode());
        };
        
        // Test each singleton type
        System.out.println("\nDCL Singleton:");
        for (int i = 0; i < 5; i++) {
            new Thread(dcl, "DCL-Thread-" + i).start();
        }
        
        try { Thread.sleep(500); } catch (InterruptedException e) { }
        
        System.out.println("\nHolder Singleton:");
        for (int i = 0; i < 5; i++) {
            new Thread(holder, "Holder-Thread-" + i).start();
        }
        
        try { Thread.sleep(500); } catch (InterruptedException e) { }
        
        System.out.println("\nEnum Singleton:");
        for (int i = 0; i < 5; i++) {
            new Thread(enumS, "Enum-Thread-" + i).start();
        }
        
        try { Thread.sleep(500); } catch (InterruptedException e) { }
        
        System.out.println("\nEager Singleton:");
        for (int i = 0; i < 5; i++) {
            new Thread(eager, "Eager-Thread-" + i).start();
        }
        
        try { Thread.sleep(500); } catch (InterruptedException e) { }
    }
    
    private static void testSerialization() throws Exception {
        // Test serialization and deserialization for each singleton type
        
        // 1. DCL Singleton
        DCLSingleton dclOriginal = DCLSingleton.getInstance();
        DCLSingleton dclDeserialized = serializeDeserialize(dclOriginal);
        System.out.println("DCL Singleton: Same instance after serialization? " + 
                          (dclOriginal == dclDeserialized));
        
        // 2. Holder Singleton
        HolderSingleton holderOriginal = HolderSingleton.getInstance();
        HolderSingleton holderDeserialized = serializeDeserialize(holderOriginal);
        System.out.println("Holder Singleton: Same instance after serialization? " + 
                          (holderOriginal == holderDeserialized));
        
        // 3. Enum Singleton (inherently serialization-safe)
        EnumSingleton enumOriginal = EnumSingleton.INSTANCE;
        EnumSingleton enumDeserialized = serializeDeserialize(enumOriginal);
        System.out.println("Enum Singleton: Same instance after serialization? " + 
                          (enumOriginal == enumDeserialized));
        
        // 4. Eager Singleton
        EagerSingleton eagerOriginal = EagerSingleton.getInstance();
        EagerSingleton eagerDeserialized = serializeDeserialize(eagerOriginal);
        System.out.println("Eager Singleton: Same instance after serialization? " + 
                          (eagerOriginal == eagerDeserialized));
    }
    
    @SuppressWarnings("unchecked")
    private static <T> T serializeDeserialize(T object) throws Exception {
        // Serialize to a byte array
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream out = new ObjectOutputStream(bos);
        out.writeObject(object);
        out.close();
        
        // Deserialize from the byte array
        ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
        ObjectInputStream in = new ObjectInputStream(bis);
        T deserialized = (T) in.readObject();
        in.close();
        
        return deserialized;
    }
    
    private static void testPerformance() {
        // Test performance of each singleton implementation
        int iterations = 10_000_000;
        
        // 1. DCL Singleton
        long start = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            DCLSingleton.getInstance();
        }
        long dclTime = System.nanoTime() - start;
        
        // 2. Holder Singleton
        start = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            HolderSingleton.getInstance();
        }
        long holderTime = System.nanoTime() - start;
        
        // 3. Enum Singleton
        start = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            EnumSingleton.INSTANCE;
        }
        long enumTime = System.nanoTime() - start;
        
        // 4. Eager Singleton
        start = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            EagerSingleton.getInstance();
        }
        long eagerTime = System.nanoTime() - start;
        
        // Print results
        System.out.println("DCL Singleton: " + dclTime / 1_000_000.0 + " ms");
        System.out.println("Holder Singleton: " + holderTime / 1_000_000.0 + " ms");
        System.out.println("Enum Singleton: " + enumTime / 1_000_000.0 + " ms");
        System.out.println("Eager Singleton: " + eagerTime / 1_000_000.0 + " ms");
    }
}
```

### 8.6 Understanding Java Memory Model and Thread Visibility

**Theory:**
- The Java Memory Model (JMM) defines how threads interact through memory
- Key concepts:
  1. **Happens-before relationship**: Guarantees that memory writes by one statement are visible to another statement
  2. **Memory visibility**: Changes made by one thread may not be immediately visible to other threads
  3. **Memory barriers**: Operations that enforce ordering constraints on memory operations
  4. **Reordering**: The JVM and CPU may reorder instructions for optimization
- Synchronization mechanisms:
  - `synchronized` blocks/methods
  - `volatile` variables
  - `final` fields
  - Atomic classes
  - Explicit locks

**Hard Example:**

```java
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicInteger;

public class MemoryVisibilityExample {
    // Regular variable - no visibility guarantees
    private static int regularCounter = 0;
    
    // Volatile variable - visibility guarantees
    private static volatile int volatileCounter = 0;
    
    // Atomic variable - visibility and atomicity guarantees
    private static AtomicInteger atomicCounter = new AtomicInteger(0);
    
    // Synchronized method - visibility and atomicity guarantees
    private static synchronized void incrementSynchronizedCounter() {
        regularCounter++;
    }
    
    public static void main(String[] args) throws InterruptedException {
        // Demonstrate visibility issues with regular variables
        demonstrateVisibilityIssue();
        
        // Compare different synchronization mechanisms
        compareVisibilityMechanisms();
        
        // Demonstrate the happens-before relationship
        demonstrateHappensBefore();
    }
    
    private static void demonstrateVisibilityIssue() throws InterruptedException {
        System.out.println("\n=== Demonstrating Visibility Issue ===");
        
        // Reset counters
        regularCounter = 0;
        
        // Flag to control the loop in the background thread
        boolean[] keepRunning = {true};
        
        // Start a background thread that increments the counter
        Thread backgroundThread = new Thread(() -> {
            System.out.println("Background thread started");
            while (keepRunning[0]) {
                regularCounter++;
            }
            System.out.println("Background thread finished, counter = " + regularCounter);
        });
        
        backgroundThread.start();
        
        // Give the background thread time to start
        Thread.sleep(100);
        
        // Main thread reads the counter value
        System.out.println("Main thread sees counter = " + regularCounter);
        
        // Signal the background thread to stop
        keepRunning[0] = false;
        
        // Wait for the background thread to finish
        backgroundThread.join();
        
        System.out.println("Final counter value = " + regularCounter);
        System.out.println("Note: Without proper synchronization, the main thread might not see");
        System.out.println("all updates made by the background thread immediately.");
    }
    
    private static void compareVisibilityMechanisms() throws InterruptedException {
        System.out.println("\n=== Comparing Visibility Mechanisms ===");
        
        // Number of threads and iterations
        int numThreads = 10;
        int iterationsPerThread = 100_000;
        
        // Reset counters
        regularCounter = 0;
        volatileCounter = 0;
        atomicCounter.set(0);
        
        // CountDownLatch to coordinate thread start and completion
        CountDownLatch startLatch = new CountDownLatch(1);
        CountDownLatch endLatch = new CountDownLatch(numThreads);
        
        // Create and start threads
        for (int i = 0; i < numThreads; i++) {
            Thread t = new Thread(() -> {
                try {
                    // Wait for the start signal
                    startLatch.await();
                    
                    // Increment each counter type
                    for (int j = 0; j < iterationsPerThread; j++) {
                        // Regular variable - no synchronization
                        regularCounter++;
                        
                        // Volatile variable - visibility but not atomicity
                        volatileCounter++;
                        
                        // Synchronized method - visibility and atomicity
                        incrementSynchronizedCounter();
                        
                        // Atomic variable - visibility and atomicity
                        atomicCounter.incrementAndGet();
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    // Signal completion
                    endLatch.countDown();
                }
            });
            t.start();
        }
        
        // Start all threads simultaneously
        startLatch.countDown();
        
        // Wait for all threads to complete
        endLatch.await();
        
        // Expected total
        int expectedTotal = numThreads * iterationsPerThread;
        
        // Print results
        System.out.println("Expected total: " + expectedTotal);
        System.out.println("Regular variable: " + regularCounter + 
                          " (Missing " + (expectedTotal - regularCounter) + " updates)");
        System.out.println("Volatile variable: " + volatileCounter + 
                          " (Missing " + (expectedTotal - volatileCounter) + " updates)");
        System.out.println("Synchronized method: " + regularCounter + 
                          " (Missing " + (expectedTotal - regularCounter) + " updates)");
        System.out.println("Atomic variable: " + atomicCounter.get() + 
                          " (Missing " + (expectedTotal - atomicCounter.get()) + " updates)");
        
        System.out.println("\nNote: Regular and volatile variables suffer from race conditions");
        System.out.println("because increment operations (x++) are not atomic.");
        System.out.println("Synchronized methods and atomic variables provide both");
        System.out.println("visibility and atomicity guarantees.");
    }
    
    private static void demonstrateHappensBefore() throws InterruptedException {
        System.out.println("\n=== Demonstrating Happens-Before Relationship ===");
        
        // Variables to demonstrate happens-before
        int[] data = new int[10];
        boolean[] dataReady = {false};
        
        // Thread that writes data
        Thread writerThread = new Thread(() -> {
            // Initialize the data
            for (int i = 0; i < data.length; i++) {
                data[i] = i * 10;
            }
            
            // Setting a volatile variable establishes a happens-before relationship
            // All writes before this point are visible to reads after the volatile write is observed
            dataReady[0] = true; // This is a volatile write (using an array to simulate)
            
            System.out.println("Writer thread: Data prepared and flag set");
        });
        
        // Thread that reads data
        Thread readerThread = new Thread(() -> {
            System.out.println("Reader thread: Waiting for data to be ready");
            
            // Spin until the flag is set
            // This is a volatile read (using an array to simulate)
            while (!dataReady[0]) {
                // Busy-wait
                Thread.yield();
            }
            
            // At this point, all writes performed by the writer thread before setting
            // the flag are visible to this thread due to the happens-before relationship
            System.out.println("Reader thread: Data is ready, reading values");
            for (int i = 0; i < data.length; i++) {
                System.out.println("data[" + i + "] = " + data[i]);
            }
        });
        
        // Start the threads
        readerThread.start();
        writerThread.start();
        
        // Wait for both threads to complete
        writerThread.join();
        readerThread.join();
        
        System.out.println("\nNote: The happens-before relationship ensures that");
        System.out.println("all memory writes by the writer thread before setting the flag");
        System.out.println("are visible to the reader thread after it observes the flag as true.");
        System.out.println("This is a fundamental concept in the Java Memory Model.");
    }
}
```

### 8.7 Implementing Parallel Processing with Fork/Join Framework

**Theory:**
- The Fork/Join Framework was introduced in Java 7 for parallel processing
- Based on the divide-and-conquer algorithm paradigm
- Key components:
  1. `ForkJoinPool`: A specialized thread pool for fork/join tasks
  2. `RecursiveTask<V>`: For tasks that return a result
  3. `RecursiveAction`: For tasks that don't return a result
  4. Work-stealing algorithm: Idle threads steal tasks from busy threads' queues
- Best suited for CPU-intensive tasks that can be broken down into smaller subtasks
- Follows the principle: if the task is small enough, solve it directly; otherwise, split it

**Hard Example:**

```java
import java.util.Arrays;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveAction;
import java.util.concurrent.RecursiveTask;
import java.util.concurrent.ThreadLocalRandom;
import java.util.concurrent.TimeUnit;

public class ForkJoinExample {
    // Common pool for all examples
    private static final ForkJoinPool POOL = ForkJoinPool.commonPool();
    
    public static void main(String[] args) throws Exception {
        try {
            // Example 1: Parallel array sorting with RecursiveAction
            demonstrateParallelArraySort();
            
            // Example 2: Parallel sum calculation with RecursiveTask
            demonstrateParallelSum();
            
            // Example 3: Parallel Fibonacci calculation with RecursiveTask
            demonstrateParallelFibonacci();
            
            // Example 4: Parallel matrix multiplication
            demonstrateMatrixMultiplication();
            
        } finally {
            // Shutdown the pool
            POOL.shutdown();
            POOL.awaitTermination(1, TimeUnit.MINUTES);
        }
    }
    
    private static void demonstrateParallelArraySort() {
        System.out.println("\n=== Parallel Array Sort Example ===");
        
        // Create a large array of random integers
        int size = 10_000_000;
        int[] array = new int[size];
        for (int i = 0; i < size; i++) {
            array[i] = ThreadLocalRandom.current().nextInt(1_000_000);
        }
        
        // Create a copy for sequential sort comparison
        int[] arrayCopy = Arrays.copyOf(array, array.length);
        
        // Sequential sort
        long start = System.nanoTime();
        Arrays.sort(arrayCopy);
        long sequentialTime = System.nanoTime() - start;
        
        // Parallel sort using Fork/Join
        start = System.nanoTime();
        ParallelArraySorter sorter = new ParallelArraySorter(array, 0, array.length - 1);
        POOL.invoke(sorter);
        long parallelTime = System.nanoTime() - start;
        
        // Verify the result
        boolean isSorted = true;
        for (int i = 1; i < array.length; i++) {
            if (array[i - 1] > array[i]) {
                isSorted = false;
                break;
            }
        }
        
        System.out.println("Array size: " + size);
        System.out.println("Sequential sort time: " + sequentialTime / 1_000_000.0 + " ms");
        System.out.println("Parallel sort time: " + parallelTime / 1_000_000.0 + " ms");
        System.out.println("Speedup: " + (double) sequentialTime / parallelTime);
        System.out.println("Is correctly sorted: " + isSorted);
    }
    
    private static void demonstrateParallelSum() {
        System.out.println("\n=== Parallel Sum Example ===");
        
        // Create a large array of random integers
        int size = 200_000_000;
        long[] array = new long[size];
        long expectedSum = 0;
        for (int i = 0; i < size; i++) {
            array[i] = ThreadLocalRandom.current().nextInt(100);
            expectedSum += array[i];
        }
        
        // Sequential sum
        long start = System.nanoTime();
        long sequentialSum = 0;
        for (long value : array) {
            sequentialSum += value;
        }
        long sequentialTime = System.nanoTime() - start;
        
        // Parallel sum using Fork/Join
        start = System.nanoTime();
        SumTask sumTask = new SumTask(array, 0, array.length);
        long parallelSum = POOL.invoke(sumTask);
        long parallelTime = System.nanoTime() - start;
        
        System.out.println("Array size: " + size);
        System.out.println("Expected sum: " + expectedSum);
        System.out.println("Sequential sum: " + sequentialSum + " (" + sequentialTime / 1_000_000.0 + " ms)");
        System.out.println("Parallel sum: " + parallelSum + " (" + parallelTime / 1_000_000.0 + " ms)");
        System.out.println("Speedup: " + (double) sequentialTime / parallelTime);
    }
    
    private static void demonstrateParallelFibonacci() {
        System.out.println("\n=== Parallel Fibonacci Example ===");
        
        int n = 45; // Large enough to benefit from parallelism, but not too large
        
        // Sequential Fibonacci
        long start = System.nanoTime();
        long sequentialResult = fibonacciSequential(n);
        long sequentialTime = System.nanoTime() - start;
        
        // Parallel Fibonacci using Fork/Join
        start = System.nanoTime();
        FibonacciTask fibTask = new FibonacciTask(n);
        long parallelResult = POOL.invoke(fibTask);
        long parallelTime = System.nanoTime() - start;
        
        System.out.println("Computing Fibonacci number for n = " + n);
        System.out.println("Sequential result: " + sequentialResult + " (" + sequentialTime / 1_000_000.0 + " ms)");
        System.out.println("Parallel result: " + parallelResult + " (" + parallelTime / 1_000_000.0 + " ms)");
        System.out.println("Speedup: " + (double) sequentialTime / parallelTime);
    }
    
    private static void demonstrateMatrixMultiplication() {
        System.out.println("\n=== Parallel Matrix Multiplication Example ===");
        
        int size = 1000; // Matrix size (size x size)
        
        // Create random matrices
        int[][] a = new int[size][size];
        int[][] b = new int[size][size];
        int[][] c = new int[size][size]; // Result matrix for sequential
        int[][] d = new int[size][size]; // Result matrix for parallel
        
        // Initialize with random values
        for (int i = 0; i < size; i++) {
            for (int j = 0; j < size; j++) {
                a[i][j] = ThreadLocalRandom.current().nextInt(10);
                b[i][j] = ThreadLocalRandom.current().nextInt(10);
            }
        }
        
        // Sequential matrix multiplication
        long start = System.nanoTime();
        sequentialMatrixMultiply(a, b, c, size);
        long sequentialTime = System.nanoTime() - start;
        
        // Parallel matrix multiplication using Fork/Join
        start = System.nanoTime();
        MatrixMultiplyTask task = new MatrixMultiplyTask(a, b, d, 0, 0, 0, 0, 0, 0, size);
        POOL.invoke(task);
        long parallelTime = System.nanoTime() - start;
        
        // Verify the result
        boolean isCorrect = true;
        outer: for (int i = 0; i < size; i++) {
            for (int j = 0; j < size; j++) {
                if (c[i][j] != d[i][j]) {
                    isCorrect = false;
                    break outer;
                }
            }
        }
        
        System.out.println("Matrix size: " + size + "x" + size);
        System.out.println("Sequential multiplication time: " + sequentialTime / 1_000_000.0 + " ms");
        System.out.println("Parallel multiplication time: " + parallelTime / 1_000_000.0 + " ms");
        System.out.println("Speedup: " + (double) sequentialTime / parallelTime);
        System.out.println("Results match: " + isCorrect);
    }
    
    // Sequential Fibonacci implementation
    private static long fibonacciSequential(int n) {
        if (n <= 1) return n;
        return fibonacciSequential(n - 1) + fibonacciSequential(n - 2);
    }
    
    // Sequential matrix multiplication
    private static void sequentialMatrixMultiply(int[][] a, int[][] b, int[][] c, int size) {
        for (int i = 0; i < size; i++) {
            for (int j = 0; j < size; j++) {
                int sum = 0;
                for (int k = 0; k < size; k++) {
                    sum += a[i][k] * b[k][j];
                }
                c[i][j] = sum;
            }
        }
    }
    
    // RecursiveAction for parallel array sorting (quicksort)
    static class ParallelArraySorter extends RecursiveAction {
        private static final int THRESHOLD = 10000; // Threshold for sequential processing
        private final int[] array;
        private final int low;
        private final int high;
        
        ParallelArraySorter(int[] array, int low, int high) {
            this.array = array;
            this.low = low;
            this.high = high;
        }
        
        @Override
        protected void compute() {
            if (high - low <= THRESHOLD) {
                // If the array segment is small enough, sort it sequentially
                Arrays.sort(array, low, high + 1);
            } else {
                // Otherwise, partition the array and sort the partitions in parallel
                int pivot = partition(array, low, high);
                invokeAll(
                    new ParallelArraySorter(array, low, pivot - 1),
                    new ParallelArraySorter(array, pivot + 1, high)
                );
            }
        }
        
        private int partition(int[] array, int low, int high) {
            // Choose the rightmost element as the pivot
            int pivot = array[high];
            int i = low - 1;
            
            // Move all elements smaller than the pivot to the left
            for (int j = low; j < high; j++) {
                if (array[j] <= pivot) {
                    i++;
                    swap(array, i, j);
                }
            }
            
            // Move the pivot to its final position
            swap(array, i + 1, high);
            return i + 1;
        }
        
        private void swap(int[] array, int i, int j) {
            int temp = array[i];
            array[i] = array[j];
            array[j] = temp;
        }
    }
    
    // RecursiveTask for parallel sum calculation
    static class SumTask extends RecursiveTask<Long> {
        private static final int THRESHOLD = 10000; // Threshold for sequential processing
        private final long[] array;
        private final int start;
        private final int end;
        
        SumTask(long[] array, int start, int end) {
            this.array = array;
            this.start = start;
            this.end = end;
        }
        
        @Override
        protected Long compute() {
            int length = end - start;
            if (length <= THRESHOLD) {
                // If the array segment is small enough, sum it sequentially
                return sumSequentially();
            }
            
            // Otherwise, split the array and sum the parts in parallel
            int mid = start + length / 2;
            SumTask leftTask = new SumTask(array, start, mid);
            SumTask rightTask = new SumTask(array, mid, end);
            
            // Fork the left task (execute asynchronously)
            leftTask.fork();
            
            // Compute the right task directly
            long rightResult = rightTask.compute();
            
            // Join with the left task (wait for its result)
            long leftResult = leftTask.join();
            
            // Combine the results
            return leftResult + rightResult;
        }
        
        private long sumSequentially() {
            long sum = 0;
            for (int i = start; i < end; i++) {
                sum += array[i];
            }
            return sum;
        }
    }
    
    // RecursiveTask for parallel Fibonacci calculation
    static class FibonacciTask extends RecursiveTask<Long> {
        private static final int THRESHOLD = 16; // Threshold for sequential processing
        private final int n;
        
        FibonacciTask(int n) {
            this.n = n;
        }
        
        @Override
        protected Long compute() {
            if (n <= THRESHOLD) {
                // If n is small enough, compute sequentially
                return fibonacciSequential(n);
            }
            
            // Otherwise, split the calculation and compute in parallel
            FibonacciTask f1 = new FibonacciTask(n - 1);
            FibonacciTask f2 = new FibonacciTask(n - 2);
            
            // Fork f1 (execute asynchronously)
            f1.fork();
            
            // Compute f2 directly
            long f2Result = f2.compute();
            
            // Join with f1 (wait for its result)
            long f1Result = f1.join();
            
            // Combine the results
            return f1Result + f2Result;
        }
    }
    
    // RecursiveAction for parallel matrix multiplication
    static class MatrixMultiplyTask extends RecursiveAction {
        private static final int THRESHOLD = 64; // Threshold for sequential processing
        private final int[][] a;
        private final int[][] b;
        private final int[][] c; // Result matrix
        private final int aRow;
        private final int aCol;
        private final int bRow;
        private final int bCol;
        private final int cRow;
        private final int cCol;
        private final int size;
        
        MatrixMultiplyTask(int[][] a, int[][] b, int[][] c, 
                          int aRow, int aCol, int bRow, int bCol, int cRow, int cCol, int size) {
            this.a = a;
            this.b = b;
            this.c = c;
            this.aRow = aRow;
            this.aCol = aCol;
            this.bRow = bRow;
            this.bCol = bCol;
            this.cRow = cRow;
            this.cCol = cCol;
            this.size = size;
        }
        
        @Override
        protected void compute() {
            if (size <= THRESHOLD) {
                // If the matrix is small enough, multiply sequentially
                multiplySequentially();
            } else {
                // Otherwise, split the matrices and multiply in parallel
                int newSize = size / 2;
                
                // Create 8 subtasks for the 8 submatrices
                invokeAll(
                    // C11 = A11 * B11 + A12 * B21
                    new MatrixMultiplyTask(a, b, c, aRow, aCol, bRow, bCol, cRow, cCol, newSize),
                    new MatrixMultiplyTask(a, b, c, aRow, aCol + newSize, bRow + newSize, bCol, cRow, cCol, newSize),
                    
                    // C12 = A11 * B12 + A12 * B22
                    new MatrixMultiplyTask(a, b, c, aRow, aCol, bRow, bCol + newSize, cRow, cCol + newSize, newSize),
                    new MatrixMultiplyTask(a, b, c, aRow, aCol + newSize, bRow + newSize, bCol + newSize, cRow, cCol + newSize, newSize),
                    
                    // C21 = A21 * B11 + A22 * B21
                    new MatrixMultiplyTask(a, b, c, aRow + newSize, aCol, bRow, bCol, cRow + newSize, cCol, newSize),
                    new MatrixMultiplyTask(a, b, c, aRow + newSize, aCol + newSize, bRow + newSize, bCol, cRow + newSize, cCol, newSize),
                    
                    // C22 = A21 * B12 + A22 * B22
                    new MatrixMultiplyTask(a, b, c, aRow + newSize, aCol, bRow, bCol + newSize, cRow + newSize, cCol + newSize, newSize),
                    new MatrixMultiplyTask(a, b, c, aRow + newSize, aCol + newSize, bRow + newSize, bCol + newSize, cRow + newSize, cCol + newSize, newSize)
                );
            }
        }
        
        private void multiplySequentially() {
            for (int i = 0; i < size; i++) {
                for (int j = 0; j < size; j++) {
                    int sum = 0;
                    for (int k = 0; k < size; k++) {
                        sum += a[aRow + i][aCol + k] * b[bRow + k][bCol + j];
                    }
                    c[cRow + i][cCol + j] += sum;
                }
            }
        }
    }
}
```
