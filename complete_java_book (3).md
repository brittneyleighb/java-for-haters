    
    public void leakViaThreadLocal() {
        threadLocal.set(new Object());
        // Should call threadLocal.remove() when done!
    }
}
```

### OutOfMemoryError Types

```java
public class OutOfMemoryExamples {
    // Heap space exhaustion
    public void heapSpaceError() {
        List<Object> objects = new ArrayList<>();
        while (true) {
            objects.add(new Object()); // Eventually: OutOfMemoryError: Java heap space
        }
    }
    
    // Metaspace exhaustion (Java 8+)
    public void metaspaceError() {
        // Dynamically load classes until metaspace is full
        // OutOfMemoryError: Metaspace
    }
    
    // Stack overflow (not OOM, but memory-related)
    public void stackOverflowError() {
        stackOverflowError(); // Infinite recursion
        // StackOverflowError
    }
}
```

---

## Chapter 26: Performance Tuning - Making Slow Code Slightly Less Slow

### JVM Performance Monitoring

```java
import java.lang.management.*;

public class PerformanceMonitoring {
    public void monitorJVMPerformance() {
        // Memory usage
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
        
        System.out.println("Heap Memory Usage:");
        System.out.println("  Used: " + (heapUsage.getUsed() / 1024 / 1024) + " MB");
        System.out.println("  Max: " + (heapUsage.getMax() / 1024 / 1024) + " MB");
        
        // Garbage Collection
        List<GarbageCollectorMXBean> gcBeans = ManagementFactory.getGarbageCollectorMXBeans();
        for (GarbageCollectorMXBean gcBean : gcBeans) {
            System.out.println("GC " + gcBean.getName() + ":");
            System.out.println("  Collections: " + gcBean.getCollectionCount());
            System.out.println("  Time: " + gcBean.getCollectionTime() + " ms");
        }
        
        // Thread information
        ThreadMXBean threadBean = ManagementFactory.getThreadMXBean();
        System.out.println("Thread count: " + threadBean.getThreadCount());
        System.out.println("Peak thread count: " + threadBean.getPeakThreadCount());
    }
}
```

### Microbenchmarking with JMH

```java
import org.openjdk.jmh.annotations.*;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@State(Scope.Thread)
public class StringConcatenationBenchmark {
    
    private static final int ITERATIONS = 1000;
    
    @Benchmark
    public String stringConcatenation() {
        String result = "";
        for (int i = 0; i < ITERATIONS; i++) {
            result += "a"; // Inefficient - creates new String each time
        }
        return result;
    }
    
    @Benchmark
    public String stringBuilderConcatenation() {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < ITERATIONS; i++) {
            sb.append("a"); // Efficient - modifies existing buffer
        }
        return sb.toString();
    }
    
    @Benchmark
    public String stringBufferConcatenation() {
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < ITERATIONS; i++) {
            sb.append("a"); // Thread-safe but slower than StringBuilder
        }
        return sb.toString();
    }
}
```

### Performance Anti-Patterns

```java
public class PerformanceAntiPatterns {
    // Anti-pattern 1: Unnecessary object creation in loops
    public void unnecessaryObjectCreation() {
        List<String> results = new ArrayList<>();
        
        // BAD: Creates new SimpleDateFormat in each iteration
        for (int i = 0; i < 1000; i++) {
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            results.add(sdf.format(new Date()));
        }
        
        // GOOD: Create once, reuse
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        for (int i = 0; i < 1000; i++) {
            results.add(sdf.format(new Date()));
        }
    }
    
    // Anti-pattern 2: Using size() in loop condition
    public void inefficientLoop() {
        List<String> items = Arrays.asList("a", "b", "c", "d", "e");
        
        // BAD: Calls size() in each iteration
        for (int i = 0; i < items.size(); i++) {
            System.out.println(items.get(i));
        }
        
        // GOOD: Cache the size
        int size = items.size();
        for (int i = 0; i < size; i++) {
            System.out.println(items.get(i));
        }
        
        // BEST: Use enhanced for loop
        for (String item : items) {
            System.out.println(item);
        }
    }
    
    // Anti-pattern 3: Excessive synchronization
    public class OverSynchronized {
        private int counter = 0;
        
        // BAD: Synchronizing simple read
        public synchronized int getCounter() {
            return counter; // Reading int is atomic, no sync needed
        }
        
        // GOOD: Only synchronize writes
        public synchronized void incrementCounter() {
            counter++; // This needs synchronization
        }
        
        public int getCounterUnsafe() {
            return counter; // Fast read
        }
    }
}
```

---

## Chapter 27: Modern Java - Lambda Expressions and Stream API

### Lambda Expressions: Functional Programming in Java

```java
import java.util.*;
import java.util.function.*;

public class LambdaExamples {
    public void basicLambdas() {
        // Old way: Anonymous inner class
        Runnable oldRunnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("Old way");
            }
        };
        
        // New way: Lambda expression
        Runnable newRunnable = () -> System.out.println("Lambda way");
        
        // With parameters
        Comparator<String> oldComparator = new Comparator<String>() {
            @Override
            public int compare(String a, String b) {
                return a.compareTo(b);
            }
        };
        
        Comparator<String> newComparator = (a, b) -> a.compareTo(b);
        
        // Even shorter with method reference
        Comparator<String> methodRef = String::compareTo;
    }
    
    public void functionalInterfaces() {
        // Predicate<T> - takes T, returns boolean
        Predicate<String> isEmpty = String::isEmpty;
        Predicate<Integer> isPositive = x -> x > 0;
        
        // Function<T, R> - takes T, returns R
        Function<String, Integer> stringLength = String::length;
        Function<Integer, String> intToString = Object::toString;
        
        // Consumer<T> - takes T, returns void
        Consumer<String> printer = System.out::println;
        
        // Supplier<T> - takes nothing, returns T
        Supplier<String> stringSupplier = () -> "Hello World";
        
        // BiFunction<T, U, R> - takes T and U, returns R
        BiFunction<String, String, String> concatenator = (a, b) -> a + b;
    }
}
```

### Stream API: Functional Data Processing

```java
import java.util.stream.*;

public class StreamExamples {
    public void basicStreamOperations() {
        List<String> words = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");
        
        // Filter and collect
        List<String> longWords = words.stream()
            .filter(word -> word.length() > 5)
            .collect(Collectors.toList());
        
        // Map (transform) elements
        List<Integer> lengths = words.stream()
            .map(String::length)
            .collect(Collectors.toList());
        
        // Chain multiple operations
        List<String> result = words.stream()
            .filter(word -> word.startsWith("a"))
            .map(String::toUpperCase)
            .sorted()
            .collect(Collectors.toList());
        
        System.out.println("Long words: " + longWords);
        System.out.println("Lengths: " + lengths);
        System.out.println("Filtered result: " + result);
    }
    
    public void streamAggregations() {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // Terminal operations
        int sum = numbers.stream().mapToInt(Integer::intValue).sum();
        OptionalDouble average = numbers.stream().mapToInt(Integer::intValue).average();
        OptionalInt max = numbers.stream().mapToInt(Integer::intValue).max();
        long count = numbers.stream().count();
        
        // Reduction
        Optional<Integer> product = numbers.stream()
            .reduce((a, b) -> a * b);
        
        // Custom collector
        String joined = numbers.stream()
            .map(String::valueOf)
            .collect(Collectors.joining(", ", "[", "]"));
        
        System.out.println("Sum: " + sum);
        System.out.println("Average: " + average.orElse(0.0));
        System.out.println("Max: " + max.orElse(0));
        System.out.println("Count: " + count);
        System.out.println("Product: " + product.orElse(0));
        System.out.println("Joined: " + joined);
    }
    
    public void groupingAndPartitioning() {
        List<Person> people = Arrays.asList(
            new Person("Alice", 30, "Engineer"),
            new Person("Bob", 25, "Designer"),
            new Person("Charlie", 35, "Engineer"),
            new Person("Diana", 28, "Designer")
        );
        
        // Group by profession
        Map<String, List<Person>> byProfession = people.stream()
            .collect(Collectors.groupingBy(Person::getProfession));
        
        // Partition by age
        Map<Boolean, List<Person>> byAge = people.stream()
            .collect(Collectors.partitioningBy(person -> person.getAge() >= 30));
        
        // Complex grouping with downstream collectors
        Map<String, Double> averageAgeByProfession = people.stream()
            .collect(Collectors.groupingBy(
                Person::getProfession,
                Collectors.averagingDouble(Person::getAge)
            ));
        
        System.out.println("By profession: " + byProfession);
        System.out.println("By age (30+): " + byAge);
        System.out.println("Average age by profession: " + averageAgeByProfession);
    }
    
    public void parallelStreams() {
        List<Integer> largeList = IntStream.rangeClosed(1, 1000000)
                                          .boxed()
                                          .collect(Collectors.toList());
        
        // Sequential processing
        long start = System.currentTimeMillis();
        long sequentialSum = largeList.stream()
            .mapToLong(Integer::longValue)
            .sum();
        long sequentialTime = System.currentTimeMillis() - start;
        
        // Parallel processing
        start = System.currentTimeMillis();
        long parallelSum = largeList.parallelStream()
            .mapToLong(Integer::longValue)
            .sum();
        long parallelTime = System.currentTimeMillis() - start;
        
        System.out.println("Sequential sum: " + sequentialSum + " (took " + sequentialTime + "ms)");
        System.out.println("Parallel sum: " + parallelSum + " (took " + parallelTime + "ms)");
        System.out.println("Speedup: " + ((double) sequentialTime / parallelTime) + "x");
    }
}

class Person {
    private String name;
    private int age;
    private String profession;
    
    public Person(String name, int age, String profession) {
        this.name = name;
        this.age = age;
        this.profession = profession;
    }
    
    // Getters
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getProfession() { return profession; }
    
    @Override
    public String toString() {
        return name + "(" + age + ", " + profession + ")";
    }
}
```

### Optional: Handling Null the Modern Way

```java
import java.util.Optional;

public class OptionalExamples {
    public void basicOptional() {
        // Creating Optionals
        Optional<String> empty = Optional.empty();
        Optional<String> notEmpty = Optional.of("Hello");
        Optional<String> nullable = Optional.ofNullable(getString()); // might be null
        
        // Checking presence
        if (notEmpty.isPresent()) {
            System.out.println("Value: " + notEmpty.get());
        }
        
        // Better way: using ifPresent
        notEmpty.ifPresent(System.out::println);
        
        // Providing default values
        String result = empty.orElse("Default value");
        String result2 = empty.orElseGet(() -> "Generated default");
        
        // Throwing exceptions
        String result3 = empty.orElseThrow(() -> new RuntimeException("No value"));
    }
    
    public void optionalChaining() {
        Optional<String> optional = Optional.of("Hello World");
        
        // Transform value if present
        Optional<Integer> length = optional.map(String::length);
        Optional<String> upper = optional.map(String::toUpperCase);
        
        // Filter based on condition
        Optional<String> longString = optional.filter(s -> s.length() > 5);
        
        // FlatMap for nested Optionals
        Optional<String> nested = optional.flatMap(this::processString);
        
        // Chaining operations
        String result = optional
            .filter(s -> s.startsWith("Hello"))
            .map(String::toUpperCase)
            .orElse("NOT FOUND");
        
        System.out.println("Result: " + result);
    }
    
    private Optional<String> processString(String input) {
        return input.isEmpty() ? Optional.empty() : Optional.of(input.trim());
    }
    
    private String getString() {
        return Math.random() > 0.5 ? "Random string" : null;
    }
    
    // Anti-pattern: Don't do this!
    public void optionalAntiPatterns() {
        Optional<String> optional = Optional.of("Hello");
        
        // BAD: Using get() without checking
        // String value = optional.get(); // Can throw NoSuchElementException
        
        // BAD: Checking isPresent() then get()
        if (optional.isPresent()) {
            String value = optional.get(); // Just use ifPresent() instead
        }
        
        // BAD: Using Optional for fields
        // private Optional<String> name; // Use null instead
        
        // GOOD: Proper usage
        optional.ifPresent(System.out::println);
        String value = optional.orElse("Default");
    }
}
```

---

## Part VIII: Survival Skills

## Chapter 28: Debugging - The Art of Controlled Crying

### Common Java Exceptions and How to Fix Them

```java
public class CommonExceptions {
    // 1. NullPointerException - The most feared
    public void nullPointerExample() {
        String text = null;
        // int length = text.length(); // NPE!
        
        // Solution: Null checks
        if (text != null) {
            int length = text.length();
        }
        
        // Better: Use Optional
        Optional<String> optionalText = Optional.ofNullable(text);
        int length = optionalText.map(String::length).orElse(0);
    }
    
    // 2. ArrayIndexOutOfBoundsException
    public void arrayIndexExample() {
        int[] array = {1, 2, 3};
        // int value = array[5]; // AIOOBE!
        
        // Solution: Bounds checking
        int index = 5;
        if (index >= 0 && index < array.length) {
            int value = array[index];
        }
    }
    
    // 3. ClassCastException
    public void classCastExample() {
        Object obj = "Hello";
        // Integer number = (Integer) obj; // CCE!
        
        // Solution: instanceof check
        if (obj instanceof Integer) {
            Integer number = (Integer) obj;
        }
    }
    
    // 4. NumberFormatException
    public void numberFormatExample() {
        String text = "not a number";
        try {
            int number = Integer.parseInt(text); // NFE!
        } catch (NumberFormatException e) {
            System.out.println("Invalid number format: " + text);
            // Provide default or handle gracefully
        }
    }
    
    // 5. ConcurrentModificationException
    public void concurrentModificationExample() {
        List<String> list = new ArrayList<>(Arrays.asList("a", "b", "c"));
        
        // BAD: Modifying while iterating
        /*
        for (String item : list) {
            if (item.equals("b")) {
                list.remove(item); // CME!
            }
        }
        */
        
        // GOOD: Use iterator
        Iterator<String> iterator = list.iterator();
        while (iterator.hasNext()) {
            String item = iterator.next();
            if (item.equals("b")) {
                iterator.remove(); // Safe removal
            }
        }
        
        // BETTER: Use removeIf (Java 8+)
        list.removeIf(item -> item.equals("b"));
    }
}
```

### Debugging Techniques

```java
public class DebuggingTechniques {
    private static final Logger logger = LoggerFactory.getLogger(DebuggingTechniques.class);
    
    public void loggingForDebugging() {
        String username = "alice";
        int age = 25;
        
        // Different log levels
        logger.debug("Processing user: {}", username);
        logger.info("User {} is {} years old", username, age);
        logger.warn("User {} age seems low: {}", username, age);
        logger.error("Failed to process user: {}", username);
        
        // Conditional logging
        if (logger.isDebugEnabled()) {
            String expensiveDebugInfo = calculateExpensiveDebugInfo();
            logger.debug("Debug info: {}", expensiveDebugInfo);
        }
    }
    
    public void assertionsForDebugging() {
        int value = getValue();
        
        // Enable assertions with -ea JVM flag
        assert value >= 0 : "Value should be non-negative: " + value;
        assert value < 100 : "Value should be less than 100: " + value;
        
        // Assertions are disabled by default in production
    }
    
    public void debuggingWithStackTraces() {
        try {
            riskyOperation();
        } catch (Exception e) {
            // Print full stack trace
            e.printStackTrace();
            
            // Log with stack trace
            logger.error("Operation failed", e);
            
            // Get stack trace as string
            StringWriter sw = new StringWriter();
            PrintWriter pw = new PrintWriter(sw);
            e.printStackTrace(pw);
            String stackTrace = sw.toString();
        }
    }
    
    private String calculateExpensiveDebugInfo() {
        // Simulate expensive operation
        return "Expensive debug calculation result";
    }
    
    private int getValue() {
        return 42;
    }
    
    private void riskyOperation() throws Exception {
        throw new RuntimeException("Something went wrong");
    }
}
```

### IDE Debugging Features

```java
public class DebuggerExample {
    public static void main(String[] args) {
        DebuggerExample example = new DebuggerExample();
        example.demonstrateDebugging();
    }
    
    public void demonstrateDebugging() {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        int sum = 0;
        
        // Set breakpoint on next line
        for (Integer number : numbers) {
            sum += number; // Step into this line
            System.out.println("Current sum: " + sum); // Step over this line
        }
        
        // Conditional breakpoint: sum > 10
        System.out.println("Final sum: " + sum);
        
        // Watch variables: numbers, sum, number
    }
    
    public void debugComplexObject() {
        Person person = new Person("Alice", 30, "Engineer");
        
        // Set breakpoint and inspect object
        String greeting = createGreeting(person);
        System.out.println(greeting);
    }
    
    private String createGreeting(Person person) {
        // Step into method and inspect parameters
        return "Hello, " + person.getName() + "!";
    }
}

/*
Debugger Actions:
- F8: Step Over - Execute current line, don't enter methods
- F7: Step Into - Enter method calls
- Shift+F8: Step Out - Exit current method
- F9: Resume - Continue execution
- Ctrl+F8: Toggle breakpoint
- Ctrl+Shift+F8: View all breakpoints

Breakpoint Types:
- Line breakpoint: Stop at specific line
- Conditional breakpoint: Stop when condition is true
- Exception breakpoint: Stop when exception is thrown
- Method breakpoint: Stop when method is entered/exited
*/
```

### Performance Debugging

```java
public class PerformanceDebugging {
    public void profileCodePerformance() {
        long startTime = System.currentTimeMillis();
        
        // Code to profile
        List<String> data = generateLargeDataSet();
        processData(data);
        
        long endTime = System.currentTimeMillis();
        System.out.println("Operation took: " + (endTime - startTime) + "ms");
        
        // Memory usage
        Runtime runtime = Runtime.getRuntime();
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        System.out.println("Memory used: " + (usedMemory / 1024 / 1024) + "MB");
    }
    
    public void nanoTimeProfiling() {
        // More precise timing
        long startNano = System.nanoTime();
        
        expensiveOperation();
        
        long endNano = System.nanoTime();
        System.out.println("Nano time: " + (endNano - startNano) + " ns");
    }
    
    private List<String> generateLargeDataSet() {
        return IntStream.range(0, 100000)
                       .mapToObj(i -> "Data item " + i)
                       .collect(Collectors.toList());
    }
    
    private void processData(List<String> data) {
        // Simulate data processing
        data.stream()
            .filter(s -> s.contains("5"))
            .map(String::toUpperCase)
            .collect(Collectors.toList());
    }
    
    private void expensiveOperation() {
        // Simulate expensive operation
        for (int i = 0; i < 1000; i++) {
            Math.sqrt(i);
        }
    }
}
```

---

## Chapter 29: Best Practices - Or How to Write Less Terrible Code

### Code Organization and Structure

```java
// GOOD: Well-structured class
public class UserService {
    // Constants first
    private static final int MAX_LOGIN_ATTEMPTS = 3;
    private static final String DEFAULT_ROLE = "USER";
    
    // Dependencies
    private final UserRepository userRepository;
    private final EmailService emailService;
    private final Logger logger = LoggerFactory.getLogger(UserService.class);
    
    // Constructor
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = Objects.requireNonNull(userRepository);
        this.emailService = Objects.requireNonNull(emailService);
    }
    
    // Public methods first
    public User createUser(CreateUserRequest request) {
        validateRequest(request);
        
        User user = User.builder()
            .username(request.getUsername())
            .email(request.getEmail())
            .role(DEFAULT_ROLE)
            .build();
        
        User savedUser = userRepository.save(user);
        emailService.sendWelcomeEmail(savedUser);
        
        logger.info("Created user: {}", savedUser.getUsername());
        return savedUser;
    }
    
    // Private methods last
    private void validateRequest(CreateUserRequest request) {
        Objects.requireNonNull(request, "Request cannot be null");
        
        if (request.getUsername() == null || request.getUsername().trim().isEmpty()) {
            throw new IllegalArgumentException("Username cannot be empty");
        }
        
        if (!isValidEmail(request.getEmail())) {
            throw new IllegalArgumentException("Invalid email format");
        }
    }
    
    private boolean isValidEmail(String email) {
        return email != null && email.contains("@") && email.contains(".");
    }
}
```

### Naming Conventions

```java
public class NamingBestPractices {
    // GOOD: Clear, descriptive names
    private int maxRetryAttempts = 3;
    private List<Customer> activeCustomers = new ArrayList<>();
    private boolean isUserLoggedIn = false;
    
    // BAD: Unclear abbreviations
    // private int mra = 3;
    // private List<Customer> actCust = new ArrayList<>();
    // private boolean usrLog = false;
    
    // GOOD: Method names are verbs
    public void calculateTotalPrice() { }
    public boolean isEligibleForDiscount() { }
    public Customer findCustomerById(Long id) { return null; }
    
    // GOOD: Class names are nouns
    public class CustomerService { }
    public class PaymentProcessor { }
    public class EmailValidator { }
    
    // GOOD: Constants are UPPER_SNAKE_CASE
    private static final int MAX_UPLOAD_SIZE_BYTES = 1024 * 1024 * 10; // 10MB
    private static final String ERROR_MESSAGE_TEMPLATE = "Error processing request: %s";
    
    // GOOD: Package names are lowercase
    // package com.company.project.service.impl;
}
```

### Error Handling Best Practices

```java
public class ErrorHandlingBestPractices {
    private static final Logger logger = LoggerFactory.getLogger(ErrorHandlingBestPractices.class);
    
    // GOOD: Specific exception handling
    public User findUserById(Long id) throws UserNotFoundException {
        Objects.requireNonNull(id, "User ID cannot be null");
        
        try {
            return userRepository.findById(id)
                .orElseThrow(() -> new UserNotFoundException("User not found with ID: " + id));
        } catch (DataAccessException e) {
            logger.error("Database error while finding user with ID: {}", id, e);
            throw new ServiceException("Unable to retrieve user", e);
        }
    }
    
    // GOOD: Resource management with try-with-resources
    public String readFileContent(String filename) throws IOException {
        try (BufferedReader reader = Files.newBufferedReader(Paths.get(filename))) {
            return reader.lines().collect(Collectors.joining("\n"));
        } catch (IOException e) {
            logger.error("Error reading file: {}", filename, e);
            throw e;
        }
    }
    
    // GOOD: Graceful degradation
    public String getUserDisplayName(User user) {
        try {
            if (user != null && user.getFullName() != null) {
                return user.getFullName();
            } else if (user != null && user.getUsername() != null) {
                return user.getUsername();
            } else {
                return "Anonymous User";
            }
        } catch (Exception e) {
            logger.warn("Error getting display name for user", e);
            return "Unknown User";
        }
    }
    
    // BAD: Swallowing exceptions
    public void badErrorHandling() {
        try {
            riskyOperation();
        } catch (Exception e) {
            // BAD: Silent failure
            // e.printStackTrace();
        }
    }
    
    private void riskyOperation() throws Exception {
        throw new RuntimeException("Something went wrong");
    }
}
```

### Performance Best Practices

```java
public class PerformanceBestPractices {
    // GOOD: Use StringBuilder for concatenation in loops
    public String concatenateStrings(List<String> strings) {
        StringBuilder sb = new StringBuilder();
        for (String str : strings) {
            sb.append(str).append(" ");
        }
        return sb.toString().trim();
    }
    
    // GOOD: Cache expensive computations
    private final Map<String, String> computationCache = new ConcurrentHashMap<>();
    
    public String expensiveComputation(String input) {
        return computationCache.computeIfAbsent(input, this::doExpensiveComputation);
    }
    
    private String doExpensiveComputation(String input) {
        // Simulate expensive operation
        try { Thread.sleep(100); } catch (InterruptedException e) {}
        return input.toUpperCase();
    }
    
    // GOOD: Use appropriate collection types
    public void collectionBestPractices() {
        // ArrayList for indexed access
        List<String> indexedList = new ArrayList<>();
        
        // LinkedList for frequent insertions/deletions
        List<String> linkedList = new LinkedList<>();
        
        // HashSet for unique elements and fast lookup
        Set<String> uniqueItems = new HashSet<>();
        
        // LinkedHashSet for unique elements with insertion order
        Set<String> orderedUniqueItems = new LinkedHashSet<>();
        
        // HashMap for key-value pairs
        Map<String, String> keyValuePairs = new HashMap<>();
        
        // TreeMap for sorted keys
        Map<String, String> sortedMap = new TreeMap<>();
    }
    
    // GOOD: Lazy initialization for expensive resources
    private volatile DataProcessor dataProcessor; // volatile for thread safety
    
    public DataProcessor getDataProcessor() {
        if (dataProcessor == null) {
            synchronized (this) {
                if (dataProcessor == null) {
                    dataProcessor = new DataProcessor(); // Expensive to create
                }
            }
        }
        return dataProcessor;
    }
}
```

### Thread Safety Best Practices

```java
public class ThreadSafetyBestPractices {
    // GOOD: Immutable objects are thread-safe
    public static class ImmutablePerson {
        private final String name;
        private final int age;
        
        public ImmutablePerson(String name, int age) {
            this.name = name;
            this.age = age;
        }
        
        public String getName() { return name; }
        public int    public void writeFile(String filename, String content) throws IOException {
        Files.writeString(Paths.get(filename), content);
    }
    
    public void copyFile(String source, String destination) throws IOException {
        Files.copy(Paths.get(source), Paths.get(destination), 
                  StandardCopyOption.REPLACE_EXISTING);
    }
    
    public void walkDirectory(String directory) throws IOException {
        Files.walk(Paths.get(directory))
             .filter(Files::isRegularFile)
             .forEach(System.out::println);
    }
}
```

### Serialization: Object Persistence

```java
import java.io.*;

public class Person implements Serializable {
    private static final long serialVersionUID = 1L;
    
    private String name;
    private int age;
    private transient String password;  // Won't be serialized
    
    // Constructors, getters, setters...
}

public class SerializationExample {
    public void saveObject(Person person, String filename) throws IOException {
        try (ObjectOutputStream oos = new ObjectOutputStream(
                new FileOutputStream(filename))) {
            oos.writeObject(person);
        }
    }
    
    public Person loadObject(String filename) throws IOException, ClassNotFoundException {
        try (ObjectInputStream ois = new ObjectInputStream(
                new FileInputStream(filename))) {
            return (Person) ois.readObject();
        }
    }
}
```

---

## Chapter 18: Reflection - Looking in the Mirror and Screaming

### Class Information at Runtime

```java
public class ReflectionExample {
    public void exploreClass(Object obj) {
        Class<?> clazz = obj.getClass();
        
        System.out.println("Class name: " + clazz.getName());
        System.out.println("Simple name: " + clazz.getSimpleName());
        System.out.println("Package: " + clazz.getPackage().getName());
        
        // Get superclass
        Class<?> superClass = clazz.getSuperclass();
        System.out.println("Superclass: " + superClass.getName());
        
        // Get interfaces
        Class<?>[] interfaces = clazz.getInterfaces();
        for (Class<?> iface : interfaces) {
            System.out.println("Implements: " + iface.getName());
        }
    }
}
```

### Field Access and Manipulation

```java
import java.lang.reflect.Field;

public class FieldReflection {
    public void manipulateFields(Object obj) throws Exception {
        Class<?> clazz = obj.getClass();
        
        // Get all fields (including private)
        Field[] fields = clazz.getDeclaredFields();
        
        for (Field field : fields) {
            field.setAccessible(true);  // Access private fields
            
            System.out.println("Field: " + field.getName());
            System.out.println("Type: " + field.getType());
            System.out.println("Value: " + field.get(obj));
            
            // Modify field value
            if (field.getType() == String.class) {
                field.set(obj, "Modified by reflection!");
            }
        }
    }
}
```

### Method Invocation

```java
import java.lang.reflect.Method;

public class MethodReflection {
    public void callMethods(Object obj) throws Exception {
        Class<?> clazz = obj.getClass();
        
        // Get all methods
        Method[] methods = clazz.getDeclaredMethods();
        
        for (Method method : methods) {
            method.setAccessible(true);
            
            System.out.println("Method: " + method.getName());
            System.out.println("Return type: " + method.getReturnType());
            
            // Call method with no parameters
            if (method.getParameterCount() == 0) {
                Object result = method.invoke(obj);
                System.out.println("Result: " + result);
            }
        }
        
        // Call specific method
        Method specificMethod = clazz.getMethod("toString");
        Object result = specificMethod.invoke(obj);
        System.out.println("toString result: " + result);
    }
}
```

### Dynamic Object Creation

```java
public class DynamicCreation {
    public Object createInstance(String className) throws Exception {
        Class<?> clazz = Class.forName(className);
        
        // Create instance using default constructor
        return clazz.getDeclaredConstructor().newInstance();
    }
    
    public Object createWithParameters(String className, Object... args) throws Exception {
        Class<?> clazz = Class.forName(className);
        
        // Find constructor with matching parameter types
        Class<?>[] paramTypes = new Class[args.length];
        for (int i = 0; i < args.length; i++) {
            paramTypes[i] = args[i].getClass();
        }
        
        return clazz.getConstructor(paramTypes).newInstance(args);
    }
}
```

---

## Chapter 19: Annotations - Metadata Madness

### Built-in Annotations

```java
public class AnnotationExamples {
    @Override
    public String toString() {  // Indicates method overrides parent
        return "AnnotationExamples{}";
    }
    
    @Deprecated
    public void oldMethod() {  // Marks method as deprecated
        // Legacy code
    }
    
    @SuppressWarnings("unchecked")
    public void suppressWarnings() {
        List rawList = new ArrayList();  // Warning suppressed
        rawList.add("item");
    }
}
```

### Custom Annotations

```java
import java.lang.annotation.*;

// Define custom annotation
@Retention(RetentionPolicy.RUNTIME)  // Available at runtime
@Target(ElementType.METHOD)          // Can only be applied to methods
public @interface Timed {
    String description() default "Execution time";
    boolean logResults() default true;
}

// Use custom annotation
public class TimedService {
    @Timed(description = "Database query", logResults = true)
    public String fetchData() {
        // Simulate database operation
        try { Thread.sleep(100); } catch (InterruptedException e) {}
        return "Data from database";
    }
    
    @Timed
    public void processData() {
        // Process data
    }
}
```

### Annotation Processing

```java
import java.lang.reflect.Method;

public class AnnotationProcessor {
    public void processAnnotations(Object obj) {
        Class<?> clazz = obj.getClass();
        
        for (Method method : clazz.getDeclaredMethods()) {
            if (method.isAnnotationPresent(Timed.class)) {
                Timed timed = method.getAnnotation(Timed.class);
                
                System.out.println("Found @Timed method: " + method.getName());
                System.out.println("Description: " + timed.description());
                System.out.println("Log results: " + timed.logResults());
                
                // Measure execution time
                long startTime = System.currentTimeMillis();
                try {
                    method.invoke(obj);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                long endTime = System.currentTimeMillis();
                
                if (timed.logResults()) {
                    System.out.println("Execution time: " + (endTime - startTime) + "ms");
                }
            }
        }
    }
}
```

---

## Part VI: Enterprise Nightmares

## Chapter 20: Design Patterns - When Simple Solutions Are Illegal

### Singleton Pattern: One Instance to Rule Them All

```java
public class Singleton {
    private static volatile Singleton instance;
    
    private Singleton() {}  // Private constructor
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {  // Double-checked locking
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
    
    // Better approach: enum singleton
    public enum BetterSingleton {
        INSTANCE;
        
        public void doSomething() {
            // Singleton behavior
        }
    }
}
```

### Factory Pattern: Object Creation Ceremony

```java
// Product interface
interface Animal {
    void makeSound();
}

// Concrete products
class Dog implements Animal {
    @Override
    public void makeSound() {
        System.out.println("Woof!");
    }
}

class Cat implements Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow!");
    }
}

// Factory
class AnimalFactory {
    public static Animal createAnimal(String type) {
        switch (type.toLowerCase()) {
            case "dog":
                return new Dog();
            case "cat":
                return new Cat();
            default:
                throw new IllegalArgumentException("Unknown animal: " + type);
        }
    }
}

// Usage
Animal dog = AnimalFactory.createAnimal("dog");
dog.makeSound();
```

### Observer Pattern: The Notification System

```java
import java.util.*;

// Observer interface
interface Observer {
    void update(String message);
}

// Subject
class NewsAgency {
    private List<Observer> observers = new ArrayList<>();
    private String news;
    
    public void addObserver(Observer observer) {
        observers.add(observer);
    }
    
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }
    
    public void setNews(String news) {
        this.news = news;
        notifyObservers();
    }
    
    private void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(news);
        }
    }
}

// Concrete observer
class NewsChannel implements Observer {
    private String name;
    
    public NewsChannel(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String news) {
        System.out.println(name + " received news: " + news);
    }
}
```

### Strategy Pattern: Swappable Algorithms

```java
// Strategy interface
interface PaymentStrategy {
    void pay(double amount);
}

// Concrete strategies
class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    
    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " using credit card " + cardNumber);
    }
}

class PayPalPayment implements PaymentStrategy {
    private String email;
    
    public PayPalPayment(String email) {
        this.email = email;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " using PayPal " + email);
    }
}

// Context
class ShoppingCart {
    private PaymentStrategy paymentStrategy;
    
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    
    public void checkout(double amount) {
        paymentStrategy.pay(amount);
    }
}
```

---

## Chapter 21: Frameworks - Spring Into Dependency Injection Hell

### Dependency Injection: The Magic Container

```java
// Without dependency injection
public class OrderService {
    private EmailService emailService = new EmailService();  // Tight coupling
    private PaymentService paymentService = new PaymentService();
    
    public void processOrder(Order order) {
        // Process order
        emailService.sendConfirmation(order);
        paymentService.processPayment(order);
    }
}

// With dependency injection
public class OrderService {
    private final EmailService emailService;
    private final PaymentService paymentService;
    
    // Constructor injection
    public OrderService(EmailService emailService, PaymentService paymentService) {
        this.emailService = emailService;
        this.paymentService = paymentService;
    }
    
    public void processOrder(Order order) {
        // Process order
        emailService.sendConfirmation(order);
        paymentService.processPayment(order);
    }
}
```

### Spring Framework Annotations

```java
import org.springframework.stereotype.*;
import org.springframework.beans.factory.annotation.*;

@Service  // Spring manages this as a service
public class UserService {
    
    @Autowired  // Spring injects the dependency
    private UserRepository userRepository;
    
    @Value("${app.default.role}")  // Inject property value
    private String defaultRole;
    
    public User createUser(String username) {
        User user = new User(username, defaultRole);
        return userRepository.save(user);
    }
}

@Repository  // Data access layer
public class UserRepository {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    public User save(User user) {
        // Save to database
        return user;
    }
}

@RestController  // Web controller
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody CreateUserRequest request) {
        User user = userService.createUser(request.getUsername());
        return ResponseEntity.ok(user);
    }
}
```

### Spring Boot: Convention Over Configuration

```java
@SpringBootApplication  // Combines @Configuration, @EnableAutoConfiguration, @ComponentScan
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}

// application.yml configuration
/*
server:
  port: 8080

spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop

app:
  default:
    role: USER
*/
```

---

## Chapter 22: Build Tools - Maven, Gradle, and XML Dependency Hell

### Maven: The XML Enthusiast

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- Project coordinates -->
    <groupId>com.example</groupId>
    <artifactId>my-awesome-app</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>My Awesome App</name>
    <description>An application that does awesome things</description>

    <!-- Properties -->
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <junit.version>5.9.2</junit.version>
        <spring.boot.version>3.0.5</spring.boot.version>
    </properties>

    <!-- Dependencies -->
    <dependencies>
        <!-- Spring Boot Starter -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring.boot.version}</version>
        </dependency>

        <!-- Database -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
            <version>${spring.boot.version}</version>
        </dependency>

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>2.1.214</version>
            <scope>runtime</scope>
        </dependency>

        <!-- Testing -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <version>${spring.boot.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <!-- Build configuration -->
    <build>
        <plugins>
            <!-- Compiler plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                </configuration>
            </plugin>

            <!-- Spring Boot plugin -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring.boot.version}</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <!-- Surefire for testing -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.0.0-M9</version>
            </plugin>
        </plugins>
    </build>
</project>
```

### Gradle: The Groovy Alternative

```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.0.5'
    id 'io.spring.dependency-management' version '1.1.0'
}

group = 'com.example'
version = '1.0.0-SNAPSHOT'
sourceCompatibility = '17'

repositories {
    mavenCentral()
}

dependencies {
    // Spring Boot
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    
    // Database
    runtimeOnly 'com.h2database:h2'
    
    // Testing
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.junit.jupiter:junit-jupiter'
}

test {
    useJUnitPlatform()
}

// Custom task
task hello {
    doLast {
        println 'Hello from Gradle!'
    }
}

// Build configuration
jar {
    enabled = false
    archiveClassifier = '' // Remove 'plain' suffix
}

bootJar {
    enabled = true
    archiveClassifier = '' // Fat JAR
}
```

### Dependency Hell: When Libraries Collide

```xml
<!-- Maven dependency conflict -->
<dependencies>
    <dependency>
        <groupId>com.library.a</groupId>
        <artifactId>library-a</artifactId>
        <version>1.0</version>
        <!-- Transitively depends on common-lib:2.0 -->
    </dependency>
    
    <dependency>
        <groupId>com.library.b</groupId>
        <artifactId>library-b</artifactId>
        <version>2.0</version>
        <!-- Transitively depends on common-lib:1.5 -->
    </dependency>
    
    <!-- Which version of common-lib will be used? -->
    <!-- Maven chooses the "nearest" one in the dependency tree -->
    
    <!-- Solution: Explicit dependency management -->
    <dependency>
        <groupId>com.common</groupId>
        <artifactId>common-lib</artifactId>
        <version>2.1</version> <!-- Force specific version -->
    </dependency>
</dependencies>
```

---

## Chapter 23: Testing - JUnit and the Art of Mocking Everything

### JUnit 5 Basics

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

public class CalculatorTest {
    
    private Calculator calculator;
    
    @BeforeEach
    void setUp() {
        calculator = new Calculator();
    }
    
    @Test
    @DisplayName("Addition should work correctly")
    void testAddition() {
        // Arrange
        int a = 5;
        int b = 3;
        
        // Act
        int result = calculator.add(a, b);
        
        // Assert
        assertEquals(8, result);
        assertTrue(result > 0);
        assertNotNull(result);
    }
    
    @Test
    void testDivisionByZero() {
        // Assert that exception is thrown
        assertThrows(ArithmeticException.class, () -> {
            calculator.divide(10, 0);
        });
    }
    
    @ParameterizedTest
    @ValueSource(ints = {1, 2, 3, 5, 8, 13})
    void testPositiveNumbers(int number) {
        assertTrue(number > 0);
    }
    
    @Test
    @Timeout(1) // Test should complete within 1 second
    void testPerformance() {
        // Some operation that should be fast
        calculator.complexCalculation();
    }
    
    @Test
    @Disabled("Not implemented yet")
    void testFutureFeature() {
        // Test for feature under development
    }
}
```

### Mockito: Mocking Framework

```java
import org.mockito.*;
import static org.mockito.Mockito.*;

public class OrderServiceTest {
    
    @Mock
    private PaymentService paymentService;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private OrderService orderService;
    
    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
    }
    
    @Test
    void testProcessOrder() {
        // Arrange
        Order order = new Order("123", 100.0);
        when(paymentService.processPayment(order)).thenReturn(true);
        
        // Act
        boolean result = orderService.processOrder(order);
        
        // Assert
        assertTrue(result);
        verify(paymentService).processPayment(order);
        verify(emailService).sendConfirmation(order);
        verifyNoMoreInteractions(paymentService, emailService);
    }
    
    @Test
    void testProcessOrderWithPaymentFailure() {
        // Arrange
        Order order = new Order("456", 200.0);
        when(paymentService.processPayment(order)).thenReturn(false);
        
        // Act
        boolean result = orderService.processOrder(order);
        
        // Assert
        assertFalse(result);
        verify(paymentService).processPayment(order);
        verifyNoInteractions(emailService); // Email should not be sent
    }
    
    @Test
    void testProcessOrderWithException() {
        // Arrange
        Order order = new Order("789", 300.0);
        when(paymentService.processPayment(order))
            .thenThrow(new PaymentException("Payment gateway down"));
        
        // Act & Assert
        assertThrows(PaymentException.class, () -> {
            orderService.processOrder(order);
        });
    }
}
```

### Integration Testing with Spring Boot

```java
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit.jupiter.SpringJUnitConfig;

@SpringBootTest
@SpringJUnitConfig
public class OrderServiceIntegrationTest {
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    void testCreateOrderEndToEnd() {
        // Test the entire application flow
        CreateOrderRequest request = new CreateOrderRequest("Product A", 2);
        
        ResponseEntity<Order> response = restTemplate.postForEntity(
            "/api/orders", request, Order.class);
        
        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getBody());
        assertEquals("Product A", response.getBody().getProductName());
    }
    
    @Test
    @DirtiesContext // Reset application context after test
    void testWithDatabaseState() {
        // Test that modifies database state
    }
}
```

---

## Part VII: The JVM and Beyond

## Chapter 24: The JVM - Your Friendly Neighborhood Overlord

### JVM Architecture

```
┌─────────────────────────────────────────────┐
│                   JVM                       │
├─────────────────────────────────────────────┤
│  Class Loader Subsystem                     │
│  ├─ Bootstrap Class Loader                  │
│  ├─ Extension Class Loader                  │
│  └─ Application Class Loader                │
├─────────────────────────────────────────────┤
│  Memory Areas                               │
│  ├─ Method Area (Metaspace)                │
│  ├─ Heap Memory                            │
│  │  ├─ Young Generation                    │
│  │  │  ├─ Eden Space                      │
│  │  │  └─ Survivor Spaces (S0, S1)       │
│  │  └─ Old Generation (Tenured)           │
│  ├─ Stack Memory                           │
│  ├─ PC Registers                           │
│  └─ Native Method Stacks                   │
├─────────────────────────────────────────────┤
│  Execution Engine                           │
│  ├─ Interpreter                            │
│  ├─ JIT Compiler                           │
│  └─ Garbage Collector                      │
└─────────────────────────────────────────────┘
```

### Memory Areas Explained

```java
public class MemoryExample {
    private static String classVariable = "Stored in Method Area";  // Method Area
    private String instanceVariable = "Stored in Heap";            // Heap
    
    public void methodExample() {
        int localVariable = 42;           // Stack
        String localString = "Also stack"; // Stack (reference), Heap (object)
        
        // Method information stored in Method Area
        // Local variables stored in Stack
        // Objects stored in Heap
    }
}
```

### Garbage Collection in Action

```java
public class GarbageCollectionExample {
    public void createGarbage() {
        // These objects become eligible for GC when method ends
        List<String> temporaryList = new ArrayList<>();
        temporaryList.add("This will be garbage collected");
        
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 1000; i++) {
            sb.append("Data " + i);
        }
        
        // Objects go out of scope here
    } // GC can collect temporaryList and sb
    
    private List<String> permanentList = new ArrayList<>(); // Lives with instance
    
    public void memoryLeak() {
        // Potential memory leak - objects keep getting added
        for (int i = 0; i < 1000000; i++) {
            permanentList.add("Item " + i);
        }
        // permanentList keeps growing, objects never eligible for GC
    }
}
```

### JVM Command Line Options

```bash
# Memory settings
java -Xms512m -Xmx2g MyApplication
# -Xms: Initial heap size
# -Xmx: Maximum heap size

# Garbage Collection settings
java -XX:+UseG1GC -XX:MaxGCPauseMillis=200 MyApplication
# -XX:+UseG1GC: Use G1 garbage collector
# -XX:MaxGCPauseMillis: Target pause time

# Debugging options
java -XX:+PrintGCDetails -XX:+PrintGCTimeStamps MyApplication
# Print GC information

# JIT Compiler options
java -XX:+PrintCompilation MyApplication
# Print JIT compilation information

# Remote debugging
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 MyApplication
```

---

## Chapter 25: Memory Management and Garbage Collection

### Heap Memory Layout

```java
public class HeapExample {
    public void demonstrateHeapUsage() {
        // Objects created in Eden space (Young Generation)
        List<String> youngObjects = new ArrayList<>();
        
        for (int i = 0; i < 1000; i++) {
            youngObjects.add("Young object " + i);
        }
        
        // After several GC cycles, surviving objects move to Old Generation
        keepReferencesAlive(youngObjects);
    }
    
    private static List<List<String>> oldGenerationObjects = new ArrayList<>();
    
    private void keepReferencesAlive(List<String> objects) {
        oldGenerationObjects.add(objects); // Prevents GC
    }
}
```

### Garbage Collection Types

```java
// Monitoring GC with JVM flags
/*
-XX:+UseSerialGC           // Single-threaded GC (small applications)
-XX:+UseParallelGC         // Multi-threaded GC (default for server)
-XX:+UseG1GC               // Low-latency GC for large heaps
-XX:+UseZGC                // Ultra-low latency GC (Java 15+)
*/

public class GCMonitoring {
    public static void triggerGC() {
        List<Object> objects = new ArrayList<>();
        
        // Create lots of objects to trigger GC
        for (int i = 0; i < 1000000; i++) {
            objects.add(new Object());
            
            if (i % 100000 == 0) {
                System.out.println("Created " + i + " objects");
                // Monitor memory usage
                Runtime runtime = Runtime.getRuntime();
                long totalMemory = runtime.totalMemory();
                long freeMemory = runtime.freeMemory();
                long usedMemory = totalMemory - freeMemory;
                
                System.out.println("Used memory: " + (usedMemory / 1024 / 1024) + " MB");
            }
        }
        
        // Clear references to allow GC
        objects.clear();
        System.gc(); // Suggest GC (not guaranteed)
    }
}
```

### Memory Leaks in Java

```java
public class MemoryLeakExamples {
    // Leak 1: Static collections
    private static List<Object> staticList = new ArrayList<>();
    
    public void leakViaStaticCollection() {
        staticList.add(new Object()); // Never removed!
    }
    
    // Leak 2: Unclosed resources
    public void leakViaUnclosedResource() {
        try {
            FileInputStream fis = new FileInputStream("file.txt");
            // If exception occurs, file not closed
            // Use try-with-resources instead!
        } catch (IOException e) {
            // Handle exception
        }
    }
    
    // Leak 3: Event listeners not removed
    private List<EventListener> listeners = new ArrayList<>();
    
    public void addListener(EventListener listener) {
        listeners.add(listener);
        // Should provide removeListener method!
    }
    
    // Leak 4: ThreadLocal variables
    private ThreadLocal<Object> threadLocal = new ThreadLocal<>();
    # ☕ Java: For the Haters
## The Complete Survival Guide to the Language That Broke A Million Dreams

> *"Write once, debug everywhere, cry indefinitely"* - Ancient Java Proverb (circa 1995)

[![Java Version](https://img.shields.io/badge/Java-8%2B-brightgreen.svg)](https://www.oracle.com/java/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Frustration Level](https://img.shields.io/badge/Frustration-Maximum-red.svg)]()
[![Coffee Required](https://img.shields.io/badge/Coffee-Essential-brown.svg)]()

---

## 📚 Table of Contents

- [Before We Begin: Making Java with Java](#-before-we-begin-making-java-with-java)
- [Part I: Welcome to Hell](#part-i-welcome-to-hell)
  - [Chapter 1: Introduction - Why We're Here](#chapter-1-introduction---why-were-here)
  - [Chapter 2: Setting Up Your Torture Chamber](#chapter-2-setting-up-your-torture-chamber-environment-setup)
  - [Chapter 3: The Ceremony of Hello World](#chapter-3-the-ceremony-of-hello-world)
- [Part II: Basic Suffering](#part-ii-basic-suffering)
  - [Chapter 4: Variables and Types - The Primitive Paradox](#chapter-4-variables-and-types---the-primitive-paradox)
  - [Chapter 5: Objects - Everything Is a Hammer](#chapter-5-objects---everything-is-a-hammer)
  - [Chapter 6: Methods - Function Junction Dysfunction](#chapter-6-methods---function-junction-dysfunction)
  - [Chapter 7: Control Flow - If This, Then Suffering](#chapter-7-control-flow---if-this-then-suffering)
- [Part III: Intermediate Torment](#part-iii-intermediate-torment)
  - [Chapter 8: The String Pool - Where Memory Goes to Die](#chapter-8-the-string-pool---where-memory-goes-to-die)
  - [Chapter 9: Arrays - Fixed-Size Nightmares](#chapter-9-arrays---fixed-size-nightmares)
  - [Chapter 10: Collections - Why Simple Things Are Hard](#chapter-10-collections---why-simple-things-are-hard)
  - [Chapter 11: Null - The Billion Dollar Mistake](#chapter-11-null---the-billion-dollar-mistake)
- [Part IV: Advanced Agony](#part-iv-advanced-agony)
  - [Chapter 12: Object-Oriented Programming - Inheritance Hierarchy Hell](#chapter-12-object-oriented-programming---inheritance-hierarchy-hell)
  - [Chapter 13: Interfaces and Abstract Classes - Contract Confusion](#chapter-13-interfaces-and-abstract-classes---contract-confusion)
  - [Chapter 14: Exception Handling - Catch Me If You Can](#chapter-14-exception-handling---catch-me-if-you-can)
  - [Chapter 15: Generics - Type Erasure and You](#chapter-15-generics---type-erasure-and-you)
- [Part V: Expert-Level Existential Crisis](#part-v-expert-level-existential-crisis)
  - [Chapter 16: Threading - Race Conditions as a Feature](#chapter-16-threading---race-conditions-as-a-feature)
  - [Chapter 17: I/O and NIO - Reading Files Shouldn't Be This Hard](#chapter-17-io-and-nio---reading-files-shouldnt-be-this-hard)
  - [Chapter 18: Reflection - Looking in the Mirror and Screaming](#chapter-18-reflection---looking-in-the-mirror-and-screaming)
  - [Chapter 19: Annotations - Metadata Madness](#chapter-19-annotations---metadata-madness)
- [Part VI: Enterprise Nightmares](#part-vi-enterprise-nightmares)
  - [Chapter 20: Design Patterns - When Simple Solutions Are Illegal](#chapter-20-design-patterns---when-simple-solutions-are-illegal)
  - [Chapter 21: Frameworks - Spring Into Dependency Injection Hell](#chapter-21-frameworks---spring-into-dependency-injection-hell)
  - [Chapter 22: Build Tools - Maven, Gradle, and XML Dependency Hell](#chapter-22-build-tools---maven-gradle-and-xml-dependency-hell)
  - [Chapter 23: Testing - JUnit and the Art of Mocking Everything](#chapter-23-testing---junit-and-the-art-of-mocking-everything)
- [Part VII: The JVM and Beyond](#part-vii-the-jvm-and-beyond)
  - [Chapter 24: The JVM - Your Friendly Neighborhood Overlord](#chapter-24-the-jvm---your-friendly-neighborhood-overlord)
  - [Chapter 25: Memory Management and Garbage Collection](#chapter-25-memory-management-and-garbage-collection)
  - [Chapter 26: Performance Tuning - Making Slow Code Slightly Less Slow](#chapter-26-performance-tuning---making-slow-code-slightly-less-slow)
  - [Chapter 27: Modern Java - Lambda Expressions and Stream API](#chapter-27-modern-java---lambda-expressions-and-stream-api)
- [Part VIII: Survival Skills](#part-viii-survival-skills)
  - [Chapter 28: Debugging - The Art of Controlled Crying](#chapter-28-debugging---the-art-of-controlled-crying)
  - [Chapter 29: Best Practices - Or How to Write Less Terrible Code](#chapter-29-best-practices---or-how-to-write-less-terrible-code)
  - [Chapter 30: Career Advice - Turning Pain Into Profit](#chapter-30-career-advice---turning-pain-into-profit)
- [Additional Resources](#additional-resources)
- [Contributing](#contributing)
- [License](#license)

---

## ☕ Before We Begin: Making Java with Java

Since you're about to embark on this journey of learning Java, let's start by doing what any reasonable person would do: make a cup of coffee using Java code. After all, you're going to need the caffeine.

```java
public class CoffeeMaker {
    // Because everything in Java must be inside a class, even coffee
    private boolean hasCoffee = false;
    private boolean hasWater = false;
    private boolean hasMilk = false;
    private int sugarCubes = 0;
    private String mood = "desperate";
    
    public static void main(String[] args) {
        // The sacred main method - where all Java adventures begin
        System.out.println("☕ Welcome to Java Coffee Brewing System™ ☕");
        System.out.println("(Enterprise Edition with Dependency Injection)");
        
        CoffeeMaker maker = new CoffeeMaker();
        try {
            Cup coffee = maker.brewCoffee();
            maker.drink(coffee);
        } catch (InsufficientCaffeineException e) {
            System.out.println("ERROR: Cannot proceed with Java learning without coffee!");
            System.out.println("Please insert coffee and try again.");
        } finally {
            System.out.println("Coffee break complete. Ready to learn Java!");
            System.out.println("(Disclaimer: Actual Java knowledge not guaranteed)");
        }
    }
    
    public Cup brewCoffee() throws InsufficientCaffeineException {
        // Method 1: Check if we have the necessary ingredients
        validateIngredients();
        
        // Method 2: Create coffee using proper object-oriented principles
        CoffeeBean beans = new CoffeeBean("Arabica", "Dark Roast");
        Water water = new Water(200, "filtered");  // 200ml of water
        
        // Method 3: Use a builder pattern because this is Java
        Cup coffee = new CoffeeBuilder()
            .withBeans(beans)
            .withWater(water)
            .withMilk(hasMinimalMilk())
            .withSugar(calculateOptimalSugar())
            .withDesperation(mood)
            .brew();
            
        return coffee;
    }
    
    private void validateIngredients() throws InsufficientCaffeineException {
        if (!hasCoffee) {
            throw new InsufficientCaffeineException(
                "No coffee beans found. Cannot compile... I mean, cannot brew."
            );
        }
        if (!hasWater) {
            throw new InsufficientCaffeineException(
                "No water found. This is like trying to run Java without the JVM."
            );
        }
    }
    
    private boolean hasMinimalMilk() {
        return hasMilk; // Java doesn't trust you to remember boolean values
    }
    
    private int calculateOptimalSugar() {
        // Complex algorithm to determine sugar needs
        switch (mood) {
            case "desperate":
                return Math.max(sugarCubes, 3);
            case "tired":
                return Math.max(sugarCubes, 2);
            case "optimistic":
                return Math.min(sugarCubes, 1);
            default:
                return sugarCubes; // Default case, because Java demands it
        }
    }
    
    public void drink(Cup coffee) {
        if (coffee != null && coffee.getTemperature() > 60) {
            System.out.println("☕ Drinking delicious Java-made coffee...");
            System.out.println("☕ Caffeine levels rising...");
            System.out.println("☕ Ready to tackle NullPointerExceptions!");
            this.mood = "caffeinated";
        } else {
            System.out.println("☕ Coffee is null or too cold. Just like my feelings about Java.");
        }
    }
}

// Because Java loves its classes, here are more classes for coffee
class CoffeeBean {
    private String variety;
    private String roast;
    
    public CoffeeBean(String variety, String roast) {
        this.variety = variety;
        this.roast = roast;
    }
    
    // Getters and setters, because direct access is forbidden in Java
    public String getVariety() { return variety; }
    public String getRoast() { return roast; }
    
    @Override
    public String toString() {
        return variety + " (" + roast + ")";
    }
}

class Water {
    private int volumeML;
    private String quality;
    
    public Water(int volumeML, String quality) {
        this.volumeML = volumeML;
        this.quality = quality;
    }
    
    public int getVolumeML() { return volumeML; }
    public String getQuality() { return quality; }
}

class Cup {
    private String contents;
    private int temperature;
    private boolean isEmpty;
    
    public Cup(String contents, int temperature) {
        this.contents = contents;
        this.temperature = temperature;
        this.isEmpty = false;
    }
    
    public String getContents() { return contents; }
    public int getTemperature() { return temperature; }
    public boolean isEmpty() { return isEmpty; }
    
    @Override
    public String toString() {
        return "Cup of " + contents + " at " + temperature + "°C";
    }
}

// Builder pattern because simple constructors are for amateurs
class CoffeeBuilder {
    private CoffeeBean beans;
    private Water water;
    private boolean withMilk = false;
    private int sugarCubes = 0;
    private String desperation = "normal";
    
    public CoffeeBuilder withBeans(CoffeeBean beans) {
        this.beans = beans;
        return this; // Method chaining, because Java loves ceremony
    }
    
    public CoffeeBuilder withWater(Water water) {
        this.water = water;
        return this;
    }
    
    public CoffeeBuilder withMilk(boolean withMilk) {
        this.withMilk = withMilk;
        return this;
    }
    
    public CoffeeBuilder withSugar(int cubes) {
        this.sugarCubes = cubes;
        return this;
    }
    
    public CoffeeBuilder withDesperation(String level) {
        this.desperation = level;
        return this;
    }
    
    public Cup brew() throws InsufficientCaffeineException {
        if (beans == null || water == null) {
            throw new InsufficientCaffeineException("Missing essential ingredients!");
        }
        
        // Simulate brewing process
        System.out.println("☕ Grinding " + beans + "...");
        System.out.println("☕ Heating " + water.getVolumeML() + "ml of " + water.getQuality() + " water...");
        System.out.println("☕ Brewing with " + desperation + " level desperation...");
        
        if (withMilk) {
            System.out.println("☕ Adding a splash of milk...");
        }
        
        if (sugarCubes > 0) {
            System.out.println("☕ Adding " + sugarCubes + " sugar cube(s)...");
        }
        
        // Calculate coffee temperature (complex algorithm)
        int temperature = 85 - (sugarCubes * 2); // Sugar cools it down, obviously
        
        String coffeeType = withMilk ? "Latte" : "Black Coffee";
        return new Cup(coffeeType, temperature);
    }
}

// Custom exception because Java loves checked exceptions
class InsufficientCaffeineException extends Exception {
    public InsufficientCaffeineException(String message) {
        super(message);
    }
}
```

### 📄 Expected Output:

```console
☕ Welcome to Java Coffee Brewing System™ ☕
(Enterprise Edition with Dependency Injection)
☕ Grinding Arabica (Dark Roast)...
☕ Heating 200ml of filtered water...
☕ Brewing with desperate level desperation...
☕ Adding 3 sugar cube(s)...
☕ Drinking delicious Java-made coffee...
☕ Caffeine levels rising...
☕ Ready to tackle NullPointerExceptions!
Coffee break complete. Ready to learn Java!
(Disclaimer: Actual Java knowledge not guaranteed)
```

### 🎓 What This Coffee-Making Adventure Teaches Us About Java:

| Concept | Example in Code | Why Java Does This |
|---------|----------------|-------------------|
| **Everything Must Be a Class** | Even making coffee requires multiple classes | Java believes in organization through separation (and verbosity) |
| **The Sacred Main Method** | `public static void main(String[] args)` | It's like the "once upon a time" of programming |
| **Exception Handling** | `InsufficientCaffeineException` | Java makes you think about what could go wrong before it happens |
| **Object Creation** | `new CoffeeBean()`, `new Water()` | Java loves the `new` keyword |
| **Method Chaining** | `builder.with().with().build()` | Fluent interfaces for "readable" code |
| **Getters and Setters** | `getVariety()`, `setRoast()` | Direct access to variables is forbidden |
| **toString() Methods** | Custom string representation | Objects need to know how to describe themselves |

> **💡 The Analogy**: Making coffee with Java code is like using a formal legal document to ask someone to pass the sugar - technically correct, thoroughly documented, but wildly overcomplicated for the task at hand.

Now that you're properly caffeinated (virtually), let's dive into the real Java journey...

---

## Part I: Welcome to Hell

## Chapter 1: Introduction - Why We're Here

### The Unfortunate Reality

So, you've decided to learn Java. Perhaps your professor assigned it, maybe your company uses it, or possibly you lost a bet. Regardless of how you arrived at this moment, you're here now, staring into the abyss of enterprise-grade verbosity and wondering where your life went wrong.

Java is like that friend who explains every single detail of how they made their morning coffee, including the geological history of the beans. It's thorough, it's comprehensive, and it makes you want to drink your coffee somewhere else.

### A Brief History of Java (Or: How We Got Into This Mess)

In 1995, James Gosling at Sun Microsystems looked at C++ and thought, "You know what this needs? More ceremony, more verbosity, and definitely more XML configuration files." Thus, Java was born, originally named "Oak" until trademark lawyers got involved – the first sign that Java would be all about legal complications.

The original promises were ambitious:

- ✅ **"Write Once, Run Anywhere"** - In practice: "Write Once, Debug Everywhere"
- ✅ **"Simple and Familiar"** - If your idea of simple involves 15 lines of code for "Hello World"
- ✅ **"Object-Oriented"** - Because everything should be an object, even when it really, really shouldn't be
- ✅ **"Robust and Secure"** - Tell that to the `ClassCastException` at 3 AM

### What Makes Java Special (And By Special, We Mean Painful)

Java's philosophy can be summarized as: **"If it can be made more complicated, it should be."** Here's what makes Java uniquely frustrating:

#### 1. 🎭 Verbosity as a Virtue

Java believes that if you can say something in 5 words, you should definitely use 50 instead. It's like having a conversation with someone who uses legal jargon to ask for the salt.

```java
// What you want to do:
print("Hello")

// What Java makes you do:
public class HelloPrinter {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```

#### 2. ⚠️ Checked Exceptions

Java is the only language that makes you acknowledge every possible way your code might fail, even the impossibly unlikely ones. It's like having to sign a waiver before opening a bag of chips.

```java
// Just trying to read a file
try {
    FileReader file = new FileReader("simple.txt");
    // Do something simple
} catch (FileNotFoundException e) {
    // Handle file not found
} catch (IOException e) {
    // Handle IO problems
} catch (SecurityException e) {
    // Handle security issues
} catch (OutOfMemoryError e) {
    // Handle memory problems
} // ... and potentially 47 other exceptions
```

#### 3. 🗂️ The Classpath

A mystical concept that determines whether your program runs or throws `ClassNotFoundException`. It's like a treasure map where X marks "your program might work here, but probably not."

```bash
# The eternal Java developer struggle
java MyProgram
Error: Could not find or load main class MyProgram

java -cp . MyProgram
Error: Could not find or load main class MyProgram

java -cp ./build/classes MyProgram
Error: NoClassDefFoundError

# 3 hours later...
java -cp ./build/classes:./lib/*:./resources MyProgram
Hello World!
# Finally!
```

#### 4. 🏭 Enterprise Patterns

Java has elevated making simple things complicated into an art form. Need to create an object? Better create a factory. Need a factory? Better create an abstract factory factory.

```java
// What you want:
User user = new User("John");

// What Java "best practices" demand:
UserFactory userFactory = UserFactoryProvider.getInstance().getUserFactory();
UserBuilder userBuilder = userFactory.createUserBuilder();
User user = userBuilder.setName("John").build();

// Or even worse:
AbstractUserFactoryBean factoryBean = 
    ApplicationContextProvider.getApplicationContext()
        .getBean(AbstractUserFactoryBean.class);
UserCreationStrategy strategy = 
    factoryBean.createUserCreationStrategy();
User user = strategy.createUser(
    UserCreationRequest.builder()
        .withName("John")
        .withCreationContext(CreationContext.DEFAULT)
        .build()
);
```

### Who This Book Is For

This book is for:

- 🎓 **Computer science students** who've been assigned Java and are questioning their life choices
- 👩‍💻 **Developers coming from sane languages** who are experiencing culture shock  
- 😵 **Anyone who's spent 4 hours debugging** a `NullPointerException` on line 1,247 of a file they didn't write
- 😈 **Masochists** who enjoy pain but want to understand why they're suffering
- 💼 **People who need to learn Java for work** but want to maintain their sanity (good luck)
- 🤷 **Anyone who's ever wondered** why a simple "Hello World" requires a class, a method, and a JVM

### Who This Book Is NOT For

- ☕ **Java evangelists** who think `AbstractSingletonProxyFactoryBean` is a reasonable class name
- 📄 **People who enjoy** writing XML configuration files for fun
- ✅ **Anyone who thinks** checked exceptions are humanity's greatest invention
- 🏭 **Developers who believe** that 47 design patterns are required to display "Hello World"
- 🤖 **Enterprise architects** who get excited about adding more layers of abstraction
- 🎯 **People looking for** a traditional, dry programming textbook

### How to Use This Book

Each chapter follows a simple structure designed to maximize learning while minimizing despair:

#### 📖 Chapter Structure:

1. **🎯 The Concept**: What Java is trying to do (its noble intentions)
2. **😱 The Reality**: What actually happens when you try to use it
3. **🔄 The Analogy**: Real-world comparisons to help you understand the absurdity
4. **💻 The Code**: Examples that will make you question your career choices
5. **🛡️ The Survival Tips**: How to cope with the madness
6. **🏋️ The Exercises**: Practice problems designed to build character (and frustration tolerance)

### A Word of Encouragement

Despite what this book's tone might suggest, Java isn't actually evil. It's just... enthusiastic. Like a golden retriever that's learned to code. It means well, it's very energetic, and it will fetch your objects whether you asked for them or not.

Java has its place in the world. It's stable, it's fast (eventually), it has excellent tooling, and it pays well. Many successful applications run on Java, and many successful careers have been built around it.

But that doesn't mean we can't have a little fun pointing out its quirks along the way.

---

## Chapter 2: Setting Up Your Torture Chamber (Environment Setup)

### The JDK vs JRE vs JVM Confusion

Before you can begin your Java journey, you need to understand the holy trinity of Java confusion. It's like trying to order coffee when the menu lists "Coffee Bean," "Ground Coffee," and "Coffee Beverage" as separate items that you somehow need to combine yourself.

- **🔧 JVM (Java Virtual Machine)**: The thing that actually runs your code. Think of it as the engine.
- **📦 JRE (Java Runtime Environment)**: The JVM plus standard libraries. Think of it as the engine plus the car.
- **🛠️ JDK (Java Development Kit)**: The JRE plus development tools (compiler, debugger, etc.). Think of it as the engine, car, and mechanic's toolbox all in one.

> **🏠 The Analogy**: It's like going to a restaurant where you need to buy the ingredients, the recipe, the kitchen, and hire the chef separately, and then assemble them yourself before you can order a sandwich.

### Installing Java: The First Test of Your Patience

Installing Java should be simple. It's not. It's like trying to assemble IKEA furniture, but the instructions are in ancient Sumerian and half the screws are missing.

#### Step 1: Choose Your Java Distribution

Because one Java wasn't confusing enough, there are now multiple distributions:

| Distribution | Provider | Cost | Best For |
|-------------|----------|------|----------|
| **Oracle JDK** | Oracle | Free for development, paid for production | Official support |
| **OpenJDK** | Open source community | Free | True open source |
| **Eclipse Temurin** | Eclipse Foundation | Free | Easy downloads |
| **Amazon Corretto** | Amazon | Free | Cloud applications |

**Recommendation**: Eclipse Temurin (it's free, easy to download, and just works)

#### Step 2: Set Environment Variables

```bash
# Windows
JAVA_HOME=C:\Program Files\Eclipse Adoptium\jdk-17.0.7.7
PATH=%PATH%;%JAVA_HOME%\bin

# macOS/Linux
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin
```

> **🔑 The Quirk**: You'll set these variables correctly, but somehow Java will still claim it can't find itself. It's like hiding your own keys and then being surprised you can't leave the house.

#### Step 3: Verify Installation

```bash
java -version
javac -version
```

If both commands work and show the same version, congratulations! You've passed the first test.

### IDE Selection: Choosing Your Weapon

#### Eclipse - The Reliable But Aging Honda Civic

**Pros**: Free, powerful, handles large projects
**Cons**: Interface designed in 2003, loading times measured in geological epochs

#### IntelliJ IDEA - The Tesla of Java IDEs

**Pros**: Excellent code completion, modern interface, great debugging
**Cons**: Ultimate edition costs money, might make you too comfortable with Java

#### Visual Studio Code - The Swiss Army Knife

**Pros**: Lightweight, free, familiar to many developers
**Cons**: Requires multiple extensions, less integrated than full Java IDEs

**Recommendation for beginners**: IntelliJ IDEA Community Edition

### Build Tools: Because Compiling Should Be Complicated

#### Maven - The Opinionated Friend

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.company</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

#### Gradle - Maven's Cooler Younger Sibling

```groovy
plugins {
    id 'java'
}

dependencies {
    testImplementation 'junit:junit:4.12'
}
```

---

## Chapter 3: The Ceremony of Hello World

### The Simplicity Comparison

Let's start with a comparison that will immediately set your expectations:

```python
# Python
print("Hello, World!")
```

```javascript
// JavaScript
console.log("Hello, World!");
```

```java
// Java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

Java's version is like being asked to fill out a form, provide three references, and undergo a background check just to say hello to someone.

### Deconstructing the Ceremony

Let's break down each piece of this seemingly simple program:

#### `public class HelloWorld`

This declares a public class named `HelloWorld`. In Java, everything must live inside a class. You can't just have floating code like some kind of anarchist.

**The Rules:**
- The class name must match the filename exactly (case-sensitive)
- The class must be `public` if you want to run it
- One public class per file (because Java believes in organization through separation)

#### `public static void main(String[] args)`

This is the magical incantation that makes Java programs start:

- `public`: Anyone can call this method
- `static`: This method belongs to the class, not to any instance
- `void`: This method returns nothing
- `main`: The sacred name that the JVM looks for
- `String[] args`: Command-line arguments you'll probably never use

#### `System.out.println("Hello, World!")`

Finally, the actual greeting! But even this simple statement has layers:

- `System`: A class representing the system
- `out`: A static variable representing standard output
- `println`: A method that prints and adds a newline

### The File Naming Tyranny

Java enforces a strict naming convention:
- File must be named `HelloWorld.java` (exact capitalization)
- Must be compiled to `HelloWorld.class`
- Must be run with `java HelloWorld`

> **The Analogy**: It's like a restaurant that refuses to serve you unless your shirt, pants, and shoes all have the exact same shade of blue.

---

## Part II: Basic Suffering

## Chapter 4: Variables and Types - The Primitive Paradox

### The Two-Class System

Java has a peculiar caste system for data types. There are the "primitives" - the working class of Java types - and the "objects" - the aristocracy.

#### The Primitive Types

```java
byte smallNumber = 42;        // -128 to 127
short mediumNumber = 1000;    // -32,768 to 32,767 
int regularNumber = 42000;    // The most popular kid in class
long bigNumber = 42000000L;   // Note the 'L' - very important!
float decimal = 3.14f;        // Note the 'f' - also very important!
double bigDecimal = 3.14159;  // The default decimal type
char letter = 'A';            // A single character
boolean flag = true;          // true or false
```

#### The Wrapper Classes

```java
Byte wrappedByte = 42;
Short wrappedShort = 1000;
Integer wrappedInt = 42000;
Long wrappedLong = 42000000L;
Float wrappedFloat = 3.14f;
Double wrappedDouble = 3.14159;
Character wrappedChar = 'A';
Boolean wrappedBoolean = true;
```

### Autoboxing and Unboxing: The Automatic Confusion Generator

```java
Integer a = 127;
Integer b = 127;
System.out.println(a == b);    // true (cached!)

Integer c = 128;
Integer d = 128;
System.out.println(c == d);    // false (not cached!)
```

**The Explanation**: Java caches Integer objects for values from -128 to 127. It's like having a special parking spot for specific numbers.

### Variable Declaration

```java
// The Classic Way
int number = 42;
String name = "Java";

// The Final Way (Constants)
final int CONSTANT = 42;

// The Modern Way (Java 10+)
var number = 42;        // Type inferred as int
var name = "Java";      // Type inferred as String
```

---

## Chapter 5: Objects - Everything Is a Hammer

### Object Creation: The Instantiation Ritual

```java
Person john = new Person("John Doe", 30);
Person jane = new Person("Jane Smith", 25);

// Each object has its own copy of instance variables
john.celebrateBirthday();  // John is now 31
System.out.println(jane.getAge());  // Jane is still 25
```

### The `this` Keyword

```java
public class BankAccount {
    private double balance;
    
    public BankAccount(double balance) {
        this.balance = balance;  // this.balance refers to the instance variable
    }
    
    public BankAccount deposit(double amount) {
        this.balance += amount;
        return this;  // Method chaining
    }
}
```

### Access Modifiers: The Privacy Police

```java
public class AccessExample {
    public String publicField = "Everyone can see this";
    protected String protectedField = "Subclasses can see this";
    String packageField = "Only same package can see this";
    private String privateField = "Only this class can see this";
}
```

### Static: The Class-Level Weirdness

```java
public class MathUtils {
    public static final double PI = 3.14159;  // Class constant
    private static int instanceCount = 0;     // Class variable
    
    public static double circleArea(double radius) {
        return PI * radius * radius;
    }
}
```

---

## Chapter 6: Methods - Function Junction Dysfunction

### Method Anatomy: Dissecting the Beast

```java
public static final synchronized String methodName(int param1, String param2) 
    throws IOException, CustomException {
    // Method body
    return "result";
}
```

- `public`: Access modifier
- `static`: Belongs to the class, not instance
- `final`: Can't be overridden
- `synchronized`: Thread-safe (sort of)
- `String`: Return type
- `methodName`: The actual name
- Parameters and exceptions

### Method Overloading: The Name Game

```java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
    
    public double add(double a, double b) {
        return a + b;
    }
    
    public int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

### Parameter Passing: The Great Confusion

```java
// Primitives: True pass-by-value
public void changePrimitive(int number) {
    number = 100;  // Changes local copy only
}

// Objects: Pass-by-value of the reference
public void changeObject(List<String> list) {
    list.add("Added");  // Modifies the original object
}

public void replaceObject(List<String> list) {
    list = new ArrayList<>();  // Only changes local reference
}
```

---

## Chapter 7: Control Flow - If This, Then Suffering

### The if Statement: Simple Decisions, Complex Syntax

```java
if (temperature > 80) {
    System.out.println("It's hot!");
} else if (temperature > 60) {
    System.out.println("It's pleasant!");
} else if (temperature > 40) {
    System.out.println("It's cold!");
} else {
    System.out.println("It's freezing!");
}
```

### Switch Statements: The Multiple Choice Quiz

```java
// Traditional switch
switch (dayOfWeek) {
    case 1:
        System.out.println("Monday");
        break;
    case 2:
        System.out.println("Tuesday");
        break;
    default:
        System.out.println("Invalid day");
        break;
}

// Modern switch expressions (Java 12+)
String message = switch (season) {
    case "spring" -> "Flowers are blooming!";
    case "summer" -> "Time for vacation!";
    case "fall", "autumn" -> "Leaves are changing!";
    case "winter" -> "Let it snow!";
    default -> "Unknown season";
};
```

### Loops: The Circle of Code

```java
// Traditional for loop
for (int i = 0; i < 10; i++) {
    System.out.println("Iteration: " + i);
}

// Enhanced for loop
int[] numbers = {1, 2, 3, 4, 5};
for (int number : numbers) {
    System.out.println("Number: " + number);
}

// While loop
int count = 0;
while (count < 5) {
    System.out.println("Count: " + count);
    count++;
}
```

---

## Part III: Intermediate Torment

## Chapter 8: The String Pool - Where Memory Goes to Die

### String Immutability: The Unchangeable Truth

```java
String original = "Hello";
String modified = original.toUpperCase();

System.out.println("Original: " + original);   // Still "Hello"
System.out.println("Modified: " + modified);   // "HELLO"
```

Strings are immutable - every "change" creates a new String object. It's like throwing away your entire car to change the radio station.

### The String Pool: Java's Memory Optimization

```java
String a = "Hello";
String b = "Hello";
String c = new String("Hello");

System.out.println(a == b);     // true (same reference)
System.out.println(a == c);     // false (different objects)
System.out.println(a.equals(c)); // true (same content)
```

### StringBuilder: The Mutable Alternative

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append("a");  // Efficient - modifies same object
}
String result = sb.toString();
```

### String Methods: The Swiss Army Knife

```java
String text = "  Hello World  ";

// Basic operations
System.out.println("Length: " + text.length());
System.out.println("Trimmed: " + text.trim());
System.out.println("Upper: " + text.toUpperCase());
System.out.println("Substring: " + text.substring(2, 7));

// Searching
System.out.println("Contains 'World': " + text.contains("World"));
System.out.println("Index of 'World': " + text.indexOf("World"));

// Splitting
String csv = "apple,banana,cherry";
String[] fruits = csv.split(",");
```

---

## Chapter 9: Arrays - Fixed-Size Nightmares

### Array Basics: The Inflexible Foundation

```java
// Different declaration styles
int[] numbers1 = new int[5];           // Java style
int numbers2[] = new int[5];           // C style
int[] numbers3 = {1, 2, 3, 4, 5};      // With initialization

// Accessing elements
System.out.println("First: " + numbers3[0]);
System.out.println("Last: " + numbers3[numbers3.length - 1]);

// Default values
boolean[] booleans = new boolean[3];   // All false
String[] strings = new String[3];      // All null
```

### Multi-Dimensional Arrays: Arrays of Arrays

```java
// 2D array
int[][] matrix = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};

// Jagged arrays (different row lengths)
int[][] jagged = {
    {1, 2},
    {3, 4, 5, 6},
    {7, 8, 9}
};

// Iterating through 2D array
for (int row = 0; row < matrix.length; row++) {
    for (int col = 0; col < matrix[row].length; col++) {
        System.out.print(matrix[row][col] + " ");
    }
    System.out.println();
}
```

### Arrays Class: The Utility Belt

```java
import java.util.Arrays;

int[] numbers = {3, 1, 4, 1, 5, 9, 2, 6};

// Sorting
Arrays.sort(numbers);
System.out.println(Arrays.toString(numbers));

// Searching (only works on sorted arrays)
int index = Arrays.binarySearch(numbers, 5);

// Copying
int[] copy = Arrays.copyOf(numbers, numbers.length);
int[] partial = Arrays.copyOf(numbers, 3);

// Filling
Arrays.fill(numbers, 0);

// Comparing
int[] other = {1, 1, 2, 3, 4, 5, 6, 9};
boolean equal = Arrays.equals(numbers, other);
```

---

## Chapter 10: Collections - Why Simple Things Are Hard

### The Collections Framework: A Hierarchy of Confusion

```java
// The main interfaces
Collection<String> collection = new ArrayList<>();
List<String> list = new ArrayList<>();        // Ordered, allows duplicates
Set<String> set = new HashSet<>();            // No duplicates
Queue<String> queue = new LinkedList<>();     // FIFO operations
Map<String, Integer> map = new HashMap<>();   // Key-value pairs
```

### Lists: Ordered Chaos

```java
List<String> fruits = new ArrayList<>();

// Adding elements
fruits.add("apple");
fruits.add("banana");
fruits.add("cherry");

// Accessing and modifying
System.out.println("First: " + fruits.get(0));
fruits.set(1, "blueberry");
fruits.add(2, "coconut");

// Removing
fruits.remove("cherry");
fruits.remove(0);

// ArrayList vs LinkedList performance
List<Integer> arrayList = new ArrayList<>();    // Fast random access
List<Integer> linkedList = new LinkedList<>();  // Fast insertion/deletion
```

### Sets: The No-Duplicates Club

```java
Set<String> uniqueWords = new HashSet<>();
uniqueWords.add("java");
uniqueWords.add("python");
uniqueWords.add("java");  // Duplicate - won't be added

System.out.println(uniqueWords.size()); // 2, not 3

// Different Set implementations
Set<Integer> hashSet = new HashSet<>();           // No order, fastest
Set<Integer> linkedHashSet = new LinkedHashSet<>(); // Insertion order
Set<Integer> treeSet = new TreeSet<>();           // Sorted order
```

### Maps: The Key-Value Store

```java
Map<String, Integer> ages = new HashMap<>();

// Adding key-value pairs
ages.put("Alice", 30);
ages.put("Bob", 25);
ages.put("Alice", 31); // Updates existing key

// Getting values
Integer aliceAge = ages.get("Alice");
Integer danAge = ages.getOrDefault("Dan", 0);

// Checking existence
if (ages.containsKey("Bob")) {
    System.out.println("Bob is in the map");
}

// Iterating
for (Map.Entry<String, Integer> entry : ages.entrySet()) {
    System.out.println(entry.getKey() + " -> " + entry.getValue());
}

// Java 8+ forEach
ages.forEach((name, age) -> 
    System.out.println(name + " is " + age + " years old")
);
```

### Stream API: Functional Programming Meets Collections

```java
List<String> words = Arrays.asList("apple", "banana", "cherry", "date");

// Filter and collect
List<String> longWords = words.stream()
    .filter(word -> word.length() > 5)
    .collect(Collectors.toList());

// Transform elements
List<String> upperCase = words.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());

// Chain operations
List<Integer> lengths = words.stream()
    .filter(word -> word.startsWith("a"))
    .map(String::length)
    .collect(Collectors.toList());

// Aggregations
OptionalInt max = words.stream()
    .mapToInt(String::length)
    .max();

long count = words.stream()
    .filter(word -> word.contains("a"))
    .count();
```

---

## Chapter 11: Null - The Billion Dollar Mistake

### The NullPointerException

```java
String name = null;
int length = name.length(); // BOOM! NullPointerException
```

**The Quirk**: It's called a "NullPointerException" even though Java doesn't have pointers.

### Null Propagation

```java
public String getMiddleName(Person person) {
    return person.getName().getMiddleName().toUpperCase();
}
```

This can explode in three ways:
1. `person` could be null
2. `person.getName()` could return null
3. `getMiddleName()` could return null

### The Optional Solution (Java 8+)

```java
Optional<String> optional = Optional.ofNullable(getString());
String result = optional.orElse("Default");

// Chaining operations
String processed = optional
    .filter(s -> s.length() > 5)
    .map(String::toUpperCase)
    .orElse("TOO SHORT");
```

### Null Safety Best Practices

```java
// BAD: Null checks everywhere
if (person != null && person.getName() != null && person.getName().length() > 0) {
    // Do something
}

// BETTER: Defensive programming
public String getDisplayName(Person person) {
    if (person == null) return "Unknown";
    String name = person.getName();
    if (name == null || name.trim().isEmpty()) return "Unnamed";
    return name;
}

// BEST: Use Optional
public Optional<String> getDisplayName(Person person) {
    return Optional.ofNullable(person)
        .map(Person::getName)
        .filter(name -> !name.trim().isEmpty());
}
```

---

## Part IV: Advanced Agony

## Chapter 12: Object-Oriented Programming - Inheritance Hierarchy Hell

### Inheritance: The Family Tree of Code

```java
// Parent class
public class Animal {
    protected String name;
    protected int age;
    
    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public void makeSound() {
        System.out.println("Some generic animal sound");
    }
    
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
}

// Child class
public class Dog extends Animal {
    private String breed;
    
    public Dog(String name, int age, String breed) {
        super(name, age);  // Call parent constructor
        this.breed = breed;
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " says: Woof!");
    }
    
    public void fetch() {
        System.out.println(name + " is fetching a ball");
    }
}
```

### The `super` Keyword

```java
public class Cat extends Animal {
    @Override
    public void makeSound() {
        super.sleep();  // Call parent method
        System.out.println(name + " says: Meow!");
    }
}
```

### Method Overriding Rules

```java
public class Parent {
    public void method1() { }
    protected void method2() { }
    void method3() { }
    private void method4() { }  // Can't be overridden
    
    public final void method5() { }  // Can't be overridden
}

public class Child extends Parent {
    @Override
    public void method1() { }     // OK: same or more visible
    
    @Override
    public void method2() { }     // OK: protected -> public
    
    // @Override
    // private void method2() { }  // ERROR: can't reduce visibility
    
    @Override
    void method3() { }            // OK: same visibility
    
    // Can't override method4() (private) or method5() (final)
}
```

### Polymorphism: One Interface, Many Forms

```java
Animal[] animals = {
    new Dog("Rex", 3, "Labrador"),
    new Cat("Whiskers", 2, "Persian"),
    new Dog("Buddy", 5, "Golden Retriever")
};

for (Animal animal : animals) {
    animal.makeSound();  // Calls the appropriate overridden method
    // Output:
    // Rex says: Woof!
    // Whiskers says: Meow!
    // Buddy says: Woof!
}
```

### The Object Class: Everyone's Ancestor

Every class implicitly extends Object:

```java
public class MyClass {  // Implicitly extends Object
    // Gets these methods for free:
    // toString(), equals(), hashCode(), getClass()
}

// Important methods to override
@Override
public String toString() {
    return "Person{name='" + name + "', age=" + age + "}";
}

@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (obj == null || getClass() != obj.getClass()) return false;
    Person person = (Person) obj;
    return age == person.age && Objects.equals(name, person.name);
}

@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```

---

## Chapter 13: Interfaces and Abstract Classes - Contract Confusion

### Interfaces: The Contract System

```java
public interface Drawable {
    void draw();  // Implicitly public abstract
    
    default void erase() {  // Java 8+ default methods
        System.out.println("Erasing...");
    }
    
    static void info() {  // Java 8+ static methods
        System.out.println("Drawable interface info");
    }
}

public class Circle implements Drawable {
    private double radius;
    
    @Override
    public void draw() {
        System.out.println("Drawing a circle with radius " + radius);
    }
}
```

### Multiple Inheritance of Interfaces

```java
interface Flyable {
    void fly();
}

interface Swimmable {
    void swim();
}

// A class can implement multiple interfaces
class Duck implements Flyable, Swimmable {
    @Override
    public void fly() {
        System.out.println("Duck is flying");
    }
    
    @Override
    public void swim() {
        System.out.println("Duck is swimming");
    }
}
```

### Abstract Classes: The Middle Ground

```java
public abstract class Shape {
    protected String color;
    
    public Shape(String color) {
        this.color = color;
    }
    
    // Abstract method - must be implemented by subclasses
    public abstract double area();
    
    // Concrete method - can be used by subclasses
    public void displayInfo() {
        System.out.println("Shape color: " + color);
        System.out.println("Area: " + area());
    }
}

public class Rectangle extends Shape {
    private double width, height;
    
    public Rectangle(String color, double width, double height) {
        super(color);
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double area() {
        return width * height;
    }
}
```

### Interface vs Abstract Class Decision Tree

**Use Interface when:**
- You want multiple inheritance
- You're defining a contract/capability
- You expect unrelated classes to implement it
- You want to support polymorphism

**Use Abstract Class when:**
- You want to share code among related classes
- You need to define non-public members
- You want to provide default implementations
- You need constructors

---

## Chapter 14: Exception Handling - Catch Me If You Can

### The Exception Hierarchy

```java
Throwable
├── Error (don't catch these)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── VirtualMachineError
└── Exception
    ├── RuntimeException (unchecked)
    │   ├── NullPointerException
    │   ├── IllegalArgumentException
    │   └── ArrayIndexOutOfBoundsException
    └── Checked Exceptions
        ├── IOException
        ├── SQLException
        └── ClassNotFoundException
```

### Try-Catch-Finally: The Safety Net

```java
public void readFile(String filename) {
    FileInputStream file = null;
    try {
        file = new FileInputStream(filename);
        // Process file
    } catch (FileNotFoundException e) {
        System.out.println("File not found: " + e.getMessage());
    } catch (IOException e) {
        System.out.println("IO error: " + e.getMessage());
    } finally {
        // Always executes (cleanup code)
        if (file != null) {
            try {
                file.close();
            } catch (IOException e) {
                System.out.println("Error closing file");
            }
        }
    }
}
```

### Try-With-Resources: The Better Way

```java
// Java 7+ automatically closes resources
public void readFileModern(String filename) {
    try (FileInputStream file = new FileInputStream(filename);
         BufferedInputStream buffer = new BufferedInputStream(file)) {
        
        // Process file
        // Resources automatically closed
        
    } catch (IOException e) {
        System.out.println("Error: " + e.getMessage());
    }
}
```

### Custom Exceptions

```java
public class InsufficientFundsException extends Exception {
    private double balance;
    private double requestedAmount;
    
    public InsufficientFundsException(double balance, double requested) {
        super("Insufficient funds: balance=" + balance + ", requested=" + requested);
        this.balance = balance;
        this.requestedAmount = requested;
    }
    
    public double getBalance() { return balance; }
    public double getRequestedAmount() { return requestedAmount; }
}

public void withdraw(double amount) throws InsufficientFundsException {
    if (amount > balance) {
        throw new InsufficientFundsException(balance, amount);
    }
    balance -= amount;
}
```

### Exception Best Practices

```java
// BAD: Catching too broadly
try {
    doSomething();
} catch (Exception e) {
    // Hides all problems
}

// BAD: Empty catch block
try {
    riskyOperation();
} catch (Exception e) {
    // Silently ignores problems
}

// GOOD: Specific exception handling
try {
    parseNumber(input);
} catch (NumberFormatException e) {
    logger.warn("Invalid number format: " + input, e);
    return DEFAULT_VALUE;
} catch (NullPointerException e) {
    logger.error("Null input provided", e);
    throw new IllegalArgumentException("Input cannot be null");
}
```

---

## Chapter 15: Generics - Type Erasure and You

### Basic Generics

```java
// Without generics (pre-Java 5)
List list = new ArrayList();
list.add("Hello");
list.add(42);  // Oops, mixed types!
String s = (String) list.get(0);  // Casting required

// With generics
List<String> strings = new ArrayList<String>();
strings.add("Hello");
// strings.add(42);  // Compile-time error!
String s = strings.get(0);  // No casting needed
```

### Generic Classes

```java
public class Box<T> {
    private T contents;
    
    public void set(T contents) {
        this.contents = contents;
    }
    
    public T get() {
        return contents;
    }
}

// Usage
Box<String> stringBox = new Box<String>();
stringBox.set("Hello");
String value = stringBox.get();

Box<Integer> intBox = new Box<>();  // Diamond operator (Java 7+)
intBox.set(42);
Integer number = intBox.get();
```

### Generic Methods

```java
public class Utility {
    // Generic method
    public static <T> void swap(T[] array, int i, int j) {
        T temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }
    
    // Bounded generic method
    public static <T extends Comparable<T>> T max(T a, T b) {
        return a.compareTo(b) > 0 ? a : b;
    }
}

// Usage
String[] names = {"Alice", "Bob", "Charlie"};
Utility.swap(names, 0, 2);  // Type inferred

Integer maximum = Utility.max(10, 20);  // Returns 20
```

### Wildcards: The Question Mark of Confusion

```java
// Upper bounded wildcard
List<? extends Number> numbers = new ArrayList<Integer>();
numbers = new ArrayList<Double>();  // Also valid
// numbers.add(42);  // Compiler error! Can't add to ? extends

// Lower bounded wildcard
List<? super Integer> integers = new ArrayList<Number>();
integers = new ArrayList<Object>();  // Also valid
integers.add(42);  // OK - can add Integer or subclasses

// Unbounded wildcard
List<?> mystery = new ArrayList<String>();
mystery = new ArrayList<Integer>();  // Any type
// mystery.add("Hello");  // Compiler error! Can't add anything except null
Object obj = mystery.get(0);  // Can only get Object
```

### Type Erasure: The Disappearing Act

```java
List<String> strings = new ArrayList<>();
List<Integer> numbers = new ArrayList<>();

// At runtime, both are just List<Object>
System.out.println(strings.getClass() == numbers.getClass()); // true

// Generic type information is erased
public void method(List<String> strings) { }
// public void method(List<Integer> numbers) { }  // Compile error!
// Both methods have same erasure: method(List)
```

**The Problem**: Generic types exist only at compile time. At runtime, they're erased for backward compatibility.

### Generic Gotchas

```java
// Can't create generic arrays
// List<String>[] arrays = new List<String>[10];  // Compile error
List<String>[] arrays = new List[10];  // Have to use raw type

// Can't use primitives
// List<int> numbers = new ArrayList<>();  // Error
List<Integer> numbers = new ArrayList<>();  // Must use wrapper

// Can't instantiate type parameter
public class GenericClass<T> {
    // T instance = new T();  // Error - don't know what T is
    
    // Workaround: pass Class<T>
    public T createInstance(Class<T> clazz) throws Exception {
        return clazz.newInstance();
    }
}
```

---

## Part V: Expert-Level Existential Crisis

## Chapter 16: Threading - Race Conditions as a Feature

### Thread Creation

```java
// Method 1: Extend Thread
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Hello from thread: " + Thread.currentThread().getName());
    }
}

// Method 2: Implement Runnable
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Hello from runnable: " + Thread.currentThread().getName());
    }
}

// Usage
Thread thread1 = new MyThread();
Thread thread2 = new Thread(new MyRunnable());
Thread thread3 = new Thread(() -> {  // Lambda (Java 8+)
    System.out.println("Hello from lambda thread");
});

thread1.start();
thread2.start();
thread3.start();
```

### The Synchronized Keyword

```java
public class Counter {
    private int count = 0;
    
    // Synchronized method
    public synchronized void increment() {
        count++;  // Thread-safe
    }
    
    // Synchronized block
    public void decrement() {
        synchronized(this) {
            count--;  // Thread-safe
        }
    }
    
    public synchronized int getCount() {
        return count;
    }
}
```

### Race Conditions: When Threads Compete

```java
public class BankAccount {
    private int balance = 100;
    
    // DANGEROUS: Race condition
    public void withdraw(int amount) {
        if (balance >= amount) {        // Check
            // Another thread might modify balance here!
            System.out.println("Withdrawing " + amount);
            balance -= amount;          // Update
            System.out.println("New balance: " + balance);
        } else {
            System.out.println("Insufficient funds");
        }
    }
    
    // SAFE: Synchronized
    public synchronized void withdrawSafe(int amount) {
        if (balance >= amount) {
            System.out.println("Withdrawing " + amount);
            balance -= amount;
            System.out.println("New balance: " + balance);
        } else {
            System.out.println("Insufficient funds");
        }
    }
}
```

### Producer-Consumer Problem

```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ArrayBlockingQueue;

class Producer implements Runnable {
    private BlockingQueue<Integer> queue;
    
    public Producer(BlockingQueue<Integer> queue) {
        this.queue = queue;
    }
    
    @Override
    public void run() {
        try {
            for (int i = 0; i < 10; i++) {
                queue.put(i);  // Blocks if queue is full
                System.out.println("Produced: " + i);
                Thread.sleep(100);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

class Consumer implements Runnable {
    private BlockingQueue<Integer> queue;
    
    public Consumer(BlockingQueue<Integer> queue) {
        this.queue = queue;
    }
    
    @Override
    public void run() {
        try {
            while (true) {
                Integer item = queue.take();  // Blocks if queue is empty
                System.out.println("Consumed: " + item);
                Thread.sleep(200);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

// Usage
BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(5);
Thread producer = new Thread(new Producer(queue));
Thread consumer = new Thread(new Consumer(queue));

producer.start();
consumer.start();
```

### Deadlock: When Threads Get Stuck

```java
public class DeadlockExample {
    private static final Object lock1 = new Object();
    private static final Object lock2 = new Object();
    
    public static void main(String[] args) {
        Thread thread1 = new Thread(() -> {
            synchronized (lock1) {
                System.out.println("Thread 1 acquired lock1");
                try { Thread.sleep(100); } catch (InterruptedException e) {}
                
                synchronized (lock2) {  // Waiting for lock2
                    System.out.println("Thread 1 acquired lock2");
                }
            }
        });
        
        Thread thread2 = new Thread(() -> {
            synchronized (lock2) {
                System.out.println("Thread 2 acquired lock2");
                try { Thread.sleep(100); } catch (InterruptedException e) {}
                
                synchronized (lock1) {  // Waiting for lock1
                    System.out.println("Thread 2 acquired lock1");
                }
            }
        });
        
        thread1.start();
        thread2.start();
        // Result: Deadlock! Both threads wait forever
    }
}
```

---

## Chapter 17: I/O and NIO - Reading Files Shouldn't Be This Hard

### Traditional I/O (java.io)

```java
import java.io.*;

// Reading a file the old way
public String readFileOldWay(String filename) throws IOException {
    FileInputStream fis = null;
    BufferedInputStream bis = null;
    StringBuilder content = new StringBuilder();
    
    try {
        fis = new FileInputStream(filename);
        bis = new BufferedInputStream(fis);
        
        int data;
        while ((data = bis.read()) != -1) {
            content.append((char) data);
        }
    } finally {
        if (bis != null) bis.close();
        if (fis != null) fis.close();
    }
    
    return content.toString();
}

// Better way with try-with-resources
public String readFileModern(String filename) throws IOException {
    StringBuilder content = new StringBuilder();
    
    try (BufferedReader reader = new BufferedReader(new FileReader(filename))) {
        String line;
        while ((line = reader.readLine()) != null) {
            content.append(line).append("\n");
        }
    }
    
    return content.toString();
}
```

### NIO.2 (java.nio.file) - The Modern Way

```java
import java.nio.file.*;
import java.util.List;

public class ModernFileIO {
    public String readFileSimple(String filename) throws IOException {
        // One line to read entire file!
        return Files.readString(Paths.get(filename));
    }
    
    public List<String> readFileLines(String filename) throws IOException {
        return Files.readAllLines(Paths.get(filename));
    }
    
    public void writeFile(String filename, String content) throws IOException {
        Files.writeString