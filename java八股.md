### 一、Java 基础与类型系统

| 知识模块 | 核心知识点 | 深度解析 (面试常挖) |
| :--- | :--- | :--- |
| **八大基本类型** | `byte/short/int/long` <br> `float/double` <br> `char` <br> `boolean` | 1. 包装类型 `Integer` 缓存池 `[-128,127]`，`valueOf()` 与 `new` 区别。<br>2. 自动装箱/拆箱底层是 `Integer.valueOf()` / `intValue()`。<br>3. `float` 精度丢失原理，`BigDecimal` 的正确使用 (`compareTo` vs `equals`)。<br>4. `boolean` 在 JVM 中的实际占用大小 (虚拟机未严格定义，多按 `int` 处理)。 |
| **引用类型** | `String` | 1. 不可变性：`final char[]`，常量池 (JDK7 后从方法区移入堆)。<br>2. `String` 拼接底层用 `StringBuilder`，但循环内拼接仍需显式使用 `StringBuilder`。<br>3. `intern()` 在不同 JDK 版本下的行为。 |
| **访问修饰符** | `private / default / protected / public` | 1. 访问权限精确表 (同类、同包、子类、任意)，尤其 `protected` 的子类可见性。<br>2. `default` (包级别) 是 Java 独特的设计，接口中 `default` 方法与此不同。<br>3. `abstract` 和 `final` 不能与 `private` 组合的原因。 |
| **关键字** | `static` / `final` / `transient` / `volatile` / `synchronized` | 1. `static` 变量在方法区 (元空间) 的存储，`<clinit>` 线程安全。<br>2. `final` 修饰类、方法、变量的区别，`final` 与不可变对象的关系。<br>3. `transient` 配合序列化，静态变量不会被序列化。 |

---

### 二、集合框架深度分析

以 **`List`、`Map`、`Set`** 为主线，掌握接口选择、实现差异、并发安全。

#### 1. List 家族

| 实现类 | 数据结构 | 扩容机制 | 线程安全 | 业务口诀 |
| :--- | :--- | :--- | :--- | :--- |
| **`ArrayList`** | `Object[]` 数组 | 默认10，每次扩容至 1.5 倍 (`newCapacity = old + old >> 1`)，`Arrays.copyOf` 复制。 | ❌ 不安全 | 读多写少，随机访问快。 |
| **`LinkedList`** | 双向链表 | 无需扩容 | ❌ 不安全 | 频繁头尾插入删除，可作 Deque 栈/队列。 |
| **`Vector`** | `Object[]` + `synchronized` | 默认10，扩容至 2 倍 (或自定义增量) | ✅ 安全 (方法级锁) | 已淘汰，用 `CopyOnWriteArrayList` 或 `Collections.synchronizedList` 代替。 |
| **`CopyOnWriteArrayList`** | 写时复制数组 | 每次写操作 `Arrays.copyOf` 创建新数组 | ✅ 安全 (读写分离) | 读极多、写极少且数组小 (如监听器列表)。 |

> **深挖**：`ArrayList` 的 `subList` 返回视图，修改原列表会导致 `ConcurrentModificationException`；`fail-fast` 与 `fail-safe` 迭代器的原理。

#### 2. Map 家族

| 实现类 (JDK8+) | 数据结构 | 哈希冲突解决 | 扩容机制 | 线程安全 | 特性 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`HashMap`** | 数组+链表+红黑树 | 链表转红黑树阈值8，树退链表阈值6 | 容量 `cap * loadFactor` (默认0.75)，每次扩容为2倍，rehash | ❌ | Key 和 Value 允许 Null |
| **`LinkedHashMap`** | `HashMap` + 双向链表 | 同 `HashMap` | 同 `HashMap` | ❌ | 保持插入/访问顺序 (LRU 缓存) |
| **`TreeMap`** | 红黑树 | 无冲突，直接比较 | – | ❌ | 按键自然排序或比较器排序 |
| **`ConcurrentHashMap`** | JDK7 分段锁，JDK8 数组+链表/树 + CAS+synchronized | 同 `HashMap`，但锁粒度细化至桶 | 支持多线程并发扩容 | ✅ 高并发 | 不允许 Null，并发安全且性能优秀 |
| **`Hashtable`** | 数组+链表 | 同早期 HashMap | 同 `HashMap` | ✅ 方法级锁 | 已淘汰 |

> **深挖**：  
> - `HashMap` 为何容量是 2 的幂？`(n-1) & hash` 即可高效取模。  
> - `HashMap` 的 `hash()` 扰动函数：高16位与低16位异或，减少冲突。  
> - `ConcurrentHashMap` 在 JDK8 中利用 `sizeCtl` 控制扩容状态，协助扩容机制。

#### 3. Set 家族

本质是对相应 `Map` 的包装 (只用 Key，Value 为 `PRESENT` 常量)：

| 实现类 | 底层依赖 |
| :--- | :--- |
| **`HashSet`** | `HashMap` |
| **`LinkedHashSet`** | `LinkedHashMap` |
| **`TreeSet`** | `TreeMap` |

---

### 三、Java 多线程并发 (结合你已学内容)

| 知识层面 | 核心知识点 | 深度解析 |
| :--- | :--- | :--- |
| **线程基础** | 6 种状态 (NEW → TERMINATED)、`Thread`/`Runnable`、`start()` vs `run()` | `BLOCKED` vs `WAITING` 底层依赖操作系统的差异；`RUNNABLE` 包含 OS 的 Ready/Running。 |
| **锁机制** | `synchronized` (对象头 Mark Word、锁升级)、`ReentrantLock` (AQS)、`volatile` (可见性+有序性) | 偏向锁撤销、轻量级锁自旋、重量级锁阻塞；AQS 的 CLH 队列、`state` 变量；`volatile` 内存屏障（StoreStore, StoreLoad, LoadLoad, LoadStore）的实际作用。 |
| **线程池** | 7 大参数、任务执行流程 (核心→队列→最大→拒绝)、拒绝策略 | 阻塞队列的选择 (`ArrayBlockingQueue` 有界必用) 决定了资源控制；`CallerRunsPolicy` 反压原理；保证线程安全编号命名 (`ThreadFactory`)。 |
| **并发工具** | `CountDownLatch`、`CyclicBarrier`、`Semaphore`、`ThreadLocal` | `ThreadLocal` 内部 `ThreadLocalMap`、Key 弱引用导致的内存泄漏，必须 `remove()`。 |
| **JMM (内存模型)** | 主内存、工作内存、happens-before 规则 | `volatile` 写 happens-before 后续读；锁释放 happens-before 锁获取；`ConcurrentHashMap` 的 `put` 如何通过 `volatile` + CAS 保证可见性与原子性。 |

---

### 四、JVM 核心原理

| 模块 | 核心知识点 | 深度要求 |
| :--- | :--- | :--- |
| **内存区域** | 堆、元空间 (方法区)、栈、程序计数器、本地方法栈 | 1.8 后方法区移至元空间 (本地内存)，字符串常量池移至堆；栈帧结构 (`局部变量表`、`操作数栈`、`动态链接`)。 |
| **对象创建** | 类加载→分配内存 (`TLAB` 优化)→初始化零值→对象头 (`Mark Word`、`Klass Pointer`)。 | 对象头在锁升级中的变化；`new` 指令与 `invokespecial` 的配合。 |
| **GC 机制** | 可达性分析、标记-清除/复制/整理、分代回收 | 1. **CMS** (老年代并发清除，浮动垃圾、提前晋升等碎片化问题)。<br>2. **G1** (Region 化、SATB、Mixed GC、Remembered Set)。<br>3. **ZGC** (染色指针，亚毫秒暂停)。 |
| **类加载** | 双亲委派模型 (`Bootstrap`/`Extension`/`Application`)、破坏与救济 (SPI `ThreadContextClassLoader`) | `Class.forName()` 与 `ClassLoader.loadClass()` 区别；Tomcat `WebappClassLoader` 独立隔离原理。 |
| **调优排查** | `jps`、`jstack`、`jmap`、`jstat`、`MAT` 分析堆 Dump | CPU 飙高排查 (`top -H -p` + 转十六进制查线程栈)；死锁检测 (`jstack -l` 查看 `Found 1 deadlock`)。 |

---

### 五、综合串联与应用

- **结合项目**：你的智能体平台中，`ConcurrentHashMap` 可用于缓存会话上下文；`CopyOnWriteArrayList` 管理实时监听器；`ThreadPoolExecutor` 自定义拒绝策略处理高并发问答。
- **设计模式**：单例用枚举避免序列化漏洞；工厂方法隔离数据源创建；策略模式消除 `if-else` (尤其支付/调度)；观察者模式用 `Guava EventBus` 解耦事件。
- **数据库**：MVCC 在并发控制中的体现；间隙锁死锁案例的排查；`EXPLAIN` 的 `type` 列主导优化方向。
