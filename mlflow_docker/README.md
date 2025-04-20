# MLflow Tracking Server with Docker Compose

This repository contains a Docker Compose configuration for setting up an MLflow tracking server with a PostgreSQL backend for storing experiment data.

## System Architecture

The setup consists of two services:

1. **PostgreSQL Database** - Used as the backend store for MLflow to persist experiment and run data
2. **MLflow Tracking Server** - Provides the MLflow UI and API for experiment tracking

## Prerequisites

- Docker and Docker Compose installed on your system
- `.env` file with necessary environment variables (see below)

## Environment Variables

Create a `.env` file in the root directory with the following variables:

```
# PostgreSQL Configuration    
POSTGRES_HOST=postgresql
POSTGRES_PORT=5432
POSTGRES_DB=<db>
POSTGRES_USER=<username>
POSTGRES_PASSWORD=<password>

# MLflow Configuration
MLFLOW_VERSION=<mlflow-version>
MLFLOW_BACKEND_STORE_URI=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgresql:5432/${POSTGRES_DB}
```

## Usage

### Starting the Services

```bash
# Start all services in detached mode
docker compose -f mlflow-docker-compose.yaml up -d
```

### Accessing MLflow UI

Once the services are running, you can access the MLflow UI at:

```
http://localhost:5001
```

### Stopping the Services

```bash
# Stop all services
docker compose -f mlflow-docker-compose.yaml down

# Stop all services and remove volumes
docker compose -f mlflow-docker-compose.yaml down -v
```

## Connecting to MLflow from Python

```python
import mlflow

# Set the MLflow tracking URI to the Docker container
mlflow.set_tracking_uri("http://localhost:5001")

# Start a new experiment
mlflow.set_experiment("my-experiment")

# Start a new run
with mlflow.start_run():
    # Log parameters
    mlflow.log_param("param1", 5)
    
    # Log metrics
    mlflow.log_metric("accuracy", 0.95)
    
    # Log artifacts
    mlflow.log_artifact("model.pkl")
```

## Volume Management

This setup uses Docker volumes to persist data:

- `mlflow_db_data`: Stores PostgreSQL database files
- `mlflow_data`: Stores MLflow artifacts

## Health Checks

Both services have health checks configured:

- PostgreSQL: Checks if the database is ready to accept connections
- MLflow: Checks if the MLflow server is responding on the health endpoint

## Project Structure

```
.
├── .env                           # Environment variables
├── README.md                      # This file
├── mlflow-docker-compose.yaml     # Docker Compose configuration
└── mlflow/                        # MLflow Dockerfile and related files
    └── Dockerfile                 # Custom MLflow Dockerfile
```

## Customization

To customize the configuration, you can modify:

- `.env` file to change environment variables
- `mlflow-docker-compose.yaml` to adjust service configurations
- `mlflow/Dockerfile` to customize the MLflow container build

## Troubleshooting

If you experience issues with the services:

1. Check container logs:
   ```bash
   docker logs mlflow
   docker logs mlflow_db
   ```

2. Verify the health status:
   ```bash
   docker inspect mlflow | grep "Health"
   docker inspect mlflow_db | grep "Health"
   ```

3. Ensure the required ports (5001 and 5432) are not already in use on your system.

