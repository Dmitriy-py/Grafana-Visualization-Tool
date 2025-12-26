# Домашнее задание к занятию  «Средство визуализации Grafana»

## ` Дмитрий Климов `

## Задание повышенной сложности

При решении задания 1 не используйте директорию help для сборки проекта. Самостоятельно разверните grafana, где в роли источника данных будет выступать prometheus, а сборщиком данных будет node-exporter:

  * grafana;
  * prometheus-server;
  * prometheus node-exporter.
```
За дополнительными материалами можете обратиться в официальную документацию grafana и prometheus.

В решении к домашнему заданию также приведите все конфигурации, скрипты, манифесты, которые вы использовали в процессе решения задания.

При решении задания 3 вы должны самостоятельно завести удобный для вас канал нотификации, например, Telegram или email, и отправить туда тестовые события.
```
В решении приведите скриншоты тестовых событий из каналов нотификаций.

Обязательные задания
Задание 1
Используя директорию help внутри этого домашнего задания, запустите связку prometheus-grafana.
Зайдите в веб-интерфейс grafana, используя авторизационные данные, указанные в манифесте docker-compose.
Подключите поднятый вами prometheus, как источник данных.
Решение домашнего задания — скриншот веб-интерфейса grafana со списком подключенных Datasource.

## Ответ:

### ` docker-compose.yml `
```yml
version: '3.8'

services:
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin 

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - --config.file=/etc/prometheus/prometheus.yml
      - --storage.tsdb.path=/prometheus

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($|/)'
volumes:
  prometheus_data:
  grafana_data:
```
### ` prometheus.yml `

```yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```

<img width="1920" height="1080" alt="Снимок экрана (2197)" src="https://github.com/user-attachments/assets/fefd76d3-cf40-4f98-b648-306d76c85b72" />

## Задание 2

Изучите самостоятельно ресурсы:

1. PromQL tutorial for beginners and humans.
2. Understanding Machine CPU usage.
3. Introduction to PromQL, the Prometheus query language.

### Создайте Dashboard и в ней создайте Panels:

   * утилизация CPU для nodeexporter (в процентах, 100-idle);
   * CPULA 1/5/15;
   * количество свободной оперативной памяти;
   * количество места на файловой системе.
### Для решения этого задания приведите promql-запросы для выдачи этих метрик, а также скриншот получившейся Dashboard.

## Ответ:

| Метрика | PromQL Запрос | Описание |
| :--- | :--- | :--- |
| **Утилизация CPU (%)** | `(1 - avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) ) * 100` | Вычисляет процент занятого процессорного времени, исключая режим простоя (idle). |
| **CPU Load Average** | `node_load1`, `node_load5`, `node_load15` | Отображает среднюю нагрузку на систему за последние 1, 5 и 15 минут. |
| **Свободная RAM** | `node_memory_MemAvailable_bytes` | Показывает объем доступной оперативной памяти в байтах. |
| **Место на диске** | `node_filesystem_avail_bytes{fstype!="rootfs", mountpoint!="/run"}` | Показывает свободное место на смонтированных файловых системах, исключая системные и временные разделы. |

<img width="1920" height="1080" alt="Снимок экрана (2203)" src="https://github.com/user-attachments/assets/6476c4c1-4a19-4b45-bcb6-818fcec91602" />










