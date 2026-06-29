---
title: Manage Multiple Datasources and Transactions
impact: HIGH
impactDescription: "Correct multi-DB injection and atomic multi-step operations"
tags: database, datasource, transaction, multi-database
---

## Manage Multiple Datasources and Transactions

For multi-database setups, name each datasource in config and target it explicitly: `@InjectEntityModel(Entity, 'dsName')` or `@InjectDataSource('dsName')`. For atomic multi-step operations, obtain the `DataSource` via `@InjectDataSource()` and call `dataSource.transaction(async (manager) => {...})` — if the callback throws, everything rolls back. Avoid running separate `save` calls outside a transaction when they must succeed together.

**Incorrect (separate saves without a transaction — partial updates on failure):**

```typescript
@Provide()
export class OrderService {
  @InjectEntityModel(Order) orderRepo: Repository<Order>;
  @InjectEntityModel(OrderItem) itemRepo: Repository<OrderItem>;

  async createOrder(userId: number, items: any[]) {
    const order = await this.orderRepo.save({ userId, status: 'pending' });
    for (const item of items) {
      await this.itemRepo.save({ orderId: order.id, ...item });
    }
    // ❌ if a later save fails, the order exists with missing items
    await this.paymentService.charge(order.id);   // ❌ payment + DB inconsistent on error
    return order;
  }
}
```

**Correct (multi-datasource injection + DataSource.transaction for atomicity):**

```typescript
import { Provide } from '@midwayjs/core';
import { InjectDataSource, InjectEntityModel } from '@midwayjs/typeorm';
import { DataSource, EntityManager } from 'typeorm';

@Provide()
export class OrderService {
  // default datasource repository
  @InjectEntityModel(Order) orderRepo: Repository<Order>;
  // named datasource (multi-DB)
  @InjectEntityModel(User, 'analytics') analyticsUserRepo: Repository<User>;
  // raw datasource for transactions
  @InjectDataSource() defaultDataSource: DataSource;
  @InjectDataSource('analytics') analyticsDataSource: DataSource;

  async createOrder(userId: number, items: any[]) {
    // atomic: all saves + payment must succeed together, else full rollback
    return this.defaultDataSource.transaction(async (manager: EntityManager) => {
      const order = await manager.save(Order, { userId, status: 'pending' });
      for (const item of items) {
        await manager.save(OrderItem, { orderId: order.id, ...item });
      }
      await this.paymentService.chargeWithManager(manager, order.id);
      return order;
    });
  }
}
```

Config for multiple datasources:
```typescript
export default {
  typeorm: {
    dataSource: {
      default: { type: 'mysql', host: '...', entities: [Order, OrderItem] },
      analytics: { type: 'postgres', host: '...', entities: [User] },
    },
  },
} as MidwayConfig;
```

Reference: [Midway TypeORM](https://midwayjs.org/docs/extensions/orm), [Data Source Management](https://midwayjs.org/docs/data_source)
