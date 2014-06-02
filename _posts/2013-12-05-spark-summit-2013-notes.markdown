---
layout: post
title: "Spark Summit 2013 notes"
date: 2013-05-12
categories: spark summit conference notes
---

# Keynote by Matei Zaharia
  
  * Spark is now one of the largest big data products out there surpassing hadoop in the number of active contributors in the last 6 months.
  * Has more contributors than Storm, Giraph, Dril and Tez combined.

## Spark 0.8.1 will have

  * MLlib - machine learning algorithms out of the box.
  * YARN 2.2 resource manager support
  * codahale metrics based metrics reporting and a new monitoring UI
  * Spark stream comes out of alpha and will be stabilized
  * Shark imporovements
  * EC2 support

## Other contributions

  * YARN support - Yahoo
  * Columnar compression of data - Yahoo
  * Metrics reporting - Quantifind
  * Fair scheduling and code generation optimization - Intel
  * New RDD operators
  * scala 2.10 support  (finally!)
  * Master HA
  * External hashing and sorting (0.9 release)
  * Better support for large number of tasks


## Current priorities
  * Standardise libraries
  * Deployment ease/automation
  * Out of the box usability - better defaults and configuration system
  * Enterprise support provided by Cloudera & Databrix

Spark Streaming and Shark will be optimised for 24/7 operation and will get other performance imporvements

SIMR - Will be released soon: Provides a seamless way to run spark as a regular hadoop job (it's not production ready just yet)
  * Recommended to upgrade to hadoop 2.2

In terms of raw performance Spark/Shark is comparable to Apache Impala

Time spent is generally as follows

90% time is spent just reading and writing in Hadoop

Int spark and other in memory systems, the data is loaded just once and all subsequent operations are performed on it

What happens when data does not fit in memory?
  * Spark manages swapping data as it is needed
Scale up vs scale out?
  *Depends on application and requires looking at the workload but given the ability to use spot instances, it is preferable to scale out and increase parallelism. 


* Value from Data

**Questions:**
     * Insights, diagnosis
          - Why is X happening?
     * Decision support
          - How do I do X?
          - What do I do to make X better?

**Solutions:**
     * Interactive queries
     * Streaming queries
     * Sophisticated data processing (in time)


**Challenge #1**: need to maintain 3 stacks - expensive and difficult to make consistent

**Challenge #2**: hard/slow to share data and code

Spark helps in both areas by allowing for the same effort to be utilised for both batch and streaming with minimal change.

**Analogy:**

  Unification of technologies

    * Step 1: first cellular phone                        --- hadoop/batch processing
    * Step 2: specialized devices (mp3 player, gps, pda)  --- storm, mahout, specialised systems
    * Step 3: unification (iphone or smartphones)         --- spark (others also there but not as mature)

* Unified realtime and historical data analysis
* Unified streaming, historical & predictive analysis.


Why streaming ML?
  * fraud detection
  * decision support
  * trend change
  * no point in suggesting a product a day later

Enterprise support by
  * Databrix
  * Cloudera
  * Wandisco
  * Tuplejump

BlinkDB will soon go out of alpha and reach stability
  * Better HIVEQL operator support
  * Performance improvements
  * Interesting numbers on conviva datasets

CrowdDB - SQL to Crowdsourcing answers (SIGMOD 2011)


# Hadoop and Spark join forces at Yahoo

Pre-2012 - Editors' decisions drive the yahoo homepage

2012 - data driven editorial decisions for homepage

  - ML driven suggestions
  - Per user customisation
  - instantly saw 300% increase in engagement

2013 - Personalised homepage

  - All parts are personalised, tracked, trained
  - Personalised mobile experience
  - Personalised properties = websearch + vertical content, deep search -> video, maps, weather when you search

Data Science @ scale requires ability to do quick turnaround discovery

  - Challenge #1: Science

    - Single model for all items in homepage stream
    - millions of items 
    - thousands of item/user features
      - categories
      - wikipedia entity names
      - Objective function
        - Relevance / User engagement
        - Freshness / popularity
        - Diversity
        - Algorithm exploration
          - Logistic regression
          - Collaborative filtering
          - Decision trees
          - Hybrid

  - Challenge#2: Speed

    - Freshness is very important
      - Surface relevant results while they are still relevant

  - Challenge #3: Scale
    - 150 PB of data on hadoop
      - data for model building and BI analytics
      - Avoid latency
      - hours of latency is not acceptable
      - 35,000 baremetal hadoop cluster
      - Store all data on a giant netapp based nfs store
      - different log sources continuously ship data

**Solution:**
  - Hadoop + Spark
  - Hadoop is and will be the core for doing most of the heavy lifting
  - Spark for iterative research and queries of most of the processed data
  - Both running in the same cluster
  - YARN brings hadoop datasets and servers at scientists' disposal

Projects that are using spark:
  - Collaborative filtering
  - Ecommerce 
    - viewed-also-viewed
    - bought-also-bought
    - bought-after-bought
  
* Huge speedup = 14 minutes vs 106 minutes
* Stream Ads
* Logistic regression algorithm implements
  - 120 LOC in scala/spark - much easier to manage vowpal/wabbi
  * 30 mins for 100M samples with 13K features and 30 iterations

**0 to production experiments - 2 hours turnaround - Big win for experimentation and hypothesis validation.**

Slides have diagrams of the architecture used by Yahoo teams
- Standalone mode
  - YARN manages master and workers
- Client mode
  - YARN manages only workers

* Future contributions
  - Dynamic resource allocation
  - Generic history server
  - Preemption

Old Architecture
  
  * Logs collection to huge NFS storage
  * Logs moved to hdfs
  * PIG/MR ETL processes - massive joins
  * Aggregations and pre-determined reports
  * Load data into DBMS
  * UI/API on top

**Problems:**
  - massive data volumes - many TBs
  - Pure hadoop throughput
  - Report latency is high
  - Culprit:
    - Row data processing through MR is slow (IO)
  - Many chained stages - IO 
  - massive joins
  - lack of interactive dsl
  - expressibility of business logic as MR is difficult

Aggregates pre-computation problems
  - Pre-calculated reports
  - counting distincts
  - Number of reports along dimensions
    - datacubes are just huge

**hadoop is not built for this**
  - No way to do data discovery
  - need a data workbench for BI

Most spark clusters are "small" and hand managed
  - 9.2 TB of addressable RAM
  - 96GB/192GB RAM per machine
  - Tableau interface to shark (OLAP)
  - overlap analysis
  - time series analysis
  - construct cubes offline and use them for fast analysis (MOLAP and HOLAP)
  - column pruning (in shark)
  - map side joins (in shark)
  - cached table compression (in shark)

* Satellite cluster pattern or per-team cluster on the same underlying data

# Making Spark Fly: Creating an elastic Spark cluster on Amazon EMR
  * EMR integrates directly with all other amazon services seamlessly (*kinesis)
  * Aggressively use spot instances to get all the processing done with the excess capacity when prices drop.
  * Task nodes can be added and removed whereas core nodes can only be added.
    * Use task nodes as spot instances (scalable workers) and use core nodes for a basic set of available workers at all times
  * Spark installed as a part of a bootstrap script
  * 1 TB memory cluster using spot prices can be very cheap.
    - 63 x m1.xlarge = $4.44/hr vs $30/hr on demand
    - 6 x  cc2.8xlarge = $4.64/hr
    - 15 x m2.4xlarge - $2.25/hr
  * Use ssd+ram for extra credit
  * Autoscaling spark
    * Using cloudwatch metrics
      - define metrics as 
      - CPU and memory (probably scale up/down on both)
    * Pick a good set of thresholds for "TotalLoad"
    * Spark 0.8.1 will provide cluster metrics that can also be used for scaling
    * Look up CloudwatchMetricsSink
    * When new nodes join the cluster the data is not eagerly rebalanced, but if tasks would get scheduled to them and they will eventually get used.
  * Kinesis integration with sparkstreaming
    * Kinesis is just like kafka
    * Upcoming KinesisReceiver using NetworkReceivers ( where to get this?)
  * C3 and i2 instance will be awesome!
  * Use placement groups and fetch data from S3 to local hdfs when working with it.

# SIMR
  * Wraps spark as a normal hadoop job so that you can run it on an existing cluster

# FLINT - Deploying BDAs on AWS (Adobe)
  * Shared nothing management
  * Use simpleDB and S3 to manage all state
  * Efficient and scalable 
  * Access to all tools
  * Pending open source

# ADATAO - R and Python with NLP over spark
  - Use R and Python with spark
  - Seamlessly translates R to spark
  - Use Cases
    - BI
    - Adhoc business query
    - Sensor network analytics
    - Ad networks
    - MLLib + Rest API

# TupleJump
  - Ubercube - distributed olap cube over spark and cassandra
  - Indexes for cassandra
  - Analytics s/w using spark
  - SnapFS
  - Calliope
    - R/W data with cassandra
    - Shark + calliope 
      - builtin indexing (clustered)
    - Stargate
    - Hydra: Common bus for messages 
    - Ops center

#  Realtime analytics processing - Intel 
  - Alibaba, youku, baidu
  - Value ?
      - descriptive analytics - SQL
      - predictive analytics - non-SQL
      - interactive
      - streaming/online
  - A series of mini batch jobs that flush aggregations to rdbms at regular intervals
  - colocate kafka and a spark worker
    - log collection is bottlenecked by n/w
    - processing is bottlenecked by cpu and mem
      - use memory_only_ser2
        - tune spark.cleaner.ttl [ throughput*spark.cleaner.ttl < memory ]
    - Don't be scared to add columns, let shark worry about that
    - Complex machine learning
      - mostly matrix/graph analysis

# Sparrow - Next-Gen spark scheduling
  - Current scheduler is not so good when jobs are very short
    - most of the time spent waiting in queue to be assigned to a worker.
  - Current scheduler bottlenecked at about 1500 tasks/sec
    - bad for big clusters with small tasks
    - Scheduler delay gives an estimate of amount of time spent in scheduler queue
    - Use: Batch sampling + Late binding
    - https://www.github.com/radlab/sparrow

# Spark as a Service - OOYALA
  - raw events in cassandra
  - spark job server
  - anyone can submit a job
  - available as a service to avoid rebuilding the universe over and over
    - REST API
      - contexts
        - allows the user to create a context and keep it alive for later use
        - low latency query now possible
        - submit subsequent querries with `&context=<id>` to run in an existing context
        - async and sync requests also possible
        - challenges/lessons
          - spark is based on contexts
          - we need a manager ( as contexts may take multiple seconds to come up and allocate lots of threads)
  - Opensource and Currently a pull request

# Spark streaming
  - Many environments require processing same data in live and as batch
    - no single framework does this
    - Traditionally, stateful event processing
      - each node has mutable state that is synchronized regularly with others
      - prone to loss of data when node is lost
  - Storm 
    - atleast once
    - gets slow
    - Trident - adds transactions by using external db (slows down further and adds external dependency/hell)
  - Built on RDDs
    - divides stream into small deterministic batches (up to 0.5 sec each) with lineage
    - code looks and feels same as the batch system code
  - Window based transformations
    - window ( length of window, frequency)
    - updateByKey( updateFunction, Key)
    - transform
      - allows us to combine historical/regular RDDs with streaming 
     - Applications
        - online ML
    - combine live and historical data
    - CEP style processing (http://en.wikipedia.org/wiki/Event-driven_architecture)
    - Data sources
      - kafk, hdfs, flume, akka actors, raw tcp
      - easy to write new receiver
    - Fault tolerance
      - Batches of input data are replicated
      - data lost due to worker is recomputed from lineage
        - how long is the data kept? - uses LRU (0.9 - active throwaway)
        - spark 0.9 will have
        - automated master failure recovery
        - perf improvement
        - better monitoring/UI for streaming specifically
    - Long term goals
      - MLlib for streaming
      - shark for streaming
      - python api

Slides: [http://spark-summit.org/agenda/](http://spark-summit.org/agenda/)

Hand notes: [Flickr](http://www.flickr.com/photos/ankurc/sets/72157638371848076/)


**Note 1:** As it goes without saying, all the information in this post is what **I** understood during the conference and whatever i could remember while writing stuff down.
There is a possibility that I got something wrong, if that is the case please let me know and I'll be happy to make the changes.
