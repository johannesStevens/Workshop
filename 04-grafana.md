# Sources
- grafana: https://grafana.com/docs/grafana/latest/setup-grafana/installation/kubernetes/
- port forwarding: https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/

# Grafana
kubectl install using yaml files

## Install
- kubectl create namespace grafana
- kubectl apply -f 01-pvc.yaml --namespace=grafana
- kubectl apply -f 02-deployment.yaml --namespace=grafana
- kubectl apply -f 03-loadbalancer.yaml --namespace=grafana

## verify install
- kubectl get pvc --namespace=grafana -o wide
- kubectl get deployments --namespace=grafana -o wide
- kubectl get svc --namespace=grafana -o wide

## Access from outside
- find external ip if possible: kubectl get all --namespace=grafana
- access using externalIP:3000
- if not use port forwarding (for debugging only): kubectl port-forward service/grafana 3000:3000 --namespace=grafana
- access using ClusterIP:3000 (login admin admin) (on localhost: http://localhost:3000/)
- stop port forwarding using ctrl+c


## access from outside using ingressRoute
- kubectl apply -f 04-ingressRoute.yaml --namespace=grafana

