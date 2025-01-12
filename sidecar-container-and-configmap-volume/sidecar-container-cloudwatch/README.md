# Kubernetes Deployment for Nginx with Sidecar Container for CloudWatch Logging

This repository contains Kubernetes manifests for deploying an **Nginx application** with a sidecar container that sends Nginx logs to **Amazon CloudWatch** for monitoring and log aggregation.

## Manifests Overview

### 1. **Nginx Deployment with CloudWatch Sidecar (`nginx-deployment.yaml`)**

- **Kind**: `Deployment`
- **Replicas**: 1 (single instance of the Nginx app)
- **Main Container (`nginx-app`)**:
  - **Image**: `rahulranjan1207015/nginx-app:latest`
  - **Ports**: Exposes port `80` for the Nginx service.
  - **Volume Mounts**:
    - Mounts a `ConfigMap` (`nginx-config`) at `/etc/nginx/nginx.conf` for custom Nginx configurations.
    - Mounts an `emptyDir` volume (`log-volume`) at `/var/log/nginx` to store Nginx logs.
- **Sidecar Container (`cloudwatch-log-sidecar-container`)**:
  - **Image**: `rahulranjan1207015/cloudwatch-agent:v4`
  - **Volume Mounts**: Shares the same `log-volume` to access Nginx logs for sending them to CloudWatch.

The sidecar container utilizes the **CloudWatch Agent** to send logs from `/var/log/nginx` (access and error logs) to CloudWatch. This provides centralized log management for the application.

- **Volumes**:
  - **nginx-config-volume**: A `ConfigMap` named `nginx-config` holds the Nginx configuration (`nginx.conf`).
  - **log-volume**: An `emptyDir` is used to store logs temporarily, shared between the main container and the sidecar.

### 2. **Nginx Service (`nginx-service.yaml`)**

- **Kind**: `Service`
- **Type**: `NodePort` (exposes the service on a high-range port on the node)
- **Ports**:
  - **Port**: 80 (internal port of the service)
  - **NodePort**: 32223 (external port exposed on the node)
  - **TargetPort**: 80 (target port on the Nginx container)

The service allows access to the Nginx application via the `NodePort` on port `32223`.

### 3. **Nginx ConfigMap (`nginx-config.yaml`)**

- **Kind**: `ConfigMap`
- **Name**: `nginx-config`
- **Data**: Contains the custom Nginx configuration (`nginx.conf`).
  - **Logging Configuration**:
    - Defines the log format for Nginx access logs.
    - Specifies the log locations for `access.log` and `error.log` at `/var/log/nginx`.

### 4. **CloudWatch IAM Policy (`cloudwatch-iam-policy.json`)**

- **Purpose**: This IAM policy allows the CloudWatch sidecar container to interact with CloudWatch logs.
  - **Actions**: Allows the sidecar container to put log events, create log groups, and describe log streams.
  - **Resources**: Allows access to all CloudWatch logs in the account (`arn:aws:logs:*:*:*`).
