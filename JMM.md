The **Java Memory Model (JMM)** defines **how threads interact through memory**. It specifies:

* **How variables are stored in memory**
* **How threads read/write those variables**
* **What guarantees exist for visibility, ordering, and atomicity**

It is the **foundation of Java concurrency** and explains why constructs like `synchronized`, `volatile`, and `Atomic` classes exist.

---

# 1. Why Java Memory Model Exists

In a **multithreaded program**, multiple threads run on different CPU cores. Each CPU core may have its own **cache**.

Problem:
If one thread updates a variable, **another thread might not see the updated value immediately**.

Example:

```java
class Example {
    static boolean flag = false;

    public static void main(String[] args) {

        Thread t1 = new Thread(() -> {
            while(!flag) { }
            System.out.println("Flag changed!");
        });

        Thread t2 = new Thread(() -> {
            flag = true;
        });

        t1.start();
        t2.start();
    }
}
```

Possible issue:

* Thread 1 may **never see the updated value**
* Because it reads from **CPU cache instead of main memory**

The **Java Memory Model defines rules to avoid these issues.**

---

# 2. Java Memory Architecture

JMM divides memory into:

### 1. Main Memory

Shared memory accessible by **all threads**.

Stores:

* Heap objects
* Static variables
* Class metadata

### 2. Thread Working Memory

Each thread has its own **local working memory (cache)**.

It stores **copies of variables from main memory**.

---

### Memory Structure

```
              MAIN MEMORY
        -------------------------
        | shared variables      |
        | objects               |
        | static fields         |
        -------------------------
           ↑            ↑
           |            |
   Thread 1 Cache   Thread 2 Cache
   (Working Memory) (Working Memory)

```

Threads operate on **local copies**, not directly on main memory.

---

# 3. The Three Core Guarantees of JMM

JMM guarantees correctness using three properties:

1. **Visibility**
2. **Atomicity**
3. **Ordering**

---

# 4. Visibility

Visibility means:

> When one thread modifies a variable, other threads should see the updated value.

Without synchronization:

Thread A:

```
x = 10
```

Thread B may still see:

```
x = 0
```

Because B reads from its **cache**.

---

### How to Guarantee Visibility

Java provides:

* `volatile`
* `synchronized`
* `final`
* `Atomic classes`
* `Lock`

Example with `volatile`:

```java
volatile boolean flag = false;
```

Now when one thread writes:

```
flag = true
```

Other threads will **immediately see the change**.

---

# 5. Atomicity

Atomicity means:

> An operation happens completely or not at all.

Example:

```java
count++
```

Looks simple but actually:

```
1. read count
2. add 1
3. write back
```

In multithreading:

```
Thread1 read 5
Thread2 read 5
Thread1 write 6
Thread2 write 6
```

Expected: 7
Actual: 6

This is called a **race condition**.

---

### Solutions

Use:

```
synchronized
AtomicInteger
Lock
```

Example:

```java
AtomicInteger count = new AtomicInteger(0);

count.incrementAndGet();
```

Now the operation is **atomic**.

---

# 6. Ordering

Compilers and CPUs **reorder instructions** for performance.

Example:

```java
int a = 10;
int b = 20;
```

Compiler might execute:

```
b = 20
a = 10
```

This is fine **within a single thread**.

But in multithreading it may break logic.

Example:

```
Thread1:
x = 1
flag = true

Thread2:
if(flag)
   print(x)
```

Due to reordering:

```
flag = true
x = 1
```

Thread2 may print **0 instead of 1**.

---

### Preventing Reordering

Use:

* `volatile`
* `synchronized`
* memory barriers

---

# 7. Happens-Before Relationship

This is the **most important concept in JMM**.

**Definition:**

If operation **A happens-before B**, then:

* All effects of **A are visible to B**
* And A executes before B

---

### Happens-before Rules

#### 1. Program Order Rule

Within a thread:

```
a = 1
b = 2
```

`a` happens-before `b`.

---

#### 2. Monitor Lock Rule

Unlock happens-before next lock.

```
synchronized(lock) {
    x = 5;
}
```

Thread B entering the same lock **will see x = 5**.

---

#### 3. Volatile Rule

Write to volatile happens-before read.

```
volatile boolean ready;

Thread1:
ready = true

Thread2:
if(ready)
```

Thread2 will see the change.

---

#### 4. Thread Start Rule

```
Thread t = new Thread();
t.start();
```

Everything before `start()` is visible to the thread.

---

#### 5. Thread Join Rule

```
t.join();
```

All actions in thread **happen-before join returns**.

---

# 8. Volatile Deep Dive

`volatile` ensures:

1. **Visibility**
2. **Ordering**

But **NOT atomicity**.

Example:

```java
volatile int count;

count++;
```

Still not safe.

Because increment is **3 operations**.

---

### What volatile actually does

When writing volatile:

```
write to main memory
invalidate caches
```

When reading volatile:

```
read directly from main memory
```

---

# 9. Memory Barriers

The JVM inserts **memory barriers** to enforce JMM rules.

Types:

| Barrier    | Purpose                   |
| ---------- | ------------------------- |
| LoadLoad   | Prevent load reordering   |
| StoreStore | Prevent store reordering  |
| LoadStore  | Prevent load before store |
| StoreLoad  | Strongest barrier         |

Example:

```
volatile write -> StoreStore barrier
volatile read -> LoadLoad barrier
```

---

# 10. Out-of-Thin-Air Problem

JMM prevents **impossible values appearing due to reordering**.

Example impossible scenario:

```
Thread1:
x = y

Thread2:
y = x
```

Both becoming **42 magically** is forbidden.

---

# 11. Double Checked Locking Problem

Classic JMM problem.

Broken version:

```java
if(instance == null) {
    synchronized(this) {
        if(instance == null)
            instance = new Singleton();
    }
}
```

Due to instruction reordering:

```
1 allocate memory
2 assign reference
3 initialize object
```

Another thread may see **half-initialized object**.

---

### Correct Version

```java
private static volatile Singleton instance;
```

---

# 12. Final Fields Guarantee

Fields marked `final` have **special visibility guarantees**.

Example:

```java
class Person {
   final int age;

   Person(int age){
      this.age = age;
   }
}
```

Once constructor finishes, **other threads will always see correct value**.

---

# 13. Summary

| Concept        | Meaning                            |
| -------------- | ---------------------------------- |
| Main Memory    | Shared memory for all threads      |
| Working Memory | Per-thread cache                   |
| Visibility     | Updates visible to other threads   |
| Atomicity      | Operation happens completely       |
| Ordering       | Instruction execution order        |
| Happens-before | Guarantee of visibility + ordering |
| volatile       | Visibility + ordering              |
| synchronized   | Visibility + atomicity + ordering  |

---

# 14. Simple Mental Model

Think of JMM like **Google Docs editing**:

```
Main Memory → Google Docs server
Thread cache → Local copy on laptop
```

Without sync:

```
Person A edits locally
Person B still sees old version
```

With sync:

```
Changes are pushed to server immediately
Everyone sees latest version
```

---
