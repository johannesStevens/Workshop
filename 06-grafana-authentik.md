# Setup Grafana to use Authentik as oauth Provider

## Authentik Settings
- Open Authentik admin: http://authentik.workshop.local/if/admin/
- Go to Applications → Applications → Create
- Name: Grafana
- Slug: grafana

- press next (for provider creation)
- Select OAuth2/OpenID Provider
Name: Grafana
Authorization flow: default-provider-authorization-explicit-consent
Client ID: -> copy this
Client Secret: -> copy this
Redirect URIs: http://grafana.workshop.local/login/generic_oauth
Scopes: openid, email, profile

## Configure Policies
- fyi: ingress to authentik from grafana was already allowed
- for grafana egress: kubectl apply -f 06-grafana-authentik.yaml -n grafana

## Configure Grafana
- kubectl delete deployment grafana -n grafana
- adapt the secret file 08-oauth-secret.yaml with the client id and secret created before
- kubectl apply -f 08-oauth-secret.yaml -n grafana
- kubectl apply -f 09-oauthdeployment.yaml -n grafana

## Check grafana:
- kubectl get pods -n grafana

## check that grafana can access the authentik token endpoint:
- kubectl exec -it -n grafana deploy/grafana -- curl -v http://authentik-server.authentik.svc.cluster.local/application/o/token/ --max-time 5