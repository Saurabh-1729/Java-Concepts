## What This Means

When you have a static method in a class, it belongs to the class itself, not to any specific instance of the class. Therefore, the proper way to call a static method is through the class name, not through an instance reference.

### Example of the Issue:

```java
// Assuming splitCommaSeperatedValues is a static method in Helper class
Helper helper = new Helper();  // Creating an instance
helper.splitCommaSeperatedValues("a,b,c");  // WRONG: Calling static method via instance
```

### Correct Way:

```java
// Correct way to call a static method
Helper.splitCommaSeperatedValues("a,b,c");  // RIGHT: Calling static method via class name
```

## Why This Matters

1. **Clarity**: Using the class name makes it clear that the method is static
2. **Maintainability**: If the method changes from static to instance or vice versa, code using it properly will need fewer changes
3. **Potential Confusion**: Calling static methods on instances can be misleading - it suggests the method might use instance state when it actually can't
