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


### Customize track

````json

{
  "version": 2,
  "description": "Track for ecommerce benchmarking with various operations",
  "indices": [
    {
      "name": "kibana_sample_data_ecommerce",
      "body": "kibana_sample_data_ecommerce.json"
    }
  ],
  "corpora": [
    {
      "name": "kibana_sample_data_ecommerce",
      "documents": [
        {
          "target-index": "kibana_sample_data_ecommerce",
          "source-file": "kibana_sample_data_ecommerce-documents.json.bz2",
          "document-count": 4675,
          "compressed-bytes": 413250,
          "uncompressed-bytes": 7960826
        }
      ]
    }
  ],
  "operations": [
    {
      "name": "delete-index",
      "operation-type": "delete-index"
    },
    {
      "name": "create-index",
      "operation-type": "create-index"
    },
    {
      "name": "bulk-index",
      "operation-type": "bulk",
      "bulk-size": 1000
    },
    {
      "name": "match-all",
      "operation-type": "search",
      "body": {
        "query": {
          "match_all": {}
        }
      }
    },
    {
      "name": "mens-clothing-search",
      "operation-type": "search",
      "body": {
        "query": {
          "match_phrase": {
            "category": "Men's Clothing"
          }
        }
      }
    },
    {
      "name": "womens-clothing-search",
      "operation-type": "search",
      "body": {
        "query": {
          "match_phrase": {
            "category": "Women's Clothing"
          }
        }
      }
    },
    {
      "name": "category-price-stats",
      "operation-type": "search",
      "body": {
        "size": 0,
        "aggs": {
          "categories": {
            "terms": {
              "field": "category.keyword",
              "size": 10
            },
            "aggs": {
              "avg_price": {
                "avg": {
                  "field": "taxful_total_price"
                }
              },
              "min_price": {
                "min": {
                  "field": "taxful_total_price"
                }
              },
              "max_price": {
                "max": {
                  "field": "taxful_total_price"
                }
              }
            }
          }
        }
      }
    }
  ],
  "challenges": [
    {
      "name": "indexing-benchmark",
      "description": "Testing bulk indexing with different concurrency",
      "default": true,
      "schedule": [
        {
          "operation": "delete-index"
        },
        {
          "operation": "create-index"
        },
        {
          "operation": "bulk-index",
          "warmup-time-period": 60,
          "clients": 8
        }
      ]
    },
    {
      "name": "search-benchmark",
      "description": "Various search operations benchmark",
      "schedule": [
        {
          "operation": "match-all",
          "warmup-iterations": 50,
          "iterations": 100,
          "clients": 4
        },
        {
          "operation": "mens-clothing-search",
          "warmup-iterations": 50,
          "iterations": 100,
          "clients": 4
        },
        {
          "operation": "womens-clothing-search",
          "warmup-iterations": 50,
          "iterations": 100,
          "clients": 4
        },
        {
          "operation": "category-price-stats",
          "warmup-iterations": 50,
          "iterations": 100,
          "clients": 2
        }
      ]
    }
  ]
}

````

### Run specific challenge

````bash
docker compose run esrally race \
--track-path=/rally/tracks/ecommerce \
--target-hosts=es01:9200 \
--pipeline=benchmark-only \
--challenge=search-benchmark \
--client-options="basic_auth_user:elastic,basic_auth_password:{ELASTIC_PASSWORD}"

````

### Sample output

````bash
   ____        ____
   / __ \____ _/ / /_  __
  / /_/ / __ `/ / / / / /
 / _, _/ /_/ / / / /_/ /
/_/ |_|\__,_/_/_/\__, /
                /____/

[INFO] Race id is [aa2495ca-df15-4b0c-9d2d-9e0c7264a301]
[INFO] Racing on track [ecommerce], challenge [search-benchmark] and car ['external'] with version [8.15.2].

Running match-all                                                              [100% done]
Running mens-clothing-search                                                   [100% done]
Running womens-clothing-search                                                 [100% done]
Running category-price-stats                                                   [100% done]

------------------------------------------------------
    _______             __   _____
   / ____(_)___  ____ _/ /  / ___/_________  ________ 
  / /_  / / __ \/ __ `/ /   \__ \/ ___/ __ \/ ___/ _ \
 / __/ / / / / / /_/ / /   ___/ / /__/ /_/ / /  /  __/
/_/   /_/_/ /_/\__,_/_/   /____/\___/\____/_/   \___/ 
------------------------------------------------------

|                                                         Metric |                   Task |         Value |   Unit |
|---------------------------------------------------------------:|-----------------------:|--------------:|-------:|
|                     Cumulative indexing time of primary shards |                        |   0           |    min |
|             Min cumulative indexing time across primary shards |                        |   0           |    min |
|          Median cumulative indexing time across primary shards |                        |   0           |    min |
|             Max cumulative indexing time across primary shards |                        |   0           |    min |
|            Cumulative indexing throttle time of primary shards |                        |   0           |    min |
|    Min cumulative indexing throttle time across primary shards |                        |   0           |    min |
| Median cumulative indexing throttle time across primary shards |                        |   0           |    min |
|    Max cumulative indexing throttle time across primary shards |                        |   0           |    min |
|                        Cumulative merge time of primary shards |                        |   0           |    min |
|                       Cumulative merge count of primary shards |                        |   0           |        |
|                Min cumulative merge time across primary shards |                        |   0           |    min |
|             Median cumulative merge time across primary shards |                        |   0           |    min |
|                Max cumulative merge time across primary shards |                        |   0           |    min |
|               Cumulative merge throttle time of primary shards |                        |   0           |    min |
|       Min cumulative merge throttle time across primary shards |                        |   0           |    min |
|    Median cumulative merge throttle time across primary shards |                        |   0           |    min |
|       Max cumulative merge throttle time across primary shards |                        |   0           |    min |
|                      Cumulative refresh time of primary shards |                        |   0.00261667  |    min |
|                     Cumulative refresh count of primary shards |                        |  64           |        |
|              Min cumulative refresh time across primary shards |                        |   0           |    min |
|           Median cumulative refresh time across primary shards |                        |   0           |    min |
|              Max cumulative refresh time across primary shards |                        |   0.00108333  |    min |
|                        Cumulative flush time of primary shards |                        |   0.0157667   |    min |
|                       Cumulative flush count of primary shards |                        |  16           |        |
|                Min cumulative flush time across primary shards |                        |   0.00015     |    min |
|             Median cumulative flush time across primary shards |                        |   0.000366667 |    min |
|                Max cumulative flush time across primary shards |                        |   0.00398333  |    min |
|                                        Total Young Gen GC time |                        |   0.836       |      s |
|                                       Total Young Gen GC count |                        |   3           |        |
|                                          Total Old Gen GC time |                        |   0           |      s |
|                                         Total Old Gen GC count |                        |   0           |        |
|                                                   Dataset size |                        |   0.00924436  |     GB |
|                                                     Store size |                        |   0.00924436  |     GB |
|                                                  Translog size |                        |   8.19564e-07 |     GB |
|                                         Heap used for segments |                        |   0           |     MB |
|                                       Heap used for doc values |                        |   0           |     MB |
|                                            Heap used for terms |                        |   0           |     MB |
|                                            Heap used for norms |                        |   0           |     MB |
|                                           Heap used for points |                        |   0           |     MB |
|                                    Heap used for stored fields |                        |   0           |     MB |
|                                                  Segment count |                        |   6           |        |
|                                    Total Ingest Pipeline count |                        |   0           |        |
|                                     Total Ingest Pipeline time |                        |   0           |      s |
|                                   Total Ingest Pipeline failed |                        |   0           |        |
|                                                 Min Throughput |              match-all |  15.43        |  ops/s |
|                                                Mean Throughput |              match-all |  15.43        |  ops/s |
|                                              Median Throughput |              match-all |  15.43        |  ops/s |
|                                                 Max Throughput |              match-all |  15.43        |  ops/s |
|                                        50th percentile latency |              match-all |   4.45062     |     ms |
|                                        90th percentile latency |              match-all |   9.03397     |     ms |
|                                        99th percentile latency |              match-all |  32.232       |     ms |
|                                       100th percentile latency |              match-all | 446.31        |     ms |
|                                   50th percentile service time |              match-all |   4.45062     |     ms |
|                                   90th percentile service time |              match-all |   9.03397     |     ms |
|                                   99th percentile service time |              match-all |  32.232       |     ms |
|                                  100th percentile service time |              match-all | 446.31        |     ms |
|                                                     error rate |              match-all |   0           |      % |
|                                                 Min Throughput |   mens-clothing-search |  24.52        |  ops/s |
|                                                Mean Throughput |   mens-clothing-search |  24.52        |  ops/s |
|                                              Median Throughput |   mens-clothing-search |  24.52        |  ops/s |
|                                                 Max Throughput |   mens-clothing-search |  24.52        |  ops/s |
|                                        50th percentile latency |   mens-clothing-search |   5.66775     |     ms |
|                                        90th percentile latency |   mens-clothing-search |   9.81704     |     ms |
|                                        99th percentile latency |   mens-clothing-search |  69.5959      |     ms |
|                                       100th percentile latency |   mens-clothing-search | 679.754       |     ms |
|                                   50th percentile service time |   mens-clothing-search |   5.66775     |     ms |
|                                   90th percentile service time |   mens-clothing-search |   9.81704     |     ms |
|                                   99th percentile service time |   mens-clothing-search |  69.5959      |     ms |
|                                  100th percentile service time |   mens-clothing-search | 679.754       |     ms |
|                                                     error rate |   mens-clothing-search |   0           |      % |
|                                                 Min Throughput | womens-clothing-search | 222.69        |  ops/s |
|                                                Mean Throughput | womens-clothing-search | 247.66        |  ops/s |
|                                              Median Throughput | womens-clothing-search | 247.66        |  ops/s |
|                                                 Max Throughput | womens-clothing-search | 272.63        |  ops/s |
|                                        50th percentile latency | womens-clothing-search |   4.97785     |     ms |
|                                        90th percentile latency | womens-clothing-search |  10.0953      |     ms |
|                                        99th percentile latency | womens-clothing-search |  92.3095      |     ms |
|                                       100th percentile latency | womens-clothing-search | 415.887       |     ms |
|                                   50th percentile service time | womens-clothing-search |   4.97785     |     ms |
|                                   90th percentile service time | womens-clothing-search |  10.0953      |     ms |
|                                   99th percentile service time | womens-clothing-search |  92.3095      |     ms |
|                                  100th percentile service time | womens-clothing-search | 415.887       |     ms |
|                                                     error rate | womens-clothing-search |   0           |      % |
|                                                 Min Throughput |   category-price-stats |  20           |  ops/s |
|                                                Mean Throughput |   category-price-stats |  20           |  ops/s |
|                                              Median Throughput |   category-price-stats |  20           |  ops/s |
|                                                 Max Throughput |   category-price-stats |  20           |  ops/s |
|                                        50th percentile latency |   category-price-stats |   3.78785     |     ms |
|                                        90th percentile latency |   category-price-stats |   6.87629     |     ms |
|                                        99th percentile latency |   category-price-stats |  24.4242      |     ms |
|                                       100th percentile latency |   category-price-stats |  45.107       |     ms |
|                                   50th percentile service time |   category-price-stats |   3.78785     |     ms |
|                                   90th percentile service time |   category-price-stats |   6.87629     |     ms |
|                                   99th percentile service time |   category-price-stats |  24.4242      |     ms |
|                                  100th percentile service time |   category-price-stats |  45.107       |     ms |
|                                                     error rate |   category-price-stats |   0           |      % |


--------------------------------
[INFO] SUCCESS (took 62 seconds)
--------------------------------

````


#### Run Nested

````bash

docker compose run esrally race \
--track-path=/rally/tracks/ecommerce_nested \
--target-hosts=es01:9200 \
--pipeline=benchmark-only \
--challenge=search-benchmark \
--client-options="basic_auth_user:elastic,basic_auth_password:{ELASTIC_PASSWORD}"

````


###


### Result

````bash

   ____        ____
   / __ \____ _/ / /_  __
  / /_/ / __ `/ / / / / /
 / _, _/ /_/ / / / /_/ /
/_/ |_|\__,_/_/_/\__, /
                /____/

[INFO] Race id is [fa27f422-edc3-460a-85f9-481b9ea36fa3]
[INFO] Racing on track [ecommerce_nested], challenge [search-benchmark] and car ['external'] with version [8.15.2].

Running match-all                                                              [100% done]
Running mens-clothing-search                                                   [100% done]
Running womens-clothing-search                                                 [100% done]
Running category-price-stats                                                   [100% done]
Running nested-product-search                                                  [100% done]

------------------------------------------------------
    _______             __   _____
   / ____(_)___  ____ _/ /  / ___/_________  ________
  / /_  / / __ \/ __ `/ /   \__ \/ ___/ __ \/ ___/ _ \
 / __/ / / / / / /_/ / /   ___/ / /__/ /_/ / /  /  __/
/_/   /_/_/ /_/\__,_/_/   /____/\___/\____/_/   \___/
------------------------------------------------------

|                                                         Metric |                   Task |          Value |   Unit |
|---------------------------------------------------------------:|-----------------------:|---------------:|-------:|
|                     Cumulative indexing time of primary shards |                        |    0           |    min |
|             Min cumulative indexing time across primary shards |                        |    0           |    min |
|          Median cumulative indexing time across primary shards |                        |    0           |    min |
|             Max cumulative indexing time across primary shards |                        |    0           |    min |
|            Cumulative indexing throttle time of primary shards |                        |    0           |    min |
|    Min cumulative indexing throttle time across primary shards |                        |    0           |    min |
| Median cumulative indexing throttle time across primary shards |                        |    0           |    min |
|    Max cumulative indexing throttle time across primary shards |                        |    0           |    min |
|                        Cumulative merge time of primary shards |                        |    0           |    min |
|                       Cumulative merge count of primary shards |                        |    0           |        |
|                Min cumulative merge time across primary shards |                        |    0           |    min |
|             Median cumulative merge time across primary shards |                        |    0           |    min |
|                Max cumulative merge time across primary shards |                        |    0           |    min |
|               Cumulative merge throttle time of primary shards |                        |    0           |    min |
|       Min cumulative merge throttle time across primary shards |                        |    0           |    min |
|    Median cumulative merge throttle time across primary shards |                        |    0           |    min |
|       Max cumulative merge throttle time across primary shards |                        |    0           |    min |
|                      Cumulative refresh time of primary shards |                        |    0.00261667  |    min |
|                     Cumulative refresh count of primary shards |                        |   64           |        |
|              Min cumulative refresh time across primary shards |                        |    0           |    min |
|           Median cumulative refresh time across primary shards |                        |    0           |    min |
|              Max cumulative refresh time across primary shards |                        |    0.00108333  |    min |
|                        Cumulative flush time of primary shards |                        |    0.0157667   |    min |
|                       Cumulative flush count of primary shards |                        |   16           |        |
|                Min cumulative flush time across primary shards |                        |    0.00015     |    min |
|             Median cumulative flush time across primary shards |                        |    0.000366667 |    min |
|                Max cumulative flush time across primary shards |                        |    0.00398333  |    min |
|                                        Total Young Gen GC time |                        |    1.372       |      s |
|                                       Total Young Gen GC count |                        |    5           |        |
|                                          Total Old Gen GC time |                        |    0           |      s |
|                                         Total Old Gen GC count |                        |    0           |        |
|                                                   Dataset size |                        |    0.00924436  |     GB |
|                                                     Store size |                        |    0.00924436  |     GB |
|                                                  Translog size |                        |    8.19564e-07 |     GB |
|                                         Heap used for segments |                        |    0           |     MB |
|                                       Heap used for doc values |                        |    0           |     MB |
|                                            Heap used for terms |                        |    0           |     MB |
|                                            Heap used for norms |                        |    0           |     MB |
|                                           Heap used for points |                        |    0           |     MB |
|                                    Heap used for stored fields |                        |    0           |     MB |
|                                                  Segment count |                        |    6           |        |
|                                    Total Ingest Pipeline count |                        |    0           |        |
|                                     Total Ingest Pipeline time |                        |    0           |      s |
|                                   Total Ingest Pipeline failed |                        |    0           |        |
|                                                 Min Throughput |              match-all |  393.68        |  ops/s |
|                                                Mean Throughput |              match-all |  393.68        |  ops/s |
|                                              Median Throughput |              match-all |  393.68        |  ops/s |
|                                                 Max Throughput |              match-all |  393.68        |  ops/s |
|                                        50th percentile latency |              match-all |    2.73399     |     ms |
|                                        90th percentile latency |              match-all |    3.68811     |     ms |
|                                        99th percentile latency |              match-all |   47.7983      |     ms |
|                                       100th percentile latency |              match-all |  339.933       |     ms |
|                                   50th percentile service time |              match-all |    2.73399     |     ms |
|                                   90th percentile service time |              match-all |    3.68811     |     ms |
|                                   99th percentile service time |              match-all |   47.7983      |     ms |
|                                  100th percentile service time |              match-all |  339.933       |     ms |
|                                                     error rate |              match-all |    0           |      % |
|                                                 Min Throughput |   mens-clothing-search |  220.71        |  ops/s |
|                                                Mean Throughput |   mens-clothing-search |  220.71        |  ops/s |
|                                              Median Throughput |   mens-clothing-search |  220.71        |  ops/s |
|                                                 Max Throughput |   mens-clothing-search |  220.71        |  ops/s |
|                                        50th percentile latency |   mens-clothing-search |    3.89353     |     ms |
|                                        90th percentile latency |   mens-clothing-search |    5.29933     |     ms |
|                                        99th percentile latency |   mens-clothing-search |   47.2285      |     ms |
|                                       100th percentile latency |   mens-clothing-search |  334.299       |     ms |
|                                   50th percentile service time |   mens-clothing-search |    3.89353     |     ms |
|                                   90th percentile service time |   mens-clothing-search |    5.29933     |     ms |
|                                   99th percentile service time |   mens-clothing-search |   47.2285      |     ms |
|                                  100th percentile service time |   mens-clothing-search |  334.299       |     ms |
|                                                     error rate |   mens-clothing-search |    0           |      % |
|                                                 Min Throughput | womens-clothing-search |   20.89        |  ops/s |
|                                                Mean Throughput | womens-clothing-search |   21.44        |  ops/s |
|                                              Median Throughput | womens-clothing-search |   20.94        |  ops/s |
|                                                 Max Throughput | womens-clothing-search |   22.49        |  ops/s |
|                                        50th percentile latency | womens-clothing-search |    3.95728     |     ms |
|                                        90th percentile latency | womens-clothing-search |   10.8358      |     ms |
|                                        99th percentile latency | womens-clothing-search |  849.036       |     ms |
|                                       100th percentile latency | womens-clothing-search | 7208.2         |     ms |
|                                   50th percentile service time | womens-clothing-search |    3.95728     |     ms |
|                                   90th percentile service time | womens-clothing-search |   10.8358      |     ms |
|                                   99th percentile service time | womens-clothing-search |  849.036       |     ms |
|                                  100th percentile service time | womens-clothing-search | 7208.2         |     ms |
|                                                     error rate | womens-clothing-search |    0           |      % |
|                                                 Min Throughput |   category-price-stats |   75.93        |  ops/s |
|                                                Mean Throughput |   category-price-stats |   75.93        |  ops/s |
|                                              Median Throughput |   category-price-stats |   75.93        |  ops/s |
|                                                 Max Throughput |   category-price-stats |   75.93        |  ops/s |
|                                        50th percentile latency |   category-price-stats |    2.30753     |     ms |
|                                        90th percentile latency |   category-price-stats |    3.82214     |     ms |
|                                        99th percentile latency |   category-price-stats |  244.567       |     ms |
|                                       100th percentile latency |   category-price-stats |  366.383       |     ms |
|                                   50th percentile service time |   category-price-stats |    2.30753     |     ms |
|                                   90th percentile service time |   category-price-stats |    3.82214     |     ms |
|                                   99th percentile service time |   category-price-stats |  244.567       |     ms |
|                                  100th percentile service time |   category-price-stats |  366.383       |     ms |
|                                                     error rate |   category-price-stats |    0           |      % |
|                                                 Min Throughput |  nested-product-search |  473.67        |  ops/s |
|                                                Mean Throughput |  nested-product-search |  473.67        |  ops/s |
|                                              Median Throughput |  nested-product-search |  473.67        |  ops/s |
|                                                 Max Throughput |  nested-product-search |  473.67        |  ops/s |
|                                        50th percentile latency |  nested-product-search |    3.37797     |     ms |
|                                        90th percentile latency |  nested-product-search |    4.15939     |     ms |
|                                        99th percentile latency |  nested-product-search |   10.9361      |     ms |
|                                       100th percentile latency |  nested-product-search |  335.415       |     ms |
|                                   50th percentile service time |  nested-product-search |    3.37797     |     ms |
|                                   90th percentile service time |  nested-product-search |    4.15939     |     ms |
|                                   99th percentile service time |  nested-product-search |   10.9361      |     ms |
|                                  100th percentile service time |  nested-product-search |  335.415       |     ms |
|                                                     error rate |  nested-product-search |    0           |      % |


--------------------------------
[INFO] SUCCESS (took 61 seconds)
--------------------------------

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