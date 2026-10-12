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

![Дашборд](skreen/dashboard.png)

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

![Сработавший алерт](skreen/alert.png)

## Что сломалось и как решил

**1. Git Bash открывался и сразу закрывался**
- **Симптом:** при запуске окно Git Bash мгновенно закрывалось, пункта «Git Bash Here» в контекстном меню не было, команды выполнить не удавалось.
- **Причина:** Git Bash не был нужен для проекта: `docker compose` работает и в PowerShell, а контекстное меню не добавилось при установке.
- **Решение:** открыл PowerShell сразу в папке проекта (адресная строка Проводника → `powershell` → Enter) и запустил `docker compose up -d`. Работоспособность проверил через `docker version` и `docker compose ps`.

**2. Импорт дашборда по ID не сработал**
- **Симптом:** дашборд Node Exporter Full (ID 1860) не загружался в Grafana через Import.
- **Причина:** импорт по ID требует доступа Grafana к grafana.com.
- **Решение:** собрал собственный дашборд в JSON (`node-dashboard.json`: статус узла, uptime, CPU, RAM, диск, сеть) и импортировал его загрузкой файла. Заодно он хранится в репозитории и воспроизводится одной операцией.

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
