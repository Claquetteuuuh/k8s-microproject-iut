### Team
- Thomas Biabiany aka "Claquetteuuuh"
- Gatta Ba aka "byood"
- Felmon Tewelde Habtay aka "filex" 
- Abd-el-rahmen Gheribi aka "TawahinBeirut"

## Steps

### Step 1 - Initial project setup

1. Clone this repository

```bash
git clone git@github.com:nas-tabchiche/k8s-microproject.git
```

2. Create your own repository on Github

3. Change the repote to your repository

```bash
git remote set-url origin git@github.com:<github-username>/<repo-name>.git
```

### Step 2 - Install and run the application

Requirements:
- Node 22+
- npm
- cURL

1. Install dependencies

```bash
npm install
```

2. Run the application

```
node app.js
```

3. Send a GET request to the exposed endpoint

```bash
curl http://localhost:3000/
```

The output should be 'Hello, Kubernetes!'

### Step 3 - Dockerize and publish the image

1. Write a Dockerfile

2. Build your image with the `k8s-microproject` tag

```bash
docker build . -t <username>/k8s-microproject
```

3. Publish the image on dockerhub

```bash
docker push <username>/k8s-microproject
```

### Step 4 - Create and expose your first deployment

1. Write a `deployment.yaml` file describing your deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-microproject-deployment
spec:
  ...
```

2. Write a `service.yaml`file describing your service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: k8s-microproject-service
spec:
```

3. Apply your deployment and service

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

4. Check that your pods are running

```bash
kubectl get pods
```

> [!NOTE]
> Their status should be 'Running'. It might take a few seconds to get there.

5. Expose your application

```bash
# If you use minikube
minikube service k8s-microproject-service --url
```

6. Send a GET request to the exposed endpoint

```bash
curl <URL of the exposed service>
```

The output should be 'Hello, Kubernetes!'

### Step 5 - Create an ingress

1. Write a `ingress.yaml` file describing your ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: k8s-microproject-ingress
spec:
  ...
```

2. Apply your ingress

```bash
kubectl apply -f ingress.yaml
```

3. Check that your ingress is operational

```bash
kubectl get ingress
```

4. Send a GET request to the ingress

```bash
curl --resolve "<ingress-host>:80:<ingress-address>" -i http://<ingress-host>/
```

### Step 6 - HTTPS

Create namespace
```bash
kubectl create namespace caddy-system
```

Create certificat
```bash
helm install --namespace=caddy-system --repo h
ttps://caddyserver.github.io/ingress/ --atomic mycaddy caddy-ingress-controller --set ingressControl
ler.config.email="<EMAIL>" --set loadBalancer.annotations."service\.beta\.kubernet
es\.io/azure-dns-label-name"="<DNS_PREFIX>"
```

### Create certificat
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=api.com/O=api.com"

kubectl create secret tls api-com-tls --cert=tls.crt --key=tls.key
```

Use -k to bypass tls verification
```bash
curl --resolve "api.com:443:192.168.49.2" -ik https://api.com/
```

### Step 7 - Add Liveness Probe
We Updated `deployment.yaml` to include a liveness probe
#### Verify the liveness probe is working
```bash
# Watch pod status
kubectl get pods -w

# Check pod details and events
kubectl describe pod <pod-name>
```
#### Test the probe by simulating a failure

```bash
# Get pod name
kubectl get pods

# Temporarily modify the probe path to trigger failure
kubectl patch deployment k8s-microproject-deployment --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/livenessProbe/httpGet/path", "value":"/non-existent"}]'

# Watch the pods restart
kubectl get pods -w

# Restore the correct probe path
kubectl patch deployment k8s-microproject-deployment --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/livenessProbe/httpGet/path", "value":"/"}]'
```
### Step 8 - Add Persistent Volume
Apply the PV and PVC:
```bash
kubectl apply -f pv-pvc.yaml

# check if it is running
kubectl get pvc k8s-microproject-pvc
```
#### Test the PVC
Verify Binding:
```bash
kubectl get pvc k8s-microproject-pvc
```
Check Volume in Pod:
```bash
kubectl exec -it <pod-name> -- sh
ls /usr/src/app/data
```
Test Persistence inside the pod:
```bash
echo "Test data" > /usr/src/app/data/test.txt
ls /usr/src/app/data
```
Delete the pod to force a restart:
```bash
kubectl delete pod <pod-name>
```
Then check the new pod to see the previous data:
```bash
kubectl exec -it <new-pod-name> -- sh
ls /usr/src/app/data
cat /usr/src/app/data/test.txt
```

### Step 9 - Add the ConfigMap
Deploy the ConfigMap to your cluster by running:
```bash
kubectl apply -f configmap.yaml

# After updating the deployment.yaml to add configMap
kubectl apply -f deployment.yaml
```
#### Test the ConfigMap Integration
```bash
# Get a Running Pod Name:
kubectl get pods

# Exec into the Pod:
kubectl exec -it <pod-name> -- sh
# Replace <pod-name> with the name of one of your running pods.

#Verify Environment Variables:
#Inside the pod, run:

echo $GREETING
echo $PORT
```
You should see the following output:
```bash
Hello Kubernetes with ConfigMap!
3000
```
### Step 10 - Add a StatefulSet
If you want each pod to get a stable identity and its own persistent storage. Create a StatefulSet Manifest in `statefulset.yaml`
Deploy the StatefulSet by running:
```bash
kubectl apply -f statefulset.yaml
```
#### Verify the StatefulSet and Pods

Check that your StatefulSet and pods are created and running:
```bash
kubectl get statefulset
kubectl get pods -l app=k8s-microproject
```
You should see pods named similar to k8s-microproject-statefulset-0, k8s-microproject-statefulset-1, etc.

#### Test Persistence and Environment Variables
```bash
# Exec into one of the Pods:

kubectl exec -it k8s-microproject-statefulset-0 -- sh

# Verify Environment Variables:
# Inside the pod, run:
echo $GREETING
echo $PORT
```bash

You should see:
```bash
Hello Kubernetes with ConfigMap!
3000
```
Check Persistent Storage:
```bash
# List the files in the mounted directory:
ls /usr/src/app/data
```

If you used the init container to copy data, you may see the pre-populated files. Otherwise, the directory might be empty until your application writes data.

## If you need to delete
```bash
kubectl delete deployment k8s-microproject-deployment
```

```bash
kubectl delete ingress k8s-microproject-ingress
```

nginx admission config
```bash
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```

## Run tunnel
```bash
minikube tunnel
```

## Minikube addon
```bash
minikube addons enable ingress
```
