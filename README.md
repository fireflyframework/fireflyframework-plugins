# Firefly Common Plugins

A simplified, well-designed plugin system for the Firefly Platform. This library provides a clean and maintainable alternative to the deprecated `lib-plugin-manager`.

## Overview

The Firefly Common Plugins library enables modular extension of the Firefly Platform through a flexible plugin architecture. It allows you to:

- **Load plugins dynamically** at runtime
- **Define extension points** that plugins can implement
- **Manage plugin lifecycles** (load, initialize, start, stop, destroy)
- **Handle dependencies** between plugins automatically
- **Register and discover extensions** through a simple registry

## Why This Design?

### Problems with the Deprecated Version

The deprecated `lib-plugin-manager` suffered from several design issues:

1. **Over-engineering**: Reactive programming (Reactor) was used everywhere, adding unnecessary complexity for a plugin system
2. **Too many features**: Git loading, hot deployment, debugging, health monitoring - features that added complexity without clear value
3. **Tight coupling**: Heavy dependence on Spring WebFlux and specific Spring patterns
4. **Complex abstractions**: Multiple loaders, event buses, security managers made the code hard to understand and maintain
5. **Unclear separation**: Mixed concerns between plugin management and runtime features

### Our Simplified Approach

This new implementation focuses on:

1. **Simplicity**: Plain Java interfaces, no reactive patterns unless actually needed
2. **Core functionality only**: Plugin lifecycle, extension points, dependency resolution
3. **Framework independence**: No tight coupling to Spring or any other framework
4. **Clean separation of concerns**: Clear boundaries between API, implementation, and client code
5. **Easy to understand**: Straightforward code that developers can read and modify

## Architecture

```
fireflyframework-plugins/
├── plugin-api/           # Core interfaces, annotations, and models
│   ├── api/             # Plugin, PluginManager, ExtensionRegistry
│   ├── annotation/      # @Plugin, @Extension, @ExtensionPoint
│   └── model/           # PluginMetadata, PluginDescriptor, PluginState
└── plugin-core/         # Default implementations
    └── core/            # DefaultPluginManager, DefaultExtensionRegistry, etc.
```

## Quick Start

### 1. Add Dependency

```xml
<dependency>
    <groupId>org.fireflyframework</groupId>
    <artifactId>plugin-core</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

### 2. Define an Extension Point

```java
@ExtensionPoint(
    id = "org.fireflyframework.banking.payment-processor",
    description = "Processes payments for financial transactions"
)
public interface PaymentProcessor {
    boolean supportsMethod(String paymentMethod);
    String processPayment(BigDecimal amount, String currency);
}
```

### 3. Create a Plugin

```java
@Plugin(
    id = "com.example.credit-card-plugin",
    name = "Credit Card Payment Plugin",
    version = "1.0.0",
    description = "Processes credit card payments",
    author = "Example Inc."
)
public class CreditCardPlugin implements Plugin {
    
    private final PluginMetadata metadata;
    
    public CreditCardPlugin() {
        this.metadata = PluginMetadata.builder()
            .id("com.example.credit-card-plugin")
            .name("Credit Card Payment Plugin")
            .version("1.0.0")
            .description("Processes credit card payments")
            .author("Example Inc.")
            .build();
    }
    
    @Override
    public PluginMetadata getMetadata() {
        return metadata;
    }
    
    @Override
    public void initialize() throws PluginException {
        // Initialize resources
    }
    
    @Override
    public void start() throws PluginException {
        // Start processing
    }
    
    @Override
    public void stop() throws PluginException {
        // Stop processing
    }
    
    @Override
    public void destroy() throws PluginException {
        // Clean up resources
    }
    
    // Extension implementation as inner class
    @Extension(
        extensionPointId = "org.fireflyframework.banking.payment-processor",
        priority = 100
    )
    public static class CreditCardProcessor implements PaymentProcessor {
        
        @Override
        public boolean supportsMethod(String paymentMethod) {
            return "CREDIT_CARD".equals(paymentMethod);
        }
        
        @Override
        public String processPayment(BigDecimal amount, String currency) {
            // Process payment
            return "TXN-" + UUID.randomUUID();
        }
    }
}
```

### 4. Use the Plugin Manager

```java
public class Application {
    public static void main(String[] args) throws PluginException {
        // Create and initialize plugin manager
        PluginManager pluginManager = new DefaultPluginManager();
        pluginManager.initialize();
        
        // Register extension point
        pluginManager.getExtensionRegistry()
            .registerExtensionPoint(
                "org.fireflyframework.banking.payment-processor", 
                PaymentProcessor.class
            );
        
        // Load and start plugin
        pluginManager.loadPlugin(CreditCardPlugin.class);
        pluginManager.startPlugin("com.example.credit-card-plugin");
        
        // Use extensions
        List<PaymentProcessor> processors = pluginManager
            .getExtensionRegistry()
            .getExtensions("org.fireflyframework.banking.payment-processor");
        
        for (PaymentProcessor processor : processors) {
            if (processor.supportsMethod("CREDIT_CARD")) {
                String txnId = processor.processPayment(
                    new BigDecimal("100.00"), "USD"
                );
                System.out.println("Transaction ID: " + txnId);
            }
        }
        
        // Shutdown
        pluginManager.shutdown();
    }
}
```

## Plugin Dependencies

Plugins can depend on other plugins:

```java
@Plugin(
    id = "com.example.advanced-plugin",
    name = "Advanced Plugin",
    version = "1.0.0",
    dependencies = {"com.example.base-plugin"} // This plugin must load first
)
public class AdvancedPlugin implements Plugin {
    // ...
}
```

The plugin manager automatically:
- Validates all dependencies exist
- Detects circular dependencies
- Starts plugins in the correct order (dependencies first)

## API Reference

### Core Interfaces

#### Plugin
The main interface all plugins must implement.
- `getMetadata()` - Returns plugin metadata
- `initialize()` - Called once when plugin is loaded
- `start()` - Called when plugin should begin work
- `stop()` - Called when plugin should cease work
- `destroy()` - Called when plugin is unloaded

#### PluginManager
Manages plugin lifecycle.
- `loadPlugin(Class)` - Loads a plugin class
- `startPlugin(String)` - Starts a plugin by ID
- `stopPlugin(String)` - Stops a plugin by ID
- `unloadPlugin(String)` - Unloads and destroys a plugin
- `getExtensionRegistry()` - Access to extension registry

#### ExtensionRegistry
Manages extension points and extensions.
- `registerExtensionPoint(String, Class)` - Register an extension point
- `registerExtension(String, Object, int)` - Register an extension with priority
- `getExtensions(String)` - Get all extensions for a point (ordered by priority)
- `getExtension(String)` - Get highest priority extension

### Annotations

#### @Plugin
Marks a class as a plugin.
- `id` - Unique plugin identifier (required)
- `name` - Human-readable name (required)
- `version` - Version string (required)
- `description` - Plugin description
- `author` - Plugin author/organization
- `dependencies` - Array of plugin IDs this depends on

#### @ExtensionPoint
Marks an interface as an extension point.
- `id` - Unique extension point identifier (required)
- `description` - Extension point description

#### @Extension
Marks a class as implementing an extension point.
- `extensionPointId` - The extension point ID (required)
- `priority` - Priority (higher = higher priority, default 0)
- `description` - Extension description

## Migration from Deprecated Version

If you're migrating from `deprecated-lib-plugin-manager`:

### Key Changes

1. **No Reactive Types**: All methods return plain types (no `Mono<>` or `Flux<>`)
   ```java
   // Old:
   Mono<PluginDescriptor> descriptor = pluginManager.installPlugin(path);
   
   // New:
   PluginDescriptor descriptor = pluginManager.loadPlugin(MyPlugin.class);
   ```

2. **Simpler Lifecycle**: Just implement 4 methods instead of complex reactive chains
   ```java
   // Old:
   public Mono<Void> start() {
       return Mono.fromRunnable(() -> { /* ... */ });
   }
   
   // New:
   public void start() throws PluginException {
       // Just do the work
   }
   ```

3. **No Spring Dependency**: Remove Spring-specific code from plugins

4. **Extension Discovery**: Extensions are now inner classes with `@Extension` annotation

5. **No Complex Loaders**: Only classpath loading (JAR/Git loading removed as over-engineering)

### Migration Steps

1. Update dependencies in `pom.xml`
2. Replace reactive return types with plain types
3. Simplify plugin implementation (remove reactive code)
4. Move extensions to inner classes with `@Extension`
5. Update extension point registration
6. Test thoroughly

## Best Practices

1. **Keep plugins focused**: Each plugin should do one thing well
2. **Use semantic versioning**: Follow semver for plugin versions
3. **Document extension points**: Clear javadoc for all extension point interfaces
4. **Handle errors gracefully**: Always clean up resources in `destroy()`
5. **Test plugins independently**: Unit test plugins before integration
6. **Use meaningful IDs**: Use reverse domain notation (e.g., `com.company.feature`)
7. **Consider thread safety**: Plugins may be accessed concurrently

## License

Copyright 2024-2026 Firefly Software Solutions Inc  
Licensed under the Apache License, Version 2.0
