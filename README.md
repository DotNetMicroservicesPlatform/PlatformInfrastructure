## Create local docker hub (container registry)
```powershell

# https://www.codeproject.com/Articles/1263817/How-to-Setup-Our-Own-Private-Docker-Registry

docker pull registry

# Create local docker hub (container registry)
docker run -d -p 5000:5000 -v D:/Dev/Docker/ContainerRegistry:/var/lib/registry --restart=always --name container-registry.local registry
```

## Kubernetes setup
```powershell
kubectl config get-contexts

# Change Kubernetes current cluster
kubectl config set current-context  docker-desktop
```

## Install NGINX Ingress
```powershell 

$ingressnamespace="ingress-nginx"

#using yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/cloud/deploy.yaml

#using helm 
helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace
```


## Create Ingress Service
```powershell 

kubectl apply -f .\kubernetes\nginx-ingress.yaml -n $ingressnamespace
```

## Installing Emissary-ingress
```powershell
helm repo add datawire https://app.getambassador.io
helm repo update


kubectl apply -f https://app.getambassador.io/yaml/emissary/3.3.1/emissary-crds.yaml
kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system

$emnamespace="emissary"
kubectl create namespace $emnamespace
$dnsname="platform"
helm install emissary-ingress datawire/emissary-ingress --set service.annotations."service\.beta\.kubernetes\.io/azure-dns-label-name"=$dnsname --namespace $emnamespace  
kubectl rollout status deployment/emissary-ingress -n $emnamespace -w

#Check pods & service
kubectl get pods -n $emnamespace
kubectl get service emissary-ingress -n $emnamespace
```

## Configuring Emissary-ingress routing
```powershell
kubectl apply -f .\emissary-ingress\listener.yaml -n $emnamespace

kubectl apply -f .\emissary-ingress\mappings.yaml -n $emnamespace

kubectl apply -f .\emissary-ingress\host.yaml -n $emnamespace
```


## Create PersistentVolumeClaim
```powershell 

kubectl apply -f .\kubernetes\local-pvc.yaml -n $ingressnamespace
```

## Create Secret
```powershell 
kubectl create secret generic mssql --from-literal=SA_PASSWORD="HardRock#7o7"

kubectl get secret mssql -o jsonpath="{.data.SA_PASSWORD}" | base64 --decode
kubectl get secret mssql -o jsonpath="{.data.SA_PASSWORD>}" | base64 --decode
kubectl get secrets/mssql --template={{.data.SA_PASSWORD}} | base64 --decode

```

## Deploy SqlServer
```powershell 
kubectl apply -f .\kubernetes\mssql-deployment.yaml
```

## Deploy RabbitMQ
```powershell 
kubectl apply -f .\kubernetes\rabbitmq-deployment.yaml
```