# **EV Smart-Charging & Micro-Grid Load Balancer**

<img width="2048" height="945" alt="image" src="https://github.com/user-attachments/assets/391e6196-e284-48f8-bdf2-5d2174143d57" />




An energy orchestration engine balancing power draw across hundreds of urban Electric Vehicle charging hubs during peak grid hours.

> * **How the stack works:** Charging stations stream voltage draw, queue depth, and battery states continuously over **MQTT**. High-volume data flows into **RabbitMQ work queues**, where a Go-based load-balancing engine runs dynamic power distribution algorithms. The hub uses persistent **gRPC streams** to push millisecond power-throttle and boost directives down to sub-station grid transformers.  
> * **Why it's awesome:** Solves a major real-world green-tech engineering challenge involving concurrent worker pools, edge-to-cloud backpressure, and reactive control streams.


### **1\. The Dynamic Grid Load Balancer & Emergency Throttler**

> * **Focus:** Preventing local power grid blackouts during peak charging hours.  
> * **Architecture:** 50+ Dockerized EV charger containers stream high-frequency voltage and current metrics over **MQTT**. A Go ingestion pipeline buffers these metrics into **RabbitMQ work queues**. A central load balancing service calculates real-time grid strain and uses persistent **gRPC streams** back to specific charger containers to dynamically dial power down (e.g., throttling fast-chargers from 150kW to 22kW in under 50 milliseconds).  
> * **Key Challenge Solved:** High-concurrency feedback loops and sub-second control commands under heavy queue load.

### **2\. Predictive Thermal & Battery Safety Sentry**

> * **Focus:** Real-time hardware health, thermal runaway prevention, and emergency intervention.  
> * **Architecture:** Containerized chargers emit continuous thermal, connector impedance, and cell health telemetry via **MQTT**. Go worker pools pull from **RabbitMQ topic exchanges**, running sliding-window spike detection algorithms. If a thermal anomaly is detected, the engine instantly fires a high-priority **gRPC kill-switch command** directly to that charger's edge agent, while publishing diagnostic fault logs to a RabbitMQ dead-letter queue (DLQ).  
> * **Key Challenge Solved:** Stream processing, event-driven alerting, and deterministic priority messaging.

### **3\. Commercial Fleet Reservation & Surge Pricing Engine**

> * **Focus:** Economic optimization and smart routing for commercial EV fleets (e.g., delivery trucks or taxis).  
> * **Architecture:** Chargers continuously report bay availability and dynamic energy costs over **MQTT**. Fleet dispatcher clients establish **bidirectional gRPC streams** with your backend to request optimal charging slots based on live route data. Non-blocking asynchronous billing events, payment receipts, and session summaries are published to **RabbitMQ** for processing by background microservices.  
> * **Key Challenge Solved:** Segregating low-latency synchronous client requests (gRPC) from asynchronous domain events (RabbitMQ).

## **System Architecture & Component Interactions**

`+---------------------------------------------------------------------------------+`  
`|                              Dockerized Edge Layer                              |`  
`|                                                                                 |`  
`|  +-------------------------+            +-------------------------+             |`  
`|  | Charger Node #1 (Go)    |            | Charger Node #N (Go)    |             |`  
`|  | - Telemetry over MQTT   |            | - Telemetry over MQTT   |             |`  
`|  | - Control via gRPC      |            | - Control via gRPC      |             |`  
`|  +------------+------------+            +------------+------------+             |`  
`+---------------+--------------------------------------+--------------------------+`  
                `| MQTT (port 1883)                     | gRPC Bi-directional Stream`  
                `v                                      v`  
`+------------------------------------+   +----------------------------------------+`  
`| RabbitMQ Broker (MQTT Plugin)      |   | Go Central Control Plane               |`  
``| - Exchange: `ecofleet.telemetry`   |   |                                        |``  
``| - Queue: `telemetry.raw.q`         |   | - gRPC Server (`ControlService`)       |``  
``+---------------+--------------------+   | - Active Stream Registry (`sync.Map`)   |``  
                `| AMQP Consumer          | - Dynamic Grid Balancer Engine         |`  
                `v                        |   (Calculates Sub-station kW budget)   |`  
`+------------------------------------+   |                                        |`  
`| Go Ingestion Workers               |---| Push Throttling Commands               |`  
`| (Aggregates load per transformer)  |   +----------------------------------------+`  
`+------------------------------------+`


