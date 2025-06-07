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


youtube-sentiment-pipeline/ <br>
│ <br>
├── docker-compose.yml <br>
├── kafka/ <br>
│   └── producer.py  # YouTube API to Kafka <br>
│ <br>
├── spark/
│   └── spark_job.py  # NLP + ML processing
│
├── elastic/
│   └── elastic_setup.sh  # Optional Elasticsearch init
│
├── model/
│   └── sentiment_model.pkl  # Trained model (or use Hugging Face live)
│
├── dashboard/
│   └── kibana_config/
│
└── requirements.txt

