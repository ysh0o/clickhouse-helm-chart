# ClickHouse Helm Chart

Helm‑чарт для быстрого развёртывания single-инстанса ClickHouse в Kubernetes через StatefulSet с persistent‑хранилищем и настраиваемыми пользователями.


## Возможности

- Один экземпляр ClickHouse на базе StatefulSet (подходит для dev/test)  
- Настройка нужной версии через values.yaml или `--set clickhouse.image.tag=...`  
- Добавление произвольного числа пользователей; пароли сохраняются как SHA256‑хеш в Kubernetes Secret  
- PersistentVolumeClaim для хранения данных  
- HTTP и Native порты, readiness/liveness‑пробы  
- Полностью параметризованный через values.yaml

## Требования

- Kubernetes 1.19+
- Helm 3+
- StorageClass в кластере (по умолчанию `standard`)

## Быстрый старт
```
git clone https://github.com/ysh0o/clickhouse-helm-chart.git
cd clickhouse-helm-chart
helm install my-clickhouse ./clickhouse-helm-chart --namespace clickhouse --create-namespace
```

**Установка с нужной версией ClickHouse:**
```
helm install my-clickhouse ./clickhouse-helm-chart
--namespace clickhouse
--create-namespace
--set clickhouse.image.tag=24.1.1-alpine
```

**Добавление своих пользователей:**
```
helm install my-clickhouse ./clickhouse-helm-chart
--namespace clickhouse
--create-namespace
--set users.name=admin,users.password=admin_pass
--set users.name=app,users.password=app_pass​
```
