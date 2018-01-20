# This document collects requirements for Apache CarbonData

Hit Count:[![HitCount](http://hits.dwyl.io/jackylk/jackylk/carbon-requested-feature.svg)](http://hits.dwyl.io/jackylk/jackylk/carbon-requested-feature)



| Priority level | Feature                                  |
| -------------- | ---------------------------------------- |
| High           | Java API, S3 Table, Data exchange between S3 and HDFS, Advisor |
| Median         | Datamap to accelerate S3 table, Different kind of Datamap, Segment Status |
| Low            | Others                                   |

## Improving usability

1. Advisor for SORT_COLUMNS
2. Advisor for pre-aggregate table

## Typical Carbon Usage on Cloud

1. Support CarbonTable on S3
2. Support loading to local CarbonTable from CSV on S3
3. Support DataMap on local HDFS
4. Support Java API to write and read CarbonTable/CarbonFile on S3
5. Support adding external segment
   - Support external segment with specified path, file format, partition value
   - Support implementing static partition using segment. User can use ALTER TABLE ADD PARTITION to add existing files to a partition (segment)

## Support non-carbonfile in Carbon

1. Support pre-aggregate datamap for Parquet/ORC
2. Support compaction for Parquet/ORC



## Datamap to accelerate S3 table

1. Support datamap on HDFS and fact table on S3. The scenario is that, user can create external table on S3 and create datamap on HDFS in EC2, to improve query performance while keeping cost low. It can be used in following scenario.
   - Pre-agg table in HDFS, fact table in S3
   - Index in HDFS, fact table in S3
   - Cache table in HDFS, fact table in S3
   - Datamap in HDFS, fact table using parquet in S3
2. Support a new type of datamap: table cache. It is a subset data of the fact table, route to it when query filter hit this subset, like time range. Datamap option should have: *event_time* to specify event time column, *expiration* to specify cache expiration time. 
3. Support assigning location when creating datamap.
4. Support creating datamap on non-carbon table
5. For batch load, support load datamap and fact table synchroniously
6. For streaming ingest, support ingest into cache datamap first then save to both datamap and fact table when handoff happen.

Example Usage:

```
// create a fact table stored in S3
CREATE TABLE fact(c1 int, c2 string) 
STORED BY 'carbondata'   // can be others
LOCATION 's3a://xx'

// create a cache table on fact table 
CREATE DATAMAP fact_cache ON TABLE fact 
USING 'cache'
LOCATION 'hdfs://yy'
DMPROPERTIES (event_time'='c2', 'expiration'='1day')

// create a preaggregate table on fact table 
CREATE DATAMAP fact_agg ON TABLE fact 
USING 'preaggregate'
LOCATION 'hdfs://yy'
AS SELECT c2, sum(c1) FROM fact GROUP BY c2

// create an text index on fact table 
CREATE DATAMAP fact_index ON TABLE fact 
USING 'lucene_index'
LOCATION 'hdfs://yy'
DMPROPERTIES ('index_column'='c2')

```

## Differnet kind of Datamap

1. Support lucene datamap for text index (it needs UDF pushdown to continue)
2. Support R tree datamap for geospatial analytic
3. Support cache table as datamap (repeated)

## File Level Input/Output

1. Support file level OuputFormat and spark/hive/presto integration, so that spark/hive/presto can write carbon as file as per other format like parquet/orc. This feature does not support dictionary.
2. Support file level InputFomat and spark/hive/presto integration. This feature can leverage index, but not support any feature needs optimizer enhancement like dictionary and datamap.
3. Support using file level integration in Hive partition related syntax. User can set to use carbon file format in hive partition

## Advisor

*TODO: add a diagram*

1. Support workload analyzer, which takes create table script and query script and stats as input.
2. Support advisor for CREATE TABLE (main index, dictionary, etc), CREATE DATAMAP (index, pre-agg, etc) 
3. Support evaluate the effect of advise output in a virtual environment by EXPLAIN command, like doing what-if analysis for virtual datamap
4. Support visualization of the workload to make it easier to tune
5. Support continuous monitor and collect workload plan to save in a store, like ES or JSON file in HDFS

## Graph

1. Support a new format (*carbongraph*) for adjacent matrix for high performance graph traversal.
2. Implement a RPC based distributed graph compute framework
3. Support prefetch of data to leverage the advantage of carbongraph's format
4. Integrate CarbonData with Apache Tinkerpop as a [**TinkerPop-enabled data system provider**](http://tinkerpop.apache.org/providers.html). Expose Gremlin language to user as counterpart of SQL in data warehouse domain.
5. Provide Graph SQL extension like [**Azure SQL**](https://blogs.technet.microsoft.com/dataplatforminsider/2017/04/20/graph-data-processing-with-sql-server-2017)

## Segment Status

1. Support segment interface and store segment related metadata in hive metastore
2. Support handle metadata correctly in all command, for cloud environment where data is in S3 and metadata in metastore.

## CarbonStore for higher performance

Since carbon has pre-agg now, many query can transform group by into point query or range query, to make it faster, we should optimize the performance for point query, for both single query and concurrent query

1. Implement a long running service based on YARN container or k8s/docker
2. Implement a RPC based execution engine for simple query: query with only projection and filter. After generating the physical plan, invoke the RPC and execute by CarbonStore process instead of toRDD and execute by spark executor.

## Timeseries Table

1. Support data temperature on cloud for S3, HDFS. CarbonStore automatically managed them according to the segment time range. Support data retention policy, so that old data is automatically spill to cooler storage, like 1 month in HDFS, others in S3.
2. In memory segment
3. Streaming pre-aggregate
4. Support pre-aggregate table loading in rollup manner to improve loading speed, like rollup to month table based on day table instead of fact table

## Streaming Table

1. Compaction: auto compact and close stream
2. Integrate with flink
3. Integrate with kafka-connect

## UDF for index column

1. Support MATCH filter UDF push down
   1. vector feature matching
   2. lucene index matching

## AI integration

1. Support feature vector in carbon/spark integration, as separate data file or datamap
2. Support similar functionality for DL/Graph

## Misc

1. merge index for global sort table
2. Support timestamp64 datatype for column that stores millisecond level timestamp
3. Make tempCSV default value as 'false' for DataframeWriter



## Image Content Search

1. CREATE TABLE on pictures, sound, binary data, etc. CREATE DATAMAP on these binary to extract metadata and build secondary index, so that user can search these pictures by metadata like time, place, camera type, etc.
2. CarbonStore should accept JPG picture as segment file format

