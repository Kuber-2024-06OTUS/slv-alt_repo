# Выполнено ДЗ №7

 - [V] Задание 7


## В процессе сделано:
 - Создан манифест объекта CustomResourceDefinition: crd.yaml
 - Создан манифест ServiceAccount, ClusterRole и ClusterRoleBinding: sa-oper.yaml
 - Создан манифест deployment для оператора: deployment.yaml
 - Создан манифест кастомного объекта: mysql.yaml


## Как запустить проект:

 - Применяем манифесты

$ kubectl apply -f crd.yaml
customresourcedefinition.apiextensions.k8s.io/mysqls.otus.homework created

$ kubectl apply -f sa-oper.yaml
serviceaccount/sa-oper created
clusterrole.rbac.authorization.k8s.io/cluster-role-oper created
clusterrolebinding.rbac.authorization.k8s.io/crb-oper created

$ kubectl apply -f deployment.yaml
deployment.apps/oper-deployment created

$ kubectl apply -f mysql.yaml
mysql.otus.homework/res-mysql created


## Как проверить работоспособность:

 - Выводим списки созданных объектов

Кастомные ресурсы
$ kubectl get crds
NAME                   CREATED AT
mysqls.otus.homework   2024-08-08T09:17:28Z

$ kubectl get mysqls
NAME        AGE
res-mysql   16m

$ kubectl describe mysqls
Name:         res-mysql
Namespace:    homework
Labels:       <none>
Annotations:  kopf.zalando.org/mysql_on_create:
                {"started":"2024-08-08T09:20:27.484659","delayed":"2024-08-08T09:41:40.889250","purpose":"create","retries":21,"success":false,"failure":f...
API Version:  otus.homework/v1
Kind:         MySQL
Metadata:
  Creation Timestamp:  2024-08-08T09:20:27Z
  Finalizers:
    kopf.zalando.org/KopfFinalizerMarker
  Generation:        1
  Resource Version:  6462
  UID:               fbdeb952-a604-40dc-9942-02b52432e3b9
Spec:
  Database:      mydb
  Image:         mysql:latest
  Password:      mypassword
  storage_size:  2Gi
Events:
  Type    Reason   Age   From  Message
  ----    ------   ----  ----  -------
  Normal  Logging  20m   kopf  Waiting for mysql deployment to become ready...
  Normal  Logging  20m   kopf  Creating pv, pvc for mysql data and svc...
  Normal  Logging  20m   kopf  Creating mysql deployment...
.......


Сервисные аккаунты
$ kubectl get sa
NAME      SECRETS   AGE
default   0         173m
sa-oper   0         10m

Кластерные роли
$ kubectl get clusterrole |grep oper
cluster-role-oper                                                      2024-08-08T09:18:40Z

Кластер роль биндинги
$ kubectl get clusterrolebindings |grep oper
crb-oper                                                        ClusterRole/cluster-role-oper

Деплоймент
$ kubectl get deploy
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
oper-deployment   1/1     1            1           15m

Поды
$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
oper-deployment-5c489cf469-8dhpn   1/1     Running   0          15m

PV
$ kubectl get pv
NAME           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
res-mysql-pv   2Gi        RWO            Retain           Released   default/res-mysql-pvc   standard       <unset>                          16m

PVC
$ kubectl get pvc
No resources found in homework namespace.

Удаляем кастомный объект mysql
$ kubectl delete -f mysql.yaml
mysql.otus.homework "res-mysql" deleted

Убеждаемся, что оператор удалил и другие объекты

$ kubectl get mysqls
No resources found in homework namespace.

$ kubectl get pv
No resources found



## PR checklist:
 - [V] Выставлен label с темой домашнего задания