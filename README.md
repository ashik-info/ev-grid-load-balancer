
 **Dynamic Grid Load Balancer & Emergency Throttler**

**System Architecture & Component Interactions**

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


