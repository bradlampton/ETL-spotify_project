# ETL-spotify_project
# Spotify End-to-End Data Engineering ETL Pipeline

An end-to-end data engineering pipeline that automates the extraction, transformation, and loading (ETL) of Spotify music data into a cloud data lakehouse. This project utilizes a serverless architecture to capture top playlists/recently played tracks daily, clean the data using Python, and expose it for analytical queries.

## 📌 Architecture Diagram

Here is a complete, production-ready README.md for a Spotify Python ETL Cloud Project. This template is tailored for the highly popular AWS Serverless architecture (Spotify API → Lambda → S3 → Athena), but it can easily be tweaked if you switch to GCP or Airflow.
Copy and paste this directly into your repository!
markdown
# Spotify End-to-End Data Engineering ETL Pipeline

An end-to-end data engineering pipeline that automates the extraction, transformation, and loading (ETL) of Spotify music data into a cloud data lakehouse. This project utilizes a serverless architecture to capture top playlists/recently played tracks daily, clean the data using Python, and expose it for analytical queries.

## 📌 Architecture Diagram

Use code with caution.
[Spotify API]
│ (Daily Trigger via Amazon EventBridge)
▼
[AWS Lambda (Extract)] ──► [Amazon S3 (Raw JSON Bucket)]
│
▼ (S3 PutObject Trigger)
[AWS Lambda (Transform)] ──► [Amazon S3 (Cleaned Parquet Bucket)]
│
▼
[AWS Glue Crawler]
│
▼
[Amazon Athena (SQL)]

## 🛠️ Tech Stack & Tools
* **Language:** Python 3.11 (Libraries: `spotipy`, `pandas`, `boto3`)
* **Compute:** AWS Lambda (Serverless execution)
* **Storage:** Amazon S3 (Object Storage Data Lake)
* **Orchestration:** Amazon EventBridge (Cron scheduling) & S3 Event Triggers
* **Catalog & Analytics:** AWS Glue Crawler, AWS Glue Data Catalog, and Amazon Athena

## 🚀 Key Pipeline Features
* **Automated Extraction:** EventBridge triggers an extraction script daily to fetch track, artist, and album metadata using the Spotify Web API.
* **Event-Driven Transformation:** Dropping raw data into the S3 landing zone automatically spins up a transformation Lambda. This isolates compute and scales horizontally.
* **Storage Optimization:** Converts raw, deeply nested JSON structures into optimized columnar **Parquet** files, drastically reducing Amazon Athena query costs.
* **Idempotency & Deduplication:** Cleans and normalizes relational data (Tracks, Albums, Artists) to maintain strict schema integrity.

## 📂 Repository Structure
```text
├── src/
│   ├── extract.py         # Lambda code: fetches Spotify API data, stores to S3 raw/
│   └── transform.py       # Lambda code: reads JSON, transforms with pandas, stores to S3 transformed/
├── requirements.txt       # Project python dependencies
├── .env.example           # Example file for local environment variables
└── README.md
```

## ⚙️ Setup & Deployment

### 1. Prerequisites
* An active [AWS Account](https://amazon.com) with administrative permissions.
* A [Spotify Developer Account](https://spotify.com) to generate your client credentials.

### 2. Spotify Credentials
1. Go to your Spotify Developer Dashboard and **Create an App**.
2. Note down your `Client ID` and `Client Secret`.

### 3. AWS Infrastructure Setup
1. **S3 Buckets:** Create two S3 buckets (or one bucket with two distinct folders):
   * `spotify-raw-data-zone`
   * `spotify-transformed-data-zone`
2. **IAM Roles:** Create an execution role for your Lambda functions granting access to `S3`, `CloudWatch Logs`, and `AWS Glue`.
3. **Lambda - Extraction Function:**
   * Create a Python Lambda function named `spotify_extraction_pipeline`.
   * Add your Spotify client keys as **Environment Variables** (`SPOTIFY_CLIENT_ID`, `SPOTIFY_CLIENT_SECRET`).
   * Add a custom **Lambda Layer** containing `spotipy` and `pandas` if using heavy data tasks.
4. **Lambda - Transformation Function:**
   * Create a Python Lambda function named `spotify_transformation_pipeline`.
   * Add an **S3 Event Trigger** to this function that fires whenever an object is added to your raw S3 bucket.

### 4. Data Catalog & Analytics
1. Create an **AWS Glue Crawler** pointed directly at your `spotify-transformed-data-zone` bucket.
2. Run the crawler to auto-generate the structural schema definitions.
3. Open **Amazon Athena**, select your new database, and query your Spotify tables instantly using standard SQL.

## 📊 Sample SQL Queries
Once data is populated, you can run analysis like finding the most popular artists in your extracted playlists:

```sql
SELECT 
    artist_name, 
    COUNT(track_id) AS total_tracks,
    AVG(track_popularity) AS average_popularity
FROM "spotify_db"."tracks_table"
GROUP BY artist_name
ORDER BY total_tracks DESC
LIMIT 10;
```

## 🤝 Contributing
Contributions, issues, and feature requests are welcome! Feel free to check the issues page if you want to expand the project (e.g., adding a Terraform script or a Looker dashboard configuration).
