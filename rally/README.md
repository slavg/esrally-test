# Rally Testing



### Create custom track
````bash

docker compose run esrally create-track \
--track=ecommerce \
--target-hosts=es01:9200 \
--client-options="basic_auth_user:elastic,basic_auth_password:{ELASTIC_PASSWORD}" \
--indices=kibana_sample_data_ecommerce \
--output-path=/rally/tracks

````

### Run Track
````bash
docker compose run esrally race \
--track-path=/rally/tracks/ecommerce \
--target-hosts=es01:9200 \
--pipeline=benchmark-only \
--client-options="timeout:60,use_ssl:false,verify_certs:false,basic_auth_user:elastic,basic_auth_password:{ELASTIC_PASSWORD}"
````





# The Seven Deadly Sins of Elasticsearch Benchmarking

https://www.elastic.co/elasticon/conf/2018/sf/the-seven-deadly-sins-of-elasticsearch-benchmarking

### Sin 1: Ignoring System Setup

Ensure hardware (CPU, memory, disk, network) and software (OS, JVM, Elasticsearch version) are identical across runs. A stable production-like environment with minimal background processes is essential for accurate benchmarking. Additionally, isolate benchmarking traffic from external interference and reduce noise through measures like enabling SSD TRIM.

### Sin 2: Cold Start

Running benchmarks without proper warmup leads to inaccurate results. The JIT Compilation optimizes code after initial execution, while CPU, disk, OS page, and Elasticsearch caches all significantly affect performance. To get realistic performance numbers, always measure throughput only after a complete warmup phase.

### Sin 3: "Hit it as Hard as Possible"

Understanding that Latency equals Waiting Time plus Service Time is crucial. Running at 100% utilization inevitably leads to increased waiting time and high request latency. Instead, use batch operations for bulk indexing and define target throughput for search operations. For more realistic testing, implement Poisson-based scheduling rather than deterministic request patterns when modeling arrival rates.

### Sin 4: Blindly Trusting Benchmarking Scripts

Never assume benchmarking tools are error-free. Proper validation of load generator behavior is essential - check response status codes, verify proper handling of request timeouts, and monitor network traffic and resource contention using tools like Wireshark to ensure accurate results.

### Sin 5: Accidental Bottlenecks

Different components of Elasticsearch (master nodes, ingest nodes, hot storage, and warm storage) have distinct performance characteristics. A common mistake is adding more Elasticsearch nodes without verifying network bandwidth capacity. Implement Brendan Gregg's USE method (Utilization, Saturation, Errors) to properly analyze performance bottlenecks.

### Sin 6: Chaos in Benchmarking Process

Maintain a structured approach to benchmarking: always reset the environment to a known stable state, modify only one variable at a time, conduct controlled experiments with clear documentation, and maintain detailed logs of every configuration detail including Elasticsearch version, JVM version, OS, and CPU specifications.

### Sin 7: Ignoring Statistical Significance

System noise causes benchmark results to vary between runs. Conduct at least 30 trials and perform statistical significance tests like t-tests for validity. Report comprehensive statistics including median, min, max, and percentiles rather than just mean values. Remember that latency distributions are often multi-modal, making normal distribution assumptions inappropriate for performance analysis.

### Key Takeaways

- Benchmark in a stable production-like environment
- Warm up caches before measuring performance
- Avoid 100% utilization, monitor latency vs. throughput
- Verify benchmarking tools, don't trust scripts blindly
- Ensure you're stressing the right component
- Change one parameter at a time in a structured process
- Validate results with statistical analysis