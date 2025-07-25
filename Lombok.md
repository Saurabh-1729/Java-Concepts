Certainly! **Lombok** is a Java library that reduces boilerplate code by generating code at compile time via annotations. Here are the **most important Lombok annotations** and what they do, with examples:

---

## 1. `@Getter` and `@Setter`

**Generates getter and setter methods for fields.**

```java
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class User {
    private String name;
    private int age;
}

// Equivalent to manually writing getName(), setName(), getAge(), setAge()
```

---

## 2. `@ToString`

**Generates a `toString()` method.**

```java
import lombok.ToString;

@ToString
public class User {
    private String name;
    private int age;
}

// Equivalent to: public String toString() { return "User(name=..., age=...)"; }
```

---

## 3. `@EqualsAndHashCode`

**Generates `equals()` and `hashCode()` methods.**

```java
import lombok.EqualsAndHashCode;

@EqualsAndHashCode
public class User {
    private String name;
    private int age;
}
```

---

## 4. `@NoArgsConstructor`, `@AllArgsConstructor`, `@RequiredArgsConstructor`

**Generates constructors:**

- `@NoArgsConstructor` – no-argument constructor.
- `@AllArgsConstructor` – constructor with all fields.
- `@RequiredArgsConstructor` – constructor with final fields and fields marked `@NonNull`.

```java
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import lombok.RequiredArgsConstructor;
import lombok.NonNull;

@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor
public class User {
    private String name;
    @NonNull private String email;
}
```

---

## 5. `@Data`

**Shortcut for `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode`, and `@RequiredArgsConstructor`.**

```java
import lombok.Data;

@Data
public class User {
    private String name;
    private int age;
}
```
*Generates getters, setters, toString, equals, hashCode, and a required-args constructor.*

---

## 6. `@Builder`

**Implements the builder pattern for object creation.**

```java
import lombok.Builder;

@Builder
public class User {
    private String name;
    private int age;
}

// Usage:
User user = User.builder().name("Alice").age(30).build();
```

---

## 7. `@Value`

**Immutable variant of `@Data`: all fields are `private final`, no setters, only getters, constructor, equals, hashCode, toString.**

```java
import lombok.Value;

@Value
public class User {
    String name;
    int age;
}
```

---

## 8. `@Slf4j`

**Adds a `private static final org.slf4j.Logger log` field for logging.**

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class MyService {
    public void doWork() {
        log.info("Work started");
    }
}
```

---

## 9. `@SneakyThrows`

**Allows you to throw checked exceptions without declaring them.**

```java
import lombok.SneakyThrows;

@SneakyThrows
public void run() {
    throw new Exception("No need to declare or catch!");
}
```

---

## 10. `@Cleanup`

**Automatically calls `close()` on resources.**

```java
import lombok.Cleanup;

public void readFile() throws IOException {
    @Cleanup InputStream in = new FileInputStream("file.txt");
    // in.close() will be called automatically
}
```

---

# **Summary Table**

| Annotation            | What it does                                    |
|-----------------------|-------------------------------------------------|
| `@Getter/@Setter`     | Generates getters and setters                   |
| `@ToString`           | Generates `toString()`                          |
| `@EqualsAndHashCode`  | Generates `equals()` and `hashCode()`           |
| `@NoArgsConstructor`  | No-args constructor                             |
| `@AllArgsConstructor` | All-args constructor                            |
| `@RequiredArgsConstructor` | Required-args constructor                  |
| `@Data`               | Combines several common annotations             |
| `@Value`              | Immutable variant of `@Data`                    |
| `@Builder`            | Implements the builder pattern                  |
| `@Slf4j`              | Adds a logger                                   |
| `@SneakyThrows`       | Throws checked exceptions without declaring     |
| `@Cleanup`            | Auto-closes resources                           |

---

**Let me know if you want deeper examples of any annotation!**
