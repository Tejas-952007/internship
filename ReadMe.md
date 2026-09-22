# ⚡ Smart Edge-AI EV Charging Station Optimizer

**ESP32 · Edge AI · MQTT · ThingsBoard · Embedded C/C++ · Python**

> An intelligent multi-bay EV charging management system that uses **Edge AI, predictive load management, thermal safety control, and dynamic current allocation** to optimize EV charging under a limited electrical capacity.

---

## 📌 Overview

The **Smart Edge-AI EV Charging Station Optimizer** is an IoT and Edge AI based system designed to intelligently manage multiple EV charging bays when the available electrical capacity is limited.

The system models a **3-bay EV charging station** controlled by an **ESP32 Edge Controller**. Each charging bay continuously provides parameters such as:

* Voltage
* Current
* Power
* State of Charge (SOC)
* Energy consumption
* Connector temperature
* Charging state

The ESP32 processes this information locally and runs lightweight machine-learning models to predict:

1. **EV arrival probability**
2. **Expected charging duration**

These predictions are combined with real-time station conditions and safety constraints by the charging optimizer.

The optimizer dynamically decides whether each charging bay should:

**ALLOW → THROTTLE → DEFER**

The resulting telemetry is published using **MQTT** to **ThingsBoard** and a mirror MQTT broker, where it can be monitored through a web dashboard.

---

# 🎯 Problem Statement

When multiple EVs charge simultaneously, the combined charging demand can exceed the available grid capacity.

For example:

```text
Bay 1 → 32 A
Bay 2 → 32 A
Bay 3 → 32 A
----------------
Total → 96 A
```

But the station has a maximum capacity of:

```text
Station Limit = 64 A
```

Therefore:

```text
96 A > 64 A
```

Uncontrolled charging can result in:

* Feeder overload
* Inefficient power distribution
* Increased peak demand
* Connector thermal stress
* Poor utilization of available charging capacity

The project addresses this problem through **predictive, safety-aware, Edge-AI-based charging optimization**.

---

# 💡 Proposed Solution

The system follows a continuous:

```text
SENSE
  ↓
PREDICT
  ↓
OPTIMIZE
  ↓
CONTROL
  ↓
MONITOR
```

workflow.

### 1. Sense

The ESP32 receives or simulates charging-station telemetry.

### 2. Predict

Edge AI predicts:

* Future EV arrival probability
* Expected charging duration

### 3. Optimize

The optimizer considers:

* Station capacity
* Bay current limits
* EV priority
* SOC
* Connector temperature
* Predicted future demand
* Minimum useful charging current

### 4. Control

Each bay receives one of three decisions:

```text
ALLOW
THROTTLE
DEFER
```

### 5. Monitor

Telemetry and decisions are published through MQTT and visualized using ThingsBoard and the web dashboard.

---

# 🏗️ System Architecture

```text
┌──────────────────────────────────────────────┐
│              EV CHARGING STATION             │
│                                              │
│   ┌────────┐   ┌────────┐   ┌────────┐      │
│   │ BAY 1  │   │ BAY 2  │   │ BAY 3  │      │
│   │  EV    │   │  EV    │   │  EV    │      │
│   └───┬────┘   └───┬────┘   └───┬────┘      │
│       └─────────────┼─────────────┘          │
│                     │                        │
│              Charging Telemetry              │
└─────────────────────┼────────────────────────┘
                      ↓
┌──────────────────────────────────────────────┐
│              ESP32 EDGE CONTROLLER            │
│                                              │
│  Telemetry Processing                        │
│          ↓                                   │
│  ┌────────────────────────────────────────┐  │
│  │             EDGE AI                    │  │
│  │                                        │  │
│  │ Logistic Regression                    │  │
│  │ → EV Arrival Probability               │  │
│  │                                        │  │
│  │ Decision Tree Regressor                │  │
│  │ → Charging Duration                   │  │
│  └────────────────────────────────────────┘  │
│          ↓                                   │
│       Optimizer                              │
│          ↓                                   │
│   ALLOW / THROTTLE / DEFER                  │
│          ↓                                   │
│   Safety + 64 A Hard Limit                  │
└──────────────────────┬───────────────────────┘
                       │
                      MQTT
                       │
              ┌────────┴────────┐
              ↓                 ↓
       ┌──────────────┐   ┌──────────────┐
       │ ThingsBoard  │   │     EMQX     │
       │    Cloud     │   │ MQTT Broker  │
       └──────┬───────┘   └──────┬───────┘
              │                  │
              └────────┬─────────┘
                       ↓
              ┌─────────────────┐
              │  Web Dashboard  │
              │                 │
              │ • Station Load  │
              │ • Bay Status    │
              │ • AI Prediction │
              │ • Power         │
              │ • Temperature   │
              │ • Alarms        │
              └─────────────────┘
```

### Architecture Principle

The key architectural concept is:

> **Critical AI inference and charging optimization happen locally on the ESP32, while the cloud is primarily used for telemetry, monitoring, communication, and visualization.**

---

# 🤖 Edge AI

The project uses two lightweight machine-learning models.

## 1. EV Arrival Prediction

### Algorithm

**Logistic Regression**

### Input Features

```text
hour_sin
hour_cos
weekday
is_weekend
is_peak
busy_rate
```

### Output

```text
Arrival Probability
0 → 1
```

The model estimates the probability that another EV may arrive within the next **15 minutes**.

This prediction is used by the optimizer to determine how much electrical capacity should be reserved for potential future demand.

---

## 2. Charging Duration Prediction

### Algorithm

**Decision Tree Regressor**

### Input Features

```text
Battery Capacity
Initial SOC
Target SOC
Ambient Temperature
Charging Current
Connector Temperature
```

### Output

```text
Predicted Charging Duration
(minutes)
```

The predicted duration helps the system understand the expected charging demand of an active session.

---

# 🧠 AI Training & Edge Deployment

The models are trained offline using:

```text
Python
   ↓
scikit-learn
   ↓
Model Training
   ↓
Model Evaluation
   ↓
Model Export
   ↓
models.h
   ↓
ESP32 Firmware
   ↓
Local Edge Inference
```

The trained model parameters are converted into lightweight C representations and included in the ESP32 firmware.

This allows the ESP32 to perform inference locally without requiring a cloud AI API for every decision.

### Benefits

* ⚡ Low-latency inference
* 🌐 Reduced cloud dependency
* 🔒 Local decision-making
* 📡 Better operation during connectivity interruptions
* 💻 Suitable for resource-constrained edge devices

---

# ⚙️ Charging Optimization

The optimizer continuously evaluates every charging bay.

## Decision Flow

```text
Safety Check
     ↓
Thermal Control
     ↓
AI Predictive Reserve
     ↓
Available Capacity
     ↓
Load Balancing
     ↓
Minimum Current Check
     ↓
64 A Hard Limit
     ↓
ALLOW / THROTTLE / DEFER
```

---

## 🌡️ Thermal Safety

The charging current is dynamically reduced according to connector temperature.

| Connector Temperature | Action                 |
| --------------------- | ---------------------- |
| `< 55°C`              | Normal charging        |
| `55–65°C`             | 75% current            |
| `65–80°C`             | 40% current            |
| `≥ 80°C`              | DEFER / charging pause |

A critical temperature condition also generates an alarm.

---

# 🔌 Current Management

The station is designed around:

```text
Maximum Station Current = 64 A
Maximum Bay Current      = 32 A
Minimum Useful Current   = 6 A
```

If total requested current exceeds the available station capacity, the optimizer performs dynamic current allocation.

Example:

```text
Requested:

Bay 1 → 32 A
Bay 2 → 32 A
Bay 3 → 32 A

Total → 96 A
```

The optimizer cannot allow 96 A because:

```text
Station Limit = 64 A
```

Instead, available current is dynamically distributed among the active bays while maintaining the hard station limit.

If a bay's calculated current falls below the **6 A minimum useful charging current**, the session can be deferred and capacity can be redistributed.

---

# 🔮 Predictive Load Management

Unlike a simple load balancer, the system also considers **future charging demand**.

The arrival prediction contributes to a predictive reserve.

Conceptually:

```text
Higher Arrival Probability
          ↓
Higher Expected Future Demand
          ↓
More Capacity Reserved
          ↓
Lower Risk of Future Overload
```

This allows the station to make decisions based not only on the current load, but also on possible near-future demand.

---

# 📡 MQTT & IoT Connectivity

MQTT is used as the primary lightweight communication protocol.

### ThingsBoard

Used for:

* Device telemetry
* Cloud monitoring
* Device attributes
* RPC communication
* IoT visualization

Main telemetry topic:

```text
v1/devices/me/telemetry
```

### EMQX MQTT Broker

A mirror MQTT communication path is also used for the web dashboard.

The dashboard communicates through MQTT/WebSockets to receive live station information.

---

# 🖥️ Web Dashboard

The project includes a web-based monitoring and control dashboard.

### Station-Level Information

* Station current
* Station power
* Charging load
* AI arrival probability
* Energy consumption

### Bay-Level Information

Each bay displays:

* SOC
* Voltage
* Current
* Power
* Connector temperature
* Charging state
* AI estimated duration
* Optimizer decision

### Optimizer States

```text
🟢 ALLOW
Normal charging

🟠 THROTTLE
Reduced charging current

🔴 DEFER
Charging temporarily paused
```

---

# 🎮 Dashboard Controls

The dashboard supports operational commands including:

```text
plugIn
plugOut
setEnabled
setMode
setMaxCurrent
setBayLimit
setPriority
fault
clearFault
resetEnergy
getStatus
```

This provides bidirectional communication between the dashboard and the ESP32 controller.

---

# 📴 Demo / Offline Support

The dashboard includes a fallback demonstration mechanism.

If live MQTT data is unavailable for approximately **12 seconds**, the dashboard can switch to a demo feed.

This allows the project to remain demonstrable even when live MQTT connectivity is temporarily unavailable.

---

# 📊 Machine Learning Performance

The project evaluates both Edge AI models.

### EV Arrival Prediction

```text
Model: Logistic Regression

ROC-AUC ≈ 0.79
```

### Charging Duration Prediction

```text
Model: Decision Tree Regressor

R² ≈ 0.91
MAE ≈ 43 minutes
```

These metrics are based on the project's model evaluation pipeline.

---

# 🛠️ Technology Stack

| Technology                  | Role                                  |
| --------------------------- | ------------------------------------- |
| **ESP32**                   | Edge controller and local inference   |
| **Embedded C/C++**          | Firmware and real-time control        |
| **Python**                  | ML training and simulation            |
| **scikit-learn**            | Machine-learning models               |
| **Logistic Regression**     | EV arrival prediction                 |
| **Decision Tree Regressor** | Charging duration prediction          |
| **MQTT**                    | IoT communication                     |
| **ThingsBoard**             | Cloud IoT monitoring                  |
| **EMQX**                    | MQTT broker / dashboard communication |
| **HTML/CSS/JavaScript**     | Web dashboard                         |
| **MQTT.js**                 | Browser MQTT communication            |
| **Wokwi**                   | ESP32/system simulation               |

---

# 🔄 End-to-End Workflow

```text
EV Charging Data
       ↓
ESP32
       ↓
Telemetry Processing
       ↓
Edge AI Inference
       ↓
┌─────────────────────────────┐
│ Arrival Probability         │
│ Charging Duration           │
└─────────────────────────────┘
       ↓
Optimization Engine
       ↓
Safety + Thermal Control
       ↓
Predictive Reserve
       ↓
Dynamic Load Balancing
       ↓
ALLOW / THROTTLE / DEFER
       ↓
MQTT
       ↓
ThingsBoard + EMQX
       ↓
Web Dashboard
```

---

# 📁 Project Structure

```text
.
├── firmware/
│   ├── ESP32 firmware
│   ├── optimizer
│   ├── MQTT communication
│   └── Edge AI inference
│
├── ai/
│   ├── arrival prediction
│   ├── charging duration prediction
│   ├── dataset generation
│   └── model evaluation
│
├── models/
│   └── models.h
│
├── dashboard/
│   ├── HTML
│   ├── CSS
│   └── JavaScript
│
├── docs/
│   └── project documentation
│
└── README.md
```

> Update the folder names above if your repository uses different directory names.

---

# 🚀 Example Optimization Scenario

Consider a station where three EVs simultaneously request maximum charging current:

```text
Bay 1 → 32 A
Bay 2 → 32 A
Bay 3 → 32 A

Total Demand → 96 A
```

Available station capacity:

```text
64 A
```

The optimizer detects:

```text
Requested Current > Available Current
```

and dynamically allocates the available current instead of allowing the station to exceed its capacity.

If connector temperature increases, thermal control further reduces the affected bay's charging current.

If future EV arrival probability is high, the optimizer can reserve additional capacity.

Therefore, the final charging decision is based on:

```text
Current Demand
+
AI Prediction
+
Temperature
+
SOC
+
Priority
+
Station Capacity
```

---

# 🌱 Future Scope

The architecture can be extended toward a larger intelligent charging ecosystem.

### ☀️ Solar-Powered Charging

Integrate solar generation data and prioritize renewable energy for EV charging.

### 🔋 Battery Energy Storage

Use stationary batteries to reduce peak grid demand.

### 💰 Dynamic Electricity Pricing

Schedule or throttle charging based on real-time electricity tariffs.

### 🏢 Multi-Station Optimization

Coordinate multiple charging stations from a centralized optimization layer.

### 📱 Mobile Application

Provide EV users with:

* Charging status
* Estimated completion time
* Notifications
* Charging control

### 🚚 Fleet Charging

Extend the optimizer for commercial EV fleets and large-scale charging depots.

---

# 🎓 Project Learning Outcomes

This project combines multiple engineering domains:

* Embedded Systems
* ESP32 Firmware Development
* Internet of Things
* MQTT Communication
* Cloud IoT
* Machine Learning
* Edge AI
* Optimization Algorithms
* Real-Time Decision Making
* Web Dashboard Development

The project demonstrates how a machine-learning model can move beyond offline prediction and become part of a **real-time embedded control workflow**.

---

# 🔑 Key Takeaway

The core idea of this project can be summarized as:

```text
┌────────┐
│ SENSE  │
└───┬────┘
    ↓
┌─────────┐
│ PREDICT │
└───┬─────┘
    ↓
┌──────────┐
│ OPTIMIZE │
└───┬──────┘
    ↓
┌─────────┐
│ CONTROL │
└───┬─────┘
    ↓
┌─────────┐
│ MONITOR │
└─────────┘
```

> **Smart EV charging is not only about supplying power — it is about predicting demand, managing constraints, protecting the hardware, and using available energy intelligently.**

---

# 👨‍💻 Author

**Tejas Ingle**

3rd Year — AI & Data Science Engineering

**Emertxe IoT Internship — 2026**

---

## ⭐ Project Highlights

**ESP32 + Edge AI + MQTT + ThingsBoard + Intelligent Load Optimization**

**Predictive. Local. Safe. Connected.**
