# NetWatch Stack

Self-hosted стек мониторинга сетевой инфраструктуры и доступности сервисов на базе **Docker Compose**.

Проект объединяет **Nginx**, **Prometheus**, **Blackbox Exporter** и **Grafana** в изолированной Docker-сети и позволяет одной командой развернуть мониторинг сетевых узлов по протоколам **ICMP**, **HTTP/HTTPS** и **DNS**.

## Архитектура

```text
                       ┌─────────────────────────────┐
  Browser (:8080) ───▶ │   Nginx (Reverse Proxy)     │
                       │   /           → Grafana     │
                       │   /prometheus → Prometheus  │
                       │   /health     → 200 OK      │
                       └───────┬─────────────┬───────┘
                               │             │
                               ▼             ▼
                         ┌──────────┐  ┌────────────┐
                         │ Grafana  │─▶│ Prometheus │ (retention 15d, alerts)
                         └──────────┘  └─────┬──────┘
                                             │ scrape /probe
                                             ▼
                                    ┌───────────────────┐
                                    │ Blackbox Exporter │
                                    └─┬───────┬───────┬─┘
                                      │       │       │
                                    ICMP   HTTP/S    DNS
                                      ▼       ▼       ▼
                                     Целевые сетевые узлы
```

## Возможности

- **Мониторинг HTTP/HTTPS** — проверка доступности, HTTP-кодов ответа, задержки и сроков действия SSL-сертификатов.
- **Мониторинг ICMP (ping)** — контроль доступности и сетевых задержек (RTT) ключевых IP-узлов.
- **Мониторинг DNS (UDP/53)** — проверка доступности и времени ответа DNS-резолверов.
- **Автоматический провижининг Grafana** — источник данных Prometheus и дашборд `NetWatch — Network & Service Availability` подключаются автоматически при первом запуске.
- **Базовые правила алертинга** (`prometheus/alerts.yml`) — падение узла, высокая задержка, ошибка DNS, истекающий SSL-сертификат.
- **Единая точка входа через Nginx** — все сервисы изолированы во внутренней Docker-сети `netwatch_net`, наружу открыт только reverse proxy.

## Быстрый старт

### Требования
- Docker Engine 24+
- Docker Compose v2 (`docker compose`)

### Запуск

```bash
git clone https://github.com/chiter52/netwatch-stack.git
cd netwatch-stack
cp .env.example .env
docker compose up -d
```

После запуска сервисы доступны по адресам:
- **Grafana:** `http://localhost:8080` (логин/пароль по умолчанию: `admin` / `netwatch`)
- **Prometheus:** `http://localhost:8080/prometheus/`
- **Healthcheck Nginx:** `http://localhost:8080/health`

### Остановка

```bash
docker compose down
```

Для полного удаления сохранённых данных Prometheus и Grafana:

```bash
docker compose down -v
```

## Структура репозитория

```text
netwatch-stack/
├── docker-compose.yml                         # Описание и оркестрация всех сервисов
├── .env.example                               # Шаблон переменных окружения
├── nginx/
│   └── nginx.conf                             # Конфигурация reverse proxy и маршрутизации
├── blackbox/
│   └── blackbox.yml                           # Модули пробинга: http_2xx, icmp, dns_udp, tcp_connect
├── prometheus/
│   ├── prometheus.yml                         # Конфигурация сбора метрик и relabeling целей
│   └── alerts.yml                             # Правила алертинга по доступности и задержкам
└── grafana/
    ├── provisioning/
    │   ├── datasources/prometheus.yml         # Автоподключение Prometheus
    │   └── dashboards/dashboards.yml          # Автоподгрузка дашбордов
    └── dashboards/
        └── netwatch-overview.json             # Готовый дашборд сетевого мониторинга
```

## Настройка целей мониторинга

Целевые хосты задаются в файле `prometheus/prometheus.yml` в секциях `blackbox_http`, `blackbox_icmp` и `blackbox_dns`. После изменения списка достаточно перезапустить Prometheus:

```bash
docker compose restart prometheus
```

## Лицензия

MIT
