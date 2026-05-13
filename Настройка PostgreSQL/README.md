# Домашнее задание:
## Спасение данных на внешнем диске.
### Цель:
- научиться подключать и настраивать дополнительный диск для хранения данных;
- освоить перенос базы данных postgresql на новое хранилище;
- обеспечить отказоустойчивость данных при помощи внешнего диска;


### Создайте виртуальную машину с Debian 13 и установите PostgreSQL 15 или выше.

![Альтернативный текст](img/screenshot.png)

```
apt update
apt upgrade -y
apt install sudo

sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh

sudo apt update
sudo apt install postgresql-18

psql --version
psql (PostgreSQL) 18.3 (Debian 18.3-1.pgdg13+1)

```
### Создайте таблицу с данными о перевозках.

```
root@postgresql-home-01:~# su postgres
postgres@postgresql-home-01:/root$ psql

create table shipments(id serial, product_name text, quantity int, destination text);

insert into shipments(product_name, quantity, destination) values('bananas', 1000, 'Europe');
insert into shipments(product_name, quantity, destination) values('bananas', 1500, 'Asia');
insert into shipments(product_name, quantity, destination) values('bananas', 2000, 'Africa');
insert into shipments(product_name, quantity, destination) values('coffee', 500, 'USA');
insert into shipments(product_name, quantity, destination) values('coffee', 700, 'Canada');
insert into shipments(product_name, quantity, destination) values('coffee', 300, 'Japan');
insert into shipments(product_name, quantity, destination) values('sugar', 1000, 'Europe');
insert into shipments(product_name, quantity, destination) values('sugar', 800, 'Asia');
insert into shipments(product_name, quantity, destination) values('sugar', 600, 'Africa');
insert into shipments(product_name, quantity, destination) values('sugar', 400, 'USA');

SELECT * FROM shipments;
 id | product_name | quantity | destination
----+--------------+----------+-------------
  1 | bananas      |     1000 | Europe
  2 | bananas      |     1500 | Asia
  3 | bananas      |     2000 | Africa
  4 | coffee       |      500 | USA
  5 | coffee       |      700 | Canada
  6 | coffee       |      300 | Japan
  7 | sugar        |     1000 | Europe
  8 | sugar        |      800 | Asia
  9 | sugar        |      600 | Africa
 10 | sugar        |      400 | USA
(10 rows)

```
### Добавьте внешний диск к виртуальной машине и перенесите туда базу данных.
### Настройте PostgreSQL для работы с новым диском.
### Проверьте, что данные сохранились и доступны.
