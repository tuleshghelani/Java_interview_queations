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
