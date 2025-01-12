# Kubernetes Deployment for Nginx with Dynamic Configuration using ConfigMap

This repository contains Kubernetes manifests for deploying an **Nginx application** where the Nginx configuration is managed using a **ConfigMap**. This allows for flexible updates to the Nginx configuration without rebuilding the Docker image.

## Manifests Overview

### 1. **Nginx Deployment with ConfigMap (`nginx-deployment.yaml`)**

- **Kind**: `Deployment`
- **Replicas**: 1 (single instance of the Nginx app)
- **Container (`nginx-app`)**:
  - **Image**: `rahulranjan1207015/nginx-app:latest`
  - **Ports**: Exposes port `80` for the Nginx service.
  - **Volume Mounts**:
    - Mounts a `ConfigMap` (`nginx-config`) at `/etc/nginx/nginx.conf` to override the default Nginx configuration with the custom `nginx.conf` from the ConfigMap.
    - Mounts an `emptyDir` volume (`log-volume`) at `/var/log/nginx` to store Nginx logs.
  
This setup allows you to dynamically manage the Nginx configuration without modifying the Docker image itself. The default `nginx.conf` from the image is ignored in favor of the custom configuration provided via the ConfigMap.

- **Volumes**:
  - **nginx-config-volume**: A `ConfigMap` named `nginx-config` holds the custom Nginx configuration (`nginx.conf`).
  - **log-volume**: An `emptyDir` is used to store logs temporarily.

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
  - **Server Configuration**:
    - Listens on port `80` and serves static files from `/usr/share/nginx/html`.

