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

![Alert Rules](https://github.com/asad-bekov/hw-29/blob/main/img/6.PNG)

![Alert Rules](https://github.com/asad-bekov/hw-29/blob/main/img/5.PNG)

#### Скриншот Telegram уведомления

![Telegram alert](https://github.com/asad-bekov/hw-29/blob/main/img/5.PNG)

---

### 5. Экспорт Dashboard (JSON MODEL)

- Dashboard → Settings → JSON Model → Copy all
- Сохранил как `mydashboard.json`

<details><summary>mydashboard.json</summary>
```json
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": {
          "type": "grafana",
          "uid": "-- Grafana --"
        },
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      }
    ]
  },
  "editable": true,
  "fiscalYearStartMonth": 0,
  "graphTooltip": 0,
  "id": 1,
  "links": [],
  "panels": [
    {
      "datasource": {
        "type": "prometheus",
        "uid": "bet6q2b8zezuoe"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "barWidthFactor": 0.6,
            "drawStyle": "line",
            "fillOpacity": 7,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "insertNulls": false,
            "lineInterpolation": "linear",
            "lineWidth": 3,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": 0
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 7,
        "w": 11,
        "x": 0,
        "y": 0
      },
      "id": 1,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "hideZeros": false,
          "mode": "single",
          "sort": "none"
        }
      },
      "pluginVersion": "12.1.0-16509090662",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "bet6q2b8zezuoe"
          },
          "editorMode": "code",
          "expr": "100 - (avg by(instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[1m])) * 100)",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "CPU Utilization (%)",
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "bet6q2b8zezuoe"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "barWidthFactor": 0.6,
            "drawStyle": "line",
            "fillOpacity": 0,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "insertNulls": false,
            "lineInterpolation": "linear",
            "lineWidth": 3,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": 0
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 7,
        "w": 10,
        "x": 11,
        "y": 0
      },
      "id": 2,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "hideZeros": false,
          "mode": "single",
          "sort": "none"
        }
      },
      "pluginVersion": "12.1.0-16509090662",
      "targets": [
        {
          "editorMode": "code",
          "expr": "node_load1",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "bet6q2b8zezuoe"
          },
          "editorMode": "code",
          "expr": "node_load5",
          "hide": false,
          "instant": false,
          "legendFormat": "__auto",
          "range": true,
          "refId": "B"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "bet6q2b8zezuoe"
          },
          "editorMode": "code",
          "expr": "node_load15",
          "hide": false,
          "instant": false,
          "legendFormat": "__auto",
          "range": true,
          "refId": "C"
        }
      ],
      "title": "CPU Load Average (1/5/15)",
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "bet6q2b8zezuoe"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "barWidthFactor": 0.6,
            "drawStyle": "line",
            "fillOpacity": 0,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "insertNulls": false,
            "lineInterpolation": "linear",
            "lineWidth": 3,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": 0
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 11,
        "x": 0,
        "y": 7
      },
      "id": 3,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "hideZeros": false,
          "mode": "single",
          "sort": "none"
        }
      },
      "pluginVersion": "12.1.0-16509090662",
      "targets": [
        {
          "editorMode": "code",
          "expr": "node_memory_MemAvailable_bytes / 1024 / 1024",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Free Memory (bytes/MB)",
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "bet6q2b8zezuoe"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisBorderShow": false,
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "barWidthFactor": 0.6,
            "drawStyle": "line",
            "fillOpacity": 0,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "insertNulls": false,
            "lineInterpolation": "linear",
            "lineWidth": 3,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": 0
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 10,
        "x": 11,
        "y": 7
      },
      "id": 4,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "hideZeros": false,
          "mode": "single",
          "sort": "none"
        }
      },
      "pluginVersion": "12.1.0-16509090662",
      "targets": [
        {
          "editorMode": "code",
          "expr": "node_filesystem_free_bytes{mountpoint=\"/etc/hostname\"}",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Free Disk Space",
      "type": "timeseries"
    }
  ],
  "preload": false,
  "schemaVersion": 41,
  "tags": [],
  "templating": {
    "list": []
  },
  "time": {
    "from": "2025-07-27T06:40:27.612Z",
    "to": "2025-07-27T10:12:28.311Z"
  },
  "timepicker": {},
  "timezone": "browser",
  "title": "New dashboard",
  "uid": "65e42d42-3973-43af-bd09-40615c66f3d1",
  "version": 9
}
```
</details>
---

## Ссылки

- [Документация Grafana](https://grafana.com/docs/)
- [Документация Prometheus](https://prometheus.io/docs/)
- [Prometheus Node Exporter](https://github.com/prometheus/node_exporter)

---