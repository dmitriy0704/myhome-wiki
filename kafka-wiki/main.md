# Изучение kafka

Работа с kafka:
Три режима работы:

- broker;
- controller;
- seever;

### Использование KRaft в режиме server.

1. Сгенерировать id для kafka кластера:

```shell
cd bin \n
sh kafka-storage.sh random-uui
```

2. Форматирование логов для совместимости с kraft режимом:

```shell
sh ./bin/kafka-storage.sh format-t <uuid> -c ./config/kraft/server.properties
```

3. Запуск kafka

```shell
sh ./bin/kafka-server-start.sh ./config/kraft/server.properties
```

_**/config/kraft/server.properties**_

> Для конфигурации нескольких серверов номер порта увеличивается на 1

node.id=1 - у каждого нового сервера должен быть уникальный номер
listeners=PLAINTEXT://:9092,CONTROLLER://:9093 - порты для брокера и контроллера
advertised.listeners=PLAINTEXT://localhost:9092 - прослушиванеи клиентов
log.dirs=/tmp/server-1/kraft-combined-logs
