Данное тестовое задание необходимо для понимание работы сетевой моедли в Kubernetes (Service discovery, DNS, Ingress, NetworkPolicy).

Результаты тестирования DNS приложены в папке tests.

Доказательство работы Service Discovery приложены в папке tests.

Результаты тестирования NetworkPolicy приложены в папке tests и screenshots.

Различия между типами Service:

ClusterIP необходим для коммуникации внутри кластера. Доступ только внутри кластера через виртуальный IP.
NodePort предоставляет внешний доступ через статический порт на каждой ноде.
LoadBalancer необходим для интеграции с облачными балансировщиками нагрузки.

Анализ сетевой архитектуры и потоков трафика:
Внешний трафик: Пользователь → LoadBalancer/NodePort → Service → Pod
Внутренний трафик: Pod A → ClusterIP Service → Pod B
Внешние зависимости: Pod → Egress Policy → External Service

Ключевые компоненты:
kube-proxy: Реализует правила iptables/IPVS для балансировки
CNI Plugin: Обеспечивает сетевое взаимодействие между подами
CoreDNS: Разрешение внутренних DNS-имен
Ingress Controller: Маршрутизация HTTP/HTTPS трафика

Команды для отладки сетевых проблем
# DNS резолюция
nslookup service-name.namespace.svc.cluster.local

# Проверка доступности
curl -v http://service-name:port/

# Сканирование портов  
nmap -p 1-65535 service-ip

# Трассировка маршрута
traceroute service-ip

# Анализ NetworkPolicy
kubectl describe networkpolicy policy-name -n namespace

# Проверка endpoints
kubectl get endpoints -n namespace