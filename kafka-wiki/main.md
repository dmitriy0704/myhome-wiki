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

controller.quorum.voters=1@localhost:9093 - определяется кто будет следующим лидером после падения первого

advertised.listeners=PLAINTEXT://localhost:9092 - порт на котором брокер будет прослушивть клиентов
log.dirs=/tmp/server-1/kraft-combined-logs

4. Создание нового топика:
```shell
sh ./bin/kafka-topics.sh --create --topic payment-created-events-topic --partitions 3 --replication-factor 3 --bootstrap-server localhost:9092,localhost:9094
```

5. Список топиков:
```shell
sh ./bin/kafka-topics.sh --list --bootstrap-server localhost:9092,localhost:9094
```

6. Delete
```shell
sh ./bin/kafka-topics.sh --delete --topic <topic-name> --bootstrap-server localhost:9092,localhost:9094
```

7. Отправка сообщения:
```shell
sh ./bin/kafka-console-producer.sh --bootstrap-server localhost:9092,localhost:9094 --topic <topic-name>
```

8. Прослушивание сообщений консюмером:
```shell
sh ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9094 --topic product-created-events-topic --property "print.key=true"
```

### Свойства spring-приложения для продюсера:  
_**От кого получать ответы:**_
>spring.kafka.producer.acks=all

_**Количество попыток отправки сообщения в случае неудачного ответа:**_
>spring.kafka.producer.retries=10

_**Интервал отправки(100):**_
>spring.kafka.producer.properties.retry.backoff.ms=1000

_**Максиальное время на повторы сообщения(12000):**_
>spring.kafka.producer.properties.delivery.timeout.ms=60000

_**Время в течение котрого накапливаются сообщения перед отправокй:**_
>spring.kafka.producer.properties.linger.ms=0

_**Время ожидания ответа от брокера:**_
>spring.kafka.producer.properties.request.timeout.ms=30000

_**Важно:**_
>delivery.timeout.ms >= linger.ms + request.timeout.ms

_**Включение свойства идемпотентности:**_
>spring.kafka.producer.properties.enable.idempotence=true