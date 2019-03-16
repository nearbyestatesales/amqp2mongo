# amqp2mongo

Connects to a rabbitmq server, declare queue and bindings to the given exchange and consume messages, saving them to a mongodb collection.

## Usage

    node amqp2mongo.js
    # or
    ./amqp2mongo.js

## Configuration

Configuration is taken from environment variables and `.env` file

* `RABBITMQ_URL`=amqp://user:password@host/%2F?heartbeat=30
    > RabbitMQ DSN

* `RABBITMQ_CONS_QUEUE`=queue
    > Queue to consume

* `RABBITMQ_CONS_EXCH`=exchange
    > Exchange to bind the queue to

* `RABBITMQ_CONS_ROUTING_KEYS`=routing keys coma separated
    > Bindings to declare, can be several, coma separated, eg: a.b.d,xx.# 

* `MONGODB`=mongodb://user:pwd@host1,host2,host3/database?replicaSet=replicaset
    > MongoDB DSN

* `MONGOCOLLECTION`=collection
    > collection to store the messages to

## Document

### format

Keys:

* `_id`: auto-generated mongodb id
* `content` Message payload interpreted if JSON, text if not
* `error`: (optionnal) exception info if message can't be interpreted as JSON (contains self explaintatory `message` and `stack` keys)
* `insertDate`: date when the message is processed by `amqp2mongo`
* `messageDate`:date extraacted from the message headers `timestamp_in_ms` field ([rabbitmq-message-timestamp] plugin required)
* `queue`: queue the message was consumed from
* `properties`: message properties
* `fields`: message fields

[rabbitmq-message-timestamp]: <https://github.com/rabbitmq/rabbitmq-message-timestamp>

### examples 

#### plain json interpreted

```json
{
    "_id" : ObjectId("5c8d5ad022b2b418cedeb91f"),
    "insertDate" : ISODate("2019-03-16T21:21:36.610+01:00"),
    "queue" : "queue",
    "fields" : {
        "consumerTag" : "amq.ctag-dGIx8npNHP_bL5pzRsqQpw",
        "deliveryTag" : 4,
        "redelivered" : false,
        "exchange" : "publish",
        "routingKey" : "a.b.c"
    },
    "properties" : {
        "headers" : {
            "timestamp_in_ms" : 1552767696580.0
        },
        "deliveryMode" : 2,
        "timestamp" : 1552767696
    },
    "messageDate" : ISODate("2019-03-16T21:21:36.580+01:00"),
    "content" : {
        "test" : true
    }
}
```

#### error at parsing

```json
{
    "_id" : ObjectId("5c8d5ae922b2b418cedeb920"),
    "insertDate" : ISODate("2019-03-16T21:22:01.519+01:00"),
    "queue" : "queue",
    "fields" : {
        "consumerTag" : "amq.ctag-dGIx8npNHP_bL5pzRsqQpw",
        "deliveryTag" : 5,
        "redelivered" : false,
        "exchange" : "publish",
        "routingKey" : "a.b.c"
    },
    "properties" : {
        "headers" : {
            "timestamp_in_ms" : 1552767721459.0
        },
        "deliveryMode" : 2,
        "timestamp" : 1552767721
    },
    "messageDate" : ISODate("2019-03-16T21:22:01.459+01:00"),
    "content" : "invalid json\n",
    "error" : {
        "message" : "Unexpected token i in JSON at position 0",
        "stack" : "SyntaxError: Unexpected token i in JSON at position 0\n    at JSON.parse (<anonymous>)\n    at /tmp/amqp2mongo/amqp2mongo.js:80:37\n    at ConfirmChannel.BaseChannel.dispatchMessage (/tmp/amqp2mongo/node_modules/amqplib/lib/channel.js:478:12)\n    at ConfirmChannel.BaseChannel.handleDelivery (/tmp/amqp2mongo/node_modules/amqplib/lib/channel.js:487:15)\n    at ConfirmChannel.emit (events.js:182:13)\n    at /tmp/amqp2mongo/node_modules/amqplib/lib/channel.js:273:10\n    at ConfirmChannel.content [as handleMessage] (/tmp/amqp2mongo/node_modules/amqplib/lib/channel.js:326:9)\n    at ConfirmChannel.C.acceptMessageFrame (/tmp/amqp2mongo/node_modules/amqplib/lib/channel.js:241:31)\n    at ConfirmChannel.C.accept (/tmp/amqp2mongo/node_modules/amqplib/lib/channel.js:394:17)\n    at Connection.mainAccept [as accept] (/tmp/amqp2mongo/node_modules/amqplib/lib/connection.js:64:33)\n    at Socket.go (/tmp/amqp2mongo/node_modules/amqplib/lib/connection.js:478:48)\n    at Socket.emit (events.js:182:13)\n    at emitReadable_ (_stream_readable.js:534:12)\n    at process._tickCallback (internal/process/next_tick.js:63:19)",
        "code" : null
    }
}
```
