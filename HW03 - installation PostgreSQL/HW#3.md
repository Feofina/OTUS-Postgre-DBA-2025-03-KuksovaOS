
# Отчет о выполнении домашнего задания
## Тема: Установка PostgreSQL в контейнере Docker

Цель:

 1. установить PostgreSQL в Docker контейнере настроить контейнер для
 2. внешнего подключения

  Описание/Пошаговая инструкция выполнения домашнего задания:

- создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным   способом
- поставить на нем Docker Engine
- сделать каталог /var/lib/postgresразвернуть контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgresql
- развернуть контейнер с клиентом postgres
- подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк
- подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов ЯО/места установки докера
- удалить контейнер с сервером
- создать его заново
- подключится снова из контейнера с клиентом к контейнеру с сервером
- проверить, что данные остались на месте

## Этапы выполнения

 1.  На личном ПК был установлен Docker согласно инструкции:  [Установка Docker Desktop на Windows](https://docs.docker.com/desktop/setup/install/windows-install/)
 2.  Для выполнения ДЗ была использована статья:
[Запускаем PostgreSQL в Docker: от простого к сложному](https://habr.com/ru/articles/578744/)
 3. Через PowerShell выполнены следующие команды, для подготовки контейнера к монтажу:
```CMD 
 docker search postgres 	# поиск пакета
 docker pull postgres 	 	# загрузить последнюю версию
 docker images				
 docker ps					# список всех запущенных контейнеров
 docker ps -a   			# список всех контейнеров, которые мы запускали
```
 4. Монтируем контейнер:
```CMD 
docker run [--name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 
5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:17]
```
Ошибка:
   > network pg-net not found

 5. Пробуем найти и исправить ошибку:
```CMD 
docker run --name pg-server-otus -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data -e POSTGRES_PASSWORD=postgres -d  postgres:17
```
Ошибка: 
> docker: Error response from daemon: failed to set up container networking: driver failed programming external connectivity on endpoint pg-server-otus (1214c0556e5d0f7201915e6b939fffa11fdc142e136fb7b673fa0b906a0cda33): Bind for 0.0.0.0:5432 failed: port is already allocated

 6.  По порту 5432 подсоединиться не смогла. Кто-то уже его прослушивает:
```CMD 
PS C:\Users\kukso> netstat -ano | findstr :5432
  TCP    0.0.0.0:5432           0.0.0.0:0              LISTENING       16976
  TCP    [::]:5432              [::]:0                 LISTENING       16976
  TCP    [::1]:5432             [::]:0                 LISTENING       18556
  ```
 7. Повторно пробуем найти и исправить ошибку:
```CMD
docker network create pg-net
mkdir -p /var/lib/postgres		# через консоль wsl создала каталог postgres
docker run --name pg-server-otus -p 5433:5432 --network pg-net -e POSTGRES_PASSWORD=postgres   -d -v /var/lib/postgres:/var/lib/postgresql/data postgres:17 
```
Успех!

8. Теперь подключаемся к БД postgres с пользователем postgres: 
```CMD
docker exec -it pg-server-otus psql --username=postgres --dbname=postgres
```
9.  Отработала скрипт создания таблицы и скрипт заполнения таблицы данными:
 ```SQL
create table public.testtable (id serial not null,
                               value varchar not null);

insert into public.testtable (value)
values('test1');

insert into public.testtable (value)
values('test2');
```	
10.  Проверяем вставку:
 ```SQL
SELECT * FROM public.testtable;
```
 id | value
----+-------
  1 | test1
  2 | test2
(2 rows)

11. Подключилась к БД postgres через pgAdmin по localhost:5433 и проверила текущее состояние БД. Таблица создана успешно.

12. Удалила контейнер с сервером pg-server-otus командами:
```CMD
docker stop pg-server-otus 
docker rm pg-server-otus  
```
13. Создала контейнер заново командой:
```CMD
docker run --name pg-server-otus -p 5433:5432 --network pg-net -e POSTGRES_PASSWORD=postgres   -d -v /var/lib/postgres:/var/lib/postgresql/data postgres:17 
docker exec -it pg-server-otus psql --username=postgres --dbname=postgres
 ```

14. Проверила наличие данных. Данные не пропали:
```CMD
postgres=# SELECT * FROM public.testtable;
 id | value
----+-------
  1 | test1
  2 | test2
(2 rows)
```
```CMD
postgres=# insert
into
    public.testtable
    (value)
values('test3');
INSERT 0 1
postgres=# SELECT * FROM public.testtable;
 id | value
----+-------
  1 | test1
  2 | test2
  3 | test3
(3 rows)
```
