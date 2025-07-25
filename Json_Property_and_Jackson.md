---

# Understanding `@JsonProperty` with a Real-World Example

Think of `@JsonProperty` as a **translator between two languages** — **JSON** and **Java**.

---

## 📨 JSON Received from an API

```json
{
  "user_first_name": "John",
  "user_age": 25,
  "is_active": true
}
```

---

## 💻 Java Class Using `@JsonProperty`

```java
import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.Data;

@Data
public class User {

    @JsonProperty("user_first_name")
    private String firstName;

    @JsonProperty("user_age")
    private int age;

    @JsonProperty("is_active")
    private boolean active;
}
```

---

## 🛫 Real-World Analogy

Imagine you're at an **international airport**, and you see a sign that says:

* `"Baggage Claim"` in **English**
* `"Réclamation des bagages"` in **French**

Both signs point to the **same place** — they're just written in **different languages**.

Similarly:

* JSON says `"user_first_name"`
* Java says `firstName`
* `@JsonProperty` tells Jackson: **"These mean the same thing"**

---

## 🔁 How Jackson Maps It

```java
ObjectMapper mapper = new ObjectMapper();

String json = "{ \"user_first_name\": \"John\" }";

User user = mapper.readValue(json, User.class);

System.out.println(user.getFirstName()); // Prints: John
```

### ✅ Jackson internally understands:

* `"user_first_name"` in JSON → `firstName` in Java
* `"user_age"` in JSON → `age` in Java
* `"is_active"` in JSON → `active` in Java

---

## 🧩 Why `@JsonProperty` Is Useful

* Smooth handling of **different naming conventions**
* Seamless integration with **external APIs**
* Keeps your Java code clean and idiomatic, even when JSON fields follow other styles (e.g., snake\_case)

---

> `@JsonProperty` makes your code more flexible, readable, and API-friendly!

---

Let me know if you'd like to include the reverse (Java → JSON) example too!
