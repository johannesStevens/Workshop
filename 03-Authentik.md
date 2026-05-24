# Sources
- https://docs.goauthentik.io/install-config/install/kubernetes/
- https://longhorn.io/docs/1.11.2/deploy/install/

# Authentik
Helm install example with some errors

## Prerequisites
- secure passwords can be generated with: openssl rand 60 | base64 -w 0

## Helm install
- Adapt the Authentik/value.yaml file
- helm repo add authentik https://charts.goauthentik.io
- helm repo update
- helm upgrade --install authentik authentik/authentik -f values.yaml

## Check Install
- helm status authentik
- kubectl get pods
- kubectl get svc
If pods are stuck, check:
- kubectl describe pod <pod-name>
- kubectl logs <pod-name>
The result of this check should be a fail due to Persistent Volume issue

## Remedy Persistent Volume issue
- Create a persisten Volume Solution using Longhorn then continue with Authentik

# Longhorn

## Prerequisites
need to be installed on all nodes that should run longhorn

### open-iscsi
check if installed:
- systemctl status iscsid
- iscsiadm --version
if fail install:
- sudo yum install -y iscsi-initiator-utils
- sudo systemctl enable --now iscsid
- recheck: systemctl status iscsid
- recheck: iscsiadm --version

### Check NFSv4 client
- rpm -q nfs-utils

### Check filesystem type (both command should prodcue ext4 or XFS)
- df -Th /
- lsblk -f

### Check mount propagation (Should show "shared")
- findmnt -o TARGET,PROPAGATION /

## Helm install
- helm repo add longhorn https://charts.longhorn.io
- helm repo update
- helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace

## check install
All pods (manager, driver, UI, etc.) should show Running/Ready. The longhorn StorageClass should appear in the list.
- kubectl get pods -n longhorn-system
- kubectl get storageclass
- kubectl get lhn -n longhorn-system
If pods are stuck, check:
- kubectl describe pod <pod-name> -n longhorn-system
- kubectl logs <pod-name> -n longhorn-system


# fix authentik using longhorn

## Make sure longhorn storage is set to default:
- check: kubectl get storageclass
if not default run:
- kubectl patch storageclass longhorn -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

## get rid of stuck PVC
kubectl delete pvc -l app.kubernetes.io/instance=authentik
helm upgrade --install authentik authentik/authentik -f values.yaml

## verify authentik again
- kubectl get pods
- kubectl get svc
If pods are stuck, check:
- kubectl describe pod <pod-name>
- kubectl logs <pod-name>

# Clean install authentik
When installing authentik we didn't provide a namespace. We need to uninstall and install in the correct namespace

## uninstall
- helm uninstall authentik
- kubectl delete pvc -l app.kubernetes.io/instance=authentik

## reinstall 
- helm upgrade --install authentik authentik/authentik -f values.yaml --namespace authentik --create-namespace

## verify authentik again
- kubectl get pods -n authentik
- kubectl get svc -n authentik
If pods are stuck, check:
- kubectl describe pod <pod-name> -n authentik
- kubectl logs <pod-name> -n authentik


# Authentik ingress

## install
- kubectl apply -f 01-ingressRoute.yaml -n authentik

## verify installation:
- open http://authentik.workshop.local in browser
- set admin user and password