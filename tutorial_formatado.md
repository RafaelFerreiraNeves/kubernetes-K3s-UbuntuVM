Transforme todo esse texto em markdown

Deployment

## Instalar K3s

Agora vem a parte boa.

Rode:

```bash
curl -sfL https://get.k3s.io | sh -
```

## Verificar

```bash
root@rafaelferreiraneves:~# kubectl get nodes
```

NAME                                  STATUS   ROLES           AGE   VERSION

rafaelferreiraneves-virtual-machine   Ready    control-plane   23m   v1.35.5+k3s1

```bash
root@rafaelferreiraneves:~# kubectl get pods -A
```

NAMESPACE     NAME                                      READY   STATUS      RESTARTS   AGE

default       nginx-deployment-59f86b59ff-nj4ls         1/1     Running     0          37m

kube-system   coredns-8db54c48d-nvc77                   1/1     Running     0          40m

kube-system   helm-install-traefik-crd-67dpx            0/1     Completed   0          40m

kube-system   helm-install-traefik-k8lsf                0/1     Completed   2          40m

kube-system   local-path-provisioner-5d9d9885bc-8dms2   1/1     Running     0          40m

kube-system   metrics-server-786d997795-fg5lk           1/1     Running     0          40m

kube-system   svclb-traefik-e9e10035-9xx8c              2/2     Running     0          39m

kube-system   traefik-9bcdbbd9-7v4cf                    1/1     Running     0          39m

## Primeiro deploy

```bash
vi  nginx-deployment.yaml
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
```

```yaml
  template:
    metadata:
      labels:
        app: nginx
```

```yaml
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

```yaml
        ports:
        - containerPort: 80
```

## Aplicar

```bash
root@rafaelferreiraneves# kubectl apply -f nginx-deployment.yaml
```

deployment.apps/nginx-deployment created

## Verificar

```bash
root@rafaelferreiraneves:~# kubectl get pods
```

NAME                                READY   STATUS    RESTARTS   AGE

nginx-deployment-59f86b59ff-nj4ls   1/1     Running   0          41m

## Criar Service

```bash
vi nginx-service.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
```

```yaml
spec:
  type: NodePort
```

```yaml
  selector:
    app: nginx
```

```yaml
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

## Aplicar

```bash
root@rafaelferreiraneves:~# kubectl apply -f nginx-service.yaml
```

service/nginx-service created

## Verificar

```bash
root@rafaelferreiraneves:~# kubectl get svc
```

NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE

kubernetes      ClusterIP   10.43.0.1      <none>        443/TCP        48m

nginx-service   NodePort    10.43.194.56   <none>        80:30080/TCP   31m

## Acessar no navegador

```bash
http://IP_DA_VM:30080
```

## Descobrir IP da VM

```bash
ip a, Procure algo como:
```

192.168.x.x

```bash
root@rafaelferreiraneves-virtual-machine:~# ip a
```

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000

```yaml
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
```

2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000

```yaml
    link/ether 00:0c:29:bb:2d:09 brd ff:ff:ff:ff:ff:ff
    altname enp2s1
    inet 192.168.37.130/24 brd 192.168.37.255 scope global dynamic noprefixroute ens33
       valid_lft 1190sec preferred_lft 1190sec
    inet6 fe80::2003:1a7c:2691:e243/64 scope link noprefixroute
```

```bash
http://IP_DA_VM:30080
```

Exemplo:

```bash
http://192.168.0.15:30080
```

Se aparecer a página do Nginx:

✅ Kubernetes funcionando

✅ Rede funcionando

✅ Service funcionando

✅ Pod funcionando

## Criar Ingress

```bash
vi nginx-ingress.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
```

```yaml
metadata:
  name: nginx-ingress
```

```yaml
spec:
  rules:
  - host: nginx.local
```

```yaml
    http:
      paths:
      - path: /
        pathType: Prefix
```

```yaml
        backend:
          service:
            name: nginx-service
```

```yaml
            port:
              number: 80
```

## Aplicar

```bash
root@rafaelferreiraneves:~# kubectl apply -f nginx-ingress.yaml
```

ingress.networking.k8s.io/nginx-ingress created

Verificar ingress

```bash
root@rafaelferreiraneves:~# kubectl get ingress
```

NAME            CLASS     HOSTS         ADDRESS          PORTS   AGE

nginx-ingress   traefik   nginx.local   192.168.37.130   80      53s

Editar hosts do Windows

No Windows abra:

C:\Windows\System32\drivers\etc\hosts

Como administrador.

Adicione:

192.168.x.x nginx.local

Exemplo:

192.168.0.15 nginx.local

Testar

No navegador Windows:

```bash
http://nginx.local
```

Veja recursos do cluster:

```bash
root@rafaelferreiraneves:~# kubectl top nodes
```

NAME                                  CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)

rafaelferreiraneves-virtual-machine   122m         6%       1605Mi          41%

```bash
root@rafaelferreiraneves:~# kubectl top pods -A
```

NAMESPACE     NAME                                      CPU(cores)   MEMORY(bytes)

default       nginx-deployment-59f86b59ff-nj4ls         0m           3Mi

kube-system   coredns-8db54c48d-nvc77                   3m           15Mi

kube-system   local-path-provisioner-5d9d9885bc-8dms2   1m           7Mi

kube-system   metrics-server-786d997795-fg5lk           6m           23Mi

kube-system   svclb-traefik-e9e10035-9xx8c              0m           0Mi

kube-system   traefik-9bcdbbd9-7v4cf                    1m           17Mi

Namespaces

Ver namespaces atuais

```bash
kubectl get namespaces
```

```bash
root@rafaelferreiraneves:~# kubectl get namespaces
```

NAME              STATUS   AGE

default           Active   94m

kube-node-lease   Active   94m

kube-public       Active   94m

kube-system       Active   94m

ou:

```bash
kubectl get ns
```

```bash
root@rafaelferreiraneves-virtual-machine:~# kubectl get ns
```

NAME              STATUS   AGE

default           Active   95m

kube-node-lease   Active   95m

kube-public       Active   95m

kube-system       Active   95m

Criar namespace monitoring

```bash
root@rafaelferreiraneves:~# kubectl create namespace monitoring
```

namespace/monitoring created

Criar namespace apps

```bash
root@rafaelferreiraneves:~# kubectl create namespace apps
```

namespace/apps created

Criar namespace dev

```bash
root@rafaelferreiraneves:~# kubectl create namespace dev
```

namespace/dev created

## Verificar

```bash
root@rafaelferreiraneves:~# kubectl get ns
```

NAME              STATUS   AGE

apps              Active   107s

default           Active   99m

dev               Active   37s

kube-node-lease   Active   99m

kube-public       Active   99m

kube-system       Active   99m

monitoring        Active   2m47s

Agora vamos mover o nginx para apps

Seu deployment atual está no namespace default.

Vamos recriar corretamente.

Remover deployment antigo

```bash
root@rafaelferreiraneves:~# kubectl delete deployment nginx-deployment
```

deployment.apps "nginx-deployment" deleted from default namespace

```bash
root@rafaelferreiraneves:~# kubectl delete service nginx-service
```

service "nginx-service" deleted from default namespace

Editar deployment

```bash
vi  nginx-deployment.yaml e adicione namespace: apps
```

```yaml
apiVersion: apps/v1
kind: Deployment
```

```yaml
metadata:
  name: nginx-deployment
  namespace: apps
```

```yaml
spec:
  replicas: 1
```

```yaml
  selector:
    matchLabels:
      app: nginx
```

```yaml
  template:
    metadata:
      labels:
        app: nginx
```

```yaml
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

```yaml
        ports:
        - containerPort: 80
```

Editar service

```bash
vi  nginx-service.yaml e adicione namespace: apps
```

```yaml
apiVersion: v1
kind: Service
```

```yaml
metadata:
  name: nginx-service
  namespace: apps
```

```yaml
spec:
  type: NodePort
```

```yaml
  selector:
    app: nginx
```

```yaml
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

Aplicar novamente

```bash
root@rafaelferreiraneves:~# kubectl apply -f nginx-deployment.yaml
```

deployment.apps/nginx-deployment created

```bash
root@rafaelferreiraneves:~# kubectl apply -f nginx-service.yaml
```

service/nginx-service created

Verificar namespace apps

```bash
root@rafaelferreiraneves:~# kubectl get all -n apps
```

NAME                                    READY   STATUS    RESTARTS   AGE

pod/nginx-deployment-59f86b59ff-mrvlq   1/1     Running   0          73s

NAME                    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE

service/nginx-service   NodePort   10.43.228.237   <none>        80:30080/TCP   41s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE

deployment.apps/nginx-deployment   1/1     1            1           73s

NAME                                          DESIRED   CURRENT   READY   AGE

replicaset.apps/nginx-deployment-59f86b59ff   1         1         1       73s

Você verá:

pod

deployment

replicaset

service

todos dentro do namespace apps.

Ver pods de um namespace

```bash
root@rafaelferreiraneves:~# kubectl get pods -n apps
```

NAME                                READY   STATUS    RESTARTS   AGE

nginx-deployment-59f86b59ff-mrvlq   1/1     Running   0          3m34s

Ver tudo de um namespace

```bash
root@rafaelferreiraneves:~# kubectl get pods -n apps
```

NAME                                READY   STATUS    RESTARTS   AGE

nginx-deployment-59f86b59ff-mrvlq   1/1     Running   0          3m34s

```bash
root@rafaelferreiraneves-virtual-machine:~# kubectl get all -n apps
```

NAME                                    READY   STATUS    RESTARTS   AGE

pod/nginx-deployment-59f86b59ff-mrvlq   1/1     Running   0          4m19s

NAME                    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE

service/nginx-service   NodePort   10.43.228.237   <none>        80:30080/TCP   3m47s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE

deployment.apps/nginx-deployment   1/1     1            1           4m19s

NAME                                          DESIRED   CURRENT   READY   AGE

replicaset.apps/nginx-deployment-59f86b59ff   1         1         1       4m19s

Ver namespaces

```bash
kubectl get ns
```

Persistent Volumes (armazenamento)

Ver StorageClass

```bash
root@rafaelferreiraneves:~# kubectl get storageclass
```

NAME                   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE

local-path (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  6h21m

Criar PVC

```bash
vi nginx-pvc.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
```

```yaml
metadata:
  name: nginx-pvc
  namespace: apps
```

```yaml
spec:
  accessModes:
    - ReadWriteOnce
```

```yaml
  resources:
    requests:
      storage: 1Gi
```

Aplicar PVC

```bash
root@rafaelferreiraneves:~# kubectl apply -f nginx-pvc.yaml
```

persistentvolumeclaim/nginx-pvc created

Verificar PVC

```bash
root@rafaelferreiraneves:~# kubectl get pvc -n apps
```

NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE

nginx-pvc   Bound    pvc-0ab336fa-6c1d-481e-84d5-eada22cd27f8   1Gi        RWO            local-path     <unset>                 19m

Editar deployment

```bash
vi nginx-deployment.yaml
```

Substitua pelo conteúdo completo:

```yaml
apiVersion: apps/v1
kind: Deployment
```

```yaml
metadata:
  name: nginx-deployment
  namespace: apps
```

```yaml
spec:
  replicas: 1
```

```yaml
  selector:
    matchLabels:
      app: nginx
```

```yaml
  template:
    metadata:
      labels:
        app: nginx
```

```yaml
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

```yaml
        ports:
        - containerPort: 80
```

```yaml
        volumeMounts:
        - name: nginx-storage
          mountPath: /usr/share/nginx/html
```

```yaml
      volumes:
      - name: nginx-storage
        persistentVolumeClaim:
          claimName: nginx-pvc
```

Aplicar deployment atualizado

```bash
root@rafaelferreiraneves:~# kubectl apply -f nginx-deployment.yaml
```

deployment.apps/nginx-deployment configured

Verificar pod

```bash
root@rafaelferreiraneves:~# kubectl get pods -n apps
```

NAME                               READY   STATUS    RESTARTS   AGE

nginx-deployment-bf54745fd-n8dnq   1/1     Running   0          12m

Entrar no container

Pegue nome do pod:

```bash
root@rafaelferreiraneves:~# kubectl get pods -n apps
```

NAME                               READY   STATUS    RESTARTS   AGE

nginx-deployment-bf54745fd-n8dnq   1/1     Running   0          13m

Entre

```bash
root@rafaelferreiraneves-virtual-machine:~# kubectl exec -it nginx-deployment-bf54745fd-n8dnq -n apps -- bash
root@nginx-deployment-bf54745fd-n8dnq:/#
```

Criar arquivo persistente

```bash
root@nginx-deployment-bf54745fd-n8dnq:/# echo "Kubernetes Persistent Volume funcionando" > /usr/share/nginx/html/index.html
root@nginx-deployment-bf54745fd-n8dnq:/#
```

Sair:

exit

Testar no navegador

```bash
http://IP_DA_VM:30080
```

Kubernetes Persistent Volume funcionando

PROVA REAL DA PERSISTÊNCIA

Agora delete o pod:

```bash
root@rafaelferreiraneves:~# kubectl delete pod -n apps --all
```

pod "nginx-deployment-bf54745fd-n8dnq" deleted from apps namespace

Kubernetes vai recriar automaticamente.

```bash
root@rafaelferreiraneves-virtual-machine:~# kubectl get pods -n apps
```

NAME                               READY   STATUS    RESTARTS   AGE

nginx-deployment-bf54745fd-npkm8   1/1     Running   0          40s

Instalar Helm

Execute:

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

```bash
root@rafaelferreiraneves-virtual-machine:~# curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
```

100 11929  100 11929    0     0   119k      0 --:--:-- --:--:-- --:--:--  120k

[WARNING] Could not find git. It is required for plugin installation.

Downloading https://get.helm.sh/helm-v3.21.0-linux-amd64.tar.gz

Verifying checksum... Done.

Preparing to install helm into /usr/local/bin

helm installed into /usr/local/bin/helm

```bash
root@rafaelferreiraneves-virtual-machine:~# helm version
```

version.BuildInfo{Version:"v3.21.0", GitCommit:"e0878d41b711792be60777fd65ad23a101e6b85f", GitTreeState:"clean", GoVersion:"go1.25.10"}

Adicionar repositório

```bash
root@rafaelferreiraneves-virtual-machine:~# helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

"prometheus-community" has been added to your repositories

Atualizar repositórios

```bash
root@rafaelferreiraneves-virtual-machine:~# helm repo update
```

Hang tight while we grab the latest from your chart repositories...

...Successfully got an update from the "prometheus-community" chart repository

Update Complete. ⎈Happy Helming!⎈

Ver charts disponíveis

```bash
root@rafaelferreiraneves-virtual-machine:~# helm search repo prometheus-community
```

NAME                                                    CHART VERSION   APP VERSION     DESCRIPTION

prometheus-community/alertmanager                       1.37.0          v0.32.1         The Alertmanager handles alerts sent by client ...

prometheus-community/alertmanager-snmp-notifier         2.1.0           v2.1.0          The SNMP Notifier handles alerts coming from Pr...

prometheus-community/jiralert                           1.8.2           v1.3.0          A Helm chart for Kubernetes to install jiralert

prometheus-community/kube-prometheus-stack              86.1.0          v0.91.0         kube-prometheus-stack collects Kubernetes manif...

prometheus-community/kube-state-metrics                 7.4.0           2.19.0          Install kube-state-metrics to generate and expo...

prometheus-community/prom-label-proxy                   0.21.0          v0.13.0         A proxy that enforces a given label in a given ...

prometheus-community/prometheus                         29.9.0          v3.12.0         Prometheus is a monitoring system and time seri...

prometheus-community/prometheus-adapter                 5.3.0           v0.12.0         A Helm chart for k8s prometheus adapter

prometheus-community/prometheus-blackbox-exporter       11.10.0         v0.28.0         Prometheus Blackbox Exporter

prometheus-community/prometheus-cloudwatch-expo...      0.28.1          0.16.0          A Helm chart for prometheus cloudwatch-exporter

prometheus-community/prometheus-conntrack-stats...      0.5.36          v0.4.43         A Helm chart for conntrack-stats-exporter

prometheus-community/prometheus-consul-exporter         1.1.1           v0.13.0         A Helm chart for the Prometheus Consul Exporter

prometheus-community/prometheus-couchdb-exporter        1.0.1           1.0             A Helm chart to export the metrics from couchdb...

prometheus-community/prometheus-druid-exporter          1.2.0           v0.11.0         Druid exporter to monitor druid metrics with Pr...

prometheus-community/prometheus-elasticsearch-e...      7.2.1           v1.10.0         Elasticsearch stats exporter for Prometheus

prometheus-community/prometheus-fastly-exporter         0.11.0          v10.2.0         A Helm chart for the Prometheus Fastly Exporter

prometheus-community/prometheus-ipmi-exporter           0.8.0           v1.10.1         This is an IPMI exporter for Prometheus.

prometheus-community/prometheus-json-exporter           0.19.2          v0.7.0          Install prometheus-json-exporter

prometheus-community/prometheus-kafka-exporter          3.0.1           v1.9.0          A Helm chart to export metrics from Kafka in Pr...

prometheus-community/prometheus-memcached-exporter      0.5.0           v0.16.0         Prometheus exporter for Memcached metrics

prometheus-community/prometheus-modbus-exporter         0.1.4           0.4.1           A Helm chart for prometheus-modbus-exporter

prometheus-community/prometheus-mongodb-exporter        3.20.0          0.51.0          A Prometheus exporter for MongoDB metrics

prometheus-community/prometheus-mysql-exporter          2.14.0          v0.19.0         A Helm chart for prometheus mysql exporter with...

prometheus-community/prometheus-nats-exporter           2.23.0          0.20.0          A Helm chart for prometheus-nats-exporter

prometheus-community/prometheus-nginx-exporter          1.22.4          1.5.1           A Helm chart for NGINX Prometheus Exporter

prometheus-community/prometheus-node-exporter           4.55.0          1.11.1          A Helm chart for prometheus node-exporter

prometheus-community/prometheus-opencost-exporter       0.1.2           1.108.0         Prometheus OpenCost Exporter

prometheus-community/prometheus-operator                9.3.2           0.38.1          DEPRECATED - This chart will be renamed. See ht...

prometheus-community/prometheus-operator-admiss...      0.40.1          0.91.0          Prometheus Operator Admission Webhook

prometheus-community/prometheus-operator-crds           29.0.0          v0.91.0         A Helm chart that collects custom resource defi...

prometheus-community/prometheus-pgbouncer-exporter      0.10.0          v0.12.0         A Helm chart for prometheus pgbouncer-exporter

prometheus-community/prometheus-pingdom-exporter        3.4.2           v0.5.6          A Helm chart for Prometheus Pingdom Exporter

prometheus-community/prometheus-pingmesh-exporter       0.4.3           v1.2.2          Prometheus Pingmesh Exporter

prometheus-community/prometheus-postgres-exporter       8.0.0           v0.19.1         A Helm chart for prometheus postgres-exporter

prometheus-community/prometheus-pushgateway             3.6.1           v1.11.3         A Helm chart for prometheus pushgateway

prometheus-community/prometheus-rabbitmq-exporter       2.1.2           1.0.0           Rabbitmq metrics exporter for prometheus

prometheus-community/prometheus-redis-exporter          6.24.0          v1.84.0         Prometheus exporter for Redis metrics

prometheus-community/prometheus-smartctl-exporter       0.16.1          v0.14.0         A Helm chart for Kubernetes

prometheus-community/prometheus-snmp-exporter           9.14.0          v0.30.1         Prometheus SNMP Exporter

prometheus-community/prometheus-sql-exporter            0.5.0           v0.8            Prometheus SQL Exporter

prometheus-community/prometheus-stackdriver-exp...      4.12.2          v0.18.0         Stackdriver exporter for Prometheus

prometheus-community/prometheus-statsd-exporter         1.0.0           v0.28.0         A Helm chart for prometheus stats-exporter

prometheus-community/prometheus-systemd-exporter        0.5.2           0.7.0           A Helm chart for prometheus systemd-exporter

prometheus-community/prometheus-to-sd                   0.5.1           v0.9.2          Scrape metrics stored in prometheus format and ...

prometheus-community/prometheus-windows-exporter        0.12.7          0.31.7          A Helm chart for prometheus windows-exporter

prometheus-community/prometheus-yet-another-clo...      0.45.0          v0.65.0         Yace - Yet Another CloudWatch Exporter

prometheus-community/yet-another-cloudwatch-exp...      0.39.1          v0.62.1         Yace - Yet Another CloudWatch Exporter

Instalar kube-prometheus-stack

Esse chart instala:

Prometheus

Grafana

Alertmanager

exporters

dashboards

Tudo pronto.

Instalar no namespace monitoring

```bash
root@rafaelferreiraneves-virtual-machine:~# helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
```

NAME: monitoring

LAST DEPLOYED: Sat May 30 16:44:27 2026

NAMESPACE: monitoring

STATUS: deployed

REVISION: 1

TEST SUITE: None

NOTES:

kube-prometheus-stack has been installed. Check its status by running:

```yaml
  kubectl --namespace monitoring get pods -l "release=monitoring"
```

Get Grafana 'admin' user password by running:

```yaml
  kubectl --namespace monitoring get secrets monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo
```

Access Grafana local instance:

```yaml
  export POD_NAME=$(kubectl --namespace monitoring get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=monitoring" -oname)
  kubectl --namespace monitoring port-forward $POD_NAME 3000
```

Get your grafana admin user password by running:

```yaml
  kubectl get secret --namespace monitoring -l app.kubernetes.io/component=admin-secret -o jsonpath="{.items[0].data.admin-password}" | base64 --decode ; echo
```

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.

Verificar pods

```bash
kubectl get pods -n monitoring
```

Vai demorar alguns minutos.

Você verá pods como:

```bash
root@rafaelferreiraneves-virtual-machine:~# kubectl get pods -n monitoring
```

NAME                                                     READY   STATUS    RESTARTS   AGE

alertmanager-monitoring-kube-prometheus-alertmanager-0   2/2     Running   0          3m6s

monitoring-grafana-6c45c69bdf-fc9f2                      3/3     Running   0          3m47s

monitoring-kube-prometheus-operator-7cd45c8468-l9rmj     1/1     Running   0          3m47s

monitoring-kube-state-metrics-868694bc4b-grwkf           1/1     Running   0          3m47s

monitoring-prometheus-node-exporter-lvb8k                1/1     Running   0          3m47s

prometheus-monitoring-kube-prometheus-prometheus-0       2/2     Running   0          3m5s

Ver services

```bash
root@rafaelferreiraneves-virtual-machine:~# kubectl get svc -n monitoring
```

NAME                                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE

alertmanager-operated                     ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   4m52s

monitoring-grafana                        ClusterIP   10.43.220.209   <none>        80/TCP                       5m33s

monitoring-kube-prometheus-alertmanager   ClusterIP   10.43.100.206   <none>        9093/TCP,8080/TCP            5m33s

monitoring-kube-prometheus-operator       ClusterIP   10.43.94.179    <none>        443/TCP                      5m33s

monitoring-kube-prometheus-prometheus     ClusterIP   10.43.240.123   <none>        9090/TCP,8080/TCP            5m33s

monitoring-kube-state-metrics             ClusterIP   10.43.167.160   <none>        8080/TCP                     5m33s

monitoring-prometheus-node-exporter       ClusterIP   10.43.248.57    <none>        9100/TCP                     5m33s

prometheus-operated                       ClusterIP   None            <none>        9090/TCP                     4m51s

Expor Grafana

Vamos transformar o service do Grafana em NodePort.

Editar servisse

```bash
kubectl edit svc monitoring-grafana -n monitoring
```

Procurar isto:

type: ClusterIP

Trocar para:

type: NodePort

Ver porta do Grafana

```bash
kubectl get svc -n monitoring
```

```bash
root@rafaelferreiraneves-virtual-machine:~# kubectl get svc -n monitoring
```

NAME                                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE

alertmanager-operated                     ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   11m

monitoring-grafana                        NodePort    10.43.220.209   <none>        80:30986/TCP                 12m

monitoring-kube-prometheus-alertmanager   ClusterIP   10.43.100.206   <none>        9093/TCP,8080/TCP            12m

monitoring-kube-prometheus-operator       ClusterIP   10.43.94.179    <none>        443/TCP                      12m

monitoring-kube-prometheus-prometheus     ClusterIP   10.43.240.123   <none>        9090/TCP,8080/TCP            12m

monitoring-kube-state-metrics             ClusterIP   10.43.167.160   <none>        8080/TCP                     12m

monitoring-prometheus-node-exporter       ClusterIP   10.43.248.57    <none>        9100/TCP                     12m

prometheus-operated                       ClusterIP   None            <none>        9090/TCP                     11m

Você verá algo como:

monitoring-grafana   NodePort   10.x.x.x   <none>   80:32141/TCP

Acessar Grafana

```bash
http://IP_DA_VM:32141
```

Exemplo:

http:/172.19.157.82:31059

Usuário e senha Grafana

Usuário:

admin

Senha:

```bash
kubectl get secret -n monitoring monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 -d
```

( Esse comando pega a senha automática criada pelo Grafana dentro do secret do Kubernetes )

O que você verá

Dashboards mostrando:

Nodes

Pods

CPU

RAM

Network

Kubernetes cluster

containers

storage

Tudo em tempo real.

Instalar Gitea + PostgreSQL

Criar namespace

fON4H0q1yPrJKdxLJEROhMKZuBpAS9n0pYBv26mzroot@rafaelferreiraneves-virtual-machine:~# kubectl create namespace gitea

namespace/gitea created

Adicionar repo Helm

```bash
root@rafaelferreiraneves-virtual-machine:~# helm repo add gitea-charts https://dl.gitea.com/charts/
```

"gitea-charts" has been added to your repositories

Atualizar repôs

```bash
root@rafaelferreiraneves-virtual-machine:~# helm repo update
```

Hang tight while we grab the latest from your chart repositories...

...Successfully got an update from the "gitea-charts" chart repository

...Successfully got an update from the "prometheus-community" chart repository

Update Complete. ⎈Happy Helming!⎈

Criar arquivo gitea-values.yaml

```bash
vi gitea-values.yaml
```

gitea:

```yaml
  admin:
    username: gitea
    password: gitea123
    email: admin@gitea.local
```

service:

```yaml
  http:
    type: NodePort
    nodePort: 30090
```

postgresql:

```yaml
  enabled: true
```

postgresql-ha:

```yaml
  enabled: false
```

persistence:

```yaml
  enabled: true
  size: 5Gi
```

Instalar

```bash
root@rafaelferreiraneves-virtual-machine:~# helm install gitea gitea-charts/gitea -n gitea -f gitea-values.yaml
```

NAME: gitea

LAST DEPLOYED: Sat May 30 20:16:29 2026

NAMESPACE: gitea

STATUS: deployed

REVISION: 1

NOTES:

1. Get the application URL by running these commands:

```yaml
  export NODE_PORT=$(kubectl get --namespace gitea -o jsonpath="{.spec.ports[0].nodePort}" services gitea)
  export NODE_IP=$(kubectl get nodes --namespace gitea -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```

Ver pods

```bash
root@rafaelferreiraneves-virtual-machine:~# kubectl get pods -n gitea
```

NAME                     READY   STATUS              RESTARTS   AGE

gitea-78b574448f-q5v56   0/1     Init:0/3            0          72s

gitea-postgresql-0       0/1     ContainerCreating   0          72s

gitea-valkey-cluster-0   0/1     ContainerCreating   0          72s

gitea-valkey-cluster-1   0/1     ContainerCreating   0          71s

gitea-valkey-cluster-2   0/1     ContainerCreating   0          71s

Ver services

```bash
root@rafaelferreiraneves-virtual-machine:~# kubectl get svc -n gitea
```

NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)              AGE

gitea-http                      NodePort    10.43.57.54     <none>        3000:30090/TCP       2m18s

gitea-postgresql                ClusterIP   10.43.26.5      <none>        5432/TCP             2m18s

gitea-postgresql-hl             ClusterIP   None            <none>        5432/TCP             2m18s

gitea-ssh                       ClusterIP   None            <none>        22/TCP               2m18s

gitea-valkey-cluster            ClusterIP   10.43.196.129   <none>        6379/TCP             2m18s

gitea-valkey-cluster-headless   ClusterIP   None            <none>        6379/TCP,16379/TCP   2m18s

Acessar Gitea

```bash
http://IP_DA_VM:32141
```

Exemplo:

```bash
http://172.19.157.82:30090/
```

Usuário

gitea

Senha

Agora vamos criar seu PRIMEIRO repositório

Clique em:

+

no canto superior direito.

Depois:

New Repository

Configure assim

Repository Name

devops-lab

Visibility

Escolha:

Public

Initialize Repository

Marque:

README.md

Criar

Clique:

Create Repository

Depois vamos conectar Git REAL

Na sua VM:

git clone http://172.19.157.82:30090/gitea/devops-lab.git

Criar namespace

```bash
root@rafaelferreiraneves-virtual-machine:~# kubectl create namespace argocd
```

namespace/argocd created

Instalar ArgoCD

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Acompanhar pods

```bash
kubectl get pods -n argocd
```

Ver servisse

```bash
kubectl get svc -n argocd
```

Pegar a porta do argocd-server

argocd-server                             NodePort    10.43.148.188   <none>        80:30556/TCP,443:32188/TCP   27m

Acessar Argo

```bash
http://IP_DA_VM:32188
```

Exemplo:

https://172.19.157.82:32188

usuário

admin

senha

Execute:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

( Esse comando pega a senha automática criada pelo Aego dentro do secret do Kubernetes )

Clonar repo

git clone http://172.19.157.82:30090/gitea/gitops-apps.git

O que isso implica

O clone funcionou corretamente

O Git criou a pasta gitops-nginx

Mas não há arquivos para baixar porque o repo ainda não tem conteúdo

Entrar no diretório

cd gitops-nginx

2. Criar um arquivo inicial (exemplo README)

echo "# gitops-nginx" > README.md

Transfira os arquivos anteriormente criados pra esse diretorio

deployment.yaml

service.yaml

ingress.yaml

Fazer o primeiro commit

git add .

git commit -m "first commit"

4. Enviar para o Gitea

git push origin main

ArgoCD precisa SABER onde estão

Crie um arquivo local

```bash
vi nginx-app.yaml
```

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
```

```yaml
metadata:
  name: nginx
  namespace: argocd
```

```yaml
spec:
  project: default
```

```yaml
  source:
    repoURL: http://172.19.157.82:30090/admin/gitops-apps.git
    targetRevision: main
    path: gitops-nginx
```

```yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: apps
```

```yaml
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Aplicar no cluster

```bash
kubectl apply -f nginx-app.yaml
```

Você está criando um objeto Application dentro do ArgoCD.

O que acontece automaticamente

O Argo CD vai:

Ler o repo Git

Entrar em:

gitops-nginx

Encontrar:

deployment.yaml

service.yaml

ingress.yaml

Aplicar no cluster

Como se fosse:

```bash
kubectl apply -f .
```

automaticamente.
