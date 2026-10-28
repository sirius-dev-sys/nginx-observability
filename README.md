# Наблюдаемость Nginx: Prometheus, Vector, Loki, Grafana

Стенд поднят локально через docker compose. Отличия от выданной ВМ:
- адреса в Docker-сети вместо localhost (например, `http://prometheus:9090`);
- остановка nginx: `docker compose stop nginx` вместо `sudo systemctl stop nginx`.

## Запуск
    cp .env.example .env   # вписать SMTP-данные
    docker compose up -d

## Скриншоты
(добавить описание после выполнения)

## Что реализовано
- **Метрики:** Prometheus собирает node_exporter и nginx_exporter (job `node_exporter` и `nginx`), конфиг в `prometheus/prometheus.yml`.
- **Логи:** Vector читает `access.log` и `error.log` nginx, парсит их, добавляет поле `log_type` (`access` или `error`) и отправляет в Loki с меткой `job="vector"`, конфиг в `vector/vector.toml`.
- **Визуализация:** дашборд "Nginx Overview" в `grafana/dashboards/nginx-overview.json`, источники данных в `grafana/provisioning/datasources`.
- **Алертинг:** Grafana Alerting, contact point (Email) и правила NginxDown (pending 1m) и InstanceDown (pending 2m) описаны кодом в `grafana/provisioning/alerting`.

## Скриншоты
1. `screenshots/01-dashboard-nginx-overview.png` - дашборд "Nginx Overview": Nginx Status (nginx_up), CPU Load и RAM Usage (node_exporter), RPS (rate по nginx_http_requests_total), Error Logs (Loki, log_type="error") и Top 10 IPs (Loki, по полю client).
2. `screenshots/02-alert-rules.png` - список правил в Grafana Alerting: NginxDown и InstanceDown.
3. `screenshots/03-alert-email.png` - письмо от Grafana об алерте при остановке nginx.

## Проверка
- `docker compose stop nginx` - через 1-2 минуты правило NginxDown переходит в Firing, на почту приходит уведомление.
- `docker compose start nginx` и нагрузка через curl (в том числе к несуществующим страницам) - рост RPS, записи 404 в Error Logs, адрес клиента в Top 10 IPs.

## Примечание
Работа выполнена на локальном стенде в Docker (Colima на macOS), так как ВМ Практикума и GitLab были недоступны. Вместо `sudo systemctl stop nginx` использовалась команда `docker compose stop nginx`. Сдаётся через Pull Request на GitHub вместо Merge Request.
