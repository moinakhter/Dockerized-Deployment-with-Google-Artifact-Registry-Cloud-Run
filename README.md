# Dockerized NLP Deployment with Google Artifact Registry & Cloud Run

## Overview
This repository automates the process of building and deploying a Dockerized NLP model to Google Cloud using Bitbucket Pipelines. It builds a Docker image, stores it in Google Artifact Registry, and deploys it to Google Cloud Run.
This repository contains a Bitbucket Pipeline configuration to automate the process of building and deploying a Dockerized NLP model to Google Artifact Registry. The pipeline is triggered on tag creation and performs the following steps:

1. Installs necessary dependencies, including Git LFS and Google Cloud SDK.
2. Authenticates with Google Cloud using a service account.
3. Configures Docker to use Google Artifact Registry.
4. Extracts the BERT model from a zip file.
5. Builds and tags the Docker image.
6. Pushes the built Docker image to Google Artifact Registry.

## Prerequisites
Before running the pipeline, ensure that you have the following:

- A Bitbucket repository with the pipeline configuration (`bitbucket-pipelines.yml`).
- Required Bitbucket secret variables configured:
  - `GCLOUD_SERVICE_KEY`: Base64-encoded Google Cloud service account key.
  - `GCP_PROJECT_ID`: Google Cloud project ID.
- A Google Cloud project with Artifact Registry and required IAM permissions.

## Pipeline Configuration
### Bitbucket Pipelines File (`bitbucket-pipelines.yml`)
```yaml
image: python:3.10.0

pipelines:
  tags:
    '*':
      - step:
          name: Build and Push Docker Image to The Google Artifact Registry.
          script:
            # Install Git LFS
            - apt-get update && apt-get install -y git-lfs
            - git lfs pull

            # Install required tools
            - apt-get update && apt-get install -y curl unzip

            # Install Google Cloud SDK
            - curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-457.0.0-linux-x86_64.tar.gz
            - tar -xf google-cloud-cli-457.0.0-linux-x86_64.tar.gz
            - export CLOUDSDK_CORE_DISABLE_PROMPTS=1
            - ./google-cloud-sdk/install.sh
            - echo "Installed Google Cloud SDK"

            # Set up Google Cloud authentication
            - echo $GCLOUD_SERVICE_KEY | base64 -d > /tmp/keyfile.json
            - export GOOGLE_APPLICATION_CREDENTIALS=/tmp/keyfile.json
            - gcloud auth activate-service-account --key-file=/tmp/keyfile.json

            # Configure Docker for Google Artifact Registry
            - gcloud auth configure-docker europe-west3-docker.pkg.dev
            - gcloud config set project $GCP_PROJECT_ID

            # Build and tag the Docker image
            - unzip ./models/bert_model.zip -d ./models/
            - docker build -t nlp:$BITBUCKET_TAG-$BITBUCKET_COMMIT .
            - docker tag nlp:$BITBUCKET_TAG-$BITBUCKET_COMMIT europe-west3-docker.pkg.dev/$GCP_PROJECT_ID/assistant-images/nlp:$BITBUCKET_TAG-$BITBUCKET_COMMIT

            # Push the Docker image to Google Artifact Registry
            - docker push europe-west3-docker.pkg.dev/$GCP_PROJECT_ID/assistant-images/nlp:$BITBUCKET_TAG-$BITBUCKET_COMMIT
            - echo "Image built and tagged successfully"

          services:
            - docker
```

## Environment Variables
Ensure the following Bitbucket repository variables are set:

| Variable Name          | Description |
|------------------------|-------------|
| `GCLOUD_SERVICE_KEY`   | Base64-encoded Google Cloud service account key |
| `GCP_PROJECT_ID`       | Google Cloud project ID |

## Deployment Process
1. Push code to the repository and create a tag.
2. The Bitbucket pipeline will trigger automatically.
3. The pipeline will build, tag, and push the Docker image to Google Artifact Registry.
4. The image will be available for deployment in Google Cloud.

## Notes
- Ensure that the service account used has sufficient permissions for Google Artifact Registry.
- Modify the `gcloud auth configure-docker` region (`europe-west3-docker.pkg.dev`) based on your Google Cloud region.
- Update the `google-cloud-cli` version if necessary to use the latest features and fixes.

## License
This project is licensed under the MIT License.

