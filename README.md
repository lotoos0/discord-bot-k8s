# [Discord bot](https://github.com/lotoos0/MusicDiscordBot) — Kubernetes (Minikube)

This repo contains the K8s manifests to run the bot image from the GitLab Container Registry.

## Requirements
- Minikube, kubectl
- GitLab Deploy Token from `read_registry`
- Image: `registry.gitlab.com/lotoos1998/dcbot/discord-bot:<TAG>`

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

## Deploy

```bash
# set tag to specific image version
export TAG=a1b2c3d4
sed "s#__TAG__#${TAG}#g" deployment.yaml | kubectl apply -f -
kubectl rollout status deploy/discord-bot
kubectl logs -f deploy/discord-bot
```

## Update (rolling update)
```bash
export TAG=<new_commit_shortcut>
kubectl set image deploy/discord-bot discord-bot=registry.gitlab.com/lotoos1998/dcbot/discord-bot:${TAG}
kubectl rollout status deploy/discord-bot
```
## Rollback
```bash
kubectl rollout undo deploy/discord-bot
```

## Why not :latest

Short paragraph: unreproducibility, cache issues, lack of auditing - we use commit tags.

## Security
- Repo/registry Private + ```imagePullSecrets```.
- Secrets via ```kubectl create secret``` (we do not commit secret.yaml).
- ```secret.example.yaml``` for local templating.


## Standard:
```bash
kubectl apply -f secret.yaml
kubectl apply -f deployment.yaml

kubectl get pods
kubectl logs deploy/discord-bot
```


