# Условие задачи
- На второй виртуалке (физическая репликация)
    1. Сделать Дамп базы из 1 виртуалки (`pg_basebackup`)
    2. Настроить репликацию на этом сервере

Как и в прошлом задании, в первую очередь создаем плейбук:
```bash
ansible-playbook -i inventory.ini main.yaml \
  --user debian \
  --private-key ~/.ssh/id_ed25519
```

После этого осуществляем копию кластера с 1 ВМ.
Для начала останавливаем postgresql, чтобы не мешал, после
этого удаляем все данные от postgres на этой ВМ и создаём
новые, чистые, выдавая нужные права.
Далее копируем кластер с нужными параметрами:
```bash
sudo systemctl stop postgresql

sudo rm -rf /pg_data/16
sudo mkdir -p /pg_data/16
sudo chmod 700 /pg_data/16
sudo chown postgres:postgres /pg_data/16

sudo -u postgres PGPASSWORD='megaparol' \
  /usr/lib/postgresql/16/bin/pg_basebackup \
  -h 84.252.137.133 \
  -p 5432 \
  -U repl_user \
  -D /pg_data/16 \
  -Fp -Xs -P -R
```

Скопировали:
![Screenshot_5.png](screens/Screenshot_5.png)

После этого без проблем запускаем реплику:
```bash
sudo -u postgres /usr/lib/postgresql/16/bin/pg_ctl -D /pg_data/16 -l /var/log/postgresql/postgresql-16-replica.log start
```

Для проверки на 1 ВМ можно использовать:
```sql
SELECT client_addr, state, sync_state
FROM pg_stat_replication;
```

На 2 ВМ:
```sql
SELECT pg_is_in_recovery();
```
