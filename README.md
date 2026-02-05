## Module 3 Homework: Data Warehousing & BigQuery

### BigQuery Setup

Create an external table using the Yellow Taxi Trip Records.

    CREATE OR REPLACE EXTERNAL TABLE `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_ext`
    OPTIONS (
        FORMAT = PARQUET, 
        URIS = ['gs://dez_hw3_2026/yellow_tripdata_2024-*.parquet']
    )
    ;

Create a (regular/materialized) table in BQ using the Yellow Taxi Trip Records (do not partition or cluster this table).

    CREATE OR REPLACE TABLE `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_materialized` AS 
    SELECT * 
    FROM `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_ext`
    ;

<b> Question 1. </b> Counting records: What is the count of records for the 2024 Yellow Taxi Data?

    Answer: 20,332,093

    SELECT COUNT(*) AS NoRecords
    FROM `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_materialized`
    ;

<b> Question 2. </b> Data read estimation: Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.

What is the estimated amount of data that will be read when this query is executed on the External Table and the Table?

    Answer: 0 MB for the External Table and 155.12 MB for the Materialized Table

    Processed data: 155.12 MB
    SELECT COUNT(*) AS PULocations_Unique
    FROM (SELECT DISTINCT(PULocationID)
        FROM `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_materialized`)
    ;

    Processed data: 0 MB
    SELECT COUNT(*) AS PULocations_Unique
    FROM (SELECT DISTINCT(PULocationID)
        FROM `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_ext`)
    ;

<b> Question 3. </b> Understanding columnar storage: Write a query to retrieve the PULocationID from the table (not the external table) in BigQuery. Now write a query to retrieve the PULocationID and DOLocationID on the same table.

Why are the estimated number of Bytes different?

    Answer: BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.

    Processed data: 155.12 MB
    SELECT PULocationID
    FROM `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_materialized`
    ;

    Processed data: 310.24 MB
    SELECT PULocationID,
        DOLocationID
    FROM `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_materialized`
    ;

<b> Question 4. </b> Counting zero fare trips: How many records have a fare_amount of 0?

    Answer: 8,333

    SELECT COUNT(*) AS fare_amount_0 
    FROM `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_materialized`
    WHERE fare_amount = 0
    ;

<b> Question 5. </b> Partitioning and clustering: What is the best strategy to make an optimized table in Big Query if your query will always filter based on tpep_dropoff_datetime and order the results by VendorID (Create a new table with this strategy)

    Answer: Partition by tpep_dropoff_datetime and Cluster on VendorID

    Strategy: Partition & Clustering. 

    CREATE OR REPLACE TABLE `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_partitioned_clustered`
    PARTITION BY
        DATE(tpep_dropoff_datetime)
    CLUSTER BY 
        VendorID  
    AS SELECT *
    FROM `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_ext`
    ;

    Processed data: 2.26 GB
    SELECT *
    FROM `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_partitioned_clustered`
    WHERE tpep_dropoff_datetime BETWEEN '2024-01-01' AND '2024-06-01'
    ORDER BY VendorID DESC NULLS LAST
    ;
    -----------------------
    Strategy: Clustering & Clustering.

    CREATE OR REPLACE TABLE `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_clustered_clustered`
    CLUSTER BY
        tpep_dropoff_datetime,
        VendorID  
    AS SELECT *
    FROM `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_ext`
    ;

    Processed data: 2.29 GB
    SELECT *
    FROM `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_clustered_clustered`
    WHERE tpep_dropoff_datetime BETWEEN '2024-01-01' AND '2024-06-01'
    ORDER BY VendorID DESC NULLS LAST
    ;

    The remaining strategies are not possible. 
    Cluster on tpep_dropoff_datetime Partition by VendorID: Its not possible to partition a column different of date format. 
    Partition by tpep_dropoff_datetime and Partition by VendorID: Partitions can only be use upon in a specific column table. 

<b> Question 6. </b> Partition benefits: Write a query to retrieve the distinct VendorIDs between tpep_dropoff_datetime 2024-03-01 and 2024-03-15 (inclusive)

Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 5 and note the estimated bytes processed. What are these values?

    Answer: 310.24 MB for non-partitioned table and 26.84 MB for the partitioned table

    Processed data: 310.24 MB
    SELECT DISTINCT(VendorID)
    FROM `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_materialized`
    WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2024-03-15'
    ;

    Processed data: 26.84 MB
    SELECT DISTINCT(VendorID)
    FROM `project-0c3c5223-416f-4242-b0f.dez_homework_3.yellow_tripdata_2024_1semester_partitioned_clustered`
    WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2024-03-15'
    ;

<b> Question 7. </b> External table storage: Where is the data stored in the External Table you created?. 

    Answer: GCP Bucket

<b> Question 8. </b> Clustering best practices: It is best practice in Big Query to always cluster your data.

    Answer: False. 
    
    Always its not recommend to cluster or partition our data for some reasons; One of them, only is suggested to do clustering/partitioning when the data table size stored exceed 1 GB, otherwise the reads of the metadata and the managment cost will not compensate the minimus improvements in performance that we could get. Another reason why its not recommended to do clustering, if we have a defined cost limit we cant measure in accurate way how much costs we will incur when processing the data; Also, its not suggested to use clustering if we only need to filter one specific column in our dataset, because its usefull when filtering multiple columns (up to 4).

<b> Question 9. </b> Understanding table scans: Write a SELECT count(*) query FROM the materialized table you created. How many bytes does it estimate will be read? Why?

    Answer: 0 MB

    The query will process 0 MB, because due the columnar storage used by Bigquery, with the aggregation function COUNT(*) we are not specifyng any column to be scanned,
    so when the query is executed the records count output is obtained from the metadata of the table without scanning other columns or additional info.  
    