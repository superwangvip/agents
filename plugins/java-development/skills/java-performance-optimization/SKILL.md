---
name: java-performance-optimization
description: Master Java performance tuning, JVM optimization, garbage collection tuning, memory management, and application profiling. Use when optimizing Java applications for throughput, latency, memory usage, or scalability.
---

# Java Performance Optimization

Comprehensive guide to Java performance optimization including JVM tuning, garbage collection optimization, memory management, profiling techniques, and enterprise application performance strategies.

## When to Use This Skill

- Optimizing Java applications for high throughput or low latency
- Tuning JVM garbage collection for specific workload characteristics
- Analyzing memory usage and identifying memory leaks
- Profiling Java applications to identify performance bottlenecks
- Configuring JVM parameters for production environments
- Implementing caching and pooling strategies
- Optimizing database access and I/O operations
- Monitoring and measuring application performance metrics

## JVM Performance Fundamentals

### Memory Management Architecture

```
Heap Memory Layout:
┌─────────────────────────────────────────────────────┐
│                    Young Generation                  │
├─────────────────┬───────────────────────────────────┤
│   Eden Space    │        Survivor Spaces             │
│                 │    S0    |    S1                  │
└─────────────────┴───────────────────────────────────┘
┌─────────────────────────────────────────────────────┐
│                  Old Generation                     │
├─────────────────────────────────────────────────────┤
│                 Tenured Objects                     │
└─────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────┐
│                Metaspace/PermGen                    │
├─────────────────────────────────────────────────────┤
│              Class Metadata                         │
└─────────────────────────────────────────────────────┘
```

### GC Algorithms Comparison

| Algorithm | Best For | Latency | Throughput | Memory Usage |
|-----------|----------|---------|-------------|--------------|
| Serial GC | Small applications, client machines | High | Low | Minimal |
| Parallel GC | Batch processing, throughput-critical | Medium | High | Medium |
| G1GC | Low latency requirements, large heaps | Low | Medium | High |
| ZGC | Ultra-low latency, very large heaps | Ultra-low | Medium | Very High |
| Shenandoah | Low latency, large heaps | Very Low | Medium | High |

## JVM Tuning Examples

### Production JVM Configuration

```bash
# G1GC for low-latency applications
java -Xms2g -Xmx2g \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -XX:G1HeapRegionSize=16m \
     -XX:+G1UseAdaptiveIHOP \
     -XX:G1MixedGCCountTarget=8 \
     -XX:InitiatingHeapOccupancyPercent=45 \
     -XX:+PrintGCDetails \
     -XX:+PrintGCTimeStamps \
     -XX:+PrintGCDateStamps \
     -Xloggc:/var/log/app/gc.log \
     -XX:+UseGCLogFileRotation \
     -XX:NumberOfGCLogFiles=5 \
     -XX:GCLogFileSize=10M \
     -jar application.jar

# Parallel GC for throughput-critical applications
java -Xms4g -Xmx4g \
     -XX:+UseParallelGC \
     -XX:ParallelGCThreads=8 \
     -XX:+UseParallelOldGC \
     -XX:MaxGCPauseMillis=300 \
     -XX:+AlwaysPreTouch \
     -XX:+PrintGCDetails \
     -XX:+PrintGCTimeStamps \
     -jar application.jar

# ZGC for ultra-low latency
java -Xms8g -Xmx8g \
     -XX:+UseZGC \
     -XX:+UnlockExperimentalVMOptions \
     -XX:ZAllocationSpikeTolerance=5 \
     -XX:+PrintGCDetails \
     -Xlog:gc*:file=/var/log/app/zgc-gc.log:time \
     -jar application.jar
```

### Container-Aware JVM Configuration

```bash
# Kubernetes/Docker optimized JVM settings
java -XX:+UseContainerSupport \
     -XX:MaxRAMPercentage=75.0 \
     -XX:InitialRAMPercentage=75.0 \
     -XX:+UseG1GC \
     -XX:G1HeapRegionSize=16m \
     -XX:+UseStringDeduplication \
     -XX:+PrintGCDateStamps \
     -XX:+PrintGCTimeStamps \
     -Xlog:gc*:file=/app/logs/gc.log:time,tid,level,tags:filecount=6,filesize=10M \
     -jar application.jar

# For microservices with small heap sizes
java -XX:+UseSerialGC \
     -XX:MaxRAMPercentage=50.0 \
     -XX:+UseStringDeduplication \
     -XX:+OptimizeStringConcat \
     -XX:+CompactStrings \
     -jar microservice.jar
```

## Memory Optimization Patterns

### Object Pooling

```java
// Generic object pool implementation
public class ObjectPool<T> {

    private final Queue<T> pool;
    private final Supplier<T> factory;
    private final Consumer<T> resetAction;
    private final int maxSize;

    public ObjectPool(Supplier<T> factory, Consumer<T> resetAction, int maxSize) {
        this.pool = new ConcurrentLinkedQueue<>();
        this.factory = factory;
        this.resetAction = resetAction;
        this.maxSize = maxSize;
    }

    public T acquire() {
        T object = pool.poll();
        return object != null ? object : factory.get();
    }

    public void release(T object) {
        if (object == null) return;

        resetAction.accept(object);

        if (pool.size() < maxSize) {
            pool.offer(object);
        }
    }

    public int size() {
        return pool.size();
    }
}

// Usage with expensive objects
public class ExpensiveObjectPool {

    private static final ObjectPool<ExpensiveObject> pool = new ObjectPool<>(
        ExpensiveObject::new,
        ExpensiveObject::reset,
        100
    );

    public void processWithPool() {
        ExpensiveObject obj = pool.acquire();
        try {
            // Use the expensive object
            obj.doExpensiveWork();
        } finally {
            pool.release(obj);
        }
    }
}

// Array pooling for primitive arrays
public class ArrayPool {

    private static final Map<Integer, Queue<byte[]>> BYTE_ARRAY_POOLS = new ConcurrentHashMap<>();

    public static byte[] acquireByteArray(int size) {
        Queue<byte[]> pool = BYTE_ARRAY_POOLS.computeIfAbsent(size, k -> new ConcurrentLinkedQueue<>());
        byte[] array = pool.poll();
        return array != null ? array : new byte[size];
    }

    public static void releaseByteArray(byte[] array) {
        if (array == null) return;

        int size = array.length;
        Arrays.fill(array, (byte) 0); // Clear sensitive data

        Queue<byte[]> pool = BYTE_ARRAY_POOLS.get(size);
        if (pool != null && pool.size() < 10) {
            pool.offer(array);
        }
    }
}
```

### Memory-Efficient Collections

```java
// Efficient primitive collections
public class PrimitiveCollectionsExample {

    // Use primitive collections instead of boxed types
    private final IntArrayList intList = new IntArrayList();
    private final Long2ObjectHashMap<String> longToStringMap = new Long2ObjectHashMap<>();
    private final ObjectArrayList<String> stringList = new ObjectArrayList<>();

    // Efficient bit set for boolean flags
    private final BitSet flags = new BitSet();

    // Custom memory-efficient cache with weak references
    private static final Map<String, WeakReference<ExpensiveObject>> cache = new ConcurrentHashMap<>();

    public ExpensiveObject getCachedObject(String key, Supplier<ExpensiveObject> factory) {
        WeakReference<ExpensiveObject> ref = cache.get(key);
        ExpensiveObject obj = ref != null ? ref.get() : null;

        if (obj == null) {
            obj = factory.get();
            cache.put(key, new WeakReference<>(obj));
        }

        return obj;
    }

    // Off-heap memory usage
    private static final ByteBuffer offHeapBuffer = ByteBuffer.allocateDirect(1024 * 1024);

    public void useOffHeapMemory() {
        offHeapBuffer.clear();
        // Write to off-heap memory
        offHeapBuffer.putInt(123);
        offHeapBuffer.putDouble(456.789);

        // Read from off-heap memory
        offHeapBuffer.flip();
        int intValue = offHeapBuffer.getInt();
        double doubleValue = offHeapBuffer.getDouble();
    }
}
```

### String Optimization

```java
public class StringOptimizationExamples {

    // String deduplication
    public void demonstrateStringDeduplication() {
        // Enable with JVM flag: -XX:+UseStringDeduplication

        List<String> strings = new ArrayList<>();
        for (int i = 0; i < 100000; i++) {
            // These strings will be deduplicated by G1GC
            strings.add("common_string_value_" + (i % 100));
        }
    }

    // Efficient string concatenation
    public String efficientConcatenation(List<String> parts) {
        // Use StringJoiner for complex concatenations
        StringJoiner joiner = new StringJoiner(", ", "[", "]");
        parts.forEach(joiner::add);
        return joiner.toString();
    }

    // StringBuilder with initial capacity
    public String optimizedStringBuilder(List<String> parts) {
        // Calculate approximate capacity to avoid resizing
        int estimatedCapacity = parts.stream()
            .mapToInt(String::length)
            .sum() + parts.size() * 2; // Add buffer

        StringBuilder builder = new StringBuilder(estimatedCapacity);
        parts.forEach(builder::append);
        return builder.toString();
    }

    // String interning for frequently used strings
    private static final Map<String, String> INTERNED_STRINGS = new ConcurrentHashMap<>();

    public String getInternedString(String original) {
        return INTERNED_STRINGS.computeIfAbsent(original, k -> k.intern());
    }

    // Compact strings (Java 9+)
    public void demonstrateCompactStrings() {
        // Enable with JVM flag: -XX:+CompactStrings

        String latin1 = "Hello World"; // Stored as LATIN-1
        String utf16 = "Hello 世界"; // Stored as UTF-16

        System.out.println("LATIN1 bytes: " + latin1.getBytes().length);
        System.out.println("UTF-16 bytes: " + utf16.getBytes().length);
    }
}
```

## I/O and Network Optimization

### NIO Buffer Management

```java
import java.nio.*;
import java.nio.channels.*;
import java.nio.file.*;

public class NIOOptimizationExamples {

    // Direct buffer for I/O operations
    private final ByteBuffer directBuffer = ByteBuffer.allocateDirect(8192);

    // Efficient file reading with memory mapping
    public void readFileWithMemoryMapping(String filePath) throws IOException {
        try (RandomAccessFile file = new RandomAccessFile(filePath, "r");
             FileChannel channel = file.getChannel()) {

            long fileSize = channel.size();
            MappedByteBuffer buffer = channel.map(
                FileChannel.MapMode.READ_ONLY, 0, fileSize
            );

            // Process file contents directly from memory
            while (buffer.hasRemaining()) {
                byte b = buffer.get();
                // Process byte
            }
        }
    }

    // Buffer pooling
    private static final ObjectPool<ByteBuffer> bufferPool = new ObjectPool<>(
        () -> ByteBuffer.allocateDirect(8192),
        ByteBuffer::clear,
        50
    );

    public void performIOWithPooledBuffers() {
        ByteBuffer buffer = bufferPool.acquire();
        try {
            // Use buffer for I/O operations
            buffer.clear();
            // Read/write operations
        } finally {
            bufferPool.release(buffer);
        }
    }

    // Asynchronous file I/O
    public CompletableFuture<Long> readFileAsync(String filePath) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                return Files.size(Paths.get(filePath));
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        });
    }

    // Efficient network server with NIO.2
    public void createAsynchronousServer() throws IOException {
        AsynchronousServerSocketChannel serverChannel = AsynchronousServerSocketChannel.open();
        serverChannel.bind(new InetSocketAddress(8080));

        serverChannel.accept(null, new CompletionHandler<AsynchronousSocketChannel, Void>() {
            @Override
            public void completed(AsynchronousSocketChannel clientChannel, Void attachment) {
                // Accept next connection
                serverChannel.accept(null, this);

                // Handle current connection
                handleClient(clientChannel);
            }

            @Override
            public void failed(Throwable exc, Void attachment) {
                exc.printStackTrace();
            }
        });
    }

    private void handleClient(AsynchronousSocketChannel clientChannel) {
        ByteBuffer buffer = ByteBuffer.allocateDirect(1024);

        clientChannel.read(buffer, buffer, new CompletionHandler<Integer, ByteBuffer>() {
            @Override
            public void completed(Integer bytesRead, ByteBuffer buffer) {
                if (bytesRead == -1) {
                    // Connection closed
                    try {
                        clientChannel.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                    return;
                }

                buffer.flip();
                // Process data
                processBuffer(buffer);
                buffer.clear();

                // Continue reading
                clientChannel.read(buffer, buffer, this);
            }

            @Override
            public void failed(Throwable exc, ByteBuffer buffer) {
                exc.printStackTrace();
                try {
                    clientChannel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        });
    }

    private void processBuffer(ByteBuffer buffer) {
        // Process received data
    }
}
```

### Connection Pooling

```java
import javax.sql.DataSource;
import org.apache.commons.dbcp2.BasicDataSource;

public class ConnectionPoolingExample {

    // Optimized database connection pool
    public DataSource createOptimizedDataSource() {
        BasicDataSource dataSource = new BasicDataSource();

        // Connection configuration
        dataSource.setUrl("jdbc:postgresql://localhost:5432/mydb");
        dataSource.setUsername("user");
        dataSource.setPassword("password");
        dataSource.setDriverClassName("org.postgresql.Driver");

        // Pool configuration
        dataSource.setInitialSize(5);
        dataSource.setMaxTotal(20);
        dataSource.setMaxIdle(10);
        dataSource.setMinIdle(5);

        // Performance tuning
        dataSource.setMaxWaitMillis(30000);
        dataSource.setValidationQuery("SELECT 1");
        dataSource.setTestOnBorrow(true);
        dataSource.setTestOnReturn(false);
        dataSource.setTestWhileIdle(true);
        dataSource.setTimeBetweenEvictionRunsMillis(60000);
        dataSource.setMinEvictableIdleTimeMillis(300000);
        dataSource.setNumTestsPerEvictionRun(3);

        return dataSource;
    }

    // HTTP connection pooling
    public HttpClient createPooledHttpClient() {
        return HttpClient.newBuilder()
            .executor(Executors.newFixedThreadPool(10))
            .connectTimeout(Duration.ofSeconds(10))
            .build();
    }

    // Redis connection pooling
    public JedisPool createJedisPool() {
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(20);
        config.setMaxIdle(10);
        config.setMinIdle(5);
        config.setTestOnBorrow(true);
        config.setTestOnReturn(false);
        config.setTestWhileIdle(true);
        config.setTimeBetweenEvictionRunsMillis(60000);
        config.setMinEvictableIdleTimeMillis(300000);

        return new JedisPool(config, "localhost", 6379);
    }
}
```

## Profiling and Monitoring

### Custom Performance Metrics

```java
import java.lang.management.*;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicLong;

public class PerformanceMonitoring {

    private final Timer requestTimer = Timer.start();
    private final AtomicLong requestCount = new AtomicLong(0);
    private final AtomicLong errorCount = new AtomicLong(0);
    private final ConcurrentLinkedQueue<Long> responseTimeSamples = new ConcurrentLinkedQueue<>();

    // Request timing
    public <T> T measureRequest(String operation, Supplier<T> supplier) {
        long startTime = System.nanoTime();
        requestCount.incrementAndGet();

        try {
            T result = supplier.get();
            long duration = System.nanoTime() - startTime;
            recordResponseTime(duration);
            return result;
        } catch (Exception e) {
            errorCount.incrementAndGet();
            throw e;
        }
    }

    private void recordResponseTime(long durationNanos) {
        // Keep only last 1000 samples
        if (responseTimeSamples.size() >= 1000) {
            responseTimeSamples.poll();
        }
        responseTimeSamples.offer(durationNanos);
    }

    // Performance statistics
    public PerformanceStats getStats() {
        long totalRequests = requestCount.get();
        long totalErrors = errorCount.get();

        // Calculate percentiles
        List<Long> samples = new ArrayList<>(responseTimeSamples);
        Collections.sort(samples);

        long p50 = samples.size() > 0 ? samples.get(samples.size() / 2) : 0;
        long p95 = samples.size() > 0 ? samples.get((int) (samples.size() * 0.95)) : 0;
        long p99 = samples.size() > 0 ? samples.get((int) (samples.size() * 0.99)) : 0;

        return new PerformanceStats(
            totalRequests,
            totalErrors,
            p50 / 1_000_000.0, // Convert to milliseconds
            p95 / 1_000_000.0,
            p99 / 1_000_000.0,
            getMemoryUsage()
        );
    }

    private MemoryUsage getMemoryUsage() {
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();

        return new MemoryUsage(
            heapUsage.getUsed(),
            heapUsage.getMax(),
            heapUsage.getUsed() * 100.0 / heapUsage.getMax()
        );
    }

    // JMX registration
    public void registerAsMBean() {
        try {
            MBeanServer mbs = ManagementFactory.getPlatformMBeanServer();
            ObjectName name = new ObjectName("com.example.performance:type=PerformanceMonitor");
            mbs.registerMBean(new PerformanceMonitorMBean(), name);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public record PerformanceStats(
        long totalRequests,
        long totalErrors,
        double responseTimeP50,
        double responseTimeP95,
        double responseTimeP99,
        MemoryUsage memoryUsage
    ) {}

    public record MemoryUsage(long used, long max, double utilizationPercent) {}
}

// MBean interface for JMX monitoring
public interface PerformanceMonitorMBeanMBean {
    long getTotalRequests();
    long getTotalErrors();
    double getResponseTimeP50();
    double getResponseTimeP95();
    double getResponseTimeP99();
    long getMemoryUsed();
    double getMemoryUtilizationPercent();
}
```

### Custom Profiling Tools

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

public class MethodProfiler {

    private final ConcurrentHashMap<String, AtomicLong> methodCounts = new ConcurrentHashMap<>();
    private final ConcurrentHashMap<String, AtomicLong> methodTotalTime = new ConcurrentHashMap<>();

    public <T> T profile(String methodName, Supplier<T> method) {
        long startTime = System.nanoTime();
        methodCounts.computeIfAbsent(methodName, k -> new AtomicLong(0)).incrementAndGet();

        try {
            T result = method.get();
            long duration = System.nanoTime() - startTime;
            methodTotalTime.computeIfAbsent(methodName, k -> new AtomicLong(0))
                .addAndGet(duration);
            return result;
        } catch (Exception e) {
            long duration = System.nanoTime() - startTime;
            methodTotalTime.computeIfAbsent(methodName, k -> new AtomicLong(0))
                .addAndGet(duration);
            throw e;
        }
    }

    public void printStats() {
        System.out.println("Method Profile:");
        System.out.println("=".repeat(50));

        methodCounts.forEach((method, count) -> {
            long totalTime = methodTotalTime.get(method).get();
            double avgTimeMs = totalTime / (double) count / 1_000_000.0;

            System.out.printf("%-30s %10d calls %12.2f ms avg%n",
                method, count.get(), avgTimeMs);
        });
    }

    public void resetStats() {
        methodCounts.clear();
        methodTotalTime.clear();
    }
}

// Usage example
public class ProfiledService {

    private final MethodProfiler profiler = new MethodProfiler();

    public String processRequest(String input) {
        return profiler.profile("processRequest", () -> {
            return doActualProcessing(input);
        });
    }

    private String doActualProcessing(String input) {
        try {
            Thread.sleep(100); // Simulate work
            return "Processed: " + input;
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException(e);
        }
    }

    public void printProfilingStats() {
        profiler.printStats();
    }
}
```

## Production Performance Checklist

### JVM Configuration Validation

```bash
#!/bin/bash
# validate-jvm-config.sh

echo "Validating JVM configuration..."

# Check if the application is running
if ! pgrep -f "application.jar" > /dev/null; then
    echo "ERROR: Application is not running"
    exit 1
fi

# Get JVM process ID
PID=$(pgrep -f "application.jar")

# Check JVM arguments
echo "JVM Arguments:"
ps -p $PID -o args

# Check heap usage
echo -e "\nHeap Usage:"
jstat -gc $PID

# Check GC activity
echo -e "\nGC Activity Summary:"
jstat -gcutil $PID

# Check memory usage
echo -e "\nMemory Usage:"
jmap -histo $PID | head -20

# Check thread count
echo -e "\nThread Count:"
jstack $PID | grep -c "java.lang.Thread.State"

# Check for deadlocks
echo -e "\nDeadlock Detection:"
jstack $PID | grep -A 10 "Found one Java-level deadlock"

echo "JVM validation completed."
```

### Performance Monitoring Dashboard

```java
@RestController
@RequestMapping("/api/performance")
public class PerformanceController {

    private final PerformanceMonitor performanceMonitor;

    @GetMapping("/stats")
    public ResponseEntity<PerformanceStats> getPerformanceStats() {
        return ResponseEntity.ok(performanceMonitor.getStats());
    }

    @GetMapping("/gc")
    public ResponseEntity<GcStats> getGcStats() {
        List<GarbageCollectorMXBean> gcBeans = ManagementFactory.getGarbageCollectorMXBeans();
        GcStats stats = new GcStats(
            gcBeans.stream()
                .mapToLong(GarbageCollectorMXBean::getCollectionCount)
                .sum(),
            gcBeans.stream()
                .mapToLong(GarbageCollectorMXBean::getCollectionTime)
                .sum()
        );
        return ResponseEntity.ok(stats);
    }

    @GetMapping("/threads")
    public ResponseEntity<ThreadStats> getThreadStats() {
        ThreadMXBean threadBean = ManagementFactory.getThreadMXBean();
        ThreadStats stats = new ThreadStats(
            threadBean.getThreadCount(),
            threadBean.getDaemonThreadCount(),
            threadBean.getPeakThreadCount(),
            threadBean.getTotalStartedThreadCount()
        );
        return ResponseEntity.ok(stats);
    }

    @GetMapping("/memory")
    public ResponseEntity<MemoryStats> getMemoryStats() {
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
        MemoryUsage nonHeapUsage = memoryBean.getNonHeapMemoryUsage();

        MemoryStats stats = new MemoryStats(
            new MemoryInfo(heapUsage.getUsed(), heapUsage.getMax()),
            new MemoryInfo(nonHeapUsage.getUsed(), nonHeapUsage.getMax())
        );
        return ResponseEntity.ok(stats);
    }

    public record GcStats(long totalCollections, long totalCollectionTime) {}
    public record ThreadStats(int threadCount, int daemonThreadCount, int peakThreadCount, long totalStarted) {}
    public record MemoryInfo(long used, long max) {}
    public record MemoryStats(MemoryInfo heap, MemoryInfo nonHeap) {}
}
```

This comprehensive Java performance optimization guide provides practical strategies for JVM tuning, memory optimization, I/O optimization, and performance monitoring in enterprise Java applications.