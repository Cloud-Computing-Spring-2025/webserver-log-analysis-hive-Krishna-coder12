# Web-Server-Log-Analysis

Your task is to analyze the web server logs using Apache Hive and perform the following tasks:

Count Total Web Requests: Calculate the total number of requests processed.
Analyze Status Codes: Determine the frequency of HTTP status codes (e.g., 200, 404, 500).
Identify Most Visited Pages: Extract the top 3 most visited URLs.
Traffic Source Analysis: Identify the most common user agents (browsers).
Detect Suspicious Activity: Identify IP addresses with more than 3 failed requests (status 404 or 500).
Analyze Traffic Trends: Calculate the number of requests per minute to observe traffic patterns.
Implement Partitioning: Use partitioning by status code to optimize query performance.

## Setup and Execution

### 1.**Start the Hadoop Cluster**

Run the following command to start the Hadoop cluster:

```bash
docker compose up -d
```

### 2. **Setup Hive Environment**

```bash
docker exec -it hive-server /bin/bash
hive
```

### 3. **Creating a Database**

Creating a database in the hive environment:

```sql
CREATE DATABASE web_logs;
USE web_logs;
```

### 4. **Creating Hive Table**

Define an External Table for Web Logs:

```sql
CREATE EXTERNAL TABLE web_server_logs (
    ip STRING,
    `timestamp` STRING,
    url STRING,
    status INT,
    user_agent STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/hive/web_logs/';
```

### 5. **Loaded the csv file manually in the hive**

Downloaded the CSV file and uploaded manually in the hive in the following path '/user/hive/web_logs/web_server_logs.csv'.
And Run the following command in the Query Tab.

```sql
LOAD DATA INPATH '/user/hive/web_logs/web_server_logs.csv' INTO TABLE web_logs;
```

### 6. **Execute Analysis Queries**

1) Total Web Requests:
   
```sql
SELECT COUNT(*) AS total_requests FROM web_server_logs;
```

2) Status Code Analysis:
   
```sql
SELECT status, COUNT(*) AS count FROM web_server_logs GROUP BY status;
```

3) Most Visited Pages:
   
```sql
SELECT url, COUNT(*) AS visits 
FROM web_server_logs 
GROUP BY url 
ORDER BY visits DESC;
```

4) Traffic Source Analysis:
   
```sql
SELECT user_agent, COUNT(*) AS count 
FROM web_server_logs 
GROUP BY user_agent 
ORDER BY count DESC;
```

5) Suspicious IP Addresses:
   
```sql
SELECT ip, COUNT(*) AS failed_requests 
FROM web_server_logs 
WHERE status IN (404, 500) 
GROUP BY ip 
HAVING COUNT(*) > 3;
```

6) Traffic Trend Over Time:
   
```sql
SELECT SUBSTR(`timestamp`, 1, 16) AS minute, COUNT(*) AS request_count 
FROM web_server_logs 
GROUP BY SUBSTR(`timestamp`, 1, 16) 
ORDER BY minute;
```


### 7. **Implement Partitioning for Performance Optimization**

Created Partitioned Table:

```sql
CREATE TABLE web_logs_partitioned (
    ip STRING,
    `timestamp` STRING,
    url STRING,
    user_agent STRING
)
PARTITIONED BY (status INT)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
```

Load Data into Partitioned Table:

```sql
SET hive.exec.dynamic.partition.mode=non-strict;
INSERT INTO TABLE web_logs_partitioned PARTITION (status)
SELECT ip, timestamp, url, user_agent, status FROM web_server_logs;
```

Verify Partitioning:

```sql
SHOW PARTITIONS web_logs_partitioned;
```

### 8. **Exported Query Results**

Save Output to HDFS:

```sql
INSERT OVERWRITE DIRECTORY '/user/hive/output/Total_requests'
SELECT COUNT(*) FROM web_server_logs;
```
And perform similarly for all the Query Analysis .

### 9. **Moving data from hive-server**

Access the Hive Server  container:

```bash
docker exec -it hive-server /bin/bash
```

### 10. **Copy Output from HDFS to Local OS**

To copy the output from HDFS to your local machine:

1. Use the following command to copy from HDFS:
    ```bash
    hdfs dfs -get /user/hive/output /tmp/output
    ```

2. use Docker to copy from the container to your local machine:
   ```bash
   exit 
   ```
    ```bash
    docker cp hive-server:/tmp/output /workspaces/webserver-log-analysis-hive-Krishna-coder12/
    ```
3. Commit and push to your repo to the github.

### 11. **Challenges Faced **

1.Configuring the Hadoop Cluster: Setting up and initializing the Hadoop cluster within a Docker environment was initially challenging, requiring extensive debugging and fine-tuning.

2.Managing Large Log Files: Processing vast amounts of log data resulted in memory and performance issues, necessitating optimization strategies to enhance efficiency.

3.Optimizing Queries: Ensuring the efficient execution of Hive queries, especially when dealing with partitioning and aggregations, required careful tuning and adjustments.

4.Implementing Partitioning Strategies: Effectively applying dynamic partitioning while maintaining proper partition placement proved to be a complex task.

5.Data Cleaning and Formatting: Addressing inconsistencies in log files, such as missing or malformed entries, added an additional layer of preprocessing work.

6.Extracting Meaningful Insights: Refining queries and analysis techniques iteratively was essential to derive accurate and valuable insights from raw log data.


### Output Expected **
1) Total Web Requests:
   
```sql

```
