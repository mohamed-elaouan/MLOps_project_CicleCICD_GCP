# MLOps Iris Classification Project

This project demonstrates an end-to-end MLOps workflow for a machine learning classification model. It trains a model on the Iris dataset, serves predictions through a Flask web application, packages the app with Docker, and deploys it to Google Kubernetes Engine using CircleCI.

The repository is designed to show practical skills across machine learning engineering, application packaging, CI/CD automation, cloud deployment, and Kubernetes operations.

## Executive Summary

The goal of this project is to build a small but complete production-style machine learning application.

Users enter flower measurements in a web form, and the application predicts the Iris species using a trained decision tree classifier. Behind that simple interface, the project includes data processing, model training, artifact management, containerization, automated image build and push, and Kubernetes deployment.

This makes the project useful for demonstrating:

- Machine learning pipeline development
- Flask-based model serving
- Docker image creation
- CI/CD with CircleCI
- Google Artifact Registry integration
- Deployment to Google Kubernetes Engine
- Kubernetes Deployment and Service configuration

## Business Value

Although the dataset is intentionally simple, the project mirrors a real ML delivery workflow:

1. Raw data is processed into clean training and testing artifacts.
2. A model is trained and evaluated.
3. The trained model is saved as a reusable artifact.
4. A web application loads the model and serves real-time predictions.
5. Docker packages the application for consistent execution.
6. CircleCI automates build and deployment.
7. Kubernetes runs the application in a scalable cloud environment.

For recruiters and managers, this project highlights the ability to move beyond notebooks and build a deployable ML system.

## Project Architecture

```text
Raw Iris Data
    |
    v
Data Processing
    - Load CSV data
    - Handle outliers
    - Split train/test data
    - Save processed artifacts
    |
    v
Model Training
    - Train DecisionTreeClassifier
    - Evaluate model
    - Save model.pkl
    - Save confusion matrix
    |
    v
Flask Application
    - Load trained model
    - Accept flower measurements
    - Return predicted Iris species
    |
    v
Docker Image
    - Package application and dependencies
    - Expose Flask app on port 5000
    |
    v
CircleCI Pipeline
    - Checkout code
    - Authenticate with Google Cloud
    - Build Docker image
    - Push image to Artifact Registry
    - Deploy to GKE
    |
    v
Google Kubernetes Engine
    - Run application with 2 replicas
    - Expose service through LoadBalancer
```

## Repository Structure

```text
.
|-- application.py                 # Flask application for real-time prediction
|-- Dockerfile                     # Container definition for the Flask app
|-- kubernetes-deployment.yaml     # Kubernetes Deployment and Service
|-- requirements.txt               # Python dependencies
|-- setup.py                       # Python package setup
|-- .circleci/
|   `-- config.yml                 # CircleCI CI/CD pipeline
|-- pipeline/
|   `-- training_pipeline.py       # Runs data processing and model training
|-- src/
|   |-- data_processing.py         # Data loading, outlier handling, train/test split
|   |-- model_training.py          # Model training and evaluation
|   |-- logger.py                  # Logging setup
|   `-- custom_exception.py        # Custom exception handling
|-- artifacts/
|   |-- raw/                       # Raw dataset
|   |-- processed/                 # Processed train/test artifacts
|   `-- models/                    # Trained model and confusion matrix
|-- template/
|   `-- index.html                 # Web form for prediction input
|-- static/
|   `-- style.css                  # Styling for the web application
`-- notebook/
    `-- iris.ipynb                 # Experimentation notebook
```

## Machine Learning Workflow

### Data Processing

The data processing logic is implemented in `src/data_processing.py`.

Main responsibilities:

- Read the raw Iris dataset from `artifacts/raw/data.csv`
- Handle outliers in the `SepalWidthCm` column using the IQR method
- Split the dataset into training and testing sets
- Save processed datasets as `.pkl` files under `artifacts/processed`

Generated artifacts:

```text
artifacts/processed/X_train.pkl
artifacts/processed/X_test.pkl
artifacts/processed/y_train.pkl
artifacts/processed/y_test.pkl
```

### Model Training

The model training logic is implemented in `src/model_training.py`.

The project uses:

```text
DecisionTreeClassifier
criterion = gini
max_depth = 30
random_state = 42
```

During training, the pipeline:

- Loads the processed train/test artifacts
- Trains a decision tree classifier
- Saves the trained model to `artifacts/models/model.pkl`
- Evaluates the model using accuracy, precision, recall, and F1 score
- Saves a confusion matrix image to `artifacts/models/confusion_matrix.png`

### Training Pipeline

To run the complete training process:

```bash
python pipeline/training_pipeline.py
```

This executes:

1. Data processing
2. Model training
3. Model evaluation
4. Artifact generation

## Web Application

The prediction service is implemented in `application.py`.

The Flask app:

- Loads the trained model from `artifacts/models/model.pkl`
- Accepts four Iris flower measurements:
  - `SepalLengthCm`
  - `SepalWidthCm`
  - `PetalLengthCm`
  - `PetalWidthCm`
- Sends the values to the model
- Displays the predicted Iris species in the browser

Run the application locally:

```bash
python application.py
```

The app listens on:

```text
http://localhost:5000
```

## Local Setup

### 1. Create a Virtual Environment

```bash
python -m venv venv
source venv/bin/activate
```

On Windows:

```bash
venv\Scripts\activate
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
pip install -e .
```

### 3. Train the Model

```bash
python pipeline/training_pipeline.py
```

### 4. Start the Flask App

```bash
python application.py
```

## Docker

The Dockerfile builds a Python 3.9 image, installs the project dependencies, exposes port `5000`, and starts the Flask application.

Build the image:

```bash
docker build -t mlops-project-iris:latest .
```

Run the container:

```bash
docker run -p 5000:5000 mlops-project-iris:latest
```

Open:

```text
http://localhost:5000
```

## CI/CD with CircleCI

The CircleCI pipeline is defined in `.circleci/config.yml`.

The workflow contains three jobs:

1. `checkout_code`
   - Pulls the repository code into the CircleCI environment.

2. `build_docker_image`
   - Installs the Docker CLI if needed.
   - Starts CircleCI remote Docker.
   - Authenticates with Google Cloud using a service account key.
   - Builds the Docker image.
   - Pushes the image to Google Artifact Registry.

3. `deploy_to_gke`
   - Authenticates with Google Cloud.
   - Fetches GKE cluster credentials.
   - Applies `kubernetes-deployment.yaml` to deploy the application.

Image target:

```text
us-central1-docker.pkg.dev/$GOOGLE_PROJECT_ID/mlops-project-iris/mlops-project-iris:latest
```

## Required CircleCI Environment Variables

Configure these variables in CircleCI project settings:

```text
GCLOUD_SERVICE_KEY
GOOGLE_PROJECT_ID
GKE_CLUSTER
GKE_REGION
```

Recommended value for this project:

```text
GKE_REGION=us-central1
```

### Environment Variable Details

`GCLOUD_SERVICE_KEY`

Base64-encoded Google Cloud service account JSON key.

`GOOGLE_PROJECT_ID`

Google Cloud project ID where Artifact Registry and GKE are hosted.

`GKE_CLUSTER`

Name of the target GKE cluster.

`GKE_REGION`

Region of the GKE cluster. For this project, the expected region is `us-central1`.

## Google Cloud Requirements

Before running the full CI/CD pipeline, the Google Cloud project should include:

- A Google Artifact Registry Docker repository named `mlops-project-iris`
- A GKE cluster in the configured region
- A service account with permissions to:
  - Authenticate with Google Cloud
  - Push Docker images to Artifact Registry
  - Fetch GKE cluster credentials
  - Deploy Kubernetes resources

Common roles include:

```text
Artifact Registry Writer
Kubernetes Engine Developer
Kubernetes Engine Cluster Viewer
Service Account User
```

Exact permissions may vary depending on the organization security policy.

## Kubernetes Deployment

The Kubernetes configuration is defined in `kubernetes-deployment.yaml`.

It creates:

- A `Deployment` named `mlops-app`
- Two application replicas
- A container listening on port `5000`
- A `LoadBalancer` service named `mlops-service`
- External access through port `80`

Apply manually:

```bash
kubectl apply -f kubernetes-deployment.yaml
```

Check deployment:

```bash
kubectl get deployments
kubectl get pods
kubectl get services
```

## Troubleshooting Notes

### `docker: command not found`

This means the CircleCI executor image does not include the Docker CLI. The pipeline installs `docker.io` before running Docker commands.

### GKE Cluster Not Found

If CircleCI shows:

```text
Could not find cluster
Did you mean ... in [us-central1]?
```

Set:

```text
GKE_REGION=us-central1
```

and make sure the `get-credentials` command uses `--region`, not a zone such as `us-central1-a`.

### Artifact Registry Push Fails

Check that:

- The repository exists in Artifact Registry.
- The region matches the image path.
- The service account has write access.
- `GOOGLE_PROJECT_ID` is correct.

### Kubernetes Deploy Fails

Check that:

- The service account can access the GKE cluster.
- `GKE_CLUSTER` exactly matches the cluster name.
- `GKE_REGION` exactly matches the cluster region.
- `kubectl` can apply resources in the target namespace.

## Technologies Used

- Python 3.9
- Flask
- Pandas
- NumPy
- Scikit-learn
- Joblib
- Matplotlib
- Seaborn
- Docker
- CircleCI
- Google Cloud SDK
- Google Artifact Registry
- Google Kubernetes Engine
- Kubernetes

## Key Skills Demonstrated

- End-to-end machine learning pipeline design
- Data preprocessing and artifact generation
- Supervised classification model training
- Flask API and web interface development
- Docker-based application packaging
- CI/CD workflow creation with CircleCI
- Cloud authentication using service accounts
- Docker image publishing to Artifact Registry
- Kubernetes deployment on GKE
- Debugging CI/CD and cloud deployment issues

## Future Improvements

Possible next steps for making the project more production-ready:

- Add automated unit tests for data processing and model training
- Add API tests for the Flask prediction route
- Add model performance reporting to the CI pipeline
- Pin dependency versions in `requirements.txt`
- Use a non-root Docker user
- Add health checks and readiness probes to Kubernetes
- Add resource requests and limits for the container
- Add model versioning
- Add monitoring and logging in GKE
- Separate development, staging, and production deployments

## Author

Mohamed Elaouan

MLOps and machine learning engineering project focused on demonstrating practical model deployment and CI/CD skills.
