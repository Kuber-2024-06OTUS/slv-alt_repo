# Выполнено ДЗ № 10

Развернут кластер на основе ОС Debian12 из 5 нод, из которых - 3 управляющих и 2 рабочих.
Облачные услуги использовать не будем. Для удобства доступа к приложениям, в т.ч. веб-серверам, которые будут развернуты на нодах, был установлен metallb.

Добавляем argo chart репозиторий в Helm:
# helm repo add argo https://argoproj.github.io/argo-helm
"argo" has been added to your repositories

Проверяем список доступных репозиториев:
# helm repo list
NAME            URL
longhorn        https://charts.longhorn.io
grafana         https://grafana.github.io/helm-charts
argo            https://argoproj.github.io/argo-helm

Смотрим список доступных чартов:
# helm search repo argo
NAME                            CHART VERSION   APP VERSION     DESCRIPTION
argo/argo                       1.0.0           v2.12.5         A Helm chart for Argo Workflows
argo/argo-cd                    7.6.7           v2.12.4         A Helm chart for Argo CD, a declarative, GitOps...
argo/argo-ci                    1.0.0           v1.0.0-alpha2   A Helm chart for Argo-CI
argo/argo-events                2.4.8           v1.9.2          A Helm chart for Argo Events, the event-driven ...
argo/argo-lite                  0.1.0                           Lighweight workflow engine for Kubernetes
argo/argo-rollouts              2.37.7          v1.7.2          A Helm chart for Argo Rollouts
argo/argo-workflows             0.42.5          v3.5.11         A Helm chart for Argo Workflows
argo/argocd-applicationset      1.12.1          v0.4.1          A Helm chart for installing ArgoCD ApplicationSet
argo/argocd-apps                2.0.2                           A Helm chart for managing additional Argo CD Ap...
argo/argocd-image-updater       0.11.0          v0.14.0         A Helm chart for Argo CD Image Updater, a tool ...
argo/argocd-notifications       1.8.1           v1.2.1          A Helm chart for ArgoCD notifications, an add-o...

Создаем файл "values.txt" с полным списком значений для чарта argo/argo-cd
helm show values argo/argo-cd >val-argo-cd.txt

Создаем файл val-argo.yaml

Установка argo-cd
helm install argo-cd argo/argo-cd --values val-argo-cd.yaml --namespace argo --create-namespace
# helm install argo-cd argo/argo-cd --values val-argo-cd.yaml --namespace argo --create-namespace
NAME: argo-cd
LAST DEPLOYED: Thu Oct  3 11:41:59 2024
NAMESPACE: argo
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
In order to access the server UI you have the following options:

1. kubectl port-forward service/argo-cd-argocd-server -n argo 8080:443

    and then open the browser on http://localhost:8080 and accept the certificate

2. enable ingress in the values file `server.ingress.enabled` and either
      - Add the annotation for ssl passthrough: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-1-ssl-passthrough
      - Set the `configs.params."server.insecure"` in the values file and terminate SSL at your ingress: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-2-multiple-ingress-objects-and-hosts


After reaching the UI the first time you can login with username: admin and the random password generated during the installation. You can find the password by running:

kubectl -n argo get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

(You should delete the initial secret afterwards as suggested by the Getting Started Guide: https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli)


 - Создание проекта Otus

Логинимся в веб-интерфейсе, создаем проект otus. Указываем source, destination.
Для получения информации при создании манифеста проекта воспользуемся argo CLI.

Установим argocd через pkgx
pkgx argocd

Логинимся в кластер
pkgx argocd login --core

Сменим текущий ns на argo
kubectl config set-context --current --namespace=argo

Получим список проектов
# pkgx argocd proj list
2024/10/09 21:39:59 maxprocs: Leaving GOMAXPROCS=2: CPU quota undefined
NAME       DESCRIPTION  DESTINATIONS                      SOURCES                                                CLUSTER-RESOURCE-WHITELIST  NAMESPACE-RESOURCE-BLACKLIST  SIGNATURE-KEYS  ORPHANED-RESOURCES  DESTINATION-SERVICE-ACCOUNTS
default                 *,*                               *                                                      */*                         <none>                        <none>          disabled            <none>
otus                    https://kubernetes.default.svc,*  https://github.com/Kuber-2024-06OTUS/slv-alt_repo.git  */*                         <none>                        <none>          disabled            <none>
otus-copy               https://kubernetes.default.svc,*  https://github.com/slv-alt/otus-copy.git               */*                         <none>                        <none>          disabled            <none>

Т.к. предыдущие ДЗ были сделаны без учета всех требований возникших к текущему ДЗ, будем использовать github-репозирторий:
https://github.com/slv-alt/otus-copy.git
 

Получим детальную информацию о проекте в формате json
pkgx argocd proj get otus -o yaml

Либо через kubectl
# kubectl describe AppProject otus-copy
Name:         otus-copy
Namespace:    argo
Labels:       <none>
Annotations:  <none>
API Version:  argoproj.io/v1alpha1
Kind:         AppProject
Metadata:
  Creation Timestamp:  2024-10-07T16:50:09Z
  Generation:          1
  Resource Version:    466564
  UID:                 d42cf121-273d-47b9-937f-7ac8997676c4
Spec:
  Cluster Resource Whitelist:
    Group:  *
    Kind:   *
  Destinations:
    Name:       in-cluster
    Namespace:  *
    Server:     https://kubernetes.default.svc
  Namespace Resource Whitelist:
    Group:  *
    Kind:   *
  Source Repos:
    https://github.com/slv-alt/otus-copy.git
Events:  <none>

По спецификации декларации проекта - https://argo-cd.readthedocs.io/en/stable/operator-manual/project-specification/
и выведенного блока данных о проекте создаем манифест, описывающий проект - project-otus-copy.yaml


 - Создание приложения ArgoCD - репозиторий kubernetes-networks

В веб-интерфейсе создаем приложение для соответствующего репозитория.
Для установки приложения необходимо установить метки на ноды (требование в манифесте deployment.yaml).
Устанвливаем соответствующие метки на рабочие ноды
kubectl label nodes node4.int homework=true
kubectl label nodes node5.int homework=true
kubectl label nodes node5.int storage=true

Получение списка созданных приложение через утилиту argocd
# pkgx argocd app list
2024/10/09 21:50:58 maxprocs: Leaving GOMAXPROCS=2: CPU quota undefined
NAME                      CLUSTER                         NAMESPACE  PROJECT    STATUS  HEALTH       SYNCPOLICY  CONDITIONS  REPO                                      PATH                 TARGET
argo/kubernetes-networks  https://kubernetes.default.svc  homework   otus-copy  Synced  Progressing  Manual      <none>      https://github.com/slv-alt/otus-copy.git  kubernetes-networks  HEAD

Получение списка созданных приложение через kubectl
# kubectl get Application
NAME                  SYNC STATUS   HEALTH STATUS
kubernetes-networks   Synced        Progressing

Получение данных приложения для создания манифеста
kubectl get Application kubernetes-networks -o yaml

По спецификации декларации проекта - https://argo-cd.readthedocs.io/en/stable/user-guide/application-specification/
и выведенного блока данных о приложении создаем манифест, описывающий приложение - app-kub-networks.yaml


 - Создание приложения ArgoCD - репозиторий kubernetes-templating

В веб-интерфейсе создаем приложение для соответствующего репозитория.

Получение списка созданных приложение через утилиту argocd
# pkgx argocd app list
2024/10/09 22:02:22 maxprocs: Leaving GOMAXPROCS=2: CPU quota undefined
NAME                        CLUSTER                         NAMESPACE     PROJECT    STATUS     HEALTH       SYNCPOLICY  CONDITIONS  REPO                                      PATH                           TARGET
argo/kubernetes-networks    https://kubernetes.default.svc  homework      otus-copy  Synced     Progressing  Manual      <none>      https://github.com/slv-alt/otus-copy.git  kubernetes-networks            HEAD
argo/kubernetes-templating  https://kubernetes.default.svc  homeworkhelm  otus-copy  OutOfSync  Healthy      Auto-Prune  SyncError   https://github.com/slv-alt/otus-copy.git  kubernetes-templating/mychart  kubernetes-templating

Получение данных приложения для создания манифеста
kubectl get Application kubernetes-templating -o yaml

По спецификации декларации проекта - https://argo-cd.readthedocs.io/en/stable/user-guide/application-specification/
и выведенного блока данных о приложении создаем манифест, описывающий приложение - app-kub-templating.yaml


## PR checklist:
 - [V] Выставлен label с темой домашнего задания