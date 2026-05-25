# Домашнее задание:
## Бэкапы.
### Цель:
- настроить надёжное резервное копирование и восстановление базы данных;

### 1. Настройте бэкапы PostgreSQL с использованием WAL-G, pg_probackup или любого другого аналогичного ПО для базы данных "Лояльность оптовиков".

Установи нужные утилиты:
```
apt update
apt install -y gpg wget
```
Добавь GPG-ключ репозитория:
```
wget -qO - https://repo.postgrespro.ru/pg_probackup/keys/GPG-KEY-PG-PROBACKUP \
  > /etc/apt/trusted.gpg.d/pg_probackup.asc
```
Добавь репозиторий для Debian 13:
```
. /etc/os-release
echo "deb [arch=amd64] https://repo.postgrespro.ru/pg_probackup/deb $VERSION_CODENAME main-$VERSION_CODENAME" > /etc/apt/sources.list.d/pg_probackup.list
```
Обнови индексы пакетов:
`apt update`

Установка пакета:
`apt install -y pg-probackup-18`

Для удобства добаляем в local bin:
ln -s /usr/bin/pg_probackup-18 /usr/local/bin/pg_probackup

Проверяем версию:
```
pg_probackup --version
pg_probackup 2.5.16 (PostgreSQL 18.1)
```

### 2. Восстановите данные на другом кластере, чтобы убедиться, что бэкапы работают.

### 3. Проверьте, что данные восстановлены корректно.

### 4. Дополнительно: Снимите бэкап под нагрузкой с реплики.

![Альтернативный текст](img/screenshot.png)
![Альтернативный текст](img/screenshot1.png)