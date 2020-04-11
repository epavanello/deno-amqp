# deno-amqp
![CI](https://github.com/lenkan/deno-amqp/workflows/CI/badge.svg)

AMQP 0.9.1 implementation for https://deno.land/. The library is implemented to connect to a RabbitMQ broker. For testing purposes, an instance can be started using docker:

```
docker run -d --rm -p 15672:15672 -p 5672:5672 rabbitmq:3-management
```

# Usage

## Consume messages

```ts
import { connect } from "https://deno.land/x/amqp/amqp.ts";

const connection = await connect();
const channel = await connection.openChannel();

const queueName = "my.queue";
await channel.declareQueue({ queue: queueName });
await channel.consume(
  { queue: queueName },
  async (args, props, data) => {
    console.log(JSON.stringify(args));
    console.log(JSON.stringify(props));
    console.log(new TextDecoder().decode(data));
    await channel.ack({ deliveryTag: args.deliveryTag });
  },
);
```

## Publish messages

```ts
import { connect } from "https://deno.land/x/amqp/amqp.ts";

const connection = await connect();
const channel = await connection.openChannel();

const queueName = "my.queue";
await channel.declareQueue({ queue: queueName });
await channel.publish(
  { routingKey: queueName },
  { contentType: "application/json" },
  new TextEncoder().encode(JSON.stringify({ foo: "bar" })),
);

await connection.close();
```

## More examples

See [examples/](examples/). Start the example consumer by running

```
deno --allow-net examples/consume_message.ts queue
```

Then, send a message to the same queue using the default exchange using the example publisher

```
deno --allow-net examples/publish_message.ts queue
```

# Development

## Testing

Run unit tests

```
deno test src/
```

Run module tests (requires an AMQP broker running on localhost)

```
deno test --allow-net module_test/
```
