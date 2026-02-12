# Firefly Framework - Plugins

[![CI](https://github.com/fireflyframework/fireflyframework-plugins/actions/workflows/ci.yml/badge.svg)](https://github.com/fireflyframework/fireflyframework-plugins/actions/workflows/ci.yml)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Java](https://img.shields.io/badge/Java-21%2B-orange.svg)](https://openjdk.org)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-green.svg)](https://spring.io/projects/spring-boot)

> Plugin system providing extension point discovery, lifecycle management, and dependency resolution for Firefly Platform modules.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Documentation](#documentation)
- [Contributing](#contributing)
- [License](#license)

## Overview

Firefly Framework Plugins provides a simplified plugin architecture for extending the Firefly Platform. It defines a plugin API with annotations for declaring plugins and extension points, along with a core implementation for plugin lifecycle management, extension registry, and dependency resolution.

The project is structured as a multi-module build with two sub-modules: plugin-api (annotations, interfaces, and models) and plugin-core (default implementations). The `@Plugin` and `@Extension` annotations enable declarative plugin definitions, while `@ExtensionPoint` marks interfaces that plugins can implement.

The `DefaultPluginManager` handles plugin discovery, initialization, and shutdown, while `DefaultExtensionRegistry` manages extension point bindings. The `PluginDependencyResolver` ensures plugins are loaded in the correct order based on their declared dependencies.

## Features

- `@Plugin` annotation for declarative plugin definition
- `@Extension` annotation for marking extension implementations
- `@ExtensionPoint` annotation for declaring extensible interfaces
- `PluginManager` interface with default implementation
- `ExtensionRegistry` for managing extension point bindings
- `PluginDependencyResolver` for dependency-ordered plugin loading
- `PluginDescriptor` and `PluginMetadata` for plugin information
- Plugin state lifecycle management (created, initialized, started, stopped)
- Multi-module architecture: plugin-api, plugin-core

## Requirements

- Java 21+
- Spring Boot 3.x
- Maven 3.9+

## Installation

```xml
<!-- Plugin API (for plugin developers) -->
<dependency>
    <groupId>org.fireflyframework</groupId>
    <artifactId>plugin-api</artifactId>
    <version>26.02.04</version>
</dependency>

<!-- Plugin Core (for host applications) -->
<dependency>
    <groupId>org.fireflyframework</groupId>
    <artifactId>plugin-core</artifactId>
    <version>26.02.04</version>
</dependency>
```

## Quick Start

```java
import org.fireflyframework.plugin.annotation.Plugin;
import org.fireflyframework.plugin.annotation.Extension;
import org.fireflyframework.plugin.annotation.ExtensionPoint;

// Define an extension point
@ExtensionPoint
public interface PaymentProcessor {
    Mono<PaymentResult> process(PaymentRequest request);
}

// Implement a plugin
@Plugin(id = "stripe-payments", version = "1.0.0")
public class StripePlugin implements org.fireflyframework.plugin.api.Plugin {

    @Extension
    public class StripePaymentProcessor implements PaymentProcessor {
        @Override
        public Mono<PaymentResult> process(PaymentRequest request) {
            return stripeApi.charge(request);
        }
    }
}
```

## Configuration

No configuration is required. Plugins are discovered and loaded automatically by the `PluginManager`.

## Documentation

No additional documentation available for this project.

## Contributing

Contributions are welcome. Please read the [CONTRIBUTING.md](CONTRIBUTING.md) guide for details on our code of conduct, development process, and how to submit pull requests.

## License

Copyright 2024-2026 Firefly Software Solutions Inc.

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for details.
