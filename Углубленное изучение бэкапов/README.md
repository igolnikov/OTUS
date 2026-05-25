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

**2.1** *Немного усложнил задачу, - на отдельном сервере поставил pg_probackup-18 
         чтобы не переносить резервную копию с Patroni-01:*

**2.2** *Cвязываю два сервера ssh ключами:*

```
ssh-keygen -t rsa -b 4096 -N "" -f /root/.ssh/id_rsa
ssh-copy-id postgres@172.34.35.156
```

**2.3** *Создаю пароль postgres на сервере Patroni-01 откуда буду забирать резервную копию и меняю pg_hba.conf:*

`ALTER USER postgres PASSWORD 'backup123';`

`nano /var/lib/postgresql/18/main/pg_hba.conf`

```
# Backup server - normal connection
host    all             postgres        172.34.35.151/32        scram-sha-256

# Backup server - replication connection (ОЧЕНЬ ВАЖНО!)
host    replication     postgres        172.34.35.151/32        scram-sha-256
# для pg_probackup
host    replication     postgres        172.34.35.151/32        scram-sha-256

```

**2.4** *Создаю фаил с паролем чтобы скрипт не спрашивал его при запуске*

```
sudo -u postgres touch /var/lib/postgresql/.pgpass
sudo -u postgres chmod 0600 /var/lib/postgresql/.pgpass
sudo -u postgres nano /var/lib/postgresql/.pgpass
172.34.35.156:5432:*:postgres:backup123
```

**2.5** *Объявляю instance**

`pg_probackup-18 backup -B /mnt/data/backups --instance=patroni_cluster -b FULL --stream --remote-host=172.34.35.156 --remote-user=postgres -U postgres`

**2.6** *Запускую первый full backup*

`sudo -u postgres pg_probackup-18 backup   -B /mnt/data/backups   --instance=patroni_cluster   -b FULL   --stream   --remote-host=172.34.35.156   --remote-user=postgres   -h 172.34.35.156   -U postgres   -d postgres`

**2.7** *Проверяем, что наш backup перешёл на наш сервер.*

```
root@postgresql-home-01:~# ip a

2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether bc:24:11:ed:06:f2 brd ff:ff:ff:ff:ff:ff
    altname enp0s18
    altname enxbc2411ed06f2
    inet 172.34.35.151/23 brd 172.34.35.255 scope global dynamic noprefixroute ens18
       valid_lft 1160sec preferred_lft 935sec
    inet6 fe80::6b24:5cd8:98e2:1b76/64 scope link
       valid_lft forever preferred_lft forever

root@postgresql-home-01:~# du -h /mnt/data/
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLLBF/database
12K     /mnt/data/backups/backups/patroni_cluster/TFLLBF
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLL6G/database
12K     /mnt/data/backups/backups/patroni_cluster/TFLL6G
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLLMO/database/pg_wal
8.0K    /mnt/data/backups/backups/patroni_cluster/TFLLMO/database
16K     /mnt/data/backups/backups/patroni_cluster/TFLLMO
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_stat
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_stat_tmp
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_logical/snapshots
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_logical/mappings
16K     /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_logical
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_notify
8.1M    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/base/4
8.1M    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/base/5
8.2M    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/base/24576
8.1M    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/base/1
33M     /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/base
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_snapshots
33M     /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_wal
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_tblspc
688K    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/global
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_replslot
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_subtrans
12K     /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_xact
12K     /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_multixact/offsets
12K     /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_multixact/members
28K     /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_multixact
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_serial
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_commit_ts
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_twophase
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLLQT/database/pg_dynshmem
66M     /mnt/data/backups/backups/patroni_cluster/TFLLQT/database
66M     /mnt/data/backups/backups/patroni_cluster/TFLLQT
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLL3J/database
12K     /mnt/data/backups/backups/patroni_cluster/TFLL3J
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLL4Q/database
12K     /mnt/data/backups/backups/patroni_cluster/TFLL4Q
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLKVV/database
12K     /mnt/data/backups/backups/patroni_cluster/TFLKVV
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLKOS/database
12K     /mnt/data/backups/backups/patroni_cluster/TFLKOS
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLL8H/database
12K     /mnt/data/backups/backups/patroni_cluster/TFLL8H
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLKQ3/database
12K     /mnt/data/backups/backups/patroni_cluster/TFLKQ3
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLL79/database
12K     /mnt/data/backups/backups/patroni_cluster/TFLL79
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLKOC/database
12K     /mnt/data/backups/backups/patroni_cluster/TFLKOC
4.0K    /mnt/data/backups/backups/patroni_cluster/TFLKZE/database
12K     /mnt/data/backups/backups/patroni_cluster/TFLKZE
66M     /mnt/data/backups/backups/patroni_cluster
66M     /mnt/data/backups/backups
4.0K    /mnt/data/backups/wal/patroni_cluster
8.0K    /mnt/data/backups/wal
66M     /mnt/data/backups
66M     /mnt/data/

```

**2.8** *Восстанавливаем все что мы забрали на новом месте.*

`sudo systemctl stop postgresql`
`sudo rm -rf /var/lib/postgresql/18/main/*`
```
sudo -u postgres pg_probackup-18 restore \
  -B /mnt/data/backups \
  --instance=patroni_cluster \
  -D /var/lib/postgresql/18/main \
  --backup-id=TFLLQT \
  --restore-command="cp /mnt/data/backups/wal/%f %p"

INFO: Validating backup TFLLQT
INFO: Backup TFLLQT data files are valid
INFO: Backup TFLLQT WAL segments are valid
INFO: Backup TFLLQT is valid.
INFO: Restoring the database from backup TFLLQT
INFO: Start restoring backup files. PGDATA size: 62MB
INFO: Backup files are restored. Transfered bytes: 62MB, time elapsed: 0
INFO: Restore incremental ratio (less is better): 100% (62MB/62MB)
INFO: Syncing restored files to disk
INFO: Restored backup files are synced, time elapsed: 9s
INFO: Restore of backup TFLLQT completed.
```
`sudo chown -R postgres:postgres /var/lib/postgresql/18/main`
`sudo systemctl start postgresql`

### 3. Проверьте, что данные восстановлены корректно.
![Альтернативный текст](img/screenshot.png)
### 4. Дополнительно: Снимите бэкап под нагрузкой с реплики.
