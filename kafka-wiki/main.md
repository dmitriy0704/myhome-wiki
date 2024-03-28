# Изучение kafka
## Использование KRaft в режиме server.
Работа с kafka:
Три режима работы:

- broker;
- controller;
- seever;

### Конфигурация kafka-server:  

> _**/config/kraft/server.properties**_  
> Для конфигурации нескольких серверов номер порта увеличивается на 1   
> 
> _**node.id=1**_ - у каждого нового сервера должен быть уникальный номер
> _**listeners=PLAINTEXT://:9092,CONTROLLER://:9093**_ - порты для брокера и контроллера
> _**controller.quorum.voters=1@localhost:9093**_ - определяется кто будет следующим лидером после падения первого
> _**advertised.listeners=PLAINTEXT://localhost:9092**_ - порт на котором брокер будет прослушивть клиентов
> _**log.dirs=/tmp/server-1/kraft-combined-logs**_ - директория логов


### Конфигурация spring-приложения для продюсера:  


```properties
# Включение идемпотентности:
spring.kafka.producer.enable.idempotence=true

# От кого получать ответы:
spring.kafka.producer.acks=all  

# Количество попыток отправки сообщения в случае неудачного ответа:
spring.kafka.producer.retries=10

# Интервал отправки(100):
spring.kafka.producer.properties.retry.backoff.ms=1000

# Максиальное время на повторы сообщения(12000):
spring.kafka.producer.properties.delivery.timeout.ms=60000

# Время в течение котрого накапливаются сообщения перед отправокй:
spring.kafka.producer.properties.linger.ms=0

# Время ожидания ответа от брокера:
spring.kafka.producer.properties.request.timeout.ms=30000

# Важно:
delivery.timeout.ms >= linger.ms + request.timeout.ms

# Включение свойства идемпотентности:
spring.kafka.producer.properties.enable.idempotence=true
```


### Команды работы с Kafka:

#### Запуск Kafka Server

```shell
# Сгенерировать id для kafka кластера:
./bin/kafka-storage.sh random-uui

# Форматирование логов для совместимости с kraft режимом:
./bin/kafka-storage.sh format -t <uuid> -c ./config/kraft/server.properties

# Запуск kafka
./bin/kafka-server-start.sh ./config/kraft/server.properties
```

#### Топики
```shell
# Создание нового топика:
./bin/kafka-topics.sh --create --topic payment-created-events-topic --partitions 3 --replication-factor 3 --bootstrap-server localhost:9092,localhost:9094

# Список топиков:
./bin/kafka-topics.sh --list --bootstrap-server localhost:9092,localhost:9094

# Удаление топика
./bin/kafka-topics.sh --delete --topic <topic-name> --bootstrap-server localhost:9092,localhost:9094
```

#### Сообщения:
```shell
# Отправка сообщения:
./bin/kafka-console-producer.sh --bootstrap-server localhost:9092,localhost:9094 --topic <topic-name>

# Прослушивание сообщений консюмером:
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9094 --topic product-created-events-topic --property "print.key=true"

# Прочитать сообщения из топика с начала:
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9094 --topic test-topic --from-beginning
```

#### Из docker/compose:
```shell
# Создать топик
docker compose exec [SERVICE_NAME] kafka-topics.sh --create --topic baeldung_linux --partitions 1 --replication-factor 1 --bootstrap-server kafka:9092

# Список топиков
docker compose exec [SERVICE_NAME] kafka-topics.sh --list --bootstrap-server kafka:9092
docker exec -it [CONTAINER_NAME] kafka-topics.sh --list --bootstrap-server kafka:9092

# Отправить сообщение
docker exec -it [CONTAINER_NAME] kafka-console-producer.sh --topic [TOPIC_NAME] --bootstrap-server kafka:9092

# Получить сообщения из топика
docker exec -it [CONTAINER_NAME] kafka-console-consumer.sh --topic [TOPIC_NAME] --bootstrap-server kafka:9092
```