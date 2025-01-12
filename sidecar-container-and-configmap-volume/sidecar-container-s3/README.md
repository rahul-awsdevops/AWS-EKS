# Kubernetes Deployment for Nginx with Sidecar Container for Amazon S3 Logging

This repository contains Kubernetes manifests for deploying an **Nginx application** with a sidecar container that uploads Nginx logs to an **Amazon S3 bucket**.

## Manifests Overview

### 1. **Nginx Deployment with Sidecar (`nginx-deployment.yaml`)**

- **Kind**: `Deployment`
- **Replicas**: 1 (single instance of the Nginx app)
- **Main Container (`nginx-app`)**:
  - **Image**: `rahulranjan1207015/nginx-app:latest`
  - **Ports**: Exposes port `80` for the Nginx service.
  - **Volume Mounts**:
    - Mounts a `ConfigMap` (`nginx-config`) at `/etc/nginx/nginx.conf` for custom Nginx configurations.
    - Mounts an `emptyDir` volume (`log-volume`) at `/var/log/nginx` to store Nginx logs.
- **Sidecar Container (`s3-log-sidecar-container`)**:
  - **Image**: `amazon/aws-cli:latest`
  - **Command**: The sidecar container uploads Nginx access and error logs to an S3 bucket every 5 minutes.
  - **Volume Mounts**: Shares the same `log-volume` for accessing logs.
  
The sidecar container uses AWS CLI to upload logs from the Nginx container:
  - Uploads `access.log` and `error.log` to the S3 bucket `nginx-logs-bucket-1234567` every 5 minutes.
  
- **Volumes**:
  - **nginx-config-volume**: A `ConfigMap` named `nginx-config` is used to configure Nginx.
  - **log-volume**: An `emptyDir` is used to store logs temporarily, shared between the main container and the sidecar.

### 2. **Nginx Service (`nginx-service.yaml`)**

- **Kind**: `Service`
- **Type**: `NodePort` (exposes the service on a high-range port on the node)
- **Ports**:
  - **Port**: 80 (internal port of the service)
  - **NodePort**: 32223 (external port exposed on the node)
  - **TargetPort**: 80 (target port on the Nginx container)
  
The service allows access to the Nginx application via the `NodePort` on port `32223`.

Ensure that the AWS CLI in the sidecar container has the necessary AWS credentials to upload logs to S3.