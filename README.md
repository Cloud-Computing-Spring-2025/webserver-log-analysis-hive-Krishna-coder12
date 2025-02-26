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

### 3. **Creating a Database**

Creating a database in the hive environment:

```bash
CREATE DATABASE web_logs;
USE web_logs;
```

### 4. **Creating Hive Table**

Define an External Table for Web Logs:

```bash
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

```bash
LOAD DATA INPATH '/user/hive/web_logs/web_server_logs.csv' INTO TABLE web_logs;
```

### 6. **Execute Analysis Queries**

1) Total Web Requests:
```bash
SELECT COUNT(*) AS total_requests FROM web_server_logs;
```

2) Status Code Analysis:
```bash
SELECT status, COUNT(*) AS count FROM web_server_logs GROUP BY status;
```

3) Most Visited Pages:
```bash
SELECT url, COUNT(*) AS visits 
FROM web_server_logs 
GROUP BY url 
ORDER BY visits DESC;
```

4) Traffic Source Analysis:
```bash
SELECT user_agent, COUNT(*) AS count 
FROM web_server_logs 
GROUP BY user_agent 
ORDER BY count DESC;
```

5) Suspicious IP Addresses:
```bash
SELECT ip, COUNT(*) AS failed_requests 
FROM web_server_logs 
WHERE status IN (404, 500) 
GROUP BY ip 
HAVING COUNT(*) > 3;
```

6) Traffic Trend Over Time:
```bash
SELECT SUBSTR(`timestamp`, 1, 16) AS minute, COUNT(*) AS request_count 
FROM web_server_logs 
GROUP BY SUBSTR(`timestamp`, 1, 16) 
ORDER BY minute;
```


### 7. **Implement Partitioning for Performance Optimization**

Created Partitioned Table:

```bash
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

```bash
SET hive.exec.dynamic.partition.mode=non-strict;
INSERT INTO TABLE web_logs_partitioned PARTITION (status)
SELECT ip, timestamp, url, user_agent, status FROM web_server_logs;
```

### 8. **Execute the MapReduce Job**

Run your MapReduce job using the following command:

```bash
hadoop jar /opt/hadoop-3.2.1/share/hadoop/mapreduce/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar.jar com.example.controller.Controller /input/dataset/input.txt /output
```

### 9. **View the Output**

To view the output of your MapReduce job, use:

```bash
hadoop fs -cat /output/*
```

### 10. **Copy Output from HDFS to Local OS**

To copy the output from HDFS to your local machine:

1. Use the following command to copy from HDFS:
    ```bash
    hdfs dfs -get /output /opt/hadoop-3.2.1/share/hadoop/mapreduce/
    ```

2. use Docker to copy from the container to your local machine:
   ```bash
   exit 
   ```
    ```bash
    docker cp resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/output/ shared-folder/output/
    ```
3. Commit and push to your repo so that we can able to see your output
