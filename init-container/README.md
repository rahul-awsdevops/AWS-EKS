# Kubernetes Deployment and Storage for Node.js App with PostgreSQL

This repository contains Kubernetes manifests for deploying a **Node.js application** connected to a **PostgreSQL database**. The application uses a persistent volume to store PostgreSQL data and is exposed via a `NodePort` service.

## Manifests Overview

### 1. **PostgreSQL Deployment (`postgres-deployment.yaml`)**

- **Kind**: `Deployment`
- **Replicas**: 1 (single instance of PostgreSQL)
- **Image**: `postgres:15`
- **Environment Variables**:
  - `POSTGRES_USER`: User for PostgreSQL.
  - `POSTGRES_PASSWORD`: Password for PostgreSQL user.
  - `POSTGRES_DB`: Name of the database.
  - `PGDATA`: Directory for storing PostgreSQL data.
- **Volume**: Persistent storage (`postgres-pvc`) is used to store data across pod restarts.
- **Service**: Exposes PostgreSQL on port `5432`.

### 2. **Node.js Application Deployment (`nodejs-deployment.yaml`)**

- **Kind**: `Deployment`
- **Replicas**: 2 (two instances of the Node.js app)
- **Image**: A custom Docker image for the Node.js app hosted on AWS ECR (`676206914267.dkr.ecr.us-east-1.amazonaws.com/app:nodejs-k8s-pv-testing`).
- **Init Container**: A `busybox` container waits for the PostgreSQL service (`postgres-service`) to be ready before starting the Node.js app.
- **Environment Variables**:
  - `PGHOST`: Host for PostgreSQL service (`postgres-service`).
  - `PGUSER`: User for PostgreSQL.
  - `PGPASSWORD`: Password for PostgreSQL user.
  - `PGDATABASE`: Database name (`userdb`).
  - `PGPORT`: Port for PostgreSQL (`5432`).
- **Service**: Exposes the Node.js application on port `80` via a `NodePort` service on port `32055`.

### 3. **PostgreSQL Persistent Volume Claim (`postgres-pvc.yaml`)**

- **Kind**: `PersistentVolumeClaim`
- **Storage Request**: 8Gi of storage.
- **Access Mode**: `ReadWriteOnce` (mountable by only one node).
- **Storage Class**: `ebs-sc` (for AWS EBS volumes).

### 4. **PostgreSQL StorageClass (`ebs-sc.yaml`)**

- **Kind**: `StorageClass`
- **Provisioner**: `ebs.csi.aws.com` (uses AWS EBS as the storage backend).
- **Parameters**:
  - `type: gp3`: General-purpose SSD storage type (`gp3`).
  - `fsType: ext4`: File system type for the volume.
- **Reclaim Policy**: `Delete` (EBS volume is deleted when PVC is deleted).
- **VolumeBindingMode**: `WaitForFirstConsumer` (the volume is provisioned only when a pod is scheduled).

## Usage Instructions

1. **Apply PostgreSQL Deployment and PVC**:
   ```bash
   kubectl apply -f postgres-deployment.yaml
   kubectl apply -f postgres-pvc.yaml
   kubectl apply -f ebs-sc.yaml
   kubectl apply -f nodejs-deployment.yaml

## Usage Instructions
Notes
Make sure your Kubernetes cluster has the necessary permissions and IAM roles to provision EBS volumes.


