---
name: posd-node-rules
description: Core design principles for TypeScript and Node.js backend development based on "A Philosophy of Software Design" by John Ousterhout. Apply when writing, reviewing, refactoring, or designing services, APIs, modules, utilities, or any server-side TypeScript code.
---

# A Philosophy of Software Design -- TypeScript / Node.js

These principles apply when working with TypeScript and Node.js backend code. They encode "A Philosophy of Software Design" by John Ousterhout, adapted for the patterns and pitfalls specific to server-side development.

---

## Strategic Mindset

Every design decision should reduce complexity. Complexity has three symptoms:

- **Change amplification**: A simple change requires modifying many files/modules
- **Cognitive load**: A developer must understand too much to make a change safely
- **Unknown unknowns**: It's not obvious what needs to change, or what might break

Be strategic, not tactical. Don't just "make it work" -- make the system better.

```typescript
// BAD (tactical): duplicate logic across route handlers
app.post('/users', async (req, res) => {
  // validation, normalization, DB insert, email send -- all inline
});
app.post('/teams/:id/members', async (req, res) => {
  // 80% identical user creation logic copy-pasted
});

// GOOD (strategic): extract a service with a clean interface
const user = await userService.create({ email, name, role });
```

When you feel pressure to "just make it work," that's the moment to pause and design. When refactoring, work incrementally -- each step should be independently shippable.

### Never Use `any`

Using `any` defeats the entire purpose of TypeScript. It hides bugs, breaks autocomplete, and creates unknown unknowns. There is always a better option:

- **Import the type** from the package if it exists
- **Create the type yourself** if the package doesn't export one
- **Use generics** if the type needs to be flexible and reusable
- **Use `unknown`** if you truly don't know the type, then narrow it with type guards

```typescript
// BAD: any hides the problem
function parseBody(body: any): User { ... }
const config = JSON.parse(raw) as any;

// GOOD: use unknown + type guard
function parseBody(body: unknown): User {
  if (!isUser(body)) throw new ValidationError('Invalid user payload');
  return body;
}

// GOOD: use generics for reusable patterns
async function fetchJson<T>(url: string): Promise<T> {
  const response = await fetch(url);
  return response.json() as Promise<T>;
}

// GOOD: create the type if the package doesn't export one
interface RedisConfig {
  host: string;
  port: number;
  password?: string;
  db?: number;
}
```

`as` type assertions are also a red flag -- they bypass the type checker. Fix the types properly instead of casting.

---

## Deep Modules

A deep module does a lot behind a simple interface. A shallow module has a complex interface relative to its functionality. Always aim for depth.

When designing new modules, consider at least two interface alternatives and evaluate their tradeoffs in terms of depth, generality, and information hiding.

### Depth in Services

```typescript
// DEEP: simple interface, complex internals (validation, DB, caching, events)
const user = await userService.create({ email: 'a@b.com', name: 'Alice' });

// SHALLOW: caller must orchestrate everything
const validated = userValidator.validate(data);
const normalized = userNormalizer.normalize(validated);
const user = await userRepo.insert(normalized);
await cache.invalidate(`users:${user.id}`);
await eventBus.emit('user.created', user);
```

### Information Hiding

Don't leak implementation decisions through your interface. Consumers should not know which database, ORM, or HTTP client you use internally.

```typescript
// BAD: leaks Prisma types through the service interface
async function getUser(id: string): Promise<PrismaUser & { posts: PrismaPost[] }> { ... }

// GOOD: return domain types -- consumer doesn't know about the ORM
async function getUser(id: string): Promise<User | null> { ... }
```

```typescript
// BAD: leaks HTTP client details
class PaymentGateway {
  async charge(amount: number, axiosConfig: AxiosRequestConfig): Promise<AxiosResponse> { ... }
}

// GOOD: clean domain interface
class PaymentGateway {
  async charge(amount: number, currency: string): Promise<ChargeResult> { ... }
}
```

### Generality

Build somewhat general-purpose interfaces even for specific use cases. They're simpler and more reusable.

```typescript
// BAD: married to one use case
async function findActiveUsersForDashboardExport(): Promise<DashboardUser[]> { ... }

// GOOD: works anywhere users are needed
async function findUsers(criteria: UserQuery): Promise<User[]> { ... }
```

### Pull Complexity Downward

The module should handle its own complexity. Don't push edge cases, retries, or defaults to the caller.

```typescript
// BAD: caller must handle retries, timeouts, defaults
const result = await httpClient.post(url, data, {
  timeout: 5000,
  retries: 3,
  retryDelay: 1000,
  headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${token}` },
});

// GOOD: service handles its own complexity
const result = await paymentService.charge(amount, currency);
// Retries, auth, timeouts, content-type -- all handled internally
```

---

## Module & Service Structure

### Single Responsibility

One module = one job. If you describe it with "and," split it.

A service that handles "user creation and email sending and audit logging" should be three collaborators behind one orchestrating function, not one monolithic service.

### Meaningful Layers

Each layer should transform the abstraction level. Pass-through layers add complexity for zero value.

```typescript
// BAD: repository that just wraps the ORM with no added value
class UserRepository {
  async findById(id: string) {
    return this.prisma.user.findUnique({ where: { id } });
  }
  async create(data: CreateUserInput) {
    return this.prisma.user.create({ data });
  }
  // Every method is a one-line pass-through to Prisma
}

// GOOD: repository that adds real value -- hides ORM, transforms data, handles concerns
class UserRepository {
  async findById(id: string): Promise<User | null> {
    const record = await this.prisma.user.findUnique({
      where: { id },
      include: { profile: true, roles: true },
    });
    return record ? this.toDomain(record) : null;
  }
}
```

### Dependency Injection

Inject dependencies through constructor or function parameters. Don't hardcode imports to concrete implementations.

```typescript
// BAD: hardcoded dependency -- can't test, can't swap
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

export async function getUser(id: string) {
  return prisma.user.findUnique({ where: { id } });
}

// GOOD: injected dependency -- testable, swappable
export function createUserService(db: Database, cache: Cache) {
  return {
    async getUser(id: string): Promise<User | null> { ... },
    async createUser(input: CreateUserInput): Promise<User> { ... },
  };
}
```

---

## Error Handling

### Define Errors Out of Existence

Design APIs so error conditions can't occur, or handle them internally.

```typescript
// BAD: caller must handle "not found" for a delete operation
async function deleteUser(id: string): Promise<void> {
  const user = await db.find(id);
  if (!user) throw new NotFoundError('User not found');
  await db.delete(id);
}

// GOOD: delete is idempotent -- deleting something that doesn't exist is fine
async function deleteUser(id: string): Promise<void> {
  await db.deleteIfExists(id);
}
```

### Use the Type System for Errors

Make error states explicit with discriminated unions instead of try/catch for expected outcomes.

```typescript
// BAD: using exceptions for expected business outcomes
try {
  const user = await userService.create(input);
} catch (e) {
  if (e instanceof DuplicateEmailError) { ... }
  if (e instanceof InvalidInputError) { ... }
  // Unknown unknowns -- what other errors can this throw?
}

// GOOD: Result type makes all outcomes explicit
type CreateUserResult =
  | { success: true; user: User }
  | { success: false; error: 'duplicate_email' | 'invalid_input'; message: string };

const result = await userService.create(input);
if (!result.success) {
  // TypeScript narrows the error -- no unknown unknowns
}
```

### Typed Error Classes

When you do use exceptions, create typed error hierarchies instead of throwing raw strings or generic Errors.

```typescript
// BAD: untyped errors
throw new Error('User not found');
throw 'something went wrong';

// GOOD: typed error classes
class AppError extends Error {
  constructor(message: string, public readonly code: string, public readonly statusCode: number) {
    super(message);
  }
}
class NotFoundError extends AppError {
  constructor(entity: string, id: string) {
    super(`${entity} ${id} not found`, 'NOT_FOUND', 404);
  }
}
```

---

## Readability & Conventions

### Naming

- **Services**: Noun describing the domain -- `UserService`, `PaymentGateway`, not `UserHelper` or `UserUtils`
- **Functions**: Verb describing the action -- `createUser`, `findOrdersByStatus`, not `userData` or `handleUser`
- **Booleans**: `is`/`has`/`should`/`can` prefix -- `isActive`, `hasPermission`, not `active`, `permission`
- **Constants**: UPPER_SNAKE_CASE for true constants -- `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT_MS`
- **Avoid vague names**: `data`, `info`, `item`, `value`, `result`, `manager`, `utils`, `helpers` -- name what it actually IS
- **File names**: Match the primary export -- `user-service.ts` exports `UserService`

```typescript
// BAD
const data = await getData();
function process(items: any[]) { ... }

// GOOD
const activeOrders = await orderRepository.findByStatus('active');
function calculateShippingCost(lineItems: LineItem[]): Money { ... }
```

### Comments

Explain WHY, not WHAT. The code shows what -- comments should add what you can't get from reading the code.

- Document the "contract" of a service (preconditions, postconditions, side effects)
- Document non-obvious performance characteristics or scaling concerns
- Document why a particular approach was chosen over the simpler alternative
- JSDoc on exported interfaces and public methods (= API documentation)

```typescript
// BAD: restates the code
// Get the user by ID
const user = await userService.getUser(id);

// GOOD: explains a non-obvious decision
// We fetch the full user with roles eagerly here because the auth middleware
// needs role data on every request. Lazy loading would cause N+1 queries
// under high concurrency.
const user = await userService.getUserWithRoles(id);
```

### Consistency

Before introducing a new pattern, check how the codebase already handles the same concern.

- Match existing error handling patterns (Result types vs exceptions)
- Match existing service/repository structure
- Match existing naming conventions and file organization
- If an existing pattern is bad, propose changing it **everywhere** rather than creating a parallel approach

---

## Red Flags

When you see these patterns, pause and consider restructuring. When reviewing code, focus on structural issues that affect complexity -- don't nitpick utility functions or simple adapters that serve a deliberate purpose.

### Module Complexity
- File over ~300 lines -- probably doing too much; split by responsibility
- Function over ~50 lines -- extract sub-functions or decompose
- Function with more than ~4 parameters -- use an options/config object
- God service that handles multiple unrelated domains -- split into focused services

### Abstraction Problems
- Pass-through layer that just delegates to another with no transformation -- remove it
- Repository that's a thin wrapper over the ORM adding no value -- question whether you need it
- Barrel file (`index.ts`) re-exporting everything -- pass-through adding no value
- `any` anywhere -- import the type, create it, or use a generic. Use `unknown` with type guards if truly unknown. Never `any`.
- `as` type assertions -- fix the types properly instead of casting
- Name includes the caller's context (`getUserForAdminDashboard`) -- too specific, generalize

### Error Handling Problems
- `catch (e) { }` -- swallowing errors silently creates unknown unknowns
- `catch (e: any)` -- use `unknown` and narrow with type guards
- Throwing raw strings -- use typed error classes
- try/catch around expected business outcomes -- use Result types instead
- No error handling at async boundaries -- unhandled rejections crash the process

### Dependency Problems
- Hardcoded imports of concrete implementations -- inject dependencies instead
- Module directly instantiates its own database/HTTP/cache clients -- accept them as parameters
- Circular dependencies between modules -- restructure the module boundary
- Config values scattered across files -- centralize configuration

### Data Flow Problems
- Business logic in route handlers / controllers -- extract to services
- Multiple modules reading/writing the same database table -- unclear ownership
- Data transformation scattered across layers -- centralize in one place (usually the repository or a mapper)
- Mutable shared state between requests -- use request-scoped context or pure functions
