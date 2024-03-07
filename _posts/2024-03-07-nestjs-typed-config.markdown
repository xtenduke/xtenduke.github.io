---
layout: post
title:  "Nestjs typed configuration"
date:   2024-03-07 00:00:00 +0800
categories: backend
---

#### Problem
The OOTB [NestJS](https://nestjs.com) configuration support leaves a lot to be desired, mainly it lacks any form of validation or schema.


 If you're reading this you probably know that having a webapp running with malformed configuration is not a good idea. The aim for this solution is that the application will fail to deploy if your configuration is wrong, as the validation will fail on boot.

#### Solution
Fortunately, you can roll your own solution with [joi](https://joi.dev/) pretty easily.
Simply, it allows you to inject typed config objects into your Injectables.

The following is an example of one of the configuration classes, each of the props are assigned from process.env. Note that it extends the `BaseConfig` class, which is below.

```
// database.config.ts
import { BaseConfig } from "@/config/config";
import * as Joi from "joi";

export class DatabaseConfig extends BaseConfig {
  public host = process.env.POSTGRES_HOST;
  public port = Number.parseInt(process.env.POSTGRES_PORT, 10);
  public username = process.env.POSTGRES_USERNAME;
  public password = process.env.POSTGRES_PASSWORD;
  public database = process.env.POSTGRES_DATABASE;

  public schema = Joi.object<DatabaseConfig>({
    host: Joi.string().min(1).required(),
    port: Joi.number().min(4).required(),
    username: Joi.string().min(1).required(),
    password: Joi.string().min(1).required(),
    database: Joi.string().min(1).required(),
  });
}

```


The BaseConfig class just handles joi schema validation. It would be best here to throw an exception that is caught by your request/response pipeline so nice errors can be returned to the user.
```
// config.ts
import { Logger } from "@nestjs/common";
import { AnySchema } from "joi";

export abstract class BaseConfig {
  public validate(): this {
    const validationResult = this.schema.validate(this, { allowUnknown: true });
    if (validationResult.error) {
      throw new Error(
        `Failed validating ${this.constructor.name} - ${validationResult.error.message}`,
      );
    }

    // don't pass the schema prop
    delete this.schema;

    Logger.log(
      `Loaded config ${this.constructor.name} - ${JSON.stringify(this)}`,
    );
    return this;
  }

  public abstract schema?: AnySchema;
}
```


This configuration can be injected into `@Injectable()` classes 
```
// database.service.ts
@Injectable()
export class DatabaseService {
  constructor(private readonly config: DatabaseConfig) {
  ...
```

The config objects are provided by the ConfigProvider, which must be used in every module like this.
```
// database.module.ts
@Module({
  imports: [],
  providers: [DatabaseService, ConfigProvider(DatabaseConfig)],
  exports: [DatabaseService]
})
export class DatabaseModule {}
```

The config provider is pretty simple, it uses the type passed into the provider function to construct the object (which pulls the values from env env), then it calls the validation function.
```
// config.provider.ts
import { Provider, Type } from "@nestjs/common";
import { BaseConfig } from "./config";

export const ConfigProvider = <T extends BaseConfig>(clazz: Type<T>) => {
  return {
    provide: clazz,
    useFactory: (): T => {
      return new clazz().validate();
    },
  } as Provider;
};

```

Arguably the most important part of the setup, since we are using process.env, we need to use something like [dotenv](https://www.npmjs.com/package/dotenv) to load config into `process.env`.

```
import { NestFactory } from "@nestjs/core";
import * as dotenv from "dotenv";
import { AppModule } from "./app.module";

async function bootstrap() {
  dotenv.config(); // load the .env file
  const app = await NestFactory.create(AppModule, { cors: true });
  await app.listen(process.env.PORT || 3000);
}
bootstrap();
```

#### What next?
 This is a good extensible starting point for most applications. You could extend this by combining it with built in NestJS `ConfigService` which would give you the ability to easily manage different environments, instead of using dotenv.

 Further, you could create custom environment files in json, with process env overrides, for the best of both worlds.