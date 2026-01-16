# Testing Strategy

## Overview

| Type        | Tool               | Target                   | Coverage    |
| ----------- | ------------------ | ------------------------ | ----------- |
| Unit        | Vitest             | Domain, Utils, Use Cases | 80%+        |
| Integration | Vitest + Supertest | API endpoints            | 70%+        |
| E2E         | Playwright         | User flows critiques     | Smoke tests |

---

## Unit Tests (Vitest)

### Use Case Test Example

```typescript
// Example: use-case test
describe('CreateFeatureUseCase', () => {
  let useCase: CreateFeatureUseCase;
  let mockRepository: MockType<FeatureRepository>;

  beforeEach(() => {
    mockRepository = createMock<FeatureRepository>();
    useCase = new CreateFeatureUseCase(mockRepository);
  });

  it('should create a feature successfully', async () => {
    const dto = { name: 'Test' };
    mockRepository.save.mockResolvedValue(Feature.create(dto).value);

    const result = await useCase.execute(dto);

    expect(result.ok).toBe(true);
    expect(mockRepository.save).toHaveBeenCalled();
  });

  it('should return error for invalid input', async () => {
    const dto = { name: '' };

    const result = await useCase.execute(dto);

    expect(result.ok).toBe(false);
    expect(result.error.code).toBe('VALIDATION_ERROR');
  });
});
```

### Domain Entity Test Example

```typescript
describe('User Entity', () => {
  it('should create valid user', () => {
    const result = User.create({
      email: 'test@example.com',
      name: 'Test User',
    });

    expect(result.ok).toBe(true);
    expect(result.value.email).toBe('test@example.com');
  });

  it('should reject invalid email', () => {
    const result = User.create({
      email: 'invalid',
      name: 'Test',
    });

    expect(result.ok).toBe(false);
  });
});
```

---

## Frontend Tests

### Component Test Example

```typescript
// feature.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('LoginForm', () => {
  it('should submit valid credentials', async () => {
    const onSubmit = vi.fn();
    render(<LoginForm onSubmit={onSubmit} />);

    await userEvent.type(screen.getByLabelText(/email/i), 'test@test.com');
    await userEvent.type(screen.getByLabelText(/password/i), 'password123');
    await userEvent.click(screen.getByRole('button', { name: /login/i }));

    expect(onSubmit).toHaveBeenCalledWith({
      email: 'test@test.com',
      password: 'password123'
    });
  });

  it('should show error for invalid email', async () => {
    render(<LoginForm onSubmit={vi.fn()} />);

    await userEvent.type(screen.getByLabelText(/email/i), 'invalid');
    await userEvent.click(screen.getByRole('button', { name: /login/i }));

    expect(screen.getByText(/email invalide/i)).toBeInTheDocument();
  });
});
```

---

## E2E Tests (Playwright)

```typescript
// e2e/features.spec.ts
test.describe('Features', () => {
  test('should create a new feature', async ({ page }) => {
    await page.goto('/features/new');
    await page.fill('[name="name"]', 'Test Feature');
    await page.click('button[type="submit"]');

    await expect(page).toHaveURL(/\/features\/[\w-]+/);
    await expect(page.locator('h1')).toContainText('Test Feature');
  });
});

// e2e/auth.spec.ts
test.describe('Authentication', () => {
  test('user can login and access dashboard', async ({ page }) => {
    await page.goto('/login');
    await page.fill('[name="email"]', 'user@test.com');
    await page.fill('[name="password"]', 'password');
    await page.click('button[type="submit"]');

    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByRole('heading', { name: /dashboard/i })).toBeVisible();
  });
});
```

---

## Commands

```bash
# Run all tests
pnpm test

# Watch mode
pnpm test:watch

# With coverage
pnpm test:coverage

# E2E tests
pnpm test:e2e
```

---

## Key Rules

1. **Write tests for business logic** — use cases, entities, value objects
2. **Edge cases covered** — null, empty, boundary values
3. **Error paths tested** — not just happy path
4. **Coverage >= 80%** on new code
5. **Use mocks for external dependencies** — repositories, services
6. **Test behavior, not implementation**
