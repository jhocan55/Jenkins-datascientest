# FastAPI DevOps Project

This project demonstrates a complete CI/CD pipeline for deploying a FastAPI application using Docker, Jenkins, and Kubernetes (with Helm).

## Project Structure

```
Jenkins_devops/
├── app/
│   └── main.py                # FastAPI application code
├── fastapi/
│   ├── .helmignore            # Helm ignore file
│   ├── Chart.yaml             # Helm chart metadata
│   ├── values.yaml            # Helm chart values (image, service, etc.)
│   └── templates/             # Helm templates for Kubernetes resources
│       ├── deployment.yaml
│       ├── hpa.yaml
│       ├── ingress.yaml
│       ├── NOTES.txt
│       ├── service.yaml
│       ├── serviceaccount.yaml
│       ├── _helpers.tpl
│       └── tests/
│           └── test-connection.yaml
├── Dockerfile                 # Docker build file for FastAPI app
├── Jenkinsfile                # Jenkins pipeline for CI/CD
├── Requirements.txt           # Python dependencies
└── README.md                  # Project documentation (this file)
```

## Main Components

### FastAPI Application
- Located in `app/main.py`.
- Simple API with a root endpoint returning a welcome message.

### Docker
- `Dockerfile` builds a production-ready image using Python 3.9, FastAPI, and Uvicorn.
- The image exposes the app on port 80.

### Jenkins Pipeline
- `Jenkinsfile` automates:
  - Docker image build and run
  - Acceptance testing (curl)
  - Docker Hub login and push
  - Helm-based deployment to Kubernetes (dev, staging, prod)
- Uses environment variables for Docker credentials and Kubernetes config.

### Helm Chart
- Located in `fastapi/`.
- Deploys the FastAPI app to Kubernetes with configurable image, service type, and scaling.
- Supports NodePort service and custom Docker image repository/tag.

## How to Use

1. **Build and Run Locally**
   ```bash
   docker build -t <your_dockerhub_id>/datascientestapi:latest .
   docker run -d -p 80:80 <your_dockerhub_id>/datascientestapi:latest
   curl http://localhost
   ```

2. **CI/CD with Jenkins**
   - Configure Jenkins with Docker and Helm.
   - Set up credentials for Docker Hub and Kubernetes config.
   - Trigger the pipeline to build, test, push, and deploy.

3. **Kubernetes Deployment**
   - Customize `fastapi/values.yaml` for your image repo/tag and service type.
   - Use Helm to deploy:
     ```bash
     helm upgrade --install app fastapi --values=fastapi/values.yaml --namespace <namespace>
     ```

## Requirements
- Docker
- Python 3.9+
- Jenkins (with Docker and Helm access)
- Kubernetes cluster
- Helm

## Authors
- Replace with your name or team

## License
- MIT (or your preferred license)

---

## Jenkinsfile Deep Explanation

The `Jenkinsfile` defines a CI/CD pipeline for building, testing, pushing, and deploying your FastAPI application using Docker and Helm. Here’s a breakdown of each part:

### 1. Environment Variables
- `DOCKER_ID`: Your Docker Hub username (e.g., `jhocan55`).
- `DOCKER_IMAGE`: The name of the Docker image (e.g., `datascientestapi`).
- `DOCKER_TAG`: The tag for the Docker image, using the Jenkins build number for versioning (e.g., `v.23.0`).

### 2. Stages

#### a. Docker Build
- Removes any running container named `jenkins` (to avoid conflicts).
- Builds a Docker image from your `Dockerfile` and tags it as `$DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG`.

#### b. Docker Run
- Runs the built Docker image as a container named `jenkins` and maps port 80.
- Waits to ensure the container is up.

#### c. Test Acceptance
- Uses `curl` to check if the FastAPI app is responding on `localhost`.
- This is a basic health check.

#### d. Docker Push
- Logs in to Docker Hub using credentials stored in Jenkins (as a secret text credential, e.g., `DOCKER_HUB_PASS`).
- Pushes the built image to your Docker Hub repository.
- **Tip:** For security, use `--password-stdin` instead of `-p` for `docker login`.

#### e. Deployment (dev, staging, prod)
- For each environment, retrieves the Kubernetes config from Jenkins credentials (secret file, e.g., `config`).
- Prepares a `.kube/config` file for `kubectl` and `helm` to use.
- Copies and updates `fastapi/values.yaml` to set the correct image tag for deployment.
- Runs `helm upgrade --install` to deploy or update the app in the specified namespace (`dev`, `staging`, `prod`).
- The production stage includes a manual approval step (with a 15-minute timeout) before deploying.

### 3. Credentials in Jenkins
- **DOCKER_HUB_PASS**: Add as a "Secret Text" credential in Jenkins (with your Docker Hub password or token).
- **config**: Add as a "Secret File" credential in Jenkins (your kubeconfig file for cluster access).

---

## Helm Setup and Usage

### What is Helm?
Helm is a package manager for Kubernetes. It uses "charts" (like templates) to define, install, and upgrade applications in a Kubernetes cluster.

### Project Helm Chart
- Located in the `fastapi/` directory.
- Contains templates for Kubernetes resources: Deployment, Service, Ingress, HPA, ServiceAccount, etc.
- `values.yaml` allows you to configure image repository, tag, service type, and more.

### How to Use Helm in This Project

1. **Install Helm**
   ```bash
   curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
   # or use your package manager
   ```

2. **Configure values.yaml**
   - Set your Docker image repository and tag:
     ```yaml
     image:
       repository: jhocan55/datascientestapi
       tag: v.1.0.0
     service:
       type: NodePort
       port: 80
     ```

3. **Deploy to Kubernetes**
   ```bash
   helm upgrade --install app fastapi --values=fastapi/values.yaml --namespace dev
   # Change --namespace as needed (dev, staging, prod)
   ```

4. **Update Image Tag**
   - The Jenkins pipeline automatically updates the image tag in `values.yaml` before each deployment.

5. **Check Deployment**
   ```bash
   kubectl get pods -n dev
   kubectl get svc -n dev
   ```

### Helm Chart Files
- `Chart.yaml`: Metadata about the chart.
- `values.yaml`: Default configuration values.
- `templates/`: Kubernetes resource templates (YAML files with Helm templating syntax).
- `.helmignore`: Patterns to exclude from the chart package.

---

For more details, see the comments in your Jenkinsfile and Helm chart files, or ask for specific examples!
