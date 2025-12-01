# ClickHouse Helm Chart

Helm chart для развёртывания single-инсталляции ClickHouse в Kubernetes.

## Описание

Этот чарт развёртывает один экземпляр ClickHouse с поддержкой:
- Настраиваемой версии образа через `values.yaml`
- Произвольного количества пользователей с паролями
- Хранилища данных через PersistentVolumeClaim
- HTTP и Native интерфейсов ClickHouse
- Проверок доступности (readiness/liveness probes)

**Структура:**
- `Deployment` — управляет запуском контейнера ClickHouse
- `Service` — предоставляет доступ к HTTP и Native портам
- `ConfigMap` — хранит конфигурацию пользователей (users.xml)
- `Secret` — хранит хеши паролей пользователей
- `PersistentVolumeClaim` — выделяет хранилище для данных

## Требования

- Kubernetes 1.19+
- Helm 3.0+
- Доступный `StorageClass` в кластере (по умолчанию используется `standard`)

## Установка

### 1. Базовая установка с параметрами по умолчанию
```
helm install my-clickhouse ./clickhouse-helm-chart
--namespace clickhouse
--create-namespace
```

### 2. Установка с переопределением версии ClickHouse
```
helm install my-clickhouse ./clickhouse-helm-chart
--namespace clickhouse
--create-namespace
--set clickhouse.image.tag=23.12.1-alpine
```

### 3. Установка с пользовательскими пользователями и паролями
```
helm install my-clickhouse ./clickhouse-helm-chart
--namespace clickhouse
--create-namespace
--set 'users.name=admin'
--set 'users.password=my_secure_password'
--set 'users.name=readonly'
--set 'users.password=readonly_pass'
```

### 4. Установка из файла values

Создай `my-values.yaml`:
```
clickhouse:
image:
tag: "24.1.1-alpine"

storage:
size: 50Gi

users:

name: admin
password: "admin_pass_123"

name: app
password: "app_pass_456"

name: readonly
password: "read_only_789"
```

Затем установи:
```
helm install my-clickhouse ./clickhouse-helm-chart
--namespace clickhouse
--create-namespace
--values my-values.yaml
```

## Конфигурация

### Версия ClickHouse

Чтобы изменить версию ClickHouse, отредактируй `values.yaml` или переопредели при установке:
```
--set clickhouse.image.tag=24.1.1-alpine
```

Доступные теги на [Docker Hub](https://hub.docker.com/r/clickhouse/clickhouse-server/tags).

### Пользователи и пароли

В `values.yaml` раздел `users`:
```
users:

name: default
password: "changeme123"
quota: "default"

name: app_user
password: "app_secure_password_456"
quota: "default"
```
Каждый пользователь:
- `name` — имя пользователя для входа
- `password` — пароль (будет захеширован в Secret)
- `quota` — квота на ресурсы (используется профиль)

Пароль автоматически:
1. Сохраняется в Kubernetes Secret в виде SHA256 хеша
2. Монтируется в контейнер как переменная окружения
3. Используется в конфигурации users.xml

### Ресурсы
```
resources:
requests:
cpu: 100m
memory: 512Mi
limits:
cpu: 2000m
memory: 2Gi


- `requests` — гарантированные ресурсы при запуске Pod
- `limits` — максимально допустимые ресурсы
```
### Хранилище
```
storage:
size: 10Gi
storageClassName: standard


- `size` — размер PersistentVolumeClaim
- `storageClassName` — класс хранилища (должен существовать в кластере)
```
### Порты
```
service:
type: ClusterIP
httpPort: 8123 # HTTP интерфейс
nativePort: 9000 # Native протокол
```

## Использование

### Подключение к ClickHouse из Pod в кластере

Подключиться через HTTP интерфейс
```
kubectl exec -it deployment/my-clickhouse-clickhouse
-n clickhouse
-- curl -s -u default:changeme123
http://localhost:8123/?query=SELECT%201
```

### Port forwarding для локального доступа

Перенаправить HTTP порт
```
kubectl port-forward -n clickhouse svc/my-clickhouse-clickhouse 8123:8123
```
Перенаправить Native порт
```
kubectl port-forward -n clickhouse svc/my-clickhouse-clickhouse 9000:9000
```
Затем подключиться локально
```
clickhouse-client --host localhost --user admin --password
```

### Проверка логов
```
kubectl logs -n clickhouse deployment/my-clickhouse-clickhouse -f
```

### Проверка статуса
```
kubectl get all -n clickhouse
kubectl describe pod -n clickhouse
```

## Обновление

### Обновление версии ClickHouse
```
helm upgrade my-clickhouse ./clickhouse-helm-chart
--namespace clickhouse
--set clickhouse.image.tag=24.2.0-alpine
```

### Обновление конфигурации
```
helm upgrade my-clickhouse ./clickhouse-helm-chart
--namespace clickhouse
--values my-values.yaml
```

## Удаление
```
helm uninstall my-clickhouse --namespace clickhouse
```

**Важно:** PersistentVolumeClaim и его данные НЕ удаляются автоматически (для защиты данных).
Чтобы удалить данные:
```
kubectl delete pvc -n clickhouse my-clickhouse-clickhouse-data
```

## Примеры команд установки

### Полный пример для стажировки

Создать namespace
```
kubectl create namespace clickhouse-dev
```
Установить ClickHouse с кастомными пользователями
```
helm install clickhouse-dev ./clickhouse-helm-chart
--namespace clickhouse-dev
--values - << EOF
clickhouse:
image:
tag: "24.1.1-alpine"

storage:
size: 20Gi

resources:
requests:
cpu: 200m
memory: 1Gi
limits:
cpu: 1000m
memory: 2Gi

users:

name: admin
password: "admin_secure_123"

name: test_app
password: "test_app_456"

name: analytics
password: "analytics_789"
EOF
```
Проверить статус
```
kubectl get all -n clickhouse-dev
```
Port forward для проверки
```
kubectl port-forward -n clickhouse-dev svc/clickhouse-dev-clickhouse 8123:8123
```
Проверить HTTP интерфейс
```
curl -u admin:admin_secure_123 http://localhost:8123/ping
```




.kube/

# Логи
*.log
