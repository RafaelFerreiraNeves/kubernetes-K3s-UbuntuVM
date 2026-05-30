# Kubernetes Lab com K3s + Ubuntu VM

# Instalar K3s

curl -sfL https://get.k3s.io | sh -

# Verificar cluster

kubectl get nodes

# Verificar pods do cluster

kubectl get pods -A

# Criar deployment nginx

vi nginx-deployment.yaml


apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx

spec:
  replicas: 1

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: nginx:latest

        ports:
        - containerPort: 80


# Aplicar deployment

kubectl apply -f nginx-deployment.yaml

# Verificar pod

kubectl get pods

# Criar service

vi nginx-service.yaml


apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  type: NodePort

  selector:
    app: nginx

  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080


# Aplicar service

kubectl apply -f nginx-service.yaml

# Verificar service

kubectl get svc

# Descobrir IP da VM

ip a

# Abrir no navegador

http://IP_DA_VM:30080

# Criar ingress

vi nginx-ingress.yaml


apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: nginx-ingress

spec:
  rules:
  - host: nginx.local

    http:
      paths:
      - path: /
        pathType: Prefix

        backend:
          service:
            name: nginx-service

            port:
              number: 80


# Aplicar ingress

kubectl apply -f nginx-ingress.yaml

# Verificar ingress

kubectl get ingress

# Configurar hosts no Windows

C:\Windows\System32\drivers\etc\hosts

Adicionar:

192.168.X.X nginx.local

# Testar ingress

http://nginx.local

# Ver uso dos nodes

kubectl top nodes

# Ver uso dos pods

kubectl top pods -A

# Ver namespaces

kubectl get ns

# Criar namespaces

kubectl create namespace monitoring
kubectl create namespace apps
kubectl create namespace dev

# Verificar namespaces

kubectl get ns

# Remover deployment antigo

kubectl delete deployment nginx
kubectl delete service nginx-service

# Editar deployment com namespace apps

vi nginx-deployment.yaml


apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment
  namespace: apps

spec:
  replicas: 1

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: nginx:latest

        ports:
        - containerPort: 80


# Editar service com namespace apps

vi nginx-service.yaml


apiVersion: v1
kind: Service

metadata:
  name: nginx-service
  namespace: apps

spec:
  type: NodePort

  selector:
    app: nginx

  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080


# Aplicar novamente

kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml

# Verificar recursos do namespace apps

kubectl get all -n apps

# Ver StorageClass

kubectl get storageclass

# Criar PVC

vi nginx-pvc.yaml


apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: nginx-pvc
  namespace: apps

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 1Gi


# Aplicar PVC

kubectl apply -f nginx-pvc.yaml

# Verificar PVC

kubectl get pvc -n apps

# Atualizar deployment com volume persistente

vi nginx-deployment.yaml


apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment
  namespace: apps

spec:
  replicas: 1

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: nginx:latest

        ports:
        - containerPort: 80

        volumeMounts:
        - name: nginx-storage
          mountPath: /usr/share/nginx/html

      volumes:
      - name: nginx-storage
        persistentVolumeClaim:
          claimName: nginx-pvc


# Aplicar deployment atualizado

kubectl apply -f nginx-deployment.yaml

# Verificar pod

kubectl get pods -n apps

# Entrar no container

kubectl exec -it NOME_DO_POD -n apps -- bash

# Criar arquivo persistente

echo "Kubernetes Persistent Volume funcionando" > /usr/share/nginx/html/index.html

# Sair do container

exit

# Testar persistência

http://IP_DA_VM:30080

# Resultado esperado

Kubernetes Persistent Volume funcionando

# Prova real da persistência

kubectl delete pod -n apps --all

# Kubernetes recriará automaticamente o pod

kubectl get pods -n apps

# Mesmo recriando o pod os dados continuarão salvos no Persistent Volume