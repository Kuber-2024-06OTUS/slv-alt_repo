# Выполнено ДЗ №5

 - [V] Основное ДЗ

## В процессе сделано:
 - Создан манифест sa-monitoring.yaml создающий service account monitoring c ClusterRole и ClusterRoleBinding
для доступа к эндпоинту /metrics кластера
 - Изменен манифест deployment.yaml так, чтобы поды запускались под service account monitoring
 - Создан манифест sa-cd.yaml, создающий в namespace homework service account с именем cd с ролью admin в рамках namespace homework 
 - Создан kubeconfig для service account cd

## Как запустить проект:
 - Переключиться в namespace homework: kubectl config set-context --current --namespace=homework
 - Установить метку на ноду: kubectl label nodes minikube homework=true
 - Установить ingress контроллер Nginx: minikube addons enable ingress
 - Применить манифесты: kubectl apply -f sa-monitoring.yaml -f deployment.yaml -f sa-cd.yaml
 - Для создания kibeconfig выполнен экпорт параметров в перменные окружения следующими командами:
export SA_SECRET_TOKEN=$(kubectl -n homework get secret/sa-cd-secret -o=go-template='{{.data.token}}' | base64 --decode)
export CLUSTER_NAME=$(kubectl config current-context)
export CURRENT_CLUSTER=$(kubectl config view --raw -o=go-template='{{range .contexts}}{{if eq .name "'''${CLUSTER_NAME}'''"}}{{ index .context "cluster" }}{{end}}{{end}}')
export CLUSTER_CA_CERT=$(kubectl config view --raw -o=go-template='{{range .clusters}}{{if eq .name "'''${CURRENT_CLUSTER}'''"}}"{{with index .cluster "certificate-authority-data" }}{{.}}{{end}}"{{ end }}{{ end }}')
export CLUSTER_ENDPOINT=$(kubectl config view --raw -o=go-template='{{range .clusters}}{{if eq .name "'''${CURRENT_CLUSTER}'''"}}{{ .cluster.server }}{{end}}{{ end }}')

Генерируем конфиг

cat << EOF > kubeconfig
apiVersion: v1
kind: Config
current-context: ${CLUSTER_NAME}
contexts:
- name: ${CLUSTER_NAME}
  context:
    cluster: ${CLUSTER_NAME}
    user: sa-cd-secret
clusters:
- name: ${CLUSTER_NAME}
  cluster:
    certificate-authority-data: ${CLUSTER_CA_CERT}
    server: ${CLUSTER_ENDPOINT}
users:
- name: sa-cd-secret
  user:
    token: ${SA_SECRET_TOKEN}
EOF

 - Создан token следущей командой: kubectl create token cd --duration=24h > token

## Как проверить работоспособность:
 - Проверить выдачу комманд:

kubectl get sa

$ kubectl get sa
NAME         SECRETS   AGE
cd           0         19h
default      0         19d
monitoring   0         22h

kubectl get clusterrole

$ kubectl get clusterrole
NAME                                                                   CREATED AT
metrics                                                                2024-07-25T16:47:40Z
Убедиться в наличии ClusterRole metrics

$ kubectl get clusterrolebinding
NAME                                                            ROLE                                                                               AGE
metrics                                                         ClusterRole/metrics                                                                22h
Убедиться в наличии ClusterRoleBinding metrics
