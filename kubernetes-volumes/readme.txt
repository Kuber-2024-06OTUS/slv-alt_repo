# Выполнено ДЗ №4

 - [V] Основное ДЗ
 - [] Задание со *

## В процессе сделано:
 - Создан манифест pvc.yaml описывающий PersistentVolumeClaim, запрашивающий хранилище с storageClass по-умолчанию
 - Создан манифест cm.yaml для объекта типа configMap, описывающий произвольный набор пар ключ-значение
 - В манифесте deployment.yaml изменена спецификация volume типа emptyDir, который монтируется в init и основной контейнер,
на pvc, созданный в предыдущем пункте
 - В манифесте deployment.yaml добавлено монтирование ранее созданного configMap как volume к основному контейнеру пода в директории
/homework/conf, так, чтобы его содержимое можно было получить, обратившись по url /conf/file

## Как запустить проект:
 - Переключиться в namespace homework: kubectl config set-context --current --namespace=homework
 - Установить метку на ноду: kubectl label nodes minikube homework=true
 - Установить ingress контроллер Nginx: minikube addons enable ingress
 - Применить манифесты: kubectl apply -f cm.yaml -f deployment.yaml -f ingress.yaml -f namespace.yaml -f pvc.yaml -f service.yaml

## Как проверить работоспособность:
 - Проверить выдачу комманд:

Ingress
$ kubectl get ingress
NAME           CLASS    HOSTS           ADDRESS        PORTS   AGE
ingress-lab3   <none>   homework.otus   192.168.49.2   80      78m

Нода
$ kubectl get nodes -o wide
NAME       STATUS   ROLES           AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION                       CONTAINER-RUNTIME
minikube   Ready    control-plane   2d18h   v1.30.0   192.168.49.2   <none>        Ubuntu 22.04.4 LTS   5.15.153.1-microsoft-standard-WSL2   docker://26.1.1

Метка на ноде
Убедиться в наличии метки homework=true
$ kubectl describe node minikube

Пространство имен "homework"
$ kubectl get ns
NAME              STATUS   AGE
default           Active   2d18h
homework          Active   2d18h
ingress-nginx     Active   75m
kube-node-lease   Active   2d18h
kube-public       Active   2d18h
kube-system       Active   2d18h

Службы
$ kubectl get svc
NAME   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
lab3   ClusterIP   10.107.82.174   <none>        80/TCP    89m

PersistentVolumeClaim
$ kubectl get pvc
NAME     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
my-pvc   Bound    pvc-9d93a1ac-abd6-46b3-bd4e-d7cedc9253e2   1Gi        RWO            standard       <unset>                 159m

configMap
$ kubectl get cm
NAME               DATA   AGE
kube-root-ca.crt   1      2d18h
my-config-map      3      159m

Deploy
$ kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   3/3     3            3           77m

Pods
$ kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
nginx-deploy-d66578ffc-9d2hh   1/1     Running   0          78m
nginx-deploy-d66578ffc-hsff8   1/1     Running   0          78m
nginx-deploy-d66578ffc-lmpdv   1/1     Running   0          78m


 - Добавить в файл hosts разрешение имени homework.otus на IP-адрес из выдачи команды kubectl get ingress или kubectl get nodes -o wide
 - Подключиться к кластеру: minikube ssh
 - Проверить выдачу страницы: curl http://homework.otus/index.html

$ minikube ssh
docker@minikube:~$ curl http://homework.otus/index.html
curl http://homework.otus/index.html
<html><head></head><body><header>
<title>http://info.cern.ch</title>
</header>

<h1>http://info.cern.ch - home of the first website</h1>
<p>From here you can:</p>
<ul>
<li><a href="http://info.cern.ch/hypertext/WWW/TheProject.html">Browse the first website</a></li>
<li><a href="http://line-mode.cern.ch/www/hypertext/WWW/TheProject.html">Browse the first website using the line-mode browser simulator</a></li>
<li><a href="http://home.web.cern.ch/topics/birth-web">Learn about the birth of the web</a></li>
<li><a href="http://home.web.cern.ch/about">Learn about CERN, the physics laboratory where the web was born</a></li>
</ul>
</body></html>

 - Проверить выдачу страницы: curl http://homework.otus/conf/file

docker@minikube:~$ curl http://homework.otus/conf/file
curl http://homework.otus/conf/file
Content file via /conf/file

## PR checklist:
 - [V] Выставлен label с темой домашнего задания
