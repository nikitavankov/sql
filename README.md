# PostgreSQL HA Cluster Deployment

Ansible-коллекция для развёртывания отказоустойчивого кластера PostgreSQL 16
на Rocky Linux 9 с использованием Patroni (управление HA) и etcd (DCS).

## Архитектура

Три узла, каждый выполняет все роли:

| Узел          | IP            | Компоненты                    |
|---------------|---------------|-------------------------------|
| `sql_server1` | 192.168.X.XX  | etcd, PostgreSQL, Patroni     |
| `sql_server2` | 192.168.X.XX  | etcd, PostgreSQL, Patroni     |
| `sql_server3` | 192.168.X.XX  | etcd, PostgreSQL, Patroni     |

Кворум etcd — 2 из 3. Кластер переживает потерю одного узла.

## Роли

| Роль         | Назначение                                              |
|--------------|---------------------------------------------------------|
| `repository` | Подключение EPEL, CRB и PGDG-репозиториев               |
| `chrony`     | Синхронизация времени (критично для etcd/Patroni)       |
| `common`     | Базовая подготовка ОС: swap, sysctl, лимиты, firewall   |
| `etcd`       | Распределённое хранилище конфигурации (DCS)             |
| `postgresql` | Установка PostgreSQL 16 из PGDG                         |
| `patroni`    | Управление HA, выборы лидера, конфигурация Patroni      |
| `haproxy`    | Балансировка трафика на текущего лидера (опционально)   |

## Требования

- Rocky Linux 9 на всех узлах
- Ansible 2.14+
- Python 3.14 (для ansible-lint в pre-commit)
- Доступ по SSH с правами root

## Быстрый старт

```bash
ansible-playbook deploy_cluster.yml
```