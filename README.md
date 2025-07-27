# Домашнее задание к занятию 14 «Средство визуализации Grafana»

> Репозиторий: hw-29\
> Выполнил: Асадбек Асадбеков\
> Дата: июль 2025

Этот проект — пример мониторинга с помощью связки **Prometheus + Grafana + Node Exporter**.\
Включены необходимые конфигурации, promQL-запросы, скриншоты и экспорт дашборда.

---

## Вопросы и ответы

### 1. Какой минимальный набор метрик вы выведите в мониторинг и почему?

**Ответ:**

- CPU Utilization — для контроля нагрузки на процессор
- Load Average — показатель средней загрузки системы
- Свободная оперативная память — чтобы видеть дефицит RAM
- Свободное место на диске — во избежание переполнения и падения сервисов

---

## Лабораторные шаги

### 1. Запуск Prometheus, Grafana и Node Exporter через docker-compose

Создаём рабочую папку и сохраняем туда `docker-compose.yml`:

<details><summary>docker-compose.yml</summary>

```yaml
version: '3.7'

services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter
    container_name: node-exporter
    ports:
      - "9100:9100"
    restart: unless-stopped
    command:
      - '--path.rootfs=/host'
    volumes:
      - '/:/host:ro,rslave'

  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
    restart: unless-stopped
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_SECURITY_ADMIN_USER=admin
```
</details>

#### prometheus.yml

<details><summary>prometheus.yml</summary>

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```
</details>

**Запуск:**

```bash
docker compose up -d
docker ps
```

#### Скриншот запуска контейнеров

![Запуск контейнеров](https://github.com/asad-bekov/hw-29/blob/main/img/1.PNG)

---

### 2. Вход в Grafana и подключение Prometheus DataSource

- Открыть [http://localhost:3000](http://localhost:3000)\
  (логин/пароль: **admin / admin**)
- В меню — Configuration → Data Sources → Add data source → Prometheus
- Указать URL: `http://prometheus:9090`
- Save & Test — Data source is working!

#### Скриншоты входа в `Graphana` и `DataSource`

![Вход в Grafana](https://github.com/asad-bekov/hw-29/blob/main/img/2.PNG)

![Вход в Grafana](https://github.com/asad-bekov/hw-29/blob/main/img/3.PNG)

---

### 3. Создание Dashboard и Panels

- Create → Dashboard → Add new panel
- Ниже — примеры promQL-запросов и как создавалось

#### **Panel 1: CPU Utilization (%)**

```promql
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)
```

#### **Panel 2: CPU Load Average (1/5/15)**

```promql
node_load1
node_load5
node_load15
```

#### **Panel 3: Free Memory (bytes/MB)**

```promql
node_memory_MemAvailable_bytes / 1024 / 1024
```

#### **Panel 4: Free Disk Space**

> Для работы node-exporter в Docker:

```promql
node_filesystem_free_bytes{mountpoint="/etc/hostname"}
```

---

#### Скриншот итогового Dashboard

![Dashboard](https://github.com/asad-bekov/hw-29/blob/main/img/4.PNG)

---

### 4. Настройка алертов и отправка уведомлений в Telegram

- Для каждой панели настроены alert rules
- В Contact points добавлен Telegram Webhook:
  - Token: `<скрыто>`
  - Chat\_id: `-1002555466252`

#### Пример Alert rule

- CPU Utilization > 90% 1 минуту:

```promql
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100) > 90
```

- Free Memory < 100 MB:

```promql
node_memory_MemAvailable_bytes / 1024 / 1024 < 100
```

#### Скриншоты `test contact point` и `Alert rules`

![Alert Rules](https://github.com/asad-bekov/hw-29/blob/main/img/5.PNG)

![Alert Rules](https://github.com/asad-bekov/hw-29/blob/main/img/6.PNG)

#### Скриншот Telegram уведомления

![Telegram alert](https://github.com/asad-bekov/hw-29/blob/main/img/7.PNG)

---

### 5. Экспорт Dashboard (JSON MODEL)

- Dashboard → Settings → JSON Model → Copy all  
- Сохранил как [mydashboard.json](https://github.com/asad-bekov/hw-29/blob/main/mydashboard.json)

---

## Ссылки

- [Документация Grafana](https://grafana.com/docs/)
- [Документация Prometheus](https://prometheus.io/docs/)
- [Prometheus Node Exporter](https://github.com/prometheus/node_exporter)

---