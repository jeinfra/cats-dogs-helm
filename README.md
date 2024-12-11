### Cats & Dogs Application Deployment Guide

This repository provides a Helm chart and workflow to deploy the **Cats & Dogs** web application on a Kubernetes cluster. The application allows users to vote for their preferred animal and displays the results in a simple UI.

---

## **Overview**

### Application Features:
- **Web UI**: Accessible via port `80`, exposing a simple interface for voting.
- **Health Endpoint**: `/health` endpoint returns `200 OK` when the application is healthy.
- **Persistent Database**: Votes are stored in a database located at `/var/lib/cats-dogs/database`.
- **Simulated Failures**: Application randomly fails after 2–15 minutes, requiring resilience mechanisms.
- **Deployment Compatibility**: Configured for Minikube with a `NodePort` service for external access.

---

## **Key Components**

### 1. **Helm Chart Structure**
- **`Chart.yaml`**: Defines the chart metadata.
- **`values.yaml`**: Allows configuration of the deployment (replicas, resource limits, persistence, etc.).
- **Templates**:
  - `deployment.yaml`: Defines the application deployment.
  - `service.yaml`: Exposes the application via a `NodePort` service.
  - `pvc.yaml`: Configures persistent storage for the database.
  - `hpa.yaml`: Configures autoscaling based on CPU/memory usage.
  - `NOTES.txt`: Instructions for accessing the deployed application.

### 2. **GitHub Workflow**
Automates the deployment process:
- Lints and packages the Helm chart.
- Transfers files to the target VM.
- Builds and deploys the application using `helm install`.

---

## **Deployment Process**

### Prerequisites
- **Kubernetes Cluster**: Tested on Minikube.
- **Docker**: For building the application image.
- **Helm**: For chart deployment.
- **Minikube**: Ensure Minikube is running.

### Steps

#### 1. **Build the Docker Image**
```bash
docker build -t cats-dogs:v1 .
```

#### 2. **Deploy Using Helm**
Update the Helm chart values (`values.yaml`) as needed:
- Enable persistence:
  ```yaml
  persistence:
    enabled: true
    storageClass: "standard"
    accessMode: ReadWriteOnce
    size: 2Gi
  ```
- Configure autoscaling:
  ```yaml
  autoscaling:
    enabled: true
    minReplicas: 3
    maxReplicas: 10
  ```

Deploy the chart:
```bash
helm upgrade --install cats-dogs ./cats-dogs
```

#### 3. **Access the Application**
- Retrieve the Minikube IP:
  ```bash
  minikube ip
  ```
- Access the application via:
  ```bash
  curl http://<minikube-ip>:30080
  ```

---

## **Fault Tolerance Configuration**

### Resilience Features
1. **Multiple Replicas**:
   - Configured in `values.yaml`:
     ```yaml
     replicaCount: 3
     ```
   - Ensures high availability.

2. **Liveness Probe**:
   - Automatically restarts failed pods:
     ```yaml
     livenessProbe:
       httpGet:
         path: /health
         port: 5000
       initialDelaySeconds: 10
       periodSeconds: 10
     ```

3. **Persistent Volume**:
   - Ensures data is retained across pod restarts:
     - PVC mounts `/var/lib/cats-dogs/database`.

4. **Horizontal Pod Autoscaler (HPA)**:
   - Scales pods based on resource usage:
     ```yaml
     autoscaling:
       enabled: true
       targetCPUUtilizationPercentage: 80
     ```

---

## **Testing and Validation**

### Validate Deployment
- Check pod and service statuses:
  ```bash
  kubectl get pods
  kubectl get svc
  ```

- View logs for troubleshooting:
  ```bash
  kubectl logs -l app.kubernetes.io/name=cats-dogs
  ```

### Simulate Failures
- Manually delete pods:
  ```bash
  kubectl delete pod -l app.kubernetes.io/name=cats-dogs
  ```
- Verify new pods are created automatically.

### Verify Data Persistence
1. Add some votes via the web UI.
2. Delete a pod and confirm data is retained in the new pod.

---

## **Enhancements**
- **Ingress Support**:
  - Optionally enable Ingress for DNS-based access:
    ```yaml
    ingress:
      enabled: true
      hosts:
        - host: cats-dogs.local
    ```

- **Monitoring**:
  - Enable `ServiceMonitor` for Prometheus integration:
    ```yaml
    metrics:
      enabled: true
      serviceMonitor:
        enabled: true
    ```

---