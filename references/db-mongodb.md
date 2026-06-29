---
title: Use MongoDB with Typegoose/Mongoose Datasource Form
impact: MEDIUM-HIGH
impactDescription: "v4 removed EntityModel; use dataSource config form"
tags: database, mongodb, typegoose, mongoose, v4
---

## Use MongoDB with Typegoose/Mongoose Datasource Form

In v4, MongoDB uses the datasource config form (`mongoose.dataSource.<name>`); the old `@EntityModel` decorator and `mongoose.clients` form are **removed**. Use `@midwayjs/typegoose` for class-based entities (`@prop()` from `@typegoose/typegoose`) injected via `@InjectEntityModel(Entity)`. For raw mongoose, get the connection via `MongooseDataSourceManager.getDataSource()` and define schemas manually.

**Incorrect (v3 EntityModel + clients config):**

```typescript
import { EntityModel } from '@midwayjs/typegoose';   // ❌ removed in v4
@EntityModel()
export class User { /* ... */ }

export default { mongoose: { clients: { default: { uri: '...' } } } };  // ❌ v3 clients form
```

**Correct (v4 dataSource form + typegoose entity + injection):**

```typescript
import * as typegoose from '@midwayjs/typegoose';
@Configuration({ imports: [typegoose] })

// src/entity/user.entity.ts — @prop from @typegoose/typegoose
import { prop } from '@typegoose/typegoose';
export class User {
  @prop() name?: string;
  @prop({ type: () => [String] }) jobs?: string[];
}

// config.default.ts — dataSource form
export default {
  mongoose: {
    dataSource: {
      default: {
        uri: 'mongodb://localhost:27017/test',
        options: { useNewUrlParser: true, useUnifiedTopology: true, user: '...', pass: '...' },
        entities: [User],    // declare entities per datasource
      },
    },
  },
} as MidwayConfig;

// service — inject typegoose model
import { InjectEntityModel } from '@midwayjs/typegoose';
import { ReturnModelType } from '@typegoose/typegoose';
@Provide()
export class UserService {
  @InjectEntityModel(User) userModel: ReturnModelType<typeof User>;

  async create() { return this.userModel.create({ name: 'John', jobs: ['dev'] } as User); }
  async find(id: string) { return this.userModel.findById(id).exec(); }
}

// raw mongoose: get the connection, define schemas yourself
import { MongooseDataSourceManager } from '@midwayjs/mongoose';
@Provide()
export class RawService {
  @Inject() dataSourceManager: MongooseDataSourceManager;
  @Init() async init() { this.conn = this.dataSourceManager.getDataSource('default'); }
  async invoke() {
    const schema = new Schema({ name: { type: String, required: true } });
    const Model = this.conn.model('User', schema);
    return new Model({ name: 'Bill' }).save();
  }
}
```

Set global schema options in `onConfigLoad` via `Typegoose.setGlobalOptions(...)`. Version matrix: MongoDB 6.x → mongoose ^7 + typegoose ^10.

Reference: [Midway MongoDB (Typegoose)](https://midwayjs.org/docs/extensions/mongodb)
