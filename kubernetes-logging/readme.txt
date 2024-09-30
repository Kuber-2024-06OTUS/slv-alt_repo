# Выполнено ДЗ №9

 - [V] Основное ДЗ


## В процессе сделано:

Развернут клаастер на основе ОС Debian12 из 5 нод, из которых - 3 управляющих и 2 рабочих.
Облачные услуги использовать не будем. Для удобства доступа к приложеням, в т.ч. вэбсерверам, которые будут развернуты на нодах,
установим был установлен metallb.

Для инфраструктурных нод добавляем taint:
kubectl taint nodes node1.int node-role=infra:NoSchedule
kubectl taint nodes node2.int node-role=infra:NoSchedule
kubectl taint nodes node3.int node-role=infra:NoSchedule

Для инфраструктурных нод добавляем метки:
kubectl label nodes node1.int infra=true
kubectl label nodes node2.int infra=true
kubectl label nodes node3.int infra=true

Инормация о нодах, выводы комманд:
# kubectl get node -o wide --show-labels
NAME        STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION   CONTAINER-RUNTIME   LABELS
node1.int   Ready    control-plane   47h   v1.30.4   192.168.1.31   <none>        Debian GNU/Linux 12 (bookworm)   6.1.0-25-amd64   cri-o://1.30.5      beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node1.int,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
node2.int   Ready    control-plane   47h   v1.30.4   192.168.1.32   <none>        Debian GNU/Linux 12 (bookworm)   6.1.0-25-amd64   cri-o://1.30.5      beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node2.int,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
node3.int   Ready    control-plane   47h   v1.30.4   192.168.1.33   <none>        Debian GNU/Linux 12 (bookworm)   6.1.0-25-amd64   cri-o://1.30.5      beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node3.int,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
node4.int   Ready    <none>          47h   v1.30.4   192.168.1.34   <none>        Debian GNU/Linux 12 (bookworm)   6.1.0-25-amd64   cri-o://1.30.5      beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node4.int,kubernetes.io/os=linux
node5.int   Ready    <none>          47h   v1.30.4   192.168.1.35   <none>        Debian GNU/Linux 12 (bookworm)   6.1.0-25-amd64   cri-o://1.30.5      beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node5.int,kubernetes.io/os=linux

# kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
NAME        TAINTS
node1.int   [map[effect:NoSchedule key:node-role value:infra] map[effect:NoSchedule key:node-role.kubernetes.io/control-plane]]
node2.int   [map[effect:NoSchedule key:node-role value:infra] map[effect:NoSchedule key:node-role.kubernetes.io/control-plane]]
node3.int   [map[effect:NoSchedule key:node-role value:infra] map[effect:NoSchedule key:node-role.kubernetes.io/control-plane]]
node4.int   <none>
node5.int   <none>


 - S3 хранилище - Minio
С сайта Minio забираем конфиг для развертывания в kubernetes
curl https://raw.githubusercontent.com/minio/docs/master/source/extra/examples/minio-dev.yaml -O

Применяем данный манифест
kubectl apply -f minio.yaml

В браузере переходим в вэб-интерфейс по ссылке http://192.168.1.31:9090/browser и настраваем корзины и доступ.


Для удобства контроля компонентов kubernetes установим k9s.

### Установка k9s через pkgx:
Установка pkgx:
curl -Ssf https://pkgx.sh | sh
Установка и запуск k9s:
pkgx k9s


Для установки в наш кластер требуего ПО, установим Helm.

### Установка helm
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
apt-get update
apt-get install helm



Добавляем Grafana’s chart репозиторий в Helm:
helm repo add grafana https://grafana.github.io/helm-charts

Проверяем список доступных:
# helm repo list
NAME    URL
grafana https://grafana.github.io/helm-charts


 - Loki

Смотрим список доступных приложений grafana репозитория:
helm search repo grafana

Создаем файл "values" с полным списком значений для пакета grafana/loki
helm show values grafana/loki >val-loki.txt

Создаем файл val-loki.yaml

Установка
helm install --values values.yaml loki grafana/loki
# helm install --values val-loki.yaml loki grafana/loki
NAME: loki
LAST DEPLOYED: Mon Sep 30 08:44:38 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
***********************************************************************
 Welcome to Grafana Loki
 Chart version: 6.15.0
 Chart Name: loki
 Loki version: 3.1.1
***********************************************************************

** Please be patient while the chart is being deployed **

Tip:

  Watch the deployment status using the command: kubectl get pods -w --namespace default

If pods are taking too long to schedule make sure pod affinity can be fulfilled in the current cluster.

***********************************************************************
Installed components:
***********************************************************************
* loki

Loki has been deployed as a single binary.
This means a single pod is handling reads and writes. You can scale that pod vertically by adding more CPU and memory                  resources.

 - PV
Выяснилось, что в моем кластере нет storageclass (не создаются автоматически PV):
# kubectl get storageclass
No resources found

Вручную делаем PersistentVolume (pv.yaml)

Созданы два манифеста PV:
Для Loki - pv-loki.yaml
Для Grafana - pv-grafana.yaml

Применяем их
kubectl apply -f pv-loki.yaml -f pv-grafana.yaml

 - Promtail

Создаем файл "values" с полным списком значений для пакета promtail
helm show values grafana/promtail >val-prom.txt

Создаем файл val-prom.yaml

Устанавливаем Promtail
helm install --values val-prom.yaml promtail grafana/promtail
# helm upgrade --values val-prom.yaml promtail grafana/promtail
Release "promtail" has been upgraded. Happy Helming!
NAME: promtail
LAST DEPLOYED: Sun Sep 29 20:24:57 2024
NAMESPACE: default
STATUS: deployed
REVISION: 4
TEST SUITE: None
NOTES:
***********************************************************************
 Welcome to Grafana Promtail
 Chart version: 6.16.6
 Promtail version: 3.0.0
***********************************************************************


- Grafana

Создаем файл "values" с полным списком значений для пакета grafana
helm show values grafana/grafana >val-grafana.txt

Создаем файл val-grafana.yaml

Устанавливаем Grafana
# helm install --values val-grafana.yaml grafana grafana/grafana
NAME: grafana
LAST DEPLOYED: Sun Sep 29 22:08:31 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get your 'admin' user password by running:

   kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo


2. The Grafana server can be accessed via port 80 on the following DNS name from within your cluster:

   grafana.default.svc.cluster.local

   Get the Grafana URL to visit by running these commands in the same shell:
     export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=grafana" -o jsonpath="{.items[0].metadata.name}")
     kubectl --namespace default port-forward $POD_NAME 3000

3. Login with the password from step 1 and the username: admin


 - Настройка в web-интерфейсе Grafana
Connections-Data sources-Add data source-Loki
  URL: http://loki-gateway:80
  Save&test
Explore-Loki-Code-Label browser

Скриншот приложен в виде файла.


Вывод информации о подах (ns default с loki, promtail и grafana)

# kubectl get pods -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP             NODE        NOMINATED NODE   READINESS GATES
grafana-78c587db65-8fh75        1/1     Running   0          7h49m   10.244.2.84    node3.int   <none>           <none>
loki-0                          2/2     Running   0          8h      10.244.1.86    node2.int   <none>           <none>
loki-canary-cpnws               1/1     Running   0          8h      10.244.2.81    node3.int   <none>           <none>
loki-canary-fmg2b               1/1     Running   0          8h      10.244.0.92    node1.int   <none>           <none>
loki-canary-vr88f               1/1     Running   0          8h      10.244.1.84    node2.int   <none>           <none>
loki-chunks-cache-0             2/2     Running   0          8h      10.244.1.85    node2.int   <none>           <none>
loki-gateway-64997f554c-h6tpm   1/1     Running   0          8h      10.244.0.93    node1.int   <none>           <none>
loki-results-cache-0            2/2     Running   0          8h      10.244.2.82    node3.int   <none>           <none>
promtail-g6xsg                  1/1     Running   0          7h57m   10.244.3.120   node4.int   <none>           <none>
promtail-gxrsd                  1/1     Running   0          7h57m   10.244.4.117   node5.int   <none>           <none>
promtail-p6w5s                  1/1     Running   0          7h57m   10.244.2.83    node3.int   <none>           <none>
promtail-pw6pf                  1/1     Running   0          7h57m   10.244.1.87    node2.int   <none>           <none>
promtail-zsvht                  1/1     Running   0          7h57m   10.244.0.94    node1.int   <none>           <none>

g# kubectl get pods -o wide -n minio-dev (ns minio-dev с minio)
NAME    READY   STATUS    RESTARTS   AGE    IP             NODE        NOMINATED NODE   READINESS GATES
minio   1/1     Running   4          5d1h   10.244.4.115   node5.int   <none>           <none>


## PR checklist:
 - [V] Выставлен label с темой домашнего задания
