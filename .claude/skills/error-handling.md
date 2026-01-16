# Error Handling

## Domain Errors (Typed)

```typescript
// common/errors/domain-error.ts
export abstract class DomainError extends Error {
  abstract readonly code: string;
  abstract readonly statusCode: number;

  constructor(
    message: string,
    public readonly metadata?: Record<string, unknown>,
  ) {
    super(message);
    this.name = this.constructor.name;
  }

  toJSON() {
    return {
      code: this.code,
      message: this.message,
      ...(this.metadata && { details: this.metadata }),
    };
  }
}

// Feature-specific errors
export class FeatureNotFoundError extends DomainError {
  readonly code = 'FEATURE_NOT_FOUND';
  readonly statusCode = 404;

  constructor(id: string) {
    super(`Feature with id "${id}" not found`, { featureId: id });
  }
}

export class FeatureValidationError extends DomainError {
  readonly code = 'FEATURE_VALIDATION_ERROR';
  readonly statusCode = 400;

  constructor(errors: string[]) {
    super('Validation failed', { errors });
  }
}

export class FeatureConflictError extends DomainError {
  readonly code = 'FEATURE_CONFLICT';
  readonly statusCode = 409;

  constructor(field: string, value: string) {
    super(`Feature with ${field} "${value}" already exists`, { field, value });
  }
}
```

---

## Result Pattern (Extended)

```typescript
// common/utils/result.ts
export type Result<T, E extends DomainError = DomainError> =
  | { ok: true; value: T }
  | { ok: false; error: E };

export const Result = {
  ok: <T>(value: T): Result<T, never> => ({ ok: true, value }),
  err: <E extends DomainError>(error: E): Result<never, E> => ({ ok: false, error }),

  // Helpers
  isOk: <T, E extends DomainError>(result: Result<T, E>): result is { ok: true; value: T } =>
    result.ok,
  isErr: <T, E extends DomainError>(result: Result<T, E>): result is { ok: false; error: E } =>
    !result.ok,

  // Map/flatMap for chaining
  map: <T, U, E extends DomainError>(result: Result<T, E>, fn: (value: T) => U): Result<U, E> =>
    result.ok ? Result.ok(fn(result.value)) : result,

  flatMap: <T, U, E extends DomainError>(
    result: Result<T, E>,
    fn: (value: T) => Result<U, E>,
  ): Result<U, E> => (result.ok ? fn(result.value) : result),
};
```

---

## Use Case Error Handling

```typescript
// application/use-cases/get-feature.use-case.ts
@Injectable()
export class GetFeatureUseCase {
  constructor(
    @Inject(FEATURE_REPOSITORY) private readonly repo: FeatureRepository,
    private readonly logger: LoggerService,
  ) {
    this.logger.setContext(GetFeatureUseCase.name);
  }

  async execute(id: string): Promise<Result<Feature, FeatureNotFoundError>> {
    this.logger.debug('Fetching feature', { featureId: id });

    const feature = await this.repo.findById(id);

    if (!feature) {
      this.logger.warn('Feature not found', { featureId: id });
      return Result.err(new FeatureNotFoundError(id));
    }

    return Result.ok(feature);
  }
}
```

---

## Global Exception Filter

```typescript
// common/filters/all-exceptions.filter.ts
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  constructor(private readonly logger: LoggerService) {
    this.logger.setContext(AllExceptionsFilter.name);
  }

  catch(exception: unknown, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    let status = 500;
    let body: Record<string, unknown> = {
      code: 'INTERNAL_ERROR',
      message: 'Internal server error',
    };

    // Domain errors (our custom errors)
    if (exception instanceof DomainError) {
      status = exception.statusCode;
      body = exception.toJSON();
    }
    // NestJS HTTP exceptions
    else if (exception instanceof HttpException) {
      status = exception.getStatus();
      const res = exception.getResponse();
      body = typeof res === 'object' ? (res as Record<string, unknown>) : { message: res };
    }
    // Prisma errors
    else if (exception instanceof Prisma.PrismaClientKnownRequestError) {
      const prismaError = this.handlePrismaError(exception);
      status = prismaError.status;
      body = prismaError.body;
    }
    // Unknown errors
    else if (exception instanceof Error) {
      this.logger.error('Unhandled exception', exception, {
        path: request.url,
        method: request.method,
      });
    }

    response.status(status).json({
      ...body,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }

  private handlePrismaError(error: Prisma.PrismaClientKnownRequestError) {
    switch (error.code) {
      case 'P2002': // Unique constraint
        return {
          status: 409,
          body: { code: 'CONFLICT', message: 'Resource already exists', field: error.meta?.target },
        };
      case 'P2025': // Record not found
        return {
          status: 404,
          body: { code: 'NOT_FOUND', message: 'Resource not found' },
        };
      default:
        return {
          status: 500,
          body: { code: 'DATABASE_ERROR', message: 'Database error' },
        };
    }
  }
}
```

---

## Controller Error Handling

```typescript
@Controller('features')
export class FeatureController {
  constructor(
    private readonly getFeature: GetFeatureUseCase,
    private readonly createFeature: CreateFeatureUseCase,
  ) {}

  @Get(':id')
  async findOne(@Param('id') id: string): Promise<FeatureResponseDto> {
    const result = await this.getFeature.execute(id);

    if (!result.ok) {
      throw new HttpException(result.error.toJSON(), result.error.statusCode);
    }

    return FeatureMapper.toResponse(result.value);
  }

  @Post()
  async create(@Body() dto: CreateFeatureRequestDto): Promise<FeatureResponseDto> {
    const result = await this.createFeature.execute(dto);

    if (!result.ok) {
      throw new HttpException(result.error.toJSON(), result.error.statusCode);
    }

    return FeatureMapper.toResponse(result.value);
  }
}
```

---

## Frontend Error Handling

```typescript
// lib/api-client.ts
class ApiError extends Error {
  constructor(
    public readonly code: string,
    message: string,
    public readonly status: number,
    public readonly details?: unknown,
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

const apiClient = {
  async request<T>(url: string, options?: RequestInit): Promise<T> {
    const response = await fetch(`${API_URL}${url}`, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...options?.headers,
      },
    });

    if (!response.ok) {
      const body = await response.json().catch(() => ({}));
      throw new ApiError(
        body.code || 'UNKNOWN_ERROR',
        body.message || 'An error occurred',
        response.status,
        body.details,
      );
    }

    return response.json();
  },
};

// TanStack Query error handling
export function useFeature(id: string) {
  return useQuery({
    queryKey: ['features', id],
    queryFn: () => apiClient.request<Feature>(`/features/${id}`),
    retry: (failureCount, error) => {
      if (error instanceof ApiError && error.status === 404) return false;
      return failureCount < 3;
    },
  });
}
```

---

## Key Rules

1. **All errors are typed `DomainError` subclasses**
2. **Error messages are user-friendly and actionable**
3. **Errors include relevant metadata for debugging**
4. **No swallowed errors (empty catch blocks)**
5. **Use `Result.err()` over throwing exceptions in use cases**
6. **Never throw generic `Error` — use typed DomainError subclasses**
