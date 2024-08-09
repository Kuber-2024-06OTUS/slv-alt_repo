# Репозиторий для выполнения домашних заданий курса "Инфраструктурная платформа на основе Kubernetes-2024-06" 
# Выполнено ДЗ №3

 - [V] Основное ДЗ
 - [V] Задание со *

## В процессе сделано:
 - Изменена проба в манифесте deployment.yaml
 - Создан манифест service.yaml
 - В кластер установлен ingress контроллер Nginx: minikube addons enable ingress
 - Создан манифест ingress.yaml

## Как запустить проект:
 - Переключиться в namespace homework: kubectl config set-context --current --namespace=homework
 - Применить манифесты

## Как проверить работоспособность:
 - Добавить в файл hosts разрешение имени homework.otus на IP-адрес из выдачи команды kubectl get ingress или kubectl get nodes -o wide
 - Подключиться к кластеру: minikube ssh
 - Проверить выдачу страницы: curl http://homework.otus/index.html
 - Проверить выдачу страницы: curl http://homework.otus/homepage

## PR checklist:
 - [V] Выставлен label с темой домашнего задания
