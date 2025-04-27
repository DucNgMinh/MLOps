# Bank Marketing Campaign API

This repository contains a FastAPI application that serves a machine learning model for predicting whether a customer will subscribe to a certificate of deposit (CD).

## Note:
+ **mlflow** folder: deploy mlflow using docker with postgresql
+ **jenkins_docker** folder: deploy jenkins using docker
+ **observable_systems_docker** folder: deploy observable systems using docker

# Setup Instructions 

## 1. API App
### Option 1: Running locally

1. **Prerequisites**:
   - Python 3.9+
   - pip

2. **Installation**:
   ```bash
   # Clone the repository
   git clone <repository-url>
   cd MLOps
   
   # Install dependencies
   pip install -r requirements.txt
   ```

3. **Running the API**:
   ```bash
   # Make sure model.pkl file is in the current directory
   uvicorn app:app --reload
   ```

   The API will be available at `http://localhost:8000`.

### Option 2: Using Docker

1. **Prerequisites**:
   - Docker

2. **Building and running the Docker container**:
   ```bash
   # Build the Docker image
   docker build -t ducngminh/bank-campaign-api:<version> .
   
   # Run the container
   docker run -p 8000:8000 ducngminh/bank-campaign-api:<version>
   ```

   The API will be available at `http://localhost:8000`.

### API Endpoints

### 1. Make a prediction for a single customer

**Endpoint**: `POST /predict`

**Request Body**:
```json
{
  "customer_age": 41,
  "tenure": 24,
  "account_balance": 2567.89,
  "previous_campaign_number_of_contacts": 3,
  "employment_type": "private",
  "marital_status": "married",
  "education_level": "tertiary",
  "credit_default": "no",
  "housing_loan": "yes",
  "personal_loan": "no",
  "contact_method": "cellular",
  "previous_campaign_outcome": "success"
}
```

**Response**:
```json
{
  "prediction": 1,
  "probability": 0.85,
  "explanation": {
    "feature1": 0.3,
    "feature2": 0.2,
    "...": "..."
  }
}
```

### 2. Make predictions for multiple customers

**Endpoint**: `POST /batch-predict`

**Request Body**:
```json
[
  {
    "customer_age": 41,
    "tenure": 24,
    "...": "..."
  },
  {
    "customer_age": 35,
    "tenure": 12,
    "...": "..."
  }
]
```

**Response**:
```json
{
  "predictions": [1, 0],
  "probabilities": [0.85, 0.23]
}
```

### 3. Get model information

**Endpoint**: `GET /model-info`

**Response**:
```json
{
  "model_type": "sklearn.pipeline.Pipeline",
  "features": {
    "numeric": ["customer_age", "tenure", "account_balance", "previous_campaign_number_of_contacts"],
    "categorical": ["employment_type", "marital_status", "education_level", "credit_default", "housing_loan", "personal_loan", "contact_method", "previous_campaign_outcome"]
  },
  "classifier": {
    "type": "sklearn.ensemble._forest.RandomForestClassifier",
    "parameters": {
      "n_estimators": 100,
      "random_state": 42,
      "class_weight": "balanced"
    }
  }
}
```

### 4. Health check

**Endpoint**: `GET /health`

**Response**:
```json
{
  "status": "healthy",
  "model_loaded": true
}
```

### Testing the API

You can test the API using curl:

```bash
curl -X POST "http://localhost:8000/predict" \
     -H "Content-Type: application/json" \
     -d '{"customer_age": 41, "tenure": 24, "account_balance": 2567.89, "previous_campaign_number_of_contacts": 3, "employment_type": "private", "marital_status": "married", "education_level": "tertiary", "credit_default": "no", "housing_loan": "yes", "personal_loan": "no", "contact_method": "cellular", "previous_campaign_outcome": "success"}'
```

## 2. Observable systems
When we have a service, we need some observable systems to monitoring our service. In this repo, we suggest Elastic Search, Grafana, Prometheus and Jaeger.

### 2.1. Elastic Search
#### How to guide
+ ```cd observable_systems_docker/elk```
+ ```docker compose -f elk-docker-compose.yml -f extensions/filebeat/filebeat-compose.yml up -d```
+ You can access kibana at port 5601 to search logs, which FileBeat pulls logs from containers and pushes to ElasticSearch. Username and password of Kibana can be found at ```monitoring_docker/elk/.env```
![](images/elastic_search.png)

### 3.2. Prometheus + Grafana + Jaeger for monitoring resources and apps
#### How to guide
+ ```cd observable_systems_docker```
+ ```docker compose -f prom-graf-docker-compose.yaml up -d```
+ Access to Prometheus, Grafana, Jaeger and enjoy!!!
+ Then, you can access Prometheus at port 9090, Grafana at 3001 and Jaeger at 16686. Username and password of Grafana is admin.

#### 3.2.1. Prometheus
+ In Prometheus UI, you can search any metrics what you want to monitor and click on the button that i highlighted border to list all metrics prometheus scraping
#### 3.2.2. Grafana
When you access to Grafana, you can create your own dashboard to monitoring or use a template on Grafana Labs
#### 3.3.3. Jaeger
Last but not least, sometimes you need to trace some block code processing time, Jaeger will help you do that.

### 4. Jenkins
#### How to guide

+ ```docker compose -f jenkins_docker/docker-compose.yml up -d```

+ Jenkins service was exposed at port 8081, we can access by this port

+ Connect to github repo using ngrok. if 200 OK, you have already connect jenkins to github.

Additionally, in **Let me select individual events** in **Setting/Webhooks/Manage webhook**, tick **Pull requests** and **Pushes** to inform jenkins start to run whenever we push or pull code from github.
