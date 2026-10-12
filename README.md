# Monitoring Stack: Prometheus + Grafana + node_exporter

Учебный проект: стек мониторинга узла, запускаемый одной командой через Docker Compose. Показывает метрики (CPU, RAM, диск, сеть) и отправляет алерт, если узел перестаёт отвечать.

## Архитектура

```
node_exporter (:9100)  <--scrape--  Prometheus (:9090)  <--query--  Grafana (:3000)
   метрики узла                      хранение + алерты               дашборды
```

- **node_exporter** отдаёт метрики хоста
- **Prometheus** собирает их каждые 15 с и вычисляет правила алертов
- **Grafana** визуализирует данные (источник данных подключается автоматически)

## Запуск

Требуется Docker Desktop.

```bash
docker compose up -d
```

| Сервис | Адрес | Доступ |
|---|---|---|
| Prometheus | http://localhost:9090 | без входа |
| Grafana | http://localhost:3000 | admin / admin (только для локального теста) |

## Дашборд

В Grafana: Dashboards → New → Import → Upload JSON → `node-dashboard.json`.

![Дашборд](screenshots/dashboard.png)

## Алерты (`prometheus/alerts.yml`)

| Алерт | Условие | Серьёзность |
|---|---|---|
| InstanceDown | узел недоступен более 1 мин | critical |
| HighCPU | CPU выше 85% более 2 мин | warning |

### Проверка алерта

```bash
docker stop node_exporter
```

Через ~1 минуту в Prometheus → Alerts алерт `InstanceDown` переходит в **Firing**, на дашборде статус узла становится **DOWN**. Возврат: `docker start node_exporter`.

![Сработавший алерт](screenshots/alert.png)

## Что сломалось и как решил

<!-- ЗАПОЛНИТЕ СВОЕЙ РЕАЛЬНОЙ ИСТОРИЕЙ: симптом → причина → как нашли → решение -->
- **Проблема:** ...
- **Причина:** ...
- **Решение:** ...

## Структура

```
.
├── docker-compose.yml
├── node-dashboard.json
├── prometheus/
│   ├── prometheus.yml
│   └── alerts.yml
├── grafana/provisioning/datasources/prometheus.yml
└── screenshots/
```

## Что дальше

- [ ] Alertmanager для уведомлений в Telegram/email
- [ ] Запуск стека в Kubernetes (kind + Helm)
- [ ] CI: проверка YAML в GitHub Actions
