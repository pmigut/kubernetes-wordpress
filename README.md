# Wordpress deployment using [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine), [Google Cloud SQL](https://cloud.google.com/sql) and [Helm](https://helm.sh)

## Introduction

The steps in this document describe the process of Wordpress deployment into GKE Kubernetes cluster using Google Cloud SQL instance as a DB and with help of a Helm chart.

## Prerequisites

- GKE cluster
- _kubectl_ configured to control the cluster
- Helm client
- instance of Cloud SQL Mysql database
- enabled Cloud SQL Admin API

## Installation

### Creating service account for SQL instance

In order to communicate with the database from within container, a service account is required.
Go to _IAM > Service accounts_ section of the GCP console and create a service account with _Cloud SQL Editor_ role. Download newly created key in JSON format. Create new secret in the cluster with the contents of downloaded JSON file.

To create secret use the command below and replace `downloaded-key.json` with the name of your key JSON file.

```console
kubectl create secret generic cloudsql-instance-credentials \
  --from-file=downloaded-key.json
```

### Setting up database

Worpress requires MySQL database.
Database setup can be accomplished through the _mysql_ client, GCP console or _gcloud_ tool.

The following command can be used to create database `wordpress` in `mysql1` instance.

```console
gcloud sql databases create wordpress -i mysql1 \
  --charset=utf8mb4 --collation=utf8mb4_unicode_ci
```

And an example command to create `wpuser` user identified by `wppass` password.

```console
gcloud sql users create wpuser --instance=mysql1 --password=wppass
```

### Installing Helm server (Tiller) in the cluster (unless it's already installed).

Create cluster-admin role for Tiller:

```console
kubectl create -f rbac-config.yaml
```

Install Tiller

```console
helm init
```

## Deploying the chart

The command below deploys Wordpress with release name set to `mywp`.

```console
helm install --name mywp ./wordpress \
  --set cloudsql.instanceConnectionName=project-1234:region:mysqlinstance \
  --set db.user=wpuser \
  --set db.password=wppass \
  --set db.database=wordpress \
  --set wordpressSkipInstall=true
```

The chart is based on the official stable [Wordpress Helm chart](https://github.com/pmigut/charts/tree/master/stable/wordpress). Additional setup options are available (excluding all mariadb and externaldb options). However, they were not tested in the process of creating this solution.

## Deleting release

To delete/uninstall `mywp` release use the following command.

```console
helm delete mywp
```

# Foot note
This solution was tested with GKE cluster v1.13.6-gke.13 and Minikube local cluster v1.1.1 running Kubernetes v1.14.3.