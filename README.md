# Kubernetes Lab com K3s + Ubuntu VM

## Instalar K3s

```bash
curl -sfL https://get.k3s.io | sh -
```

---

## Verificar cluster

```bash
kubectl get nodes
```

---

## Verificar pods do cluster

```bash
kubectl get pods -A
```

---

# Criar deployment nginx

```bash
vi nginx-deployment.yaml
```

```yaml
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
```

---

# Aplicar deployment

```bash
kubectl apply -f nginx-deployment.yaml
```

---

# Verificar pod

```bash
kubectl get pods
```

---

# Criar service

```bash
vi nginx-service.yaml
```

```yaml
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
```

---

# Aplicar service

```bash
kubectl apply -f nginx-service.yaml
```

---

# Verificar service

```bash
kubectl get svc
```

---

# Descobrir IP da VM

```bash
ip a
```

---

# Abrir no navegador

```text
http://IP_DA_VM:30080
```

---

# Criar ingress

```bash
vi nginx-ingress.yaml
```

```yaml
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
```

---

# Aplicar ingress

```bash
kubectl apply -f nginx-ingress.yaml
```

---

# Verificar ingress

```bash
kubectl get ingress
```

---

# Configurar hosts no Windows

```text
C:\Windows\System32\drivers\etc\hosts
```

Adicionar:

```text
192.168.X.X nginx.local
```

---

# Testar ingress

```text
http://nginx.local
```

---

# Ver uso dos nodes

```bash
kubectl top nodes
```

---

# Ver uso dos pods

```bash
kubectl top pods -A
```

---

# Ver namespaces

```bash
kubectl get ns
```

---

# Criar namespaces

```bash
kubectl create namespace monitoring
kubectl create namespace apps
kubectl create namespace dev
```

---

# Verificar namespaces

```bash
kubectl get ns
```

---

# Remover deployment antigo

```bash
kubectl delete deployment nginx
kubectl delete service nginx-service
```

---

# Editar deployment com namespace apps

```bash
vi nginx-deployment.yaml
```

```yaml
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
```

---

# Editar service com namespace apps

```bash
vi nginx-service.yaml
```

```yaml
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
```

---

# Aplicar novamente

```bash
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
```

---

# Verificar recursos do namespace apps

```bash
kubectl get all -n apps
```

---

# Ver StorageClass

```bash
kubectl get storageclass
```

---

# Criar PVC

```bash
vi nginx-pvc.yaml
```

```yaml
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
```

---

# Aplicar PVC

```bash
kubectl apply -f nginx-pvc.yaml
```

---

# Verificar PVC

```bash
kubectl get pvc -n apps
```

---

# Atualizar deployment com volume persistente

```bash
vi nginx-deployment.yaml
```

```yaml
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
```

---

# Aplicar deployment atualizado

```bash
kubectl apply -f nginx-deployment.yaml
```

---

# Verificar pod

```bash
kubectl get pods -n apps
```

---

# Entrar no container

```bash
kubectl exec -it NOME_DO_POD -n apps -- bash
```

---

# Criar arquivo persistente

```bash
echo "Kubernetes Persistent Volume funcionando" > /usr/share/nginx/html/index.html
```

---

# Sair do container

```bash
exit
```

---

# Testar persistência

```text
http://IP_DA_VM:30080
```

---

# Resultado esperado

```text
Kubernetes Persistent Volume funcionando
```

---

# Prova real da persistência

```bash
kubectl delete pod -n apps --all
```

---

# Kubernetes recriará automaticamente o pod

```bash
kubectl get pods -n apps
```

---

# Mesmo recriando o pod os dados continuarão salvos no Persistent Volume
```

---


# Autor

Rafael Ferreira Neves

---

# Licença

Projeto desenvolvido para fins educacionais e portfólio DevOps/Cloud.
