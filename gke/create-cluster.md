# Deploy a cluster in GKE

## Install the Google Cloud SDK

```
brew install caskroom/cask/google-cloud-sdk
```

## Cluster configuration

```
export GKE_PROJECT_NAME=fbelzunc-training
export GKE_MIN_NODES=1
export GKE_MAX_NODES=3
export GKE_DOMAIN_NAME=fbelzunc.$GKE_PROJECT_NAME.gke.k8s.local
export GKE_CLUSTER_NAME=cluster-$GKE_PROJECT_NAME
export GKE_ZONE=europe-west1-b
export GKE_ACCOUNT=<THE_GKE_ACCOUNT> // Enterprise e-mail
export GKE_USER_ID=<USER_ID> // support
gcloud config set project $GKE_PROJECT_NAME
gcloud config set compute/zone $GKE_ZONE
gcloud config set account $GKE_ACCOUNT
gcloud auth login
```

## Enable Google Kubernetes Engine API

[Enable Google Kubernetes Engine API](https://console.cloud.google.com/apis/library/container.googleapis.com?q=kubernetes%20engine&_ga=2.80379410.-405337588.1547913753)

## Create the Kubernetes cluster

```
gcloud beta container --project "$GKE_PROJECT_NAME" clusters create "$GKE_CLUSTER_NAME" --zone "$GKE_ZONE" --username "admin" --cluster-version "1.11.6-gke.2" --machine-type "n1-standard-2" --image-type "COS" --disk-size "50" --scopes "https://www.googleapis.com/auth/compute","https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --num-nodes "3" --network "default" --enable-cloud-logging --enable-cloud-monitoring --subnetwork "default" --enable-autoscaling --min-nodes "$GKE_MIN_NODES" --max-nodes "$GKE_MAX_NODES" --addons HorizontalPodAutoscaling,HttpLoadBalancing,KubernetesDashboard --enable-autorepair
```

