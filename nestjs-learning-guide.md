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
