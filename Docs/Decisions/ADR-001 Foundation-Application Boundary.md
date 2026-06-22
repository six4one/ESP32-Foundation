# ADR-001: Foundation/Application Boundary

## Status

Accepted

## Date

2026-06-22

---

## Context

ESP32_Foundation is intended to provide a reusable firmware framework for multiple ESP32-based projects.

Many ESP32 applications require the same foundational capabilities:

* WiFi connectivity
* Secure WiFi / TLS support
* MQTT communication
* Time synchronization
* Device provisioning
* Configuration storage
* Logging
* Diagnostics
* Firmware update support
* System health monitoring

Historically, these capabilities tend to become embedded directly into application firmware. This results in duplicated code, inconsistent implementations, and increasing maintenance effort across projects.

A reusable foundation framework requires a clear separation between:

* generic infrastructure services
* project-specific application logic

Without this separation, ESP32_Foundation risks gradually accumulating knowledge of the projects built on top of it, reducing its usefulness as a general-purpose platform.

ESP32_Foundation must therefore remain completely application-agnostic.

---

# Decision

ESP32_Foundation shall provide reusable firmware infrastructure only.

It shall provide:

* communication services
* configuration management
* module lifecycle management
* diagnostic services
* system utilities

It shall not provide:

* project-specific control logic
* sensor interpretation
* actuator behavior
* application calculations
* application fault handling
* project-specific state machines

The dependency relationship shall always be:

```text
Application Firmware
        |
        v
ESP32_Foundation
        |
        v
ESP32 Hardware / Platform Libraries
```

The reverse dependency is prohibited:

```text
ESP32_Foundation
        |
        v
Application Firmware
```

Foundation modules shall never require knowledge of the applications consuming them.

---

# Architecture Model

The framework shall be organized around generic reusable services.

```text
ESP32_Foundation

├── Core
│
│   ├── ModuleManager
│   ├── EventBus
│   ├── ConfigManager
│   ├── Logger
│   └── SystemStatus
│
├── Modules
│
│   ├── WiFi
│   ├── SecureWiFi
│   ├── MQTT
│   ├── TimeService
│   ├── SetupPortal
│   ├── OTA
│   ├── Watchdog
│   └── Diagnostics
│
└── Applications
        (external projects)
```

Applications are responsible for:

* sensors
* actuators
* measurements
* calculations
* decisions
* operating modes
* safety logic

The foundation provides capabilities.

Applications provide meaning.

---

# Event Bus Boundary

ESP32_Foundation may provide a generic publish/subscribe EventBus.

The EventBus is responsible only for:

* event transport
* decoupling modules
* message routing

Foundation events may include:

```text
WIFI_CONNECTED
WIFI_DISCONNECTED

MQTT_CONNECTED
MQTT_DISCONNECTED

TIME_SYNCHRONIZED

CONFIG_CHANGED

SYSTEM_WARNING
SYSTEM_ERROR
```

Foundation shall not define application events.

Examples of prohibited foundation events:

```text
SENSOR_TRIGGERED

TEMPERATURE_ALARM

MOTOR_FAULT

POSITION_CHANGED
```

Application projects shall define their own event namespaces.

---

# MQTT Boundary

The MQTT module shall provide messaging infrastructure only.

The MQTT module may provide:

```cpp
mqtt.publish(topic, payload);

mqtt.subscribe(topic, callback);
```

It shall not provide application-specific functions.

Invalid:

```cpp
mqtt.publishTemperature();

mqtt.publishAlarm();

mqtt.publishSensorData();
```

The MQTT module does not understand the meaning of messages.

Topic structure, payload format, and message semantics belong to the application.

---

# Configuration Boundary

ESP32_Foundation shall provide generic configuration services.

Foundation-owned configuration includes:

```text
device.*
wifi.*
mqtt.*
time.*
system.*
```

Example:

```json
{
    "device":
    {
        "name": "ESP32_Node"
    },

    "wifi":
    {
        "ssid": "",
        "password": ""
    },

    "mqtt":
    {
        "server": "",
        "port": 1883
    },

    "application":
    {
        "...": "application-owned"
    }
}
```

Application-specific configuration shall remain application-owned.

ESP32_Foundation may store application configuration data but shall not interpret its meaning.

---

# Setup Portal Boundary

A SetupPortal module shall be included as part of ESP32_Foundation.

The SetupPortal shall provide:

* temporary WiFi access point mode
* configuration web interface
* parameter entry
* configuration validation
* persistent storage of settings

The SetupPortal may expose application-provided configuration fields.

However:

* the application defines those fields
* the application validates their meaning
* the foundation only provides the mechanism

---

# Consequences

## Benefits

* Framework remains reusable across unrelated projects
* Common services are implemented once
* Firmware behavior becomes more consistent
* New projects begin from a tested foundation
* Bugs and improvements benefit all projects
* Clear ownership boundaries reduce complexity

## Tradeoffs

* Requires more initial architecture
* Requires disciplined module boundaries
* Very small projects may require more structure than necessary
* Applications must explicitly define their own behaviors

---

# Guiding Principle

ESP32_Foundation provides capability.

Applications provide purpose.

A foundation module should answer:

> "What service do I provide?"

An application module should answer:

> "What problem am I solving?"
