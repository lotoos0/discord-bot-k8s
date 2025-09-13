# [Discord bot](https://github.com/lotoos0/MusicDiscordBot) — Kubernetes (Minikube)

Ten repo zawiera manifesty K8s do uruchomienia obrazu bota z GitLab Container Registry.

## Wymagania
- Minikube, kubectl
- Obraz: registry.gitlab.com/lotoos1998/dcbot/discord-bot:latest

## Szybki start
```bash
minikube start --driver=docker

# (opcjonalnie) jeśli chcesz trzymać sekret poza repo:
kubectl create secret generic discord-bot-secret \
  --from-literal=DISCORD_TOKEN='WSTAW_TUTAJ_TOKEN' \
  -n default
# wówczas nie stosuj secret.yaml

# standardowo:
kubectl apply -f secret.yaml
kubectl apply -f deployment.yaml

kubectl get pods
kubectl logs deploy/discord-bot

