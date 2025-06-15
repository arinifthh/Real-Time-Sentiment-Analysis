# Real-Time-Sentiment-Analysis

YouTube API → Kafka Producer → Kafka Topic → Spark Streaming Consumer → NLP Sentiment Model → Elasticsearch/Druid → Dashboard (e.g., Kibana or Superset)

## Step by step

1. Extract YouTube comments using the YouTube Data API v3.
   - try extract 100k then tya Dr. how many records ?
  
🔁 STREAMING Workflow (Real-Time):
If you're handling live or continuous data, like live Twitter/YouTube/Kafka stream:

✅ Recommended Order:
Extract (from API)
→ e.g., via Tweepy (Twitter) or Kafka Producer (YouTube comments stream)

Load into Kafka Topic
→ Raw, possibly noisy text is streamed into Kafka
→ Create kafka topic 

Kafka Consumer with PySpark Streaming
→ Read data in real-time from Kafka

Clean using PySpark Streaming logic
→ Tokenization, stopword removal, etc., all in-stream

Apply ML Model
→ Sentiment classification using pre-trained model (e.g., Hugging Face or scikit-learn)

Send to Elasticsearch
→ Store results for analytics and visualization in Kibana/Superset

Sure! Here's a complete **step-by-step guide** to run your **Kafka + PySpark streaming job without Docker**, on your **local machine**.



## Details Step by Step 

### 1. **Install Java**

Apache Spark needs Java (JDK). Install:

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
```

### 2. **Install Apache Spark**

* Download from: [https://spark.apache.org/downloads](https://spark.apache.org/downloads)

  * Choose Spark version: `3.5.1` (recommended)
  * Package type: Pre-built for Apache Hadoop `3`

* Extract and set environment variables:

```bash
export SPARK_HOME=~/spark-3.5.1-bin-hadoop3
export PATH=$SPARK_HOME/bin:$PATH
```

Add these lines to `~/.bashrc` or `~/.zshrc`.

✅ Test: `spark-submit --version`

---

### 3. **Install Kafka (Local)**

* Download from: [https://kafka.apache.org/downloads](https://kafka.apache.org/downloads)

* Extract, then run:

```bash
# Start Zookeeper (1st terminal)
bin/zookeeper-server-start.sh config/zookeeper.properties

# Start Kafka (2nd terminal)
bin/kafka-server-start.sh config/server.properties
```

✅ Test: Create a topic

```bash
bin/kafka-topics.sh --create --topic youtube_sentiment --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

---

### 4. **Prepare Python Environment**

Install dependencies:

```bash
pip install pyspark kafka-python
```

---

## 🧠 Your PySpark Kafka Script: `kafka-spark.py`

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, from_json
from pyspark.sql.types import StructType, StringType

schema = StructType() \
    .add("text", StringType()) \
    .add("sentiment", StringType())

spark = SparkSession.builder \
    .appName("YouTubeSentimentKafkaConsumer") \
    .getOrCreate()

spark.sparkContext.setLogLevel("WARN")

df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "youtube_sentiment") \
    .load()

json_df = df.selectExpr("CAST(value AS STRING)") \
    .select(from_json(col("value"), schema).alias("data")) \
    .select("data.*")

query = json_df.writeStream \
    .format("console") \
    .outputMode("append") \
    .start()

query.awaitTermination()
```

---

## 🚀 Run Spark with Kafka Integration

```bash
spark-submit \
--packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.1 \
kafka-spark.py
```

---

## 🛰️ Send Data from Producer (Python)

```python
from kafka import KafkaProducer
import json
import time

producer = KafkaProducer(
    bootstrap_servers='localhost:9092',
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

messages = [
    {"text": "I love this!", "sentiment": "positive"},
    {"text": "This is bad", "sentiment": "negative"},
    {"text": "It's okay", "sentiment": "neutral"}
]

for msg in messages:
    producer.send('youtube_sentiment', value=msg)
    print(f"Sent: {msg}")
    time.sleep(1)

producer.flush()
```

Save as `producer.py` and run:

```bash
python producer.py
```

---

## ✅ If Everything Works

You’ll see the JSON records printed to the console by `kafka-spark.py`.

Great initiative! Here's an improved and more **detailed architecture flow (No Docker)**, with **clarity on where the Kafka Consumer fits in**, especially for **manual cleaning or debugging**:

---

## 📊 Full Architecture Flow 

### 1. 📝 `producer.py` – Data Ingestion

* Reads the `youtube_comments.csv` file (or real-time API in future).
* Converts each row to JSON.
* Sends the data to **Kafka topic: `youtube_sentiment`**.
* ✅ You are doing this step successfully now.

---

### 2. 📡 Apache Kafka – Message Broker

* Acts as a **buffer and pipeline**.
* Stores and streams data from **producers** to **consumers**.
* Topic: `youtube_sentiment`
* Kafka ensures **high-throughput**, **fault-tolerant** delivery of real-time messages.

---

### 3. 🧪 `consumer_cleaner.py` – (Optional) Kafka Consumer for Pre-Cleaning or Monitoring

> 🧼 **This script is optional** and mostly for:

* Debugging: See what's coming in real-time.
* Cleaning only: Preprocess before handing to Spark.
* Educational purpose: Show raw → cleaned data.

If you're using **PySpark to do all transformations**, this consumer isn't required in production.

✅ This fits **between Step 2 and 4** if you need manual inspection.

---

### 4. ⚡ PySpark – Real-Time Data Processing

```python
spark.readStream \
  .format("kafka") \
  .option("subscribe", "youtube_sentiment") \
```

* Spark reads **Kafka topic: `youtube_sentiment`**.
* Steps handled:

  * Deserialize the JSON
  * 🔡 Clean text (lowercase, remove stopwords, etc.)
  * 🤖 Optionally reclassify sentiment using an ML model
* Spark DataFrame becomes real-time mini-pipeline.

---

### 5. 📤 Spark Output to Elasticsearch

* Transformed + classified records are sent to:

  * **Elasticsearch (via REST or Spark connector)**.
  * Index: `youtube_cleaned_comments` or similar.

---

### 6. 📈 Kibana – Sentiment Visualization Dashboard

* Kibana connects to Elasticsearch.
* Visualizes:

  * 🔼 Positive/Negative/Neutral over time (line graph)
  * 📌 Sentiment by keyword/topic
  * 🔍 Filter by hour/date/keyword
* You’ll create time-based aggregations on `published_at` or `processed_at`.
