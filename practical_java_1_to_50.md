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
