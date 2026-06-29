---
title: Use MQTT for IoT Messaging
impact: MEDIUM
impactDescription: "Correct subscriber/publisher patterns with separate configs"
tags: microservices, mqtt, iot, messaging, pub-sub
---

## Use MQTT for IoT Messaging

`@midwayjs/mqtt` splits into **subscribers** (`@MqttSubscriber(name)` + `IMqttSubscriber`) and **publishers** (service-factory pattern via `DefaultMqttProducer`). They are independent — use either alone. Subscribers are keyed by name matching the config; publishers support multi-instance via `mqtt.pub.clients`. The subscriber context (`ctx`) carries `topic`, `message` (Buffer), and `packet`. Serverless supports **publish only**.

**Incorrect (raw mqtt client, no lifecycle, no DI):**

```typescript
import mqtt from 'mqtt';   // ❌ bypasses the component
const client = mqtt.connect('mqtt://broker');
client.on('message', (topic, message) => { /* ❌ no DI, no graceful shutdown */ });
```

**Correct (Midway subscriber + publisher factory):**

```typescript
import * as mqtt from '@midwayjs/mqtt';
@Configuration({ imports: [mqtt] })

// config.default.ts — separate sub/pub configs
export default {
  mqtt: {
    sub: {
      sub1: { connectOptions: { host: 'broker.hivemq.com', port: 1883 }, subscribeOptions: { topicObject: 'sensors/temp' } },
    },
    pub: {
      clients: { default: { host: 'broker.hivemq.com', port: 1883 } },
    },
  },
} as MidwayConfig;

// subscriber — name must match config key
import { MqttSubscriber, IMqttSubscriber, Context } from '@midwayjs/mqtt';
@MqttSubscriber('sub1')
export class TempSubscriber implements IMqttSubscriber {
  @Inject() ctx: Context;
  async subscribe() {
    const temp = JSON.parse(this.ctx.message.toString()).value;  // message is Buffer
    // process reading
  }
}

// publisher — default or named instance
import { DefaultMqttProducer, MqttProducerFactory } from '@midwayjs/mqtt';
import { InjectClient } from '@midwayjs/core';
@Provide()
export class CommandPublisher {
  @Inject() producer: DefaultMqttProducer;                       // default
  @InjectClient(MqttProducerFactory, 'default') named: DefaultMqttProducer;
  async send(topic: string, msg: string) {
    await this.producer.publishAsync(topic, msg, { qos: 1 });
  }
}
```

Reference: [Midway MQTT](https://midwayjs.org/docs/extensions/mqtt)
