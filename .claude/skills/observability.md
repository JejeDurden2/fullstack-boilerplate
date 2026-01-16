# Observability - OpenTelemetry

## Setup

```typescript
// tracing.ts (load BEFORE app imports)
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
import { OTLPMetricExporter } from '@opentelemetry/exporter-metrics-otlp-http';
import { PeriodicExportingMetricReader } from '@opentelemetry/sdk-metrics';
import { Resource } from '@opentelemetry/resources';
import { ATTR_SERVICE_NAME, ATTR_SERVICE_VERSION } from '@opentelemetry/semantic-conventions';

const sdk = new NodeSDK({
  resource: new Resource({
    [ATTR_SERVICE_NAME]: process.env.SERVICE_NAME || 'api',
    [ATTR_SERVICE_VERSION]: process.env.npm_package_version || '1.0.0',
  }),
  traceExporter: new OTLPTraceExporter({
    url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT + '/v1/traces',
  }),
  metricReader: new PeriodicExportingMetricReader({
    exporter: new OTLPMetricExporter({
      url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT + '/v1/metrics',
    }),
    exportIntervalMillis: 30000,
  }),
  instrumentations: [getNodeAutoInstrumentations()],
});

sdk.start();
```

---

## Structured Logging with Pino + OpenTelemetry

```typescript
// common/logger/logger.service.ts
import { pino } from 'pino';
import { trace, context } from '@opentelemetry/api';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => ({ level: label }),
  },
  mixin() {
    const span = trace.getSpan(context.active());
    if (span) {
      const { traceId, spanId } = span.spanContext();
      return { traceId, spanId };
    }
    return {};
  },
  transport: process.env.NODE_ENV === 'development' ? { target: 'pino-pretty' } : undefined,
});

@Injectable()
export class LoggerService {
  private context?: string;

  setContext(context: string): void {
    this.context = context;
  }

  info(message: string, data?: Record<string, unknown>): void {
    logger.info({ context: this.context, ...data }, message);
  }

  error(message: string, error?: Error, data?: Record<string, unknown>): void {
    logger.error(
      {
        context: this.context,
        err: error,
        stack: error?.stack,
        ...data,
      },
      message,
    );
  }

  warn(message: string, data?: Record<string, unknown>): void {
    logger.warn({ context: this.context, ...data }, message);
  }

  debug(message: string, data?: Record<string, unknown>): void {
    logger.debug({ context: this.context, ...data }, message);
  }
}
```

---

## Custom Spans

```typescript
// Usage in use cases
import { trace } from '@opentelemetry/api';

const tracer = trace.getTracer('feature-module');

async execute(dto: CreateFeatureDto): Promise<Result<Feature, FeatureError>> {
  return tracer.startActiveSpan('CreateFeatureUseCase.execute', async (span) => {
    try {
      span.setAttribute('feature.name', dto.name);
      // ... logic
      span.setStatus({ code: SpanStatusCode.OK });
      return Result.ok(feature);
    } catch (error) {
      span.setStatus({ code: SpanStatusCode.ERROR, message: error.message });
      span.recordException(error);
      throw error;
    } finally {
      span.end();
    }
  });
}
```

---

## Recommended Stack

- **Local**: Jaeger (docker-compose)
- **Production**: Grafana Tempo, Datadog, or Honeycomb

---

## Key Rules

1. **Add structured logging with context** (traceId, spanId, relevant metadata)
2. **Never log sensitive data** (passwords, tokens, PII)
3. **Use log levels appropriately**: debug, info, warn, error
4. **Include relevant metadata** for debugging
