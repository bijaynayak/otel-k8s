# otel-k8s
openelemetry-kubernetes setup with load-generators

## Architecture Diagram

                       +-------------------------------------------------------------------------------------------------------------+                                 
                       | load-generator deployment                                                                                   |                                 
                       |+------------------+  +--------------------------+  +--------------------------+ +--------------------------+|                                 
                       || OpenCensus       |  | omnition                 |  |omnition                  | | freshtracks.io           ||                                 
                       || trace + metrics  |  | synthetic-load-generator |  |synthetic-load-generator  | | avalanche                ||                                 
                       || generator        |  | jaeger-emtter            |  |zipkin-emitter            | | prom metrics generator   ||                                 
                       |+----------\-------+  +-------------|------------+  +-----------/--------------+ +-------------|------------+|                                 
                       +---------------\--------------------|-----------------------/----------------------------------|-------------+                                 
                                        --\                 |                   /---                     +-------------|------------+                                  
                                           ---\             |               /---                         |    prometheus-avalanche  |                                  
                                       :55678  --\          |:14268     /---  :9411                      |    service               |                                  
                                                  ---\      |       /---                                 +-------------|------------+                                  
                                                 +----------|----------+                                               |                                               
                                                 | opentelemetry-agent |                                               |                                               
                                                 | headless service    |                                               |                                               
                                                 +---/------|----------+                                               |                                               
                                               /-----       |       \---                                               |                                               
                       +-----------------/------------------|-----------\----------------------------------------------|--------------+                                
                       |+----------/------------------------|---------------\------------------------------------------|-------------+|                                
                       || +-----------+                +----|---+            +--\-----+     +-------------+  +---------|---------+   ||                                
                       || |opencensus |                | jaeger |            |zipkin  |     | prometheus  ---- scrape-config     |   ||                                
                       || +-----------+                +--------+            +--------+     +-------------+  +-------------------+   ||                                
                       || Receivers                                                                                                  ||                                
                       |+------------------------------------------------------------------------------------------------------------+|                                
                       |+------------------------------------------------------------------------------------------------------------+|                                
                       ||                              +-------+                            +--------------+                         ||                                
                       ||                              | batch |                            | queued-retry |                         ||                                
                       || Processors                   +-------+                            +--------------+                         ||                                
                       |+------------------------------------------------------------------------------------------------------------+|                                
                       |+------------------------------------------------------------------------------------------------------------+|                                
                       ||                                             +------------------+                                           ||                                
                       ||                                             |    opencensus    |                                           ||                                
                       || Exporters                                   +---------|--------+                                           ||                                
                       |+-------------------------------------------------------|----------------------------------------------------+|                                
                       |  open-telemetry agent daemonset                        |                                                     |                                
                       +--------------------------------------------------------|-----------------------------------------------------+                                
                                                                                | :55678                                                                               
                                                                   +------------|------------+                                                                         
                                                                   |opentelemetry-collector  |                                                                         
                                                                   |headless service         |                                                                         
                                                                   +------------|------------+                                                                         
                                                                                |                                                                                      
                       +--------------------------------------------------------|-----------------------------------------------------+                                
                       |+-------------------------------------------------------|---------------------------------------------------+ |                                
                       ||                                                +------|-----+                                             | |                                
                       ||                                                | opencensus |                                             | |                                
                       ||  Receivers                                     +------------+                                             | |                                
                       |+-----------------------------------------------------------------------------------------------------------+ |                                
                       |+-----------------------------------------------------------------------------------------------------------+ |                                
                       ||                              +-------+                             +--------------+                       | |                                
                       ||                              | batch |                             | queued-retry |                       | |                                
                       ||  Processors                  +-------+                             +--------------+                       | |                                
                       |+-----------------------------------------------------------------------------------------------------------+ |                                
                       |+-----------------------------------------------------------------------------------------------------------+ |                                
                       ||                  +------------+            +------------+            +--------------------+               | |                                
                       ||                  |  jaeger    |            |  zipkin    |            | prometheus         |               | |                                
                       ||  Exporters       +------|-----+            +------|-----+            +----------|---------+               | |                                
                       |+-------------------------|-------------------------|-----------------------------|-------------------------+ |                                
                       +--------------------------|-------------------------|-----------------------------|---------------------------+                                
                                                  |                         |                +------------|-----------+                                                
                                                  |                         |                |opentelemetry-collector |                                                
                                                  |                         |                |headless service        |                                                
                                                  |                         |                +------------|-----------+                                                
                                                  |:14250                   |:9411                        | :8889                                                      
                                      +-----------|----------+  +-----------|-----------+    +------------|-----------+                                                
                                      |   jaeger-all-in-one  |  |  zipkin-all-in-one    |    |     prometheus         |                                                
                                      +----------------------+  +-----------------------+    +------------------------+                                                
