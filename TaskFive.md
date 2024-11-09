создал кластер и реплику на одной машине
```
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-16 unzip atop iotop

sudo pg_createcluster 16 main2

pg_lsclusters

Ver Cluster Port Status Owner    Data directory               Log file
16  main    5432 online postgres /var/lib/postgresql/16/main  /var/log/postgresql/postgresql-16-main.log
16  main2   5433 down   postgres /var/lib/postgresql/16/main2 /var/log/postgresql/postgresql-16-main2.log
root@ilinsinc:~# pg_lsclusters
```

```
postgres@postgres:~$ cat >> /etc/postgresql/16/main/postgresql.conf << EOL
listen_addresses = '192.168.1.2'
EOL
postgres@postgres:~$ cat >> /etc/postgresql/16/main/pg_hba.conf << EOL
host replication replicator 198.168.0.0/16 scram-sha-256
EOL
```

```
postgres@postges:/root$ pg_lsclusters
Ver Cluster Port Status        Owner    Data directory              Log file
16  main    5432 down,recovery postgres /var/lib/postgresql/16/main /var/log/postgresql/postgresql-16-main.log
```
```
 cat > ~/workload2.sql << EOL
INSERT INTO book.tickets (fkRide, fio, contact, fkSeat)
VALUES (
        ceil(random()*100)
        , (array(SELECT fam FROM book.fam))[ceil(random()*110)]::text || ' ' ||
    (array(SELECT nam FROM book.nam))[ceil(random()*110)]::text
    ,('{"phone":"+7' || (1000000000::bigint + floor(random()*9000000000)::bigint)::text || '"}')::jsonb
    , ceil(random()*100));

EOL
postgres@postges:~$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload2.sql -n -U postgres -p 5432 thai
```
```
number of transactions actually processed: 71587

после старта реплики

number of transactions actually processed: 66345 

cat > ~/workload2.sql << EOL
INSERT INTO book.tickets (fkRide, fio, contact, fkSeat)
VALUES (
        ceil(random()*100)
        , (array(SELECT fam FROM book.fam))[ceil(random()*110)]::text || ' ' ||
    (array(SELECT nam FROM book.nam))[ceil(random()*110)]::text
    ,('{"phone":"+7' || (1000000000::bigint + floor(random()*9000000000)::bigint)::text || '"}')::jsonb
    , ceil(random()*100));

EOL
postgres@postges:~$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload2.sql -n -U postgres -p 5432 thai

number of transactions actually processed: 285524

после старта реплики

number of transactions actually processed: 285703
```

Проверяем на реплике
```
cat > ~/workload.sql << EOL

\set r random(1, 5000000)
SELECT id, fkRide, fio, contact, fkSeat FROM book.tickets WHERE id = :r;

EOL

cd

/usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres thai

number of transactions actually processed: 254309

```
