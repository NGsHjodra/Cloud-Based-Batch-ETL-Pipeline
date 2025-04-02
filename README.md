# Cloud-Based Batch ETL Pipeline (GCP)

This project implements a cloud-native ETL pipeline that extracts schedule data from the [Harbour.Space API](https://harbour.space), stores it in Google Cloud Storage, and automatically loads it into BigQuery for analysis.

---

## Tech Stack

- **Cloud Run** – Serverless execution for ETL services
- **Cloud Scheduler** – Triggers extraction jobs on a schedule
- **Google Cloud Storage (GCS)** – Temporary storage of raw JSON files
- **BigQuery** – Scalable data warehouse for analysis
- **Eventarc** – Connects GCS events to Cloud Run functions

---

## Components

### 1. `extract-to-gcs` (Extract)
- Fetches schedule data from the Harbour.Space API
- Saves it as newline-delimited JSON (NDJSON)
- Uploads the file to a GCS bucket
- Triggered by: **Cloud Scheduler**

### 2. `gcs-to-bigquery` (Load)
- Triggered automatically via **Eventarc** when a file is uploaded to GCS
- Loads the JSON file into a **BigQuery table**
- Uses `autodetect` schema or custom schema (provided)

---

## Deployment (Docker + Cloud Build)

Both functions are Dockerized and can be deployed to Cloud Run:

```bash
# Build & push extract-to-gcs
docker build -t gcr.io/YOUR_PROJECT_ID/extract-to-gcs ./extract_to_gcs
docker push gcr.io/YOUR_PROJECT_ID/extract-to-gcs

gcloud run deploy extract-to-gcs \
  --image gcr.io/YOUR_PROJECT_ID/extract-to-gcs \
  --region us-central1 \
  --allow-unauthenticated

# Build & push gcs-to-bigquery
docker build -t gcr.io/YOUR_PROJECT_ID/gcs-to-bigquery ./gcs_to_bigquery
docker push gcr.io/YOUR_PROJECT_ID/gcs-to-bigquery

gcloud run deploy gcs-to-bigquery \
  --image gcr.io/YOUR_PROJECT_ID/gcs-to-bigquery \
  --region us-central1 \
  --no-allow-unauthenticated
