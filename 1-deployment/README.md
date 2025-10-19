# Module 5 - Kubernetes deployments

**Goal**: Deploy a single environment to a cloud Kubernetes cluster using CI/CD.

## What to review

- Application code: `app.py`
- Container build: `Dockerfile`
- Kubernetes manifests: `deployment.yml`, `service.yml`

## Goal

Your goal is to create a Semaphore pipeline that builds the Docker image, pushes it to Docker Hub, and deploys the updated image to the Kubernetes cluster.

## Preparation

1. Review `deployment.yml` and `service.yml` so you understand how the application is deployed and exposed.
2. Pick a unique Kubernetes namespace for your group (for example: `team-sharks`).
3. Edit `deployment.yml` and replace `DOCKER_USERNAME` with your Docker Hub username
4. Edit `service.yml` and change `nodePort` to either 30040 or 30080 (each group should pick a different port)
4. Commit the changes to the repository

## Exercise tasks

Define a Semaphore workflow with three blocks:

1. **Build and Push Image**
   - Check out the repository.
   - Build the application image with name: `$DOCKERHUB_USERNAME/my-flask-image:latest`
   - Push image your Docker Hub repository.
2. **Deploy to Kubernetes**
   - Configure `kubectl` using the provided `kubeconfig` secret.
   - Create the namespace if it does not already exist: `kubectl create namespace "your-chosen-name" || true`.
   - Switch to the namespace: `kubectl config set-context --current --namespace="your-chosen-name"`
   - Deploy the manifests into that namespace:
     - `kubectl apply -f deployment.yml`
     - `kubectl apply -f service.yml`
   - Wait for the rollout to complete (`kubectl rollout status deployment/flask-app`)
3. **Verify**
   - Retrieve the service information: `kubectl get svc flask-app`
   - Note the `NODE-PORT` value (for example `30040`) from the command output.

When the pipeline finishes, visit `http://devops.tomfern.com:<nodePort>` (replace with the worker node address and the NodePort obtained above) to confirm the greeting appears in your browser.

## Semaphore secrets

1. Docker Hub credentials — you must create this secret yourself and include:
   - `DOCKERHUB_USERNAME`
   - `DOCKERHUB_PASSWORD`
2. Kubernetes access — already provided by the instructor. Import the existing shared secret `kubeconfig` into your job
   - This secret exposes `KUBECONFIG_BASE64` and, optionally, `KUBE_CONTEXT`.
