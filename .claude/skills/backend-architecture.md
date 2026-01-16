# Backend Architecture - NestJS + Hexagonal

## Folder Structure

```
apps/api/src/
├── modules/
│   └── [feature]/
│       ├── domain/
│       │   ├── entities/           # Domain entities
│       │   ├── value-objects/      # Value objects
│       │   ├── events/             # Domain events
│       │   └── repositories/       # Repository interfaces (ports)
│       ├── application/
│       │   ├── use-cases/          # Application services
│       │   ├── ports/              # Input/Output ports
│       │   └── dto/                # Application DTOs
│       ├── infrastructure/
│       │   ├── persistence/        # Prisma repositories (adapters)
│       │   ├── http/               # External API clients
│       │   └── messaging/          # Event publishers
│       └── presentation/
│           ├── controllers/        # HTTP controllers
│           ├── dto/                # Request/Response DTOs
│           └── guards/             # Feature-specific guards
├── common/
│   ├── decorators/
│   ├── filters/
│   ├── guards/
│   ├── interceptors/
│   ├── pipes/
│   └── utils/
├── config/
│   └── configuration.ts
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts
├── app.module.ts
└── main.ts
```

---

## NestJS Patterns

## NestJS Dependency Injection Rules

When creating or modifying NestJS modules:

1. Always check that new providers are added to the `providers` array
2. Always check that services used in other modules are in `exports` array
3. When using @Inject() with tokens, verify the token is registered
4. Run build after any module change to catch DI errors at compile time
5. For circular dependencies, use `forwardRef(() => ModuleName)`

### Module Structure

```typescript
// feature.module.ts
@Module({
  imports: [PrismaModule],
  controllers: [FeatureController],
  providers: [
    // Use cases
    CreateFeatureUseCase,
    GetFeatureUseCase,
    // Ports → Adapters binding
    {
      provide: FEATURE_REPOSITORY,
      useClass: PrismaFeatureRepository,
    },
  ],
  exports: [FEATURE_REPOSITORY],
})
export class FeatureModule {}
```

### Repository Pattern (Port)

```typescript
// domain/repositories/feature.repository.ts
export const FEATURE_REPOSITORY = Symbol('FEATURE_REPOSITORY');

export interface FeatureRepository {
  findById(id: string): Promise<Feature | null>;
  findAll(filter: FeatureFilter): Promise<Feature[]>;
  save(feature: Feature): Promise<Feature>;
  delete(id: string): Promise<void>;
}
```

### Repository Implementation (Adapter)

```typescript
// infrastructure/persistence/prisma-feature.repository.ts
@Injectable()
export class PrismaFeatureRepository implements FeatureRepository {
  constructor(private readonly prisma: PrismaService) {}

  async findById(id: string): Promise<Feature | null> {
    const data = await this.prisma.feature.findUnique({ where: { id } });
    return data ? FeatureMapper.toDomain(data) : null;
  }
}
```

### Use Case Pattern

```typescript
// application/use-cases/create-feature.use-case.ts
@Injectable()
export class CreateFeatureUseCase {
  constructor(
    @Inject(FEATURE_REPOSITORY)
    private readonly featureRepository: FeatureRepository,
  ) {}

  async execute(dto: CreateFeatureDto): Promise<Result<Feature, FeatureError>> {
    const feature = Feature.create(dto);
    if (feature.isErr()) return feature;

    await this.featureRepository.save(feature.value);
    return Result.ok(feature.value);
  }
}
```

### Result Pattern (Error Handling)

```typescript
// common/utils/result.ts
type Result<T, E> = { ok: true; value: T } | { ok: false; error: E };

export const Result = {
  ok: <T>(value: T): Result<T, never> => ({ ok: true, value }),
  err: <E>(error: E): Result<never, E> => ({ ok: false, error }),
};
```

### Controller Pattern

```typescript
@Controller('features')
@ApiTags('Features')
export class FeatureController {
  constructor(private readonly createFeature: CreateFeatureUseCase) {}

  @Post()
  @ApiOperation({ summary: 'Create a feature' })
  async create(@Body() dto: CreateFeatureRequestDto): Promise<FeatureResponseDto> {
    const result = await this.createFeature.execute(dto);
    if (!result.ok) throw new BadRequestException(result.error);
    return FeatureMapper.toResponse(result.value);
  }
}
```

---

## Global Configuration

### Interceptors & Filters

```typescript
// main.ts
app.useGlobalPipes(
  new ValidationPipe({
    transform: true,
    whitelist: true,
    forbidNonWhitelisted: true,
  }),
);
app.useGlobalInterceptors(new LoggingInterceptor());
app.useGlobalFilters(new AllExceptionsFilter());
```

---

---

## Domain Events & Event-Driven Architecture

### Domain Event Pattern

```typescript
// shared/domain/domain-event.base.ts
export abstract class DomainEvent {
  public readonly occurredAt: Date;
  public readonly aggregateId: string;
  public readonly eventName: string;

  constructor(aggregateId: string) {
    this.occurredAt = new Date();
    this.aggregateId = aggregateId;
    this.eventName = this.constructor.name;
  }
}

// Example: keepsake/domain/events/keepsake-delivered.event.ts
export class KeepsakeDeliveredEvent extends DomainEvent {
  constructor(
    public readonly keepsakeId: string,
    public readonly vaultId: string,
    public readonly triggerCondition: TriggerCondition,
  ) {
    super(keepsakeId);
  }
}
```

### Aggregate Root with Events

```typescript
// shared/domain/aggregate-root.base.ts
export abstract class AggregateRoot<T> extends Entity<T> {
  private _domainEvents: DomainEvent[] = [];

  protected addDomainEvent(event: DomainEvent): void {
    this._domainEvents.push(event);
  }

  public pullDomainEvents(): DomainEvent[] {
    const events = [...this._domainEvents];
    this._domainEvents = [];
    return events;
  }
}

// Example usage in entity
export class Keepsake extends AggregateRoot<KeepsakeProps> {
  deliver(): Result<void, string> {
    this.props.status = KeepsakeStatus.DELIVERED;
    this.addDomainEvent(new KeepsakeDeliveredEvent(this.id, this.vaultId, this.triggerCondition));
    return ok(undefined);
  }
}
```

### Event Handler Pattern

```typescript
// notification/application/handlers/keepsake-delivered.handler.ts
@Injectable()
export class KeepsakeDeliveredHandler {
  constructor(private readonly orchestrator: NotificationOrchestratorService) {}

  async handle(event: KeepsakeDeliveredEvent): Promise<void> {
    await this.orchestrator.scheduleNotificationsForKeepsake({
      keepsakeId: event.keepsakeId,
      vaultId: event.vaultId,
    });
  }
}
```

---

## Job Queue with BullMQ

### Queue Configuration

```typescript
// shared/queue/queue.module.ts
@Module({
  imports: [
    BullModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        connection: {
          host: configService.get('REDIS_HOST', 'localhost'),
          port: configService.get('REDIS_PORT', 6379),
        },
        defaultJobOptions: {
          attempts: 3,
          backoff: { type: 'exponential', delay: 1000 },
        },
      }),
    }),
  ],
})
export class QueueModule {}
```

### Job Processor

```typescript
// notification/infrastructure/processors/notification.processor.ts
@Processor('notification')
export class NotificationProcessor extends WorkerHost {
  async process(job: Job<NotificationJobData>): Promise<void> {
    const { notificationLogId, type } = job.data;

    switch (type) {
      case NotificationType.BENEFICIARY_INVITATION:
        await this.sendBeneficiaryInvitation(notificationLogId);
        break;
      // ... other cases
    }
  }
}
```

### Scheduling Delayed Jobs

```typescript
// Schedule job with delay
await this.notificationQueue.add(
  'send_email',
  { notificationLogId, beneficiaryId },
  {
    jobId: `notification-${logId}`,
    delay: delayInMilliseconds, // 72h for trusted person, 168h for beneficiaries
  },
);
```

---

## Pluggable Services (Port-Adapter Pattern)

### Email Service Port

```typescript
// shared/ports/email.port.ts
export interface IEmailService {
  sendEmail(input: SendEmailInput): Promise<void>;
  sendBeneficiaryInvitation(input: BeneficiaryInvitationEmailInput): Promise<void>;
}

export const EMAIL_SERVICE = Symbol('EMAIL_SERVICE');
```

### Console Email Adapter (Development)

```typescript
// shared/adapters/console-email.adapter.ts
@Injectable()
export class ConsoleEmailAdapter implements IEmailService {
  async sendEmail(input: SendEmailInput): Promise<void> {
    console.log('📧 EMAIL (Console Mode)');
    console.log(`To: ${input.to}`);
    console.log(`Subject: ${input.subject}`);
    console.log(input.html);
  }
}
```

### Module Configuration

```typescript
@Module({
  providers: [
    {
      provide: EMAIL_SERVICE,
      useClass: ConsoleEmailAdapter, // Swap for ResendEmailAdapter in production
    },
  ],
})
export class SharedModule {}
```

---

## Multi-Role User System

### User Role Enum

```typescript
export enum UserRole {
  VAULT_OWNER = 'VAULT_OWNER', // Has own vault
  BENEFICIARY = 'BENEFICIARY', // Receives keepsakes
  BOTH = 'BOTH', // Both roles simultaneously
}
```

### Role Methods in Entity

```typescript
export class User extends AggregateRoot<UserProps> {
  isVaultOwner(): boolean {
    return this.props.role === UserRole.VAULT_OWNER || this.props.role === UserRole.BOTH;
  }

  isBeneficiary(): boolean {
    return this.props.role === UserRole.BENEFICIARY || this.props.role === UserRole.BOTH;
  }

  linkBeneficiaryProfile(): Result<void, string> {
    if (this.props.role === UserRole.VAULT_OWNER) {
      this.props.role = UserRole.BOTH;
    }
    return ok(undefined);
  }
}
```

### JWT with Role

```typescript
// JWT payload includes role
export interface JwtPayload {
  sub: string;
  email: string;
  role?: UserRole;
}

// JWT strategy validates and returns role
export class JwtStrategy extends PassportStrategy(Strategy) {
  validate(payload: JwtPayload): AuthenticatedUser {
    return {
      id: payload.sub,
      email: payload.email,
      role: payload.role,
    };
  }
}
```

---

## Key Rules

1. **Domain layer has zero imports from infrastructure**
2. **Use cases return `Result<T, E>`, not throwing exceptions**
3. **Controllers only orchestrate, no business logic**
4. **Use dependency injection for all services**
5. **Repository interfaces (ports) live in domain/, implementations (adapters) in infrastructure/**
6. **Domain events for cross-aggregate communication (loose coupling)**
7. **BullMQ for delayed/async operations (emails, notifications)**
8. **Port-Adapter pattern for external services (email, payment, etc.)**
9. **User roles support multiple simultaneous states (BOTH = VAULT_OWNER + BENEFICIARY)**
