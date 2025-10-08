# NestJS Learning Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Quick Setup](#quick-setup)
3. [Project Structure](#project-structure)
4. [First Controller, Service, Module](#first-controller-service-module)
5. [Providers & Dependency Injection](#providers--dependency-injection)
6. [Pipes, Guards, Interceptors, Filters](#pipes-guards-interceptors-filters)
7. [Modules & Organization](#modules--organization)
8. [Database Integration (TypeORM / Prisma)](#database-integration-typeorm--prisma)
9. [Validation and DTOs](#validation-and-dtos)
10. [Authentication & Authorization](#authentication--authorization)
11. [Testing](#testing)
12. [Deployment & Docker](#deployment--docker)
13. [Best Practices & Patterns](#best-practices--patterns)
14. [Useful CLI Commands](#useful-cli-commands)
15. [Resources](#resources)

## Introduction

NestJS is a progressive Node.js framework for building efficient, reliable and scalable server-side applications. It uses TypeScript and draws on concepts from Angular (modules, providers, decorators) to provide a structured architecture.

## Quick Setup

Prerequisites: Node.js (LTS), npm or pnpm, and a code editor.

Install Nest CLI globally (optional):

```bash
npm i -g @nestjs/cli
```

Create a new project using the CLI:

```bash
nest new my-nest-app
cd my-nest-app
npm run start:dev
```

Or with pnpm:

```bash
pnpm create @nestjs/app my-nest-app
cd my-nest-app
pnpm install
pnpm run start:dev
```

## Project Structure

Typical Nest project layout:

```
src/
├─ app.module.ts
├─ main.ts
├─ modules/
│  └─ users/
│     ├─ users.controller.ts
│     ├─ users.service.ts
│     └─ users.module.ts
├─ common/
│  ├─ pipes/
│  ├─ guards/
│  └─ filters/
test/
package.json
nest-cli.json
tsconfig.json
```

## First Controller, Service, Module

Create a simple `Users` resource example.

```ts
// src/modules/users/users.controller.ts
import { Controller, Get, Post, Body, Param } from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll() {
    return this.usersService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(+id);
  }

  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }
}
```

```ts
// src/modules/users/users.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';

@Injectable()
export class UsersService {
  private users = [];
  private idCounter = 1;

  findAll() {
    return this.users;
  }

  findOne(id: number) {
    const user = this.users.find(u => u.id === id);
    if (!user) throw new NotFoundException('User not found');
    return user;
  }

  create(dto: CreateUserDto) {
    const user = { id: this.idCounter++, ...dto };
    this.users.push(user);
    return user;
  }
}
```

```ts
// src/modules/users/users.module.ts
import { Module } from '@nestjs/common';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';

@Module({
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

Add module to root `AppModule`:

```ts
// src/app.module.ts
import { Module } from '@nestjs/common';
import { UsersModule } from './modules/users/users.module';

@Module({
  imports: [UsersModule],
})
export class AppModule {}
```

## Providers & Dependency Injection

Providers are injectable classes. Use `@Injectable()` and add them to `providers` array of a module. The DI system resolves instances for controllers and other providers automatically.

## Pipes, Guards, Interceptors, Filters

- Pipes: transform and validate input (e.g., `ValidationPipe`).
- Guards: control access (e.g., `AuthGuard`).
- Interceptors: transform responses, logging, caching.
- Filters: handle exceptions and map them to HTTP responses.

Example: enable global validation in `main.ts`:

```ts
import { ValidationPipe } from '@nestjs/common';
app.useGlobalPipes(new ValidationPipe({ whitelist: true, transform: true }));
```

## Modules & Organization

Group related controllers and providers into feature modules. Create shared modules for common providers and exports.

## Database Integration (TypeORM / Prisma)

TypeORM example install:

```bash
npm install --save @nestjs/typeorm typeorm sqlite3
```

Basic TypeORM setup in `AppModule`:

```ts
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'sqlite',
      database: 'db.sqlite',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

Prisma example (recommended for type-safe DB access):

```bash
npm install prisma --save-dev
npx prisma init
npm install @prisma/client
```

Then define schema in `prisma/schema.prisma` and use the generated client in providers.

## Validation and DTOs

Use `class-validator` and `class-transformer`.

```bash
npm install class-validator class-transformer
```

Example DTO:

```ts
// src/modules/users/dto/create-user.dto.ts
import { IsString, IsEmail, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;

  @MinLength(6)
  password: string;
}
```

## Authentication & Authorization

Common approaches:
- JWT (passport-jwt)
- Session-based (express-session)
- OAuth providers

Install and configure Passport and JWT strategies for authentication.

## Testing

Nest provides testing helpers using Jest.

```bash
npm run test
npm run test:watch
npm run test:cov
```

Example unit test for `UsersService`:

```ts
import { Test, TestingModule } from '@nestjs/testing';
import { UsersService } from './users.service';

describe('UsersService', () => {
  let service: UsersService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [UsersService],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  it('should create and find a user', () => {
    const user = service.create({ name: 'A', email: 'a@example.com', password: 'secret' });
    expect(service.findOne(user.id)).toEqual(user);
  });
});
```

## Deployment & Docker

Example `Dockerfile`:

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build
CMD ["node", "dist/main.js"]
```

Compose example (with a database):

```yaml
version: '3'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: mydb
```

## Best Practices & Patterns

- Prefer feature modules
- Keep controllers thin; business logic in services
- Use DTOs to validate and type inputs
- Use async/await and proper error handling
- Centralize shared logic in providers or shared modules
- Use CI to run tests and linters

## Useful CLI Commands

- Create a new controller: `nest g controller users` or `nest generate controller users`
- Create a service: `nest g service users`
- Create a module: `nest g module users`
- Run in dev: `npm run start:dev`
- Run tests: `npm run test`

## Resources

- Official docs: https://docs.nestjs.com/
- GitHub: https://github.com/nestjs/nest
- Examples: https://github.com/nestjs/nest/tree/master/sample

Happy building with NestJS! 🚀

## Advanced NestJS Functionality (Extras)

This section covers common advanced features you will likely use in production NestJS apps, with install notes and small examples.

### WebSockets (Realtime)

Nest has built-in support for WebSockets via `@nestjs/websockets` and adapters (Socket.IO, ws).

Install:

```bash
npm install --save @nestjs/websockets @nestjs/platform-socket.io socket.io
```

Basic gateway example:

```ts
import { WebSocketGateway, SubscribeMessage, MessageBody, WebSocketServer } from '@nestjs/websockets';
import { Server } from 'socket.io';

@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('message')
  handleMessage(@MessageBody() payload: any) {
    this.server.emit('message', payload);
  }
}
```

### GraphQL

Nest supports GraphQL with code-first (decorators) or schema-first approaches.

Install (code-first):

```bash
npm install @nestjs/graphql graphql apollo-server-express
```

Basic code-first setup in `AppModule`:

```ts
import { GraphQLModule } from '@nestjs/graphql';

@Module({
  imports: [
    GraphQLModule.forRoot({
      autoSchemaFile: true,
    }),
  ],
})
export class AppModule {}
```

Resolver example:

```ts
import { Resolver, Query, Args, Mutation } from '@nestjs/graphql';

@Resolver()
export class UsersResolver {
  @Query(() => [User])
  users() {
    return this.usersService.findAll();
  }
}
```

### Microservices

Nest provides microservice transport drivers (TCP, Redis, NATS, Kafka, gRPC).

Example (TCP microservice):

```ts
// main.ts (microservice)
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
    transport: Transport.TCP,
    options: { port: 4000 },
  });
  await app.listen();
}

bootstrap();
```

### Scheduling (cron jobs)

Install:

```bash
npm install --save @nestjs/schedule
```

Example:

```ts
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class TasksService {
  @Cron(CronExpression.EVERY_MINUTE)
  handleCron() {
    console.log('Called every minute');
  }
}
```

### Caching

Install:

```bash
npm install --save cache-manager
```

Enable cache in `AppModule`:

```ts
import { CacheModule } from '@nestjs/common';

@Module({
  imports: [CacheModule.register({ ttl: 5 })],
})
export class AppModule {}
```

### File Uploads

Using `@nestjs/platform-express` and `multer`:

```bash
npm install --save @nestjs/platform-express multer
```

Controller example:

```ts
import { Controller, Post, UploadedFile, UseInterceptors } from '@nestjs/common';
import { FileInterceptor } from '@nestjs/platform-express';

@Controller('upload')
export class UploadController {
  @Post()
  @UseInterceptors(FileInterceptor('file'))
  uploadFile(@UploadedFile() file: Express.Multer.File) {
    return { filename: file.originalname };
  }
}
```

### Swagger (API docs)

Install:

```bash
npm install --save @nestjs/swagger swagger-ui-express
```

Setup in `main.ts`:

```ts
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

const config = new DocumentBuilder()
  .setTitle('My API')
  .setDescription('API description')
  .setVersion('1.0')
  .build();

const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api', app, document);
```

### Health Checks

Install:

```bash
npm install --save @nestjs/terminus
```

Basic health check:

```ts
import { TerminusModule } from '@nestjs/terminus';

@Module({ imports: [TerminusModule] })
export class AppModule {}
```

### Mailer

Install (example with nodemailer):

```bash
npm install --save @nestjs-modules/mailer nodemailer
```

Basic setup uses the MailerModule. See package docs for configuration.

### Rate Limiting / Throttling

Install:

```bash
npm install --save @nestjs/throttler
```

Enable globally:

```ts
import { ThrottlerModule } from '@nestjs/throttler';

@Module({
  imports: [ThrottlerModule.forRoot({ ttl: 60, limit: 10 })],
})
export class AppModule {}
```

### Logging

Nest uses a built-in Logger; you can also plug in `winston` or `pino` for structured logging.

Example using built-in Logger:

```ts
import { Logger } from '@nestjs/common';

const logger = new Logger('Bootstrap');
logger.log('Application starting...');
```

### Configuration (dotenv)

Install:

```bash
npm install --save @nestjs/config
```

Usage:

```ts
import { ConfigModule } from '@nestjs/config';

@Module({ imports: [ConfigModule.forRoot({ isGlobal: true })] })
export class AppModule {}
```

### Metrics (Prometheus)

Use libraries like `nestjs-prometheus` to expose metrics.

### CQRS (Command Query Responsibility Segregation)

Install:

```bash
npm install --save @nestjs/cqrs
```

CQRS helps structure complex applications with commands, events, and query handlers.

### Migrations & Transactions

- For TypeORM use `typeorm` CLI or `nestjs/typeorm` migrations.
- For Prisma use `prisma migrate dev` and the generated client supports transactions.

### Internationalization (i18n)

Install an i18n package:

```bash
npm install --save nestjs-i18n
```

Basic usage allows decorators to translate messages and load JSON resource files.

---

If you want, I can scaffold sample code for any of the above features (for example: a small app with WebSockets + Swagger + Rate limiting and Dockerfile), or implement a Users CRUD with Prisma and migrations. Tell me which feature(s) to scaffold and I'll create the files and run a quick smoke test.

## Operational & Integration Features

Below are additional operational topics and integrations commonly used in production NestJS services.

### Graceful Shutdown

- Handle SIGINT/SIGTERM to close database connections, stop accepting new requests, and finish in-flight requests.
- Nest supports lifecycle hooks: `beforeApplicationShutdown()` and `onModuleDestroy()` in providers.

Example:

```ts
import { Injectable, Logger, OnModuleDestroy } from '@nestjs/common';

@Injectable()
export class AppService implements OnModuleDestroy {
  private readonly logger = new Logger(AppService.name);

  async onModuleDestroy() {
    this.logger.log('Cleaning up resources...');
    // close DB connections, flush queues
  }
}
```

In `main.ts` enable explicit shutdown (Nest >= 8):

```ts
const app = await NestFactory.create(AppModule);
app.enableShutdownHooks();
await app.listen(3000);
```

### Server-Sent Events (SSE)

Nest supports SSE via `@Sse()` decorator.

```ts
import { Controller, Sse } from '@nestjs/common';
import { interval, map } from 'rxjs';

@Controller('events')
export class EventsController {
  @Sse('time')
  time() {
    return interval(1000).pipe(map(() => ({ data: new Date().toISOString() })));
  }
}
```

### gRPC

- Use `@nestjs/microservices` with gRPC transport and define `.proto` files.

Install:

```bash
npm install --save @grpc/grpc-js @grpc/proto-loader
```

Example gRPC options in `main.ts`:

```ts
import { Transport } from '@nestjs/microservices';

const app = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.GRPC,
  options: {
    package: 'users',
    protoPath: join(__dirname, 'users.proto'),
  },
});
```

### OpenTelemetry (Tracing)

Add tracing to correlate requests across services.

Install OpenTelemetry packages and configure an exporter (OTLP, Jaeger).

High-level steps:

1. Install `@opentelemetry/sdk-node`, `@opentelemetry/instrumentation-http`, and exporter.
2. Initialize SDK at process start before Nest bootstrap.

### Redis Integration (Cache, Job store, Sessions)

Install Redis client and use as a cache store or queue backend:

```bash
npm install --save ioredis
```

Use `CacheModule.register({ store: redisStore, host, port })` or use `@nestjs/bull` with Redis for queues.

### S3 (File Storage) & Signed Uploads

Use AWS SDK v3 for uploads and pre-signed URLs.

```bash
npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
```

Generate a presigned URL to upload directly from the client and keep your server stateless for large files.

### Webhook Handling & Security

- Validate incoming webhooks with HMAC signatures and timestamps to prevent replay attacks.
- Use background jobs to process webhook payloads asynchronously.

Example HMAC validation:

```ts
import * as crypto from 'crypto';

function verifySignature(secret: string, payload: string, signature: string) {
  const hmac = crypto.createHmac('sha256', secret).update(payload).digest('hex');
  return crypto.timingSafeEqual(Buffer.from(hmac), Buffer.from(signature));
}
```

### Feature Flags

- Integrate LaunchDarkly, Unleash, or simple in-house flag checks to enable gradual rollouts.

### Serverless (AWS Lambda, Vercel)

- Nest can run in serverless environments using adapters (serverless-http). Keep cold starts low by minimizing dependencies and using AWS Lambda provisioned concurrency if necessary.

### Readiness and Liveness Probes

- Expose `/health/live` and `/health/ready` endpoints via Terminus and include checks for DB connectivity, Redis, and other dependencies.

Example:

```ts
import { HealthCheckService, HttpHealthIndicator, HealthCheck } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(private health: HealthCheckService, private http: HttpHealthIndicator) {}

  @Get('ready')
  @HealthCheck()
  ready() {
    return this.health.check([() => this.http.pingCheck('google', 'https://www.google.com')]);
  }
}
```

### Secrets Management

- Use a secrets manager (AWS Secrets Manager, Azure Key Vault) or HashiCorp Vault in production. Avoid storing secrets in repo or plain env files.

### Blue-Green / Canary Deployment Patterns

- Use Kubernetes deployments with readiness/liveness probes to do rolling updates.
- For canary releases, use traffic-splitting via service mesh (Istio) or ingress rules.

---

I added operational and integration topics. I'll mark this task completed. If you'd like, I can scaffold an example implementing one of these features (for example: Redis-backed idempotency + ETag + readiness probe). Tell me which feature to scaffold and I'll create the project files and run a smoke test.
