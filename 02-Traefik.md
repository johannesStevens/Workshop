# Sources:
- https://doc.traefik.io/traefik/getting-started/kubernetes/

# Traefik
Helm install example

## Prerequisites:
- gatway crds: kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.5.1/standard-install.yaml

## Install
- helm repo add traefik https://traefik.github.io/charts
- helm repo update
- helm install traefik traefik/traefik -f values.yaml --namespace traefik --create-namespace --wait

## check install
- kubectl describe GatewayClass traefik
- kubectl get pods -n traefik
- kubectl get svc -n traefik

# Test Application whoami

## Install

- kubectl create namespace whoami
- kubectl apply -f 01-deployment.yaml -n whoami
- kubectl apply -f 02-service.yaml -n whoami
- kubectl apply -f 03-ingressRoute.yaml -n whoami

# check install

- kubectl get pods -n whoami
- curl http://whoami.workshop.local