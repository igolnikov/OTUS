# Домашнее задание:
## Бэкапы.
### Цель:
- настроить надёжное резервное копирование и восстановление базы данных;

### 1. Настройте бэкапы PostgreSQL с использованием WAL-G, pg_probackup или любого другого аналогичного ПО для базы данных "Лояльность оптовиков".

**1.1** *Установи нужные утилиты:*

```
apt update
apt install -y gpg wget
```
**1.2** *Добавь GPG-ключ репозитория:*

```
wget -qO - https://repo.postgrespro.ru/pg_probackup/keys/GPG-KEY-PG-PROBACKUP \
  > /etc/apt/trusted.gpg.d/pg_probackup.asc
```
**1.3** *Добавь репозиторий для Debian 13:*

```
. /etc/os-release
echo "deb [arch=amd64] https://repo.postgrespro.ru/pg_probackup/deb $VERSION_CODENAME main-$VERSION_CODENAME" > /etc/apt/sources.list.d/pg_probackup.list
```
**1.4** *Обнови индексы пакетов:*

`apt update`

**1.5** *Установка пакета:*

`apt install -y pg-probackup-18`

**1.6** *Для удобства добаляем в local bin:*

`ln -s /usr/bin/pg_probackup-18 /usr/local/bin/pg_probackup`

**1.7** *Проверяем версию:*

```
pg_probackup --version
pg_probackup 2.5.16 (PostgreSQL 18.1)
```

**1.8** *Инициализируем новый инстанс резервного копирования:*

`sudo -u postgres /usr/bin/pg_probackup-18 add-instance -B /mnt/data/ -D /var/lib/postgresql/18/main --instance=patroni_cluster`

**1.9** *Запускаем полное резервное копирование PostgreSQL:*

`sudo -u postgres /usr/bin/pg_probackup-18 backup -B /mnt/data/ --instance=patroni_cluster -b FULL --stream`

**1.10** *Проверяем состояние завершённой задачи:*

![Альтернативный текст](img/screenshot.png)

### 2. Восстановите данные на другом кластере, чтобы убедиться, что бэкапы работают.

### 3. Проверьте, что данные восстановлены корректно.

### 4. Дополнительно: Снимите бэкап под нагрузкой с реплики.
