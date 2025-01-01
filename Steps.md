CI-Server : t2.micro 

Deployment-Server : t2.xlarge

CI-Server :
```
# For Docker Installation
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER && newgrp docker
```

git clone https://github.com/Shabbirsyed05/reddit-clone-k8s-ingress.git

```
cd reddit-clone-yt
vim Dockerfile
```
```
FROM node:19-alpine3.15

WORKDIR /reddit-clone

COPY . /reddit-clone

RUN npm install 

EXPOSE 3000

CMD ["npm","run","dev"]
```
```
docker build -t <DockerHub_Username>/reddit-clone  .
docker build -t shabbirsyed103/reddit-clone .
docker login
docker push <DockerHub_Username>/<Imagename>
docker push shabbirsyed103/reddit-clone
```

Deployment-Server :
```
# For Docker Installation
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER && newgrp docker

# For Minikube & Kubectl
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube 

sudo snap install kubectl --classic
minikube start --driver=docker
```

kubectl get pods
mkdir k8s
cd k8s

vim deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reddit-clone-deployment
  labels:
    app: reddit-clone
spec:
  replicas: 2
  selector:
    matchLabels:
      app: reddit-clone
  template:
    metadata:
      labels:
        app: reddit-clone
    spec:
      containers:
      - name: reddit-clone
        image: trainwithshubham/reddit-clone
        ports:
        - containerPort: 3000
```
```
kubectl apply -f deployment.yaml
kubectl get deployment
kubectl get deployment
```

vim service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: reddit-clone-service
  labels:
    app: reddit-clone
spec:
  type: NodePort
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 31000
  selector:
    app: reddit-clone
```
```
kubectl apply -f service.yaml
minikube service reddit-clone-service --url (it will provide the url)
curl -L url (it will be running the instance)
kubectl expose deployment reddit-clone-deployment --type=NodePort
kubectl port-forward svc/reddit-clone-service 3000:3000 --address 0.0.0.0 & (security group -> open port 3000)
ip:3000 
```
```
minikube addons enable ingress
minikube addons list
```
vim ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-reddit-app
spec:
  rules:
  - host: "domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000
  - host: "*.domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000
```

curl -L domain.com/test (it will open app in code)
kubectl apply -f ingress.yml
