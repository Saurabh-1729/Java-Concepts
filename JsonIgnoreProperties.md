Ah — you’re talking about the **reverse of `@JsonIgnore`**:

* **`@JsonIgnore`** → Ignore a field **when serializing Java → JSON**.
* Your case → Ignore unknown fields **when deserializing JSON → Java** (API sends extra fields not in your class).

---

## **How to Handle Extra JSON Fields When Deserializing**

If you’re using Jackson (Spring Boot’s default JSON mapper), you can use:

### **1. `@JsonIgnoreProperties(ignoreUnknown = true)`**

* Placed on the class.
* Tells Jackson to ignore any JSON fields not mapped to Java fields.

```java
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class User {
    private String id;
    private String name;
    // getters & setters
}
```

**Example**
Incoming JSON:

```json
{
  "id": "101",
  "name": "Alice",
  "extraField": "This will be ignored"
}
```

`extraField` won’t cause an error — Jackson just skips it.

---

### **2. Global Setting in `application.properties`**

If you want to apply this behavior for **all** objects without adding the annotation everywhere:

```properties
spring.jackson.deserialization.fail-on-unknown-properties=false
```

This sets Jackson to **never fail** when extra fields are present.

---

### **3. ObjectMapper Configuration (Manual)**

If you’re configuring Jackson yourself:

```java
ObjectMapper mapper = new ObjectMapper();
mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
```

---

## **Best Practice**

* Use **`@JsonIgnoreProperties(ignoreUnknown = true)`** for **specific DTOs** where you expect extra API fields.
* Use **global config** if your entire app frequently interacts with APIs that send unpredictable fields.

---

If you want, I can also show you **the reverse case** where you want to **capture all extra fields** into a `Map<String, Object>` instead of ignoring them — that’s useful when the extra fields might still be needed later. Would you like me to show that?
