# Выполнено ДЗ №6

 - [V] Задание 1

## В процессе сделано:
 - Установлен Helm
 - Создан helm-chart для деплоя приложения из предыдущих ДЗ в соответствии с требованиями
## Файлы:
 - Основной деплой: mychart\templates\deployment.yaml
 - Chart.yaml
 - Переменные mychart\values.yaml
 - Сообщение после установки релиза: mychert\NOTES.txt

## Как запустить проект:
 - Добавляем репозиторий для выполнения условия зависимости: helm repo add stable https://charts.helm.sh/stable
 - Обновляем репозиторий: helm repo update
 - Поиск добавляемой зависимости (redis) и его версии:
$ helm search repo redis
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION
stable/prometheus-redis-exporter        3.5.1           1.3.4           DEPRECATED Prometheus exporter for Redis metrics
stable/redis                            10.5.7          5.0.7           DEPRECATED Open source, advanced key-value stor...
stable/redis-ha                         4.4.6           5.0.6           DEPRECATED - Highly available Kubernetes implem...
stable/sensu                            0.2.5           0.28            DEPRECATED Sensu monitoring framework backed by...

 - Обновление зависимостей в чарте: helm dependency update
 - Проверка синтаксиса: helm lint mychart/
 - Запаковка чарта: helm package mychart
 - Уствновка релиза: helm install my-release my-app-1.0.0.tgz


## Как проверить работоспособность:

После установки релиза убедиться в выдаче информации, соответствущей NOTES.txt:
$ helm install my-release my-app-1.0.0.tgz
NAME: my-release
LAST DEPLOYED: Tue Jul 30 23:39:09 2024
NAMESPACE: homework
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Release deploy: my-release
Application install in namespace:
- homework
The web interface is available at:
http://homework.otus

Проверяем наличие и статус подов:
$ kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
my-release-redis-master-0       1/1     Running   0          28m
my-release-redis-slave-0        1/1     Running   0          28m
my-release-redis-slave-1        1/1     Running   0          28m
nginx-deploy-84885949df-5nsbb   1/1     Running   0          28m
nginx-deploy-84885949df-9kcdw   1/1     Running   0          28m
nginx-deploy-84885949df-gpmlg   1/1     Running   0          28m



 - [V] Задание 2

## В процессе сделано:
 - Установлен helmfile под Windows
1) Установка scoop в PowerShell:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
2) Установка helmfile
scoop install helmfile
helmfile init

 - Установка kafka из bitnami helm-чарта

Добавляем репозиторий
$ helm repo add bitnami https://charts.bitnami.com/bitnami

Проверяем список доступных:
$ helm repo list
NAME    URL
stable  https://charts.helm.sh/stable
bitnami https://charts.bitnami.com/bitnami

Смотрим список доступных приложений bitnami репозитория:
helm search repo bitnami

Устанавливаем kafka (вариант prod)
helm -n prod install kafka-release bitnami/kafka --create-namespace -f values-prod.yaml

Проверяем наличие и статус подов:
$ kubectl -n prod get pods
NAME                        READY   STATUS    RESTARTS   AGE
kafka-release-broker-0      1/1     Running   0          63s
kafka-release-broker-1      1/1     Running   0          63s
kafka-release-broker-2      1/1     Running   0          63s
kafka-release-broker-3      1/1     Running   0          63s
kafka-release-broker-4      1/1     Running   0          63s
kafka-release-zookeeper-0   1/1     Running   0          63s

Удаляем релиз
$ helm -n prod delete kafka-release

Устанавливаем kafka (вариант dev)
helm -n dev install kafka-release bitnami/kafka --create-namespace -f values-dev.yaml

Проверяем наличие и статус подов:
$ kubectl -n dev get pods
NAME                        READY   STATUS    RESTARTS      AGE
kafka-release-broker-0      1/1     Running   1 (18s ago)   39s
kafka-release-zookeeper-0   1/1     Running   0             39s

Удаляем релиз:
helm -n dev delete kafka-release

 - Helmfile
Формируем helmfile.yaml

Устанавливаем релиз (вариант prod)
helmfile -e prod apply

Проверяем наличие и статус подов:
$ kubectl -n prod get pods
NAME                     READY   STATUS    RESTARTS   AGE
kafka-prod-broker-0      1/1     Running   0          87s
kafka-prod-broker-1      1/1     Running   0          87s
kafka-prod-broker-2      1/1     Running   0          87s
kafka-prod-broker-3      1/1     Running   0          87s
kafka-prod-broker-4      1/1     Running   0          87s
kafka-prod-zookeeper-0   1/1     Running   0          87s

Удаляем релиз
helmfile -e prod delete

Устанавливаем релиз (вариант dev)
helmfile -e dev apply

Проверяем наличие и статус подов:
$ kubectl -n dev get pods
NAME                    READY   STATUS    RESTARTS   AGE
kafka-dev-broker-0      1/1     Running   0          64s
kafka-dev-zookeeper-0   1/1     Running   0          64s

Удаляем релиз
helmfile -e dev delete
