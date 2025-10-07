# Real-Time Anomaly Detection with Streaming Data (MVP)
>     “The minimum viable product is that version of a new product which allows a team to collect the maximum amount of validated learning about customers with the least effort.”

    ERIC RIES

## 1. Problem Statement
Modern systems (websites, sensors, spacecraft, finance, etc.) generate continuous data streams.  
Hidden inside these streams are **anomalies**: unusual spikes, drops, or patterns that could signal errors, attacks, or important events.

Goal: **detect anomalies in real-time** as data arrives, rather than after the fact.

We use the **Yahoo S5 dataset**, a well-known benchmark of time series with labeled anomalies (like sudden traffic spikes or dips).

---

## 2. Solution
Design: a **simple, streaming anomaly detection pipeline**:

1. **Replay historical data as a stream** (Yahoo S5 CSVs → Kafka topic).
2. **Detection logic** (e.g., rolling statistics, Median Absolute Deviation, Isolation Forest) runs on each new data point.
3. **Flag anomalies in real time** and publish them to another Kafka topic.
4. **Store anomalies in PostgreSQL** for later querying and analysis.
5. **Visualize results** using a lightweight dashboard (Streamlit or Grafana).

This MVP focuses on getting a working end-to-end system quickly.  
Later, we can extend it with Spark jobs, advanced ML models, drift monitoring, and cloud deployment.

---

### 2.1. Technologies
- **Kafka** → message broker to simulate streaming data
- **Python** → anomaly detection logic
- **PostgreSQL** → store flagged anomalies
- **Streamlit / Grafana** → simple dashboards
- (Future: Spark Structured Streaming, MLflow for model tracking)

---

### 2.2. Diagram
```
[Yahoo S5 CSVs] --> [Kafka Producer] --> [Kafka Topic: telemetry.raw]
                                         |
                                         v
                            [Detector Consumer: MAD/Z-score/IForest]
                                         |
                                         v
                          [Kafka Topic: telemetry.anomalies] --> [PostgreSQL]
                                                                    |
                                                                    v
                                                         [Dashboard: Streamlit/Grafana]
```

---

## 3. Directory Layout
```
anomaly-detector/
├─ configs/           # YAML config files (detector thresholds, dataset paths, broker info)
├─ data/              # Local sample data (Yahoo S5 CSVs)
│  ├─ raw/
│  └─ processed/
├─ docs/              # Documentation, diagrams, screenshots
├─ notebooks/         # Jupyter notebooks for EDA and prototyping
├─ scripts/           # Utilities (download dataset, replay to Kafka, eval metrics)
├─ src/               # Source code
│  ├─ common/         # IO, schema, utilities
│  ├─ detector/       # Algorithms: MAD, Z-score, Isolation Forest
│  ├─ pipeline/       # Kafka consumer/producer, Postgres sink
│  └─ api/            # Optional FastAPI service for on-demand scoring
├─ tests/             # Unit and integration tests
├─ infra/             # Dockerfiles, docker-compose, infra configs
│  ├─ docker/
│  └─ grafana/
└─ .github/workflows/ # CI pipelines
```

---

## 4. Steps to Replicate (MVP)

1. **Clone the repo**  
   ```bash
   git clone <repo-url>
   cd anomaly-detector
   ```

2. **Set up environment**  
   - Install Python 3.10+  
   - Install dependencies:  
     ```bash
     pip install -r requirements.txt
     ```

3. **Download Yahoo S5 dataset**  
   ```bash
   python scripts/download_s5.py
   ```

4. **Start infrastructure** (Kafka, Zookeeper, Postgres)  
   ```bash
   docker compose up -d
   ```

5. **Replay dataset into Kafka**  
   ```bash
   python scripts/replay_s5_to_kafka.py --config configs/data.yaml
   ```

6. **Run detector consumer**  
   ```bash
   python src/pipeline/consumer.py --config configs/app.yaml
   ```

7. **Check anomalies in Postgres**  
   - Connect to DB:  
     ```bash
     docker exec -it postgres psql -U user -d anomalies
     SELECT * FROM events LIMIT 10;
     ```

---

## 5. Benefits (even at MVP stage)
- **Understandable**: small, clear components (producer → detector → sink).
- **Extensible**: detectors can be swapped out (rule-based → ML → deep learning).
- **Practical**: connects anomaly detection theory with real streaming infra.
- **Teachable**: works well for demos and portfolio projects.

---

## 6. Future Improvements
- Add **Spark Structured Streaming** for scalable processing.
- Integrate **MLflow** for experiment tracking and model registry.
- Add **drift detection** (Evidently) to monitor data distributions.
- Deploy to **cloud (AWS/GCP/Azure)** using Docker/Cloud Run.
- Support **multivariate anomalies** (NASA SMAP telemetry).

---

## Acknowledgements
- Yahoo Webscope S5 dataset: [Yahoo Research](https://webscope.sandbox.yahoo.com/catalog.php?datatype=s)  
- Inspiration: NAB benchmark, PyOD library, and real-world anomaly detection systems.
