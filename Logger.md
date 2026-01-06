Let me explain how the **Log4j2 configuration** works in your application, breaking down each part with examples:

## Overview
Log4j2 is a logging framework that captures application events and outputs them to various destinations (console, files, etc.) with different formatting.

## Configuration Breakdown:

### 1. **Global Configuration**
```properties:src/main/resources/log4j2.properties
status = warn
name = PropertiesConfig
```
- **`status = warn`**: Log4j2's internal logging level (only shows warnings/errors about Log4j2 itself)
- **`name`**: Just a label for this configuration

### 2. **Console Appender**
```properties:src/main/resources/log4j2.properties
appender.console.type = Console
appender.console.name = ConsoleAppender
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = %d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
```

**What it does**: Prints logs to the terminal/console

**Pattern breakdown**:
- `%d{yyyy-MM-dd HH:mm:ss}` - Date/time: `2024-01-05 14:30:45`
- `%-5p` - Log level (left-aligned, 5 chars): `INFO `, `ERROR`, `DEBUG`
- `%c{1}` - Logger name (last part only): `VaultBO` instead of `com.mathworks.trweb.config.Vault.bo.VaultBO`
- `%L` - Line number: `42`
- `%m` - The actual log message
- `%n` - New line

**Example output**:
```
2024-01-05 14:30:45 INFO  DataSourceConfig:38 - Creating webquery datasource
2024-01-05 14:30:45 ERROR VaultBO:67 - Failed to read vault secret
```

### 3. **File Appender (Application Logs)**
```properties:src/main/resources/log4j2.properties
appender.file.type = File
appender.file.name = FileAppender
appender.file.fileName = /opt/logs/trweb_api/application/application.log
appender.file.layout.type = PatternLayout
appender.file.layout.pattern = %d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
```

**What it does**: Writes logs to `/opt/logs/trweb_api/application/application.log`

**In your code**:
```java:src/main/java/com/mathworks/trweb/config/database/DataSourceConfig.java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class DataSourceConfig {
    private static final Logger logger = LoggerFactory.getLogger(DataSourceConfig.class);
    
    public DataSource webqueryDataSource() {
        logger.info("Fetching database credentials from Vault");
        try {
            String url = vaultReader.getSecretByKey(...);
            logger.debug("Successfully retrieved DB URL: {}", url);
        } catch (Exception e) {
            logger.error("Failed to fetch credentials: {}", e.getMessage());
        }
    }
}
```

**File output** (`application.log`):
```
2024-01-05 14:30:45 INFO  DataSourceConfig:25 - Fetching database credentials from Vault
2024-01-05 14:30:45 ERROR DataSourceConfig:30 - Failed to fetch credentials: Status 403 Forbidden
```

### 4. **Access Log Appender**
```properties:src/main/resources/log4j2.properties
appender.access.type = File
appender.access.name = AccessAppender
appender.access.fileName = /opt/logs/trweb_api/access/access.log
appender.access.layout.type = PatternLayout
appender.access.layout.pattern = %d{yyyy-MM-dd HH:mm:ss} %msg%n
```

**What it does**: Logs HTTP requests/responses to a separate file

**Example access.log**:
```
2024-01-05 14:30:45 GET /api/v1/sessions 200 125ms
2024-01-05 14:30:46 POST /api/v1/results 201 340ms
2024-01-05 14:30:47 GET /api/v1/health 200 5ms
```

### 5. **Root Logger**
```properties:src/main/resources/log4j2.properties
rootLogger.level = info
rootLogger.appenderRefs = console
rootLogger.appenderRef.console.ref = ConsoleAppender
```

**What it does**: Default configuration for ALL loggers not explicitly configured
- Only logs `INFO` level and above (INFO, WARN, ERROR, FATAL)
- Outputs only to console

**Hierarchy example**:
```java
// This uses root logger (not under com.mathworks)
Logger logger = LoggerFactory.getLogger("org.springframework.boot");
logger.debug("This won't show"); // Below INFO level
logger.info("This will show in console only");
```

### 6. **Application Logger**
```properties:src/main/resources/log4j2.properties
logger.application.name = com.mathworks
logger.application.level = info
logger.application.additivity = false
logger.application.appenderRefs = file, console
logger.application.appenderRef.file.ref = FileAppender
logger.application.appenderRef.console.ref = ConsoleAppender
```

**What it does**: Specific configuration for your application code
- Applies to ALL classes under `com.mathworks` package
- Logs to BOTH file and console
- `additivity = false`: Doesn't inherit from root logger (prevents duplicate logs)

**Example**:
```java:src/main/java/com/mathworks/trweb/config/Vault/bo/VaultBO.java
package com.mathworks.trweb.config.Vault.bo;

public class VaultBO {
    private static final Logger logger = LoggerFactory.getLogger(VaultBO.class);
    
    public Object getSecretByKey(String path, String key, VaultPath secretPath) {
        logger.info("Reading secret from path: {}", path); // Goes to console AND file
        Map<String, Object> secrets = getAllSecretsByPath(path, secretPath);
        if (secrets == null) {
            logger.error("No secrets found at path: {}", path); // Goes to console AND file
        }
        return secrets.get(key);
    }
}
```

### 7. **Access Logger (Spring Web)**
```properties:src/main/resources/log4j2.properties
logger.access.name = org.springframework.web
logger.access.level = info
logger.access.additivity = false
logger.access.appenderRefs = access
logger.access.appenderRef.access.ref = AccessAppender
```

**What it does**: Captures Spring Web/REST endpoint access
- Only goes to `access.log` file
- Typically logs HTTP requests

## Log Levels (in order):
1. **TRACE** - Most detailed (method entry/exit)
2. **DEBUG** - Detailed info for debugging
3. **INFO** - General information
4. **WARN** - Warning messages
5. **ERROR** - Error messages
6. **FATAL** - Critical errors

**Example with different levels**:
```java
logger.trace("Entering method with params: {}", params); // Won't show (below INFO)
logger.debug("SQL Query: {}", sqlQuery);                 // Won't show (below INFO)
logger.info("Processing request for user: {}", userId);  // Will show
logger.warn("Vault response slow: {}ms", responseTime);  // Will show
logger.error("Database connection failed", exception);   // Will show
```

## How it works in your Docker setup:

Looking at your `docker-compose.dev.yaml`:
```yaml:docker-compose.dev.yaml
volumes:
  - /opt/logs/trweb_api:/opt/logs/trweb_api
```

This mounts the log directory so:
- Logs persist even if container restarts
- You can access logs from the host machine
- Multiple containers can write to same location

**To view logs**:
```bash
# Application logs
tail -f /opt/logs/trweb_api/application/application.log

# Access logs
tail -f /opt/logs/trweb_api/access/access.log

# Spring Boot console logs (from your entrypoint)
tail -f /opt/logs/trweb_api/spring-boot.log
```

This configuration ensures you have:
- **Console output** for development/debugging
- **Persistent file logs** for production monitoring
- **Separated access logs** for request tracking
- **Proper log levels** to control verbosity

## How It decides where to append the logs?

Excellent question! Log4j2 decides which file to write to based on the **package/class hierarchy** of where the logger is created. Let me explain the routing logic:

## How Log4j2 Routes Your Logs

### 1. **Logger Name Matching**

When you create a logger:
```java
private static final Logger logger = LoggerFactory.getLogger(DataSourceConfig.class);
```

The logger gets a name based on the **fully qualified class name**:
- `DataSourceConfig.class` → `com.mathworks.trweb.config.database.DataSourceConfig`

### 2. **Configuration Matching Process**

Log4j2 looks at your configuration and matches loggers by name:

```properties:src/main/resources/log4j2.properties
# This catches ALL classes under com.mathworks package
logger.application.name = com.mathworks
logger.application.appenderRefs = file, console

# This catches Spring Web classes
logger.access.name = org.springframework.web
logger.access.appenderRefs = access

# This is the fallback for everything else
rootLogger.appenderRefs = console
```

## Matching Examples:

### Example 1: Your Application Code
```java:src/main/java/com/mathworks/trweb/config/database/DataSourceConfig.java
package com.mathworks.trweb.config.database;

public class DataSourceConfig {
    private static final Logger logger = LoggerFactory.getLogger(DataSourceConfig.class);
    
    public void someMethod() {
        logger.info("This message goes where?");
    }
}
```

**Routing Decision**:
1. Logger name: `com.mathworks.trweb.config.database.DataSourceConfig`
2. Matches: `com.mathworks` (from `logger.application.name`)
3. **Output**: Goes to BOTH `console` AND `application.log` file

### Example 2: Spring Framework Code
```java
// Inside Spring's code
package org.springframework.web.servlet;

public class DispatcherServlet {
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);
    
    public void doDispatch() {
        logger.info("Processing request");
    }
}
```

**Routing Decision**:
1. Logger name: `org.springframework.web.servlet.DispatcherServlet`
2. Matches: `org.springframework.web` (from `logger.access.name`)
3. **Output**: Goes ONLY to `access.log` file

### Example 3: Third-party Library
```java
// Some external library
package com.zaxxer.hikari;

public class HikariDataSource {
    private static final Logger logger = LoggerFactory.getLogger(HikariDataSource.class);
    
    public void getConnection() {
        logger.info("Getting database connection");
    }
}
```

**Routing Decision**:
1. Logger name: `com.zaxxer.hikari.HikariDataSource`
2. Doesn't match `com.mathworks` or `org.springframework.web`
3. Falls back to `rootLogger`
4. **Output**: Goes ONLY to `console`

## The Hierarchy Matching Rules:

Log4j2 uses **longest prefix matching**:

```properties
# More specific wins over less specific
logger.specific.name = com.mathworks.trweb.config.database
logger.general.name = com.mathworks
```

If you log from `DataSourceConfig`:
- Matches both, but `com.mathworks.trweb.config.database` is more specific
- Uses the more specific configuration

## Visual Decision Tree:

```
Your logger.info("message") call
            ↓
    What's the logger name?
            ↓
┌───────────────────────────────────┐
│ com.mathworks.trweb.config...     │
└───────────────────────────────────┘
            ↓
    Check configurations:
            ↓
┌─────────────────────────────────────────┐
│ 1. Is there exact match?                │
│    com.mathworks.trweb.config.database  │ → No
└─────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────┐
│ 2. Is there parent match?               │
│    com.mathworks.trweb.config           │ → No
│    com.mathworks.trweb                  │ → No
│    com.mathworks ✓                      │ → YES!
└─────────────────────────────────────────┘
            ↓
    Use com.mathworks configuration:
    - appenderRefs = file, console
            ↓
    Write to:
    - /opt/logs/trweb_api/application/application.log
    - Console (stdout)
```

## Additivity Property:

The `additivity` flag is important:

```properties
logger.application.additivity = false  # Doesn't inherit from parent
logger.access.additivity = false
```

If `additivity = true` (default):
```java
// If you had:
logger.child.name = com.mathworks.trweb
logger.child.level = debug
logger.child.additivity = true  # or omitted

logger.parent.name = com.mathworks
logger.parent.appenderRefs = file
```

Then logs from `com.mathworks.trweb` would go to:
1. Its own appenders (if any)
2. PLUS parent's appenders (file)

With `additivity = false`, it ONLY uses its own appenders.

## Practical Example - Adding Custom Logger:

Say you want database queries in a separate file:

```properties:src/main/resources/log4j2.properties
# Add new appender
appender.database.type = File
appender.database.name = DatabaseAppender
appender.database.fileName = /opt/logs/trweb_api/database/queries.log
appender.database.layout.type = PatternLayout
appender.database.layout.pattern = %d{ISO8601} [%thread] %msg%n

# Add specific logger
logger.database.name = com.mathworks.trweb.config.database
logger.database.level = debug
logger.database.additivity = false
logger.database.appenderRefs = database, console
logger.database.appenderRef.database.ref = DatabaseAppender
logger.database.appenderRef.console.ref = ConsoleAppender
```

Now:
```java:src/main/java/com/mathworks/trweb/config/database/DataSourceConfig.java
logger.info("This goes to queries.log AND console");
logger.debug("Debug info also goes to queries.log AND console");
```

But:
```java:src/main/java/com/mathworks/trweb/service/SomeService.java
logger.info("This still goes to application.log AND console");
// Because it matches com.mathworks, not com.mathworks.trweb.config.database
```

## Summary:

The file destination is determined by:
1. **Logger name** (usually the fully qualified class name)
2. **Configuration matching** (longest prefix match)
3. **Appender references** in the matched configuration
4. **Additivity** setting (whether to also use parent's appenders)
