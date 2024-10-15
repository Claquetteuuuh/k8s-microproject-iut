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
node app.ts
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
