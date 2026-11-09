# 🛣️ RoadSense: Intelligent Road-Safety & Infrastructure Monitoring

RoadSense is a multi-platform ecosystem designed to prevent vehicle-bridge collisions by combining Edge AI, IoT sensing, and real-time cloud supervision. The system detects oversized vehicles in real time and triggers an immediate physical and digital alert chain to ensure driver diversion and infrastructure safety.

> **Note:** This repository is a technical case study (architecture and documentation only). The source code is proprietary and is not publicly available. See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## 🏗️ System Architecture

The system is engineered as a distributed pipeline across three primary modules.

### 1. Detection & Sensing (Edge Layer)

Located 300–400 m before the infrastructure, this layer performs multi-sensor fusion to eliminate false positives.

- **Sensors**
  - **LiDAR (Benewake TF03-180):** high-precision laser Time-of-Flight measurement of vehicle height (±1 cm accuracy).
  - **IP Camera (Hikvision ColorVu):** high-resolution visual feed for vehicle classification.
  - **Environmental sensing (BME280):** real-time temperature and humidity monitoring to dynamically adjust safety thresholds (δ_météo).
- **Edge processing:** NVIDIA Jetson Nano running YOLOv8s (exported to ONNX Runtime) for fast inference (< 100 ms).
- **Fusion logic:** an alert is triggered only when the measured height exceeds the threshold **and** the AI confirms the object is a heavy vehicle:

$$
\text{Alert} = \left(h > H_{\text{seuil}} + \delta_{\text{météo}}\right) \wedge \left(\text{Confidence}_{\text{IA}} > \theta_{\text{IA}}\right)
$$

### 2. Local Alerting (Physical Layer)

To ensure immediate driver reaction, the system uses a low-latency wireless link.

- **Communication:** 433 MHz LoRa link (≈ 500 m range) carrying lightweight JSON packets.
- **Actuators:** Module B drives a P10 LED matrix, a 110 dB industrial siren, and traffic-diversion lights to force immediate vehicle diversion.

### 3. Supervision & Cloud (Management Layer)

Simultaneous data streaming over 4G (SIM7600G-H) for multi-stakeholder monitoring.

- **Backend:** Node.js & Express.js server managing a relational schema in PostgreSQL.
- **Real-time stream:** Socket.IO for non-blocking, sub-second alert delivery.
- **Platforms**
  - **B2G Dashboard (React):** public authorities monitor infrastructure health.
  - **B2B Dashboard (React):** logistics fleet managers track vehicle compliance.
  - **B2C Mobile App (Flutter):** "SafePass" app with GPS-based rerouting and push notifications.

---

## 🛠️ Technical Stack

| Layer | Technologies |
|---|---|
| AI & ML | Python, YOLOv8s, OpenCV, ONNX Runtime, PyTorch |
| Embedded / IoT | NVIDIA Jetson Nano, ESP32-CAM, LoRa (SX1262/SX1276), LiDAR |
| Backend | Node.js, Express.js, PostgreSQL, Socket.IO, JWT |
| Frontend | React.js, Flutter, Dart, Tailwind CSS |
| Infrastructure | 4G LTE, REST APIs, JSON |

---

## 🚀 Engineering Challenges Solved

### ⚡ Latency Optimization
- **Challenge:** meeting a strict 2-second safety window for end-to-end response.
- **Solution:** exported the model to ONNX Runtime and implemented a 15-frame circular buffer, bringing total response time under 500 ms.

### 🔌 Hardware Stability (LoRa Receiver)
- **Challenge:** the SX1262 radio module froze after receiving its first packet.
- **Solution:** re-engineered driver initialization with a structured `RadioEvents_t` callback table and a recursive `Radio.Rx(0)` loop with a 5-second fail-safe timeout.

### 🖼️ Memory Management (ESP32-CAM)
- **Challenge:** PSRAM saturation during MQTT streaming caused HTTP timeouts and crashes.
- **Solution:** migrated from a polling-based MQTT model to an event-driven HTTP POST architecture, uploading images only upon LiDAR detection.

### 🧪 AI Robustness (Data Augmentation)
- **Challenge:** a breaking change in Albumentations v2.0.8 caused YOLOv8 to silently skip weather augmentations (fog/rain), reducing accuracy in adverse conditions.
- **Solution:** wrote a Python monkey-patch bridging the signature mismatch, restoring weather-based augmentation and improving model reliability.

---

## 📊 Database Domain Model

- **Users & Roles:** B2G Admin, B2B Manager, B2C Driver.
- **Infrastructure:** bridge/tunnel coordinates, clearance limits, operational status.
- **Detections & Alerts:** logs of height readings, confidence scores, and incident resolution status.
- **Fleet Management:** vehicle dimensions and driver certification tracking.

---

## 👩‍💻 My Role

I was the backend lead and a core contributor on the hardware and AI side of this project:

- **Backend & integration:** designed the backend architecture, the B2B routes and middleware, and handled end-to-end integration between the edge, cloud, and client layers.
- **IoT:** carried most of the IoT work, including hardware testing and validation of the sensing and communication chain (LiDAR, LoRa, ESP32-CAM).
- **AI vehicle recognition:** built the entire vehicle-recognition pipeline, for both the prototype and the real-world deployment scenario (dataset and augmentation, YOLOv8 training, ONNX export, and edge inference).

---

*This repository is a technical case study. The source code is proprietary and is not available for public distribution.*
