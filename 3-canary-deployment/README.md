# Module 6 - Canary on Kubernetes

**Goal**: Practise a canary rollout on k3s by serving a new version to a small percentage of traffic before promoting it to all users.

## What to review

- Application code: `app.py`
- Container build: `Dockerfile`
- Kubernetes manifests: `deployment-stable.yml`, `deployment-canary.yml`, `service.yml`

## Preparation

1. Pick a unique Kubernetes namespace for your group (for example: `team-dolphins`).
2. Replace the `DOCKER_USERNAME` placeholder in both deployment manifests with your Docker Hub username.
3. Commit those changes so your pipeline references the correct image repository.

## Exercise tasks

Define a Semaphore workflow with at least three stages that model a canary rollout:

1. **Build and Push Image**
   - Check out the repository.
   - Build the application image.
   - Tag it twice: `docker tag <image> DOCKER_USERNAME/my-flask-image:latest` and `DOCKER_USERNAME/my-flask-image:<short-sha>`, where `<short-sha>` is the first seven characters of `SEMAPHORE_GIT_SHA`.
   - Push both tags to Docker Hub.
   - Store the fully qualified image reference so later stages can deploy it.
2. **Deploy Stable Baseline**
   - Import the shared `kubeconfig` secret and configure `kubectl`.
   - Ensure your namespace exists: `kubectl create namespace <namespace> || true`.
   - Apply `service.yml` (once) so traffic flows through a single Service.
   - Deploy or refresh the stable version: `kubectl apply -f deployment-stable.yml --namespace <namespace>` and update the image to the new tag (`kubectl set image deployment/flask-app-stable flask-app=<image>`).
   - Wait for the rollout to finish (`kubectl rollout status deployment/flask-app-stable --namespace <namespace>`).
3. **Roll Out Canary**
   - Deploy the canary manifest: `kubectl apply -f deployment-canary.yml --namespace <namespace>`.
   - Update the canary Deployment to the new image tag (`kubectl set image deployment/flask-app-canary flask-app=<image>`).
   - Scale the canary to 1 replica and the stable deployment to 9 replicas to maintain a 90/10 traffic split (`kubectl scale deployment/flask-app-canary --replicas=1` and `kubectl scale deployment/flask-app-stable --replicas=9`).
   - Watch the rollout status for both deployments.
   - (Optional) Add verification commands—`kubectl get pods -l track=canary` and `kubectl logs`—to confirm the new pods are healthy.
4. **Promotion / Rollback (Optional Challenge)**
   - Use additional steps or manual commands to scale canary up and stable down for full promotion, or delete the canary deployment to roll back.

## Verifying traffic

1. Retrieve the Service details: `kubectl get svc flask-app --namespace <namespace>`.
2. Note the NodePort (defaults to `30082`) and list node IPs using `kubectl get nodes -o wide`.
3. Visit `http://<node-ip>:<node-port>` repeatedly or run a loop with `curl`; you should see the stable greeting most of the time with occasional canary responses. Adjust replica counts to change the traffic split.

## Semaphore secrets

1. **Docker Hub** — create a secret with `DOCKERHUB_USERNAME` and `DOCKERHUB_PASSWORD`.
2. **kubeconfig** — already supplied; import the shared secret so `kubectl` can access the cluster (`KUBECONFIG_BASE64`, optionally `KUBE_CONTEXT`).
