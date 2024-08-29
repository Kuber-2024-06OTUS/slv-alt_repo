# Выполнено ДЗ №8

 - [V] Основное ДЗ


## В процессе сделано:

 - Создан кастомный образ nginx отдающий метрики
Образ генерируется следущей командой:
$ docker build -t nginx-metric:v1 .
Для построения образа требуются файлы:
Dockerfile - манифест для построения образа
nginx-default.conf - конфигурационный файл nginx для поддержки метрик
Загрузить образ в minikube:
$ minikube cache add nginx-metric:v1
Отобразить образы добавленные в кеш:
$ minikube cache list

 - Установлен в кластер Prometheus-operator
$ helm install my-release oci://registry-1.docker.io/bitnamicharts/kube-prometheus
Pulled: registry-1.docker.io/bitnamicharts/kube-prometheus:9.5.11
Digest: sha256:8c96f904674eb1aa52a6da94c5ecb9243a0293a3faab9513082c77ddd0d71def
NAME: my-release
LAST DEPLOYED: Fri Aug  9 22:55:52 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: kube-prometheus
CHART VERSION: 9.5.11
APP VERSION: 0.75.2

 - Создан deployment (deployment.yaml) для запуска созданного ранее образа nginx
и сервис для него (service.yaml)

Применяем манифест для деплоймента
$ kubectl apply -f deployment.yaml

Проверяем наличие данного деплоймента, а также ранее созданных для оператора:
$ kubectl get deploy
NAME                                           READY   UP-TO-DATE   AVAILABLE   AGE
my-release-kube-prometheus-blackbox-exporter   1/1     1            1           18h
my-release-kube-prometheus-operator            1/1     1            1           18h
my-release-kube-state-metrics                  1/1     1            1           18h
nginx-deploy                                   1/1     1            1           62m

Проверяем наличие подов:
$ kubectl get pod
NAME                                                            READY   STATUS    RESTARTS      AGE
alertmanager-my-release-kube-prometheus-alertmanager-0          2/2     Running   2 (18h ago)   19h
my-release-kube-prometheus-blackbox-exporter-757bfc8dd9-fwq4h   1/1     Running   1 (18h ago)   19h
my-release-kube-prometheus-operator-c59c55bb-zmhvl              1/1     Running   2 (89m ago)   19h
my-release-kube-state-metrics-5fd7d47c94-rvnpg                  1/1     Running   2 (89m ago)   19h
my-release-node-exporter-6bv9p                                  1/1     Running   1 (18h ago)   19h
nginx-deploy-cc5dbd7f8-sjqw6                                    1/1     Running   0             67m
prometheus-my-release-kube-prometheus-prometheus-0              2/2     Running   2 (18h ago)   19h

Применяем манифест для сервиса
$ kubectl apply -f service.yaml

Проверяем наличие сервисов
$ kubectl get svc
NAME                                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
alertmanager-operated                          ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP   19h
kubernetes                                     ClusterIP   10.96.0.1        <none>        443/TCP                      40d
my-release-kube-prometheus-alertmanager        ClusterIP   10.96.209.174    <none>        9093/TCP                     19h
my-release-kube-prometheus-blackbox-exporter   ClusterIP   10.98.75.217     <none>        19115/TCP                    19h
my-release-kube-prometheus-operator            ClusterIP   10.102.121.174   <none>        8080/TCP                     19h
my-release-kube-prometheus-prometheus          ClusterIP   10.106.138.228   <none>        9090/TCP                     19h
my-release-kube-state-metrics                  ClusterIP   10.97.137.13     <none>        8080/TCP                     19h
my-release-node-exporter                       ClusterIP   10.106.146.65    <none>        9100/TCP                     19h
prometheus-operated                            ClusterIP   None             <none>        9090/TCP                     19h
svc-nginx                                      ClusterIP   10.99.125.3      <none>        80/TCP                       20m

Подключаемся к кластеру и проверяем выдачу метрик сервером nginx, IP берем из выдачи svc-nginx
$ minikube ssh
docker@minikube:~$ curl http://10.99.125.3/nginx/status
curl http://10.99.125.3/nginx/status
Active connections: 1
server accepts handled requests
 3 3 3
Reading: 0 Writing: 1 Waiting: 0


 - Создан манифест (exporter-deploy.yaml) для запуска nginx prometheus exporter

Применяем данный манифест
$ kubectl apply -f exporter-deploy.yaml

Проверяем наличие деплоймента
$ kubectl get deployment |grep nginx-prometheus-exporter
nginx-prometheus-exporter                      1/1     1            1           16d


 - Создан манифест serviceMonitor

Применяем манифест
$ kubectl apply -f serviceMonitor.yaml
service/nginx-prometheus-exporter created

Проверяем наличие сервиса
$ kubectl get svc |grep nginx-prometheus-exporter
nginx-prometheus-exporter                      ClusterIP   10.108.105.143   <none>        80/TCP                       31m

Подключаемся к кластеру и проверяем выдачу метрик в формате Prometheus:
$ minikube ssh
docker@minikube:~$ curl http://10.108.105.143/metrics
curl http://10.108.105.143/metrics
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0
go_gc_duration_seconds{quantile="0.25"} 0
go_gc_duration_seconds{quantile="0.5"} 0
go_gc_duration_seconds{quantile="0.75"} 0
go_gc_duration_seconds{quantile="1"} 0
go_gc_duration_seconds_sum 0
go_gc_duration_seconds_count 0
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 11
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.22.5"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 227200
# HELP go_memstats_alloc_bytes_total Total number of bytes allocated, even if freed.
# TYPE go_memstats_alloc_bytes_total counter
go_memstats_alloc_bytes_total 227200
# HELP go_memstats_buck_hash_sys_bytes Number of bytes used by the profiling bucket hash table.
# TYPE go_memstats_buck_hash_sys_bytes gauge
go_memstats_buck_hash_sys_bytes 8154
# HELP go_memstats_frees_total Total number of frees.
# TYPE go_memstats_frees_total counter
go_memstats_frees_total 0
# HELP go_memstats_gc_sys_bytes Number of bytes used for garbage collection system metadata.
# TYPE go_memstats_gc_sys_bytes gauge
go_memstats_gc_sys_bytes 1.394152e+06
# HELP go_memstats_heap_alloc_bytes Number of heap bytes allocated and still in use.
# TYPE go_memstats_heap_alloc_bytes gauge
go_memstats_heap_alloc_bytes 227200
# HELP go_memstats_heap_idle_bytes Number of heap bytes waiting to be used.
# TYPE go_memstats_heap_idle_bytes gauge
go_memstats_heap_idle_bytes 1.72032e+06
# HELP go_memstats_heap_inuse_bytes Number of heap bytes that are in use.
# TYPE go_memstats_heap_inuse_bytes gauge
go_memstats_heap_inuse_bytes 1.851392e+06
# HELP go_memstats_heap_objects Number of allocated objects.
# TYPE go_memstats_heap_objects gauge
go_memstats_heap_objects 708
# HELP go_memstats_heap_released_bytes Number of heap bytes released to OS.
# TYPE go_memstats_heap_released_bytes gauge
go_memstats_heap_released_bytes 1.72032e+06
# HELP go_memstats_heap_sys_bytes Number of heap bytes obtained from system.
# TYPE go_memstats_heap_sys_bytes gauge
go_memstats_heap_sys_bytes 3.571712e+06
# HELP go_memstats_last_gc_time_seconds Number of seconds since 1970 of last garbage collection.
# TYPE go_memstats_last_gc_time_seconds gauge
go_memstats_last_gc_time_seconds 0
# HELP go_memstats_lookups_total Total number of pointer lookups.
# TYPE go_memstats_lookups_total counter
go_memstats_lookups_total 0
# HELP go_memstats_mallocs_total Total number of mallocs.
# TYPE go_memstats_mallocs_total counter
go_memstats_mallocs_total 708
# HELP go_memstats_mcache_inuse_bytes Number of bytes in use by mcache structures.
# TYPE go_memstats_mcache_inuse_bytes gauge
go_memstats_mcache_inuse_bytes 9600
# HELP go_memstats_mcache_sys_bytes Number of bytes used for mcache structures obtained from system.
# TYPE go_memstats_mcache_sys_bytes gauge
go_memstats_mcache_sys_bytes 15600
# HELP go_memstats_mspan_inuse_bytes Number of bytes in use by mspan structures.
# TYPE go_memstats_mspan_inuse_bytes gauge
go_memstats_mspan_inuse_bytes 44160
# HELP go_memstats_mspan_sys_bytes Number of bytes used for mspan structures obtained from system.
# TYPE go_memstats_mspan_sys_bytes gauge
go_memstats_mspan_sys_bytes 48960
# HELP go_memstats_next_gc_bytes Number of heap bytes when next garbage collection will take place.
# TYPE go_memstats_next_gc_bytes gauge
go_memstats_next_gc_bytes 4.194304e+06
# HELP go_memstats_other_sys_bytes Number of bytes used for other system allocations.
# TYPE go_memstats_other_sys_bytes gauge
go_memstats_other_sys_bytes 981534
# HELP go_memstats_stack_inuse_bytes Number of bytes in use by the stack allocator.
# TYPE go_memstats_stack_inuse_bytes gauge
go_memstats_stack_inuse_bytes 622592
# HELP go_memstats_stack_sys_bytes Number of bytes obtained from system for stack allocator.
# TYPE go_memstats_stack_sys_bytes gauge
go_memstats_stack_sys_bytes 622592
# HELP go_memstats_sys_bytes Number of bytes obtained from system.
# TYPE go_memstats_sys_bytes gauge
go_memstats_sys_bytes 6.642704e+06
# HELP go_threads Number of OS threads created.
# TYPE go_threads gauge
go_threads 9
# HELP nginx_connections_accepted Accepted client connections
# TYPE nginx_connections_accepted counter
nginx_connections_accepted 2
# HELP nginx_connections_active Active client connections
# TYPE nginx_connections_active gauge
nginx_connections_active 1
# HELP nginx_connections_handled Handled client connections
# TYPE nginx_connections_handled counter
nginx_connections_handled 2
# HELP nginx_connections_reading Connections where NGINX is reading the request header
# TYPE nginx_connections_reading gauge
nginx_connections_reading 0
# HELP nginx_connections_waiting Idle client connections
# TYPE nginx_connections_waiting gauge
nginx_connections_waiting 0
# HELP nginx_connections_writing Connections where NGINX is writing the response back to the client
# TYPE nginx_connections_writing gauge
nginx_connections_writing 1
# HELP nginx_exporter_build_info A metric with a constant '1' value labeled by version, revision, branch, goversion from which nginx_exporter was built, and the goos and goarch for the build.
# TYPE nginx_exporter_build_info gauge
nginx_exporter_build_info{branch="HEAD",goarch="amd64",goos="linux",goversion="go1.22.5",revision="9522f4e39ee1aed817d7d70a89514ccc0ae1594a",tags="unknown",version="1.3.0"} 1
# HELP nginx_http_requests_total Total http requests
# TYPE nginx_http_requests_total counter
nginx_http_requests_total 2
# HELP nginx_up Status of the last metric scrape
# TYPE nginx_up gauge
nginx_up 1
# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
process_cpu_seconds_total 0.05
# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1.048576e+06
# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 10
# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 1.0100736e+07
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.7247868507e+09
# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 1.26636032e+09
# HELP process_virtual_memory_max_bytes Maximum amount of virtual memory available in bytes.
# TYPE process_virtual_memory_max_bytes gauge
process_virtual_memory_max_bytes 1.8446744073709552e+19
# HELP promhttp_metric_handler_requests_in_flight Current number of scrapes being served.
# TYPE promhttp_metric_handler_requests_in_flight gauge
promhttp_metric_handler_requests_in_flight 1
# HELP promhttp_metric_handler_requests_total Total number of scrapes by HTTP status code.
# TYPE promhttp_metric_handler_requests_total counter
promhttp_metric_handler_requests_total{code="200"} 0
promhttp_metric_handler_requests_total{code="500"} 0
promhttp_metric_handler_requests_total{code="503"} 0


## PR checklist:
 - [V] Выставлен label с темой домашнего задания