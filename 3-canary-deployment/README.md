# Module 5 - Canary on Kubernetes

**Goal**: Practise a canary rollout on Kubernetes by serving a new version to a small percentage of traffic before promoting it to all users.

## What to review

- Application code: `app.py`
- Container build: `Dockerfile`
- Kubernetes manifests: `deployment-stable.yml`, `deployment-canary.yml`, `service.yml`

## Preparation

1. Pick a unique Kubernetes namespace for your group (for example: `team-dolphins`).
2. Replace the `DOCKER_USERNAME` placeholder in both deployment manifests with your Docker Hub username.
3. Replace the nodePort in `service.yml` for your team's port (30040 or 30080)
4. Commit those changes so your pipeline references the correct image repository.

## Exercise tasks

Update the pipeline from exercise 3 to do a Canary deployment:

1. Keep the first block (**Build and Push Image**)
2. Block 2 should now **Deploy Stable**
   - Import the shared `kubeconfig` secret, configure `kubectl`, switch to your groups namespace (`kubectl config set-context --current --namespace="your-chosen-name"`)
   - Apply `deployment-stable.yml` (`kubectl apply -f deployment-stable.yml`)
   - Apply `service.yml` (`kubectl apply -f service.yml`)
   - Wait for the rollout to complete (`kubectl rollout status deployment/flask-app-stable`)
3. **Verify Stable**
   - Retrieve the service information: `kubectl get svc flask-app`
   - Note the `NODE-PORT` value (for example `30040`) from the command output.
4. Create a manual promotion (a second pipeline) to deploy canary
5. **Deploy Canary**
   - Import the shared `kubeconfig` secret, configure `kubectl`, switch to your groups namespace (`kubectl config set-context --current --namespace="your-chosen-name"`)
   - Apply `deployment-canary.yml` (`kubectl apply -f deployment-canary.yml`)
   - Wait for the rollout to complete (`kubectl rollout status deployment/flask-app-canary`)
6. **Verify Canary**
   - Retrieve the service information: `kubectl get svc flask-app`
   - Note the `NODE-PORT` value (for example `30040`) from the command output.

## Verifying the rollout manually

Visit `http://devops.tomfern.com:30040` (or 30080) after deploying stable and canary, you should see the canary application about 1/6th of the times

## Semaphore secrets

1. **Docker Hub** — create a secret with `DOCKERHUB_USERNAME` and `DOCKERHUB_PASSWORD`.
2. **kubeconfig** — already supplied; import the shared secret so `kubectl` can access the cluster (`KUBECONFIG_BASE64`, optionally `KUBE_CONTEXT`).
