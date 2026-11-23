# repository

Настройка репозиториев на Rocky Linux 9 для офлайн-развёртывания кластера
PostgreSQL + Patroni + etcd.

Роль регистрирует EPEL, включает CRB (CodeReady Builder) и подключает PGDG,
используя **локальные RPM-файлы** из `files/`. Импортирует GPG-ключи,
отключает встроенный модуль PostgreSQL из AppStream.

## Требования

- Rocky Linux 9
- Ansible 2.14+
- Права root (или `become: true`)

## Файлы

Положите в `files/` перед запуском:

| Файл | Откуда взять |
| :--- | :--- |
| `epel-release-latest-9.noarch.rpm` | https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm |
| `pgdg-redhat-repo-latest.noarch.rpm` | https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm |
| `RPM-GPG-KEY-EPEL-9` | https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-9 |
| `PGDG-RPM-GPG-KEY-RHEL` | https://download.postgresql.org/pub/repos/yum/keys/PGDG-RPM-GPG-KEY-RHEL |

## Переменные

| Переменная | По умолчанию | Описание |
| :--- | :--- | :--- |
| `repository_enable_epel` | `true` | Подключать EPEL |
| `repository_enable_crb` | `true` | Включать CRB |
| `repository_enable_pgdg` | `true` | Подключать PGDG |
| `repository_pgdg_version` | `16` | Версия PostgreSQL |
| `repository_epel_rpm` | `epel-release-latest-9.noarch.rpm` | EPEL RPM в `files/` |
| `repository_pgdg_rpm` | `pgdg-redhat-repo-latest.noarch.rpm` | PGDG repo RPM в `files/` |
| `repository_epel_gpg_key` | `RPM-GPG-KEY-EPEL-9` | GPG-ключ EPEL |
| `repository_pgdg_gpg_key` | `PGDG-RPM-GPG-KEY-RHEL` | GPG-ключ PGDG |
| `repository_staging_dir` | `/var/tmp/ansible-repo-rpms` | Каталог для RPM на хосте |
| `repository_disable_repos` | `pgdg*` | Репозитории, игнорируемые при установке RPM |

## Теги

`repository`, `epel`, `crb`, `pgdg`.

## Что делает роль

1. Создаёт staging-каталог.
2. Копирует и импортирует GPG-ключ EPEL, устанавливает EPEL RPM локально
   (с `disablerepo: pgdg*`, чтобы не обращаться к недоступным PGDG-репозиториям).
3. Включает CRB через `dnf config-manager`.
4. Копирует и импортирует GPG-ключ PGDG, устанавливает PGDG repo RPM локально
   (также с `disablerepo: pgdg*`).
5. Отключает встроенный модуль `postgresql` из AppStream.

## Ограничения

- Роль **не делает** систему способной ставить пакеты PostgreSQL офлайн.
  Пакеты доставляются в роли `postgresql` через `files/rpms/`.
- `Refresh dnf cache` намеренно отсутствует: в офлайн-среде PGDG-репозитории
  недоступны, и обновление кэша падает с `Cannot download repomd.xml`.
- `disablerepo: pgdg*` обязателен при повторных прогонах: после первого
  запуска PGDG-репозитории уже зарегистрированы, и без этого параметра
  `dnf` пытается обновить их метаданные и падает.

## Проверка

```bash
dnf repolist enabled | grep -E 'pgdg|epel|crb'
dnf module list postgresql    # disabled
```