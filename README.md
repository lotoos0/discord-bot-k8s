# [Discord bot](https://github.com/lotoos0/MusicDiscordBot) — Kubernetes (Minikube) [![pipeline status](https://gitlab.com/lotoos1998/dcbot/badges/main/pipeline.svg)](https://gitlab.com/lotoos1998/dcbot/-/pipelines) 
This repo contains the K8s manifests to run the bot image from the GitLab Container Registry.

---

## Requirements
- Minikube, kubectl
- GitLab Deploy Token from `read_registry`
- Image: `registry.gitlab.com/lotoos1998/dcbot/discord-bot:<TAG>`

---

## Quick start
```bash
minikube start --driver=docker

# Image Pull Secret (imagePullSecret)
kubectl create secret docker-registry regcred \
  --docker-server=registry.gitlab.com \
  --docker-username="k8s-pull" \
  --docker-password="<DEPLOY_TOKEN>" \
  --docker-email="ci@example.com"

Application secret (Discord token)
kubectl create secret generic discord-bot-secret \
  --from-literal=DISCORD_TOKEN="YOUR_TOKEN"
```

---

## Deploy

```bash
# set tag to specific image version
export TAG=a1b2c3d4
sed "s#__TAG__#${TAG}#g" deployment.yaml | kubectl apply -f -
kubectl rollout status deploy/discord-bot
kubectl logs -f deploy/discord-bot
```

---

## Update (rolling update)
```bash
export TAG=<new_commit_hash>
kubectl set image deploy/discord-bot discord-bot=registry.gitlab.com/lotoos1998/dcbot/discord-bot:${TAG}
kubectl rollout status deploy/discord-bot
```

---

## Rollback
```bash
kubectl rollout undo deploy/discord-bot
```

---

## Why not :latest

Short paragraph: unreproducibility, cache issues, lack of auditing - we use commit tags.

---

## Security
- Repo/registry Private + ```imagePullSecrets```.
- Secrets via ```kubectl create secret``` (we do not commit secret.yaml).
- ```secret.example.yaml``` for local templating.

---

## Direct apply:
```bash
kubectl apply -f secret.yaml
kubectl apply -f deployment.yaml

kubectl get pods
kubectl logs deploy/discord-bot
```
---

## Troubleshooting

### 1) ImagePullBackOff / ErrImagePull

**Causes:**
- private registry without `imagePullSecrets`,
- wrong image path/name or missing tag,
- using `:latest` and cache stores old version.

**Check:**
```bash
kubectl describe pod -l app=discord-bot | sed -n '/Events:/,$p'
kubectl get secret regcred -o yaml # is there a secret for the registry 
```
**Fix:**
- Create a registry secret (Deploy Token from `read_registry`):
```bash
kubectl create secret docker-registry regcred \
  --docker-server=registry.gitlab.com \
  --docker-username="k8s-pull" \
  --docker-password="<DEPLOY_TOKEN>" \
  --docker-email="ci@example.com"
```
- In `deployment.yaml` set:
```yaml
spec:
  template:
    spec:
      imagePullSecrets:
        - name: regcred
```
- Use specific tag (SHA), not `latest`:
```bash
TAG=<commit_hash>
kubectl set image deploy/discord-bot \
  discord-bot=registry.gitlab.com/lotoos1998/dcbot/discord-bot:${TAG}
kubectl rollout status deploy/discord-bot
```

### 2) CrashLoopBackOff

**Causes:** application error, missing `DISCORD_TOKEN`, missing `ffmpeg` in image.

Check the logs:
```bash
kubectl logs -f deploy/discord-bot
kubectl get secret discord-bot-secret -o yaml   # is there DISCORD_TOKEN?
```
### 3) Pod Pending / does not assign to node
```bash 
kubectl get events --sort-by=.lastTimestamp | tail -n 20
minikube status
```
Often `minikube stop && minikube start` helps.

### 4) New version not rolling out (old image)

Always change the tag to the new SHA (avoid `:latest`) or restart the rollout:

```bash
kubectl rollout restart deploy/discord-bot
kubectl rollout history deploy/discord-bot
kubectl rollout undo deploy/discord-bot   # rollback
```

### 5) Secret not found

Make sure you create it in the same namespace:
```bash
kubectl get ns
kubectl -n default get secret
```
```yaml

---

# Minor cleanup (for completeness)

- **Replicas = 1** (bot without sharding):
  ```bash
  kubectl scale deploy/discord-bot --replicas=1
```

- Registry back to Private → use `imagePullSecrets` as above.

- Without `:latest`: rollout after the SHA tag (deterministic, easy rollback).

---

