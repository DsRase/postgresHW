# Условие задачи
- (логическая репликация)
    1. поднять еще один инстанс БД на второй виртуалке с данными в /pg_data/16-one-base/
    1. Поднять слот логической репликации на мастере только для одной таблицы
    2. Подключиться к слоту логической репликации на второй виртуалке

Опять же, создаем playbook:
```bash
ansible-playbook -i inventory.ini main.yaml
```

Для начала создаем таблицу логической репликации:
```bash
CREATE TABLE IF NOT EXISTS public.orders_logical (
    id BIGSERIAL PRIMARY KEY,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    amount NUMERIC(12,2) NOT NULL,
    status TEXT NOT NULL
);

CREATE PUBLICATION pub_orders 
FOR TABLE public.orders_logical;

SELECT * FROM pg_create_logical_replication_slot('orders_slot', 'pgoutput');
```

Подключаемся:
```bash
CREATE TABLE IF NOT EXISTS public.orders_logical (
    id BIGSERIAL PRIMARY KEY,
    created_at TIMESTAMPTZ NOT NULL,
    amount NUMERIC(12,2) NOT NULL,
    status TEXT NOT NULL
);

CREATE SUBSCRIPTION sub_orders
    CONNECTION 'host=84.252.137.133 port=5432 dbname=bench user=repl_user password=megaparol'
    PUBLICATION pub_orders
    WITH (
        copy_data = true,
        create_slot = false,
        slot_name = 'orders_slot'
    );
```