# monitoring-app

# Flask App on Kubernetes (Minikube) via AWS ECR

This project demonstrates deploying a containerized Python Flask web application to a local Kubernetes cluster (Minikube) after pushing its Docker image to a private AWS ECR repository.

## ✨ Key Features

* **Containerization:** Flask app packaged with Docker.
* **Private Registry:** Image hosted on AWS ECR.
* **Kubernetes Deployment:** Orchestrated on Minikube.
* **Secure Image Pulls:** Using Kubernetes `ImagePullSecrets` for ECR.
* **Service Exposure:** Application accessible from host.

## 🛠️ Technologies Used

* Python (Flask, Gunicorn)
* Docker
* AWS CLI
* Minikube
* kubectl
* AWS ECR

## 🚀 Quick Start & Deployment

Follow these steps to get the Flask app running on Minikube:

1.  **Prerequisites:** Ensure you have **AWS CLI (configured)**, **Docker Desktop**, **Minikube**, and **kubectl** installed.
2.  **Flask App Files:** Make sure `app.py` and `requirements.txt` are in your project root.
3.  **Dockerfile:** Create your `Dockerfile` (see example below or in a separate file).
4.  **Build Docker Image:**
    ```bash
    docker build -t flask-web-app .
    ```
5.  **Configure ECR Credential Helper:**
    * **Crucial for Windows:** Find your `docker-credential-ecr-login.exe` (e.g., `C:\Program Files\Docker\Docker\resources\bin\docker-credential-ecr-login.exe`).
    * **Edit `C:\Users\<YourUser>\.docker\config.json` manually:**
        Delete existing `config.json` and create new one with:
        ```json
        {
          "credHelpers": {
            "YOUR_AWS_ACCOUNT_ID.dkr.ecr.YOUR_AWS_REGION.amazonaws.com": "C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker-credential-ecr-login.exe"
          }
        }
        ```
        (Remember double backslashes and your actual AWS details).
6.  **Login to ECR & Push Image:**
    ```bash
    docker login https://YOUR_AWS_ACCOUNT_ID.dkr.ecr.YOUR_AWS_REGION.amazonaws.com
    docker tag flask-web-app:latest YOUR_AWS_ACCOUNT_ID.dkr.ecr.YOUR_AWS_REGION.amazonaws.com/flask-web-app:latest
    docker push YOUR_AWS_ACCOUNT_ID.dkr.ecr.YOUR_AWS_REGION.amazonaws.com/flask-web-app:latest
    ```
7.  **Start Minikube:**
    ```bash
    minikube stop && minikube start --driver=docker --embed-certs
    ```
8.  **Create Kubernetes Secret:**
    ```bash
    kubectl delete secret ecr-pull-secret --ignore-not-found && kubectl create secret generic ecr-pull-secret --from-file=.dockerconfigjson=C:\Users\<YourUser>\.docker\config.json --type=kubernetes.io/dockerconfigjson
    ```
9.  **Apply Kubernetes Manifests:**
    * Create `flask-deployment.yaml` (specifying your ECR image path and `imagePullSecrets`).
    * Create `flask-service.yaml` (exposing `targetPort: 8000` via `LoadBalancer`).
    ```bash
    kubectl apply -f flask-deployment.yaml -f flask-service.yaml
    kubectl rollout restart deployment/flask-app-deployment # To ensure fresh pull
    ```
10. **Monitor Pods:**
    ```bash
    kubectl get pods # Wait for 1/1 Running
    ```
11. **Access Application:**
    ```bash
    minikube service flask-app-service # Opens in browser
    ```

## ⚠️ Troubleshooting

* **`ImagePullBackOff` / `no basic auth credentials`:**
    * **Re-verify `config.json`:** Ensure it's correctly configured with the `credHelpers` entry as detailed in step 5 above. This is the most common cause.
    * Ensure Docker Desktop is running and Minikube was started with `--embed-certs`.
* **`minikube service` errors:** Often related to `config.json` parsing issues or Minikube's internal state. Re-check `config.json` and restart Minikube.

## 🧹 Cleanup

To remove Kubernetes resources and stop Minikube:

```bash
kubectl delete -f flask-deployment.yaml -f flask-service.yaml
minikube stop
# minikube delete (to completely remove cluster data)
```

---
