
# Real-Time Spacecraft Telemetry Anomaly Detection (MVP)
>     “The minimum viable product is that version of a new product which allows a team to collect the maximum amount of validated learning about customers with the least effort.”

    — ERIC RIES

## 1. Problem Statement
Modern spacecraft generate continuous streams of telemetry data from hundreds of onboard sensors — voltages, temperatures, currents, positions, and more.  
Hidden inside these streams are **anomalies**: subtle changes in sensor behavior that can indicate faults, system drift, or upcoming failures.

Goal: **detect anomalies in real time** as telemetry data arrives, rather than diagnosing them after a mission-critical event has already occurred.

We use the **NASA SMAP/MSL Telemanom dataset**, a benchmark collection of real spacecraft telemetry from NASA’s **Soil Moisture Active Passive (SMAP)** satellite and **Mars Science Laboratory (MSL)** rover.  
Each subsystem, or *channel*, contains readings from multiple sensors over time, along with labeled anomaly intervals derived from NASA’s fault analysis.


## 2. Solution
Design: a **streaming anomaly detection pipeline** that simulates real-time fault monitoring for spacecraft telemetry.

1. **Replay telemetry data as a live stream** (NASA Telemanom `.npy` arrays → Kafka topic).  
2. **Detection logic** (e.g., rolling statistics, Median Absolute Deviation, Isolation Forest) runs on each new data point.  
3. **Flag anomalies in real time** and publish them to another Kafka topic.  
4. **Store anomalies in PostgreSQL** for later review and analysis.  
5. **Visualize results** using a simple dashboard (Streamlit or Grafana).

This MVP focuses on building a working end-to-end system — from raw telemetry to real-time anomaly flags — in a modular and easily extendable way.



### 2.1. Technologies
- **Kafka** → message broker to simulate streaming telemetry data  
- **Python** → anomaly detection logic  
- **PostgreSQL** → storage for flagged anomaly events  
- **Streamlit / Grafana** → dashboards for visualization  
- *(Future: Spark Structured Streaming, MLflow, Evidently for drift tracking)*



### 2.2. Architecture Diagram
```

[NASA Telemanom Data (.npy)] --> [Kafka Producer] --> [Kafka Topic: telemetry.raw]
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



## 3. Directory Layout
```

anomaly-detector/
├─ configs/           # YAML config files (detector thresholds, dataset paths, broker info)
├─ data/              # Local sample data (NASA Telemanom SMAP/MSL)
│  ├─ raw/
│  └─ processed/
├─ docs/              # Documentation, diagrams, screenshots
├─ notebooks/         # Jupyter notebooks for EDA and model prototyping
├─ scripts/           # Utilities (dataset download, replay to Kafka, metrics)
├─ src/               # Source code
│  ├─ common/         # IO, schema, utilities
│  ├─ detector/       # Algorithms: MAD, Z-score, Isolation Forest
│  ├─ pipeline/       # Kafka producer/consumer and Postgres sink
│  └─ api/            # Optional FastAPI service for on-demand scoring
├─ tests/             # Unit and integration tests
├─ infra/             # Dockerfiles, docker-compose, infra configs
│  ├─ docker/
│  └─ grafana/
└─ .github/workflows/ # CI pipelines

```


## 4. Steps to Replicate (MVP)

1. **Clone the repo**
  
   git clone <repo-url>
   cd anomaly-detector


2. **Set up environment**

   * Install Python 3.10+
   * Install dependencies:

     pip install -r requirements.txt
     

3. **Download NASA Telemanom dataset**

   
   python scripts/download_telemanom.py
  

4. **Start infrastructure** (Kafka, Zookeeper, Postgres)

  
   docker compose up -d
   

5. **Replay telemetry data into Kafka**

   
   python scripts/replay_telemanom_to_kafka.py --config configs/data.yaml
   

6. **Run detector consumer**

  
   python src/pipeline/consumer.py --config configs/app.yaml
   

7. **Check anomalies in Postgres**

   
   docker exec -it postgres psql -U user -d anomalies
   SELECT * FROM events LIMIT 10;
   

## 5. Benefits (even at MVP stage)

* **Understandable:** clean, modular components (producer → detector → sink).
* **Extensible:** supports multiple anomaly detection methods (rule-based → ML → deep learning).
* **Realistic:** based on real NASA spacecraft telemetry data.
* **Practical:** bridges data science, MLOps, and streaming infrastructure.
* **Teachable:** ideal for portfolio or educational demonstrations.



## 6. Future Improvements

* Add **Spark Structured Streaming** for large-scale processing.
* Integrate **MLflow** for experiment tracking and model registry.
* Add **drift detection** (Evidently) to monitor data quality over time.
* Deploy to **cloud (AWS/GCP/Azure)** with Docker or Cloud Run.
* Extend to **multivariate models** (Autoencoder, LSTM) for deeper anomaly analysis.



## Acknowledgements

* NASA Telemanom Dataset (SMAP & MSL telemetry): [Hundman et al., KDD 2018, *Detecting Spacecraft Anomalies Using LSTMs and Nonparametric Dynamic Thresholding*](https://www.kdd.org/kdd2018/accepted-papers/view/detecting-spacecraft-anomalies-using-lstms-and-nonparametric-dynamic-thresholdin)
* Inspiration: Telemanom project, PyOD library, and NASA’s open data programs.
