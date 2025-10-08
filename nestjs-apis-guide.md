# NestJS APIs Guide

This guide focuses on building robust REST APIs with NestJS. It covers controllers, DTOs, validation, pagination, filtering, versioning, error handling, caching, security, testing, and practical examples.

## Table of Contents
1. [Design Principles](#design-principles)
2. [Controllers & Routes](#controllers--routes)
3. [DTOs and Validation](#dtos-and-validation)
4. [Pagination, Sorting & Filtering](#pagination-sorting--filtering)
5. [Versioning APIs](#versioning-apis)
6. [Error Handling & Exceptions](#error-handling--exceptions)
7. [Response Shaping & Serialization](#response-shaping--serialization)
8. [Caching Responses](#caching-responses)
9. [Security Best Practices](#security-best-practices)
10. [Rate Limiting / Throttling](#rate-limiting--throttling)
11. [File Uploads & Streams](#file-uploads--streams)
12. [Testing APIs](#testing-apis)
13. [API Documentation (Swagger)](#api-documentation-swagger)
14. [Examples & Patterns](#examples--patterns)
15. [Useful Libraries & Tools](#useful-libraries--tools)

## Design Principles

- Follow RESTful conventions: resources, nouns not verbs, proper HTTP methods
- Keep controllers thin; implement business logic in services
- Use DTOs for input validation and typing
- Handle errors explicitly and map them to appropriate HTTP status codes
- Version your API early (URI or header-based)

## Controllers & Routes

A typical controller:

```ts
import { Controller, Get, Post, Param, Body, Query, ParseIntPipe } from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll(@Query() query) {
    return this.usersService.findAll(query);
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.usersService.findOne(id);
  }

  @Post()
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }
}
```

## DTOs and Validation

Use `class-validator` and `class-transformer` to validate incoming requests.

```ts
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

Enable global validation in `main.ts`:

```ts
app.useGlobalPipes(new ValidationPipe({ whitelist: true, transform: true }));
```

## Pagination, Sorting & Filtering

Common pattern: use query parameters and a `PaginationDto`.

```ts
export class PaginationDto {
  page?: number = 1;
  limit?: number = 20;
  sort?: string;
  order?: 'ASC' | 'DESC' = 'ASC';
}
```

Service example handles pagination and filtering using the ORM query builder or repository.

## Versioning APIs

Enable versioning in `main.ts`:

```ts
app.enableVersioning({ type: VersioningType.URI });
```

Then use routes like `/v1/users` or use `@Version('1')` decorator on controllers.

## Error Handling & Exceptions

Use Nest's built-in exceptions and create custom exceptions when needed.

```ts
throw new NotFoundException('User not found');
```

Global exception filter example (optional):

```ts
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception instanceof HttpException ? exception.getStatus() : 500;

    response.status(status).json({
      statusCode: status,
      message: (exception as any).message || 'Internal server error',
    });
  }
}
```

## Response Shaping & Serialization

Use `class-transformer` Expose/Exclude decorators and `ClassSerializerInterceptor` to control responses.

```ts
@Exclude()
export class UserEntity {
  @Expose()
  id: number;

  @Expose()
  name: string;

  password: string; // not exposed
}
```

Enable serialization:

```ts
app.useGlobalInterceptors(new ClassSerializerInterceptor(app.get(Reflector)));
```

## Caching Responses

Use `@nestjs/cache-manager` or the `CacheModule`.

```ts
import { CacheInterceptor, UseInterceptors } from '@nestjs/common';

@UseInterceptors(CacheInterceptor)
@Get()
findAll() {}
```

## Security Best Practices

- Use helmet (Nest provides `helmet()` integration)
- Validate inputs and sanitize where needed
- Escape output and avoid leaking sensitive fields
- Store secrets in environment variables and use `@nestjs/config`
- Use HTTPS in production and secure cookies if using sessions

## Rate Limiting / Throttling

Use `@nestjs/throttler`:

```ts
import { ThrottlerModule } from '@nestjs/throttler';

@Module({ imports: [ThrottlerModule.forRoot({ ttl: 60, limit: 10 })] })
export class AppModule {}
```

## File Uploads & Streams

See the `File Uploads` section in the main NestJS guide. Use streams for large files.

## Testing APIs

- Unit tests for services with mocks
- e2e tests using `supertest` against the compiled app

Example e2e test:

```ts
import * as request from 'supertest';
import { Test } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import { AppModule } from './../src/app.module';

describe('Users (e2e)', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleRef = await Test.createTestingModule({ imports: [AppModule] }).compile();
    app = moduleRef.createNestApplication();
    await app.init();
  });

  it('/users (GET)', () => {
    return request(app.getHttpServer()).get('/users').expect(200);
  });

  afterAll(async () => app.close());
});
```

## API Documentation (Swagger)

See `Swagger` section in the main guide to set up `@nestjs/swagger` and annotate DTOs/controllers with decorators like `@ApiTags()` and `@ApiProperty()`.

## Examples & Patterns

- Controller delegates to service
- Service handles DB and business logic
- Use repositories or query builders for complex queries
- Create a `FiltersService` or `QueryBuilder` helper to apply filtering and sorting

## Useful Libraries & Tools

- `class-validator`, `class-transformer`
- `@nestjs/swagger` for docs
- `@nestjs/throttler` for rate limiting
- `@nestjs/jwt` + `passport-jwt` for auth
- `prisma` or `typeorm` for DB access
- `supertest` for e2e tests

---

If you want, I can scaffold a small, runnable example API (Users CRUD) in this workspace with validation, pagination, Swagger, and tests. Tell me which stack you prefer (Prisma or TypeORM) and I'll scaffold it and run a quick smoke test.
