# Module 5 - Blue/Green on Kubernetes

**Goal**: Practise blue/green deployments on a k3s cluster by promoting traffic between two Kubernetes Deployments.

## What to review

- Application code: `app.py`
- Container build: `Dockerfile`
- Kubernetes manifests: `deployment-blue.yml`, `deployment-green.yml`, `service.yml`

## Preparation

1. Pick a unique Kubernetes namespace for your group (for example: `team-sharks`).
2. Replace the `DOCKER_USERNAME` placeholder in both deployment manifests with your Docker Hub username.
3. Replace the nodePort in `service-blue.yml` and `service-green.yml` for your team's port (30040 or 30080)
4. Commit those manifest changes so your pipeline uses the right image repository.

## Exercise tasks

Update the pipeline from exercise 1 to do a blue-green deployment:

1. Keep the first block (**Build and Push Image**)
2. Block 2 should now **Deploy Blue**
   - Import the shared `kubeconfig` secret, configure `kubectl`, switch to your groups namespace (`kubectl config set-context --current --namespace="your-chosen-name"`)
   - Apply `deployment-blue.yml` (`kubectl apply -f deployment-blue.yml`)
   - Apply `service-blue.yml` (`kubectl apply -f service-blue.yml`)
   - Wait for the rollout to complete (`kubectl rollout status deployment/flask-app-blue`)
3. **Verify Blue**
   - Retrieve the service information: `kubectl get svc flask-app`
   - Note the `NODE-PORT` value (for example `30040`) from the command output.
4. Create a manual promotion (a second pipeline) to deploy green
5. **Deploy Green**
   - Import the shared `kubeconfig` secret, configure `kubectl`, switch to your groups namespace (`kubectl config set-context --current --namespace="your-chosen-name"`)
   - Apply `deployment-green.yml` (`kubectl apply -f deployment-blue.yml`)
   - Apply `service-green.yml` (`kubectl apply -f service-green.yml`)
   - Wait for the rollout to complete (`kubectl rollout status deployment/flask-app-green`)
6. **Verify Green**
   - Retrieve the service information: `kubectl get svc flask-app`
   - Note the `NODE-PORT` value (for example `30040`) from the command output.

## Verifying the rollout manually

1. Visit `http://devops.tomfern.com:30040` (or 30080) after deploying blue, you should see the message: "Greetings from Blue"
2. Visit `http://devops.tomfern.com:30040` (or 30080) after deploying green, you should see the message: "Greetings from Green"

## Semaphore secrets

1. **Docker Hub** — create a secret with `DOCKERHUB_USERNAME` and `DOCKERHUB_PASSWORD`.
2. **kubeconfig** — already provided by the instructor
