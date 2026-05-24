# Sources:
- prometheus-operator: https://github.com/prometheus-operator/prometheus-operator

# Network Policies
Right now all traffic between all pods is allowed. Fix it:

## Whoami 

### Default Deny
- kubectl apply -f 01-default-deny.yaml -n whoami
- test connection: curl http://whoami.workshop.local/
should result in timeout

### whoami allow ingress traffic from traefik namespace
- kubectl apply -f 02-whoami.yaml -n whoami
- test connection: curl http://whoami.workshop.local/


## Authentik

### apply policies
- kubectl apply -f 01-default-deny.yaml -n authentik
- kubectl apply -f 03-authentik.yaml -n authentik
- test connection in browser: http://authentik.workshop.local


### test pod connections
- Test inner namespace connection (should work): kubectl exec -it -n authentik deploy/authentik-server -- curl -v telnet://authentik-postgresql:5432 --max-time 3
- Test outer namespace connection to whoami (shouldn't work): kubectl exec -it -n authentik deploy/authentik-server -- curl -v http://whoami.whoami.svc.cluster.local --max-time 3


## Grafana

### apply policies
- kubectl apply -f 01-default-deny.yaml -n grafana
- kubectl apply -f 03-grafana.yaml -n grafana
- test connection in browser: http://grafana.workshop.local


### test pod connections
- Test outer namespace connection to authentik (shouldn't work): kubectl exec -it -n grafana deploy/grafana -- curl -v http://authentik-server.authentik.svc.cluster.local --max-time 3


## traefik 

### apply policies
- kubectl apply -f 01-default-deny.yaml -n traefik
(now nothing should be reachable)
- kubectl apply -f 05-traefik.yaml -n traefik

# Remove default deny
- kubectl delete networkpolicy default-deny-ingress -n <namespace>