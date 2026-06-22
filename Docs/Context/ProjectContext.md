# ESP32_Foundation Project Context

## Project Name

ESP32_Foundation

---

## Project Purpose

ESP32_Foundation is a reusable firmware framework intended to provide a common starting point for ESP32-based projects.

The goal of the project is to avoid repeatedly rewriting common infrastructure code and to establish a consistent architecture for future embedded applications.

ESP32_Foundation provides generic services such as:

* WiFi connectivity
* Secure WiFi / TLS support
* MQTT communication
* Time synchronization
* Device configuration
* Runtime parameter management
* Logging
* Diagnostics
* Firmware maintenance
* Module lifecycle management

The framework is intended to become the base layer upon which multiple independent ESP32 projects can be built.

---

# Primary Architectural Principle

## Foundation Provides Capability. Applications Provide Meaning.

ESP32_Foundation must remain completely application-agnostic.

It must not contain knowledge of:

* sensors
* actuators
* machines
* processes
* measurements
* control algorithms
* safety logic
* application-specific faults

Those responsibilities belong exclusively to the consuming application.

The relationship shall always be:

```text
Application Firmware
        |
        v
ESP32_Foundation
        |
        v
ESP32 Platform Libraries
```

The foundation never depends on the application.

---

# Intended Users of the Framework

Examples of projects that may consume ESP32_Foundation include:

* monitoring devices
* automation controllers
* data acquisition devices
* communication gateways
* IoT nodes
* custom embedded controllers

These projects may define their own hardware, sensors, calculations, and control behavior.

ESP32_Foundation provides only the reusable infrastructure.

---

# Core Framework Components

## ModuleManager

Responsible for:

* registering modules
* managing initialization order
* managing module lifecycle
* monitoring module health
* providing consistent startup behavior

Modules should expose a common interface.

Conceptual example:

```cpp
class Module
{
public:
    begin();
    update();
    status();
};
```

---

## Configuration Manager

Responsible for:

* storing configuration data
* retrieving parameters
* managing persistent settings
* providing configuration services to modules

Configuration shall be separated into:

1. Compile-time configuration

Determines which modules are included.

Example:

```cpp
ENABLE_WIFI
ENABLE_MQTT
ENABLE_TIME_SERVICE
```

2. Runtime configuration

User-editable parameters stored in persistent memory.

Examples:

```text
WiFi SSID

MQTT server

Device name

Runtime options
```

---

# Setup Portal

ESP32_Foundation shall include a SetupPortal module.

The SetupPortal provides a user-friendly method for provisioning devices.

Responsibilities:

* create temporary WiFi access point
* host configuration interface
* allow entry of required parameters
* validate foundation settings
* save configuration
* restart services if required

Typical first boot process:

```text
Device Starts

      |
      v

Config Exists?

      |
      +---- Yes ---> Normal Startup

      |
      No

      |
      v

Setup Portal Mode

      |
      v

User Configures Device

      |
      v

Save Configuration

      |
      v

Restart Normally
```

---

# Planned Foundation Modules

Initial candidate modules:

## Connectivity

* WiFi Module
* Secure WiFi Module
* MQTT Module

## Configuration

* Configuration Manager
* Setup Portal
* Persistent Storage Handler

## System Services

* Time Service / NTP
* Logger
* Diagnostics
* Watchdog
* Health Monitor
* OTA Update Support

---

# Event Bus Architecture

ESP32_Foundation shall provide a generic EventBus.

Purpose:

* reduce coupling between modules
* allow publish/subscribe communication
* avoid direct dependencies

Foundation events may include:

```text
WIFI_CONNECTED

WIFI_DISCONNECTED

MQTT_CONNECTED

MQTT_DISCONNECTED

TIME_SYNCHRONIZED

CONFIG_CHANGED

SYSTEM_WARNING
```

Application-specific events are not defined by the foundation.

Applications may create their own event namespaces.

---

# MQTT Philosophy

MQTT support is provided only as a transport service.

The MQTT module provides:

```cpp
publish(topic, payload);

subscribe(topic, callback);
```

The MQTT module does not define:

* topic meanings
* payload meanings
* telemetry structures
* alarms
* device behavior

Those belong to applications.

---

# Repository Structure

Proposed structure:

```text
ESP32_Foundation

├── src
│
│   ├── core
│   │
│   │   ├── ModuleManager
│   │   ├── EventBus
│   │   ├── ConfigManager
│   │   └── Logger
│   │
│   └── modules
│
│       ├── wifi
│       ├── secureWifi
│       ├── mqtt
│       ├── setupPortal
│       ├── time
│       └── ota
│

├── examples
│
│   └── basic_node

└── docs

    ├── ADR
    ├── DesignJournal.md
    ├── ChangeLog.md
    └── ProjectContext.md
```

---

# Architectural Decision Records

## Existing

### ADR-001: Foundation/Application Boundary

Decision:

ESP32_Foundation shall remain strictly application independent.

Foundation provides reusable services.

Applications provide project-specific behavior.

---

## Planned ADR Topics

### ADR-002: Modular Firmware Architecture and Lifecycle

Define:

* module interface
* startup sequence
* initialization dependencies
* fault handling
* health reporting

---

### ADR-003: Configuration System and Setup Portal

Define:

* persistent storage approach
* configuration schema
* provisioning workflow
* runtime updates

---

### ADR-004: Event Bus Architecture

Define:

* event ownership
* message structure
* publish/subscribe rules

---

### ADR-005: Connectivity Services

Define:

* WiFi behavior
* secure connections
* MQTT ownership
* reconnection handling

---

# Current Project State

The project is in architectural definition phase.

No implementation decisions have yet been finalized regarding:

* ESP-IDF vs Arduino framework
* filesystem selection
* configuration storage format
* MQTT library selection
* OTA mechanism
* event implementation

Current priority is defining architectural boundaries before writing code.

---

# Design Philosophy

Prefer:

* clear ownership
* reusable components
* loose coupling
* explicit interfaces
* predictable startup behavior
* maintainable code

Avoid:

* hidden dependencies
* application logic inside foundation modules
* tightly coupled services
* duplicated infrastructure code

The long-term goal is a reliable ESP32 firmware foundation that can support many projects without modification.
