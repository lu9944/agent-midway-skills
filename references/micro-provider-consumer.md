---
title: Use the Correct Microservice Provider/Consumer Pattern
impact: MEDIUM
impactDescription: "gRPC/RabbitMQ/Kafka each have specific decorator conventions"
tags: microservices, grpc, rabbitmq, kafka, provider, consumer
---

## Use the Correct Microservice Provider/Consumer Pattern

Each Midway transport has distinct decorators. gRPC uses `@Provider(MSProviderType.GRPC)` + `@GrpcMethod()`. RabbitMQ/Kafka use `@Consumer(MSListenerType.X)` + `@RabbitMQListener`/`@KafkaConsumer`. Kafka producers use the service-factory pattern (`@InjectClient(KafkaProducerFactory, 'name')`). Each transport has its own `Context` type, framework, and (for most) an independent logger. Match the decorator to the transport — do not mix conventions.

**Incorrect (wrong decorator for the transport, ignoring context typing):**

```typescript
// ❌ using @Provide for a gRPC service (should be @Provider)
@Provide()
export class Greeter { async sayHello(req) { return {}; } }

// ❌ RabbitMQ consumer that never acks the message (redelivered forever)
@Consumer(MSListenerType.RABBITMQ)
export class UserConsumer {
  @RabbitMQListener('tasks')
  async gotData(msg: ConsumeMessage) {
    await this.process(msg);   // ❌ missing ctx.channel.ack(msg)
  }
}
```

**Correct (transport-specific decorators + typed context + ack):**

```typescript
// === gRPC Provider (server side) ===
import { MSProviderType, Provider, GrpcMethod, Inject } from '@midwayjs/core';
import { Context } from '@midwayjs/grpc';

@Provider(MSProviderType.GRPC, { package: 'helloworld' })
export class Greeter implements helloworld.Greeter {
  @Inject() ctx: Context;
  @GrpcMethod()
  async sayHello(request: helloworld.HelloRequest) {
    return { message: 'Hello ' + request.name };
  }
}
// gRPC client: this.grpcClients.getService<helloworld.GreeterClient>('helloworld.Greeter')

// === RabbitMQ Consumer (must ack!) ===
import { Consumer, MSListenerType, RabbitMQListener } from '@midwayjs/core';
import { Context } from '@midwayjs/rabbitmq';
import { ConsumeMessage } from 'amqplib';

@Consumer(MSListenerType.RABBITMQ)
export class TaskConsumer {
  @Inject() ctx: Context;   // { channel, requestContext }
  @RabbitMQListener('tasks', {
    exchange: 'logs',
    exchangeOptions: { type: 'fanout', durable: false },
    consumeOptions: { noAck: false },
  })
  async gotData(msg: ConsumeMessage) {
    try {
      await this.process(msg.content.toString());
    } finally {
      this.ctx.channel.ack(msg);   // ✓ always ack (or nack for retry)
    }
  }
}

// === Kafka Consumer + Producer ===
import { KafkaConsumer, IKafkaConsumer } from '@midwayjs/kafka';
import { KafkaJS } from '@midwayjs/kafka';   // KafkaJS namespace re-exports kafkajs types

@KafkaConsumer('sub1')
export class OrderConsumer implements IKafkaConsumer {
  async eachMessage(payload: KafkaJS.EachMessagePayload) { /* ... */ }
}

// Kafka producer via service-factory
import { InjectClient } from '@midwayjs/core';
import { KafkaProducerFactory } from '@midwayjs/kafka';
@Provide()
export class OrderProducer {
  @InjectClient(KafkaProducerFactory, 'pub1') producer: KafkaJS.Producer;
  async publish(topic: string, key: string, value: string) {
    await this.producer.send({ topic, messages: [{ key, value }] });
  }
}
```

> **Kafka ctx fields:** `ctx.topic` / `ctx.partition` / `ctx.message` / `ctx.commitOffsets()` on the kafka `Context` are **deprecated** in v4 — read from `ctx.payload` (the `EachMessagePayload`) instead, and commit offsets via `ctx.consumer.commitOffsets(...)`.

WebSocket: `@WSController(namespace)` + `@OnWSMessage`/`@WSEmit` (socket.io) or `@WSController()` + `@OnWSMessage`/`@WSBroadCast` (ws).

Reference: [Midway gRPC](https://midwayjs.org/docs/extensions/grpc), [RabbitMQ](https://midwayjs.org/docs/extensions/rabbitmq), [Kafka](https://midwayjs.org/docs/extensions/kafka)
