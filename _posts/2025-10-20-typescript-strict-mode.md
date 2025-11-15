---
layout: post
title: "TypeScript Strict Mode: Why We Enabled It (And How)"
date: 2025-10-20 14:30:00 +0800
categories: frontend typescript
author: Jordan Kim
---

Enabling TypeScript strict mode in a legacy codebase with 50k+ lines? Painful. Worth it? Absolutely.

## What is Strict Mode?

`strict: true` in `tsconfig.json` enables ALL strict type-checking options:

```json
{
  "compilerOptions": {
    "strict": true  // Enables all below:
    // "noImplicitAny": true,
    // "strictNullChecks": true,
    // "strictFunctionTypes": true,
    // "strictBindCallApply": true,
    // "strictPropertyInitialization": true,
    // "noImplicitThis": true,
    // "alwaysStrict": true
  }
}
```

## Why Bother?

**Before strict mode**, our codebase had:
- 847 runtime `undefined` errors in production (6 months)
- Frequent type coercion bugs
- `any` types everywhere (defeating TypeScript's purpose)

## The Migration Strategy

### Phase 1: Gradual Rollout (Don't Flip the Switch)

**Bad idea**: Enable `strict: true` and fix 10,000 errors

**Our approach**: Enable one strict flag at a time

```json
{
  "compilerOptions": {
    "noImplicitAny": true  // Start here
    // Others: false (for now)
  }
}
```

### Phase 2: Fix Module by Module

Create `tsconfig.strict.json`:

```json
{
  "extends": "./tsconfig.json",
  "include": [
    "src/utils/**/*",  // Start with leaf modules
    "src/components/Button/**/*"
  ],
  "compilerOptions": {
    "strict": true
  }
}
```

CI checks this config. Gradually add modules to `include`.

### Phase 3: Common Patterns & Fixes

#### Pattern 1: Implicit Any

**Before**:
```typescript
function getUser(id) {  // Error: Parameter 'id' implicitly has 'any'
  return users.find(u => u.id === id);
}
```

**After**:
```typescript
function getUser(id: string): User | undefined {
  return users.find(u => u.id === id);
}
```

#### Pattern 2: Null Checks

**Before**:
```typescript
function greet(name?: string) {
  return `Hello ${name.toUpperCase()}`;  // Runtime error if undefined
}
```

**After**:
```typescript
function greet(name?: string) {
  if (!name) return "Hello stranger";
  return `Hello ${name.toUpperCase()}`;
}

// Or use optional chaining
function greet(name?: string) {
  return `Hello ${name?.toUpperCase() ?? "stranger"}`;
}
```

#### Pattern 3: Type Assertions vs Type Guards

**Bad** (type assertion):
```typescript
const user = data as User;  // Trust me bro
user.email.toLowerCase();   // Crashes if email is null
```

**Good** (type guard):
```typescript
function isUser(data: unknown): data is User {
  return (
    typeof data === 'object' &&
    data !== null &&
    'email' in data &&
    typeof data.email === 'string'
  );
}

if (isUser(data)) {
  data.email.toLowerCase();  // Safe
}
```

#### Pattern 4: Strict Property Initialization

**Before**:
```typescript
class UserService {
  private apiKey: string;  // Error: not initialized

  async init() {
    this.apiKey = await fetchApiKey();
  }
}
```

**After (Option A)**: Definite assignment assertion
```typescript
class UserService {
  private apiKey!: string;  // I promise to initialize this

  async init() {
    this.apiKey = await fetchApiKey();
  }
}
```

**After (Option B)**: Make it nullable
```typescript
class UserService {
  private apiKey: string | null = null;

  async init() {
    this.apiKey = await fetchApiKey();
  }

  private getApiKey(): string {
    if (!this.apiKey) throw new Error("Not initialized");
    return this.apiKey;
  }
}
```

## Tooling to Help

### 1. Auto-fix with ESLint

```bash
npm install -D @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

`.eslintrc.js`:
```javascript
module.exports = {
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint'],
  rules: {
    '@typescript-eslint/no-explicit-any': 'error',
    '@typescript-eslint/strict-boolean-expressions': 'warn'
  }
};
```

Run: `eslint --fix src/`

### 2. Type Coverage Monitoring

```bash
npm install -D type-coverage
```

Track progress:
```bash
npx type-coverage --detail
```

We started at 64% type coverage, now at 97%.

## Results After 3 Months

**Before strict mode**:
- Runtime errors: 847 (6 months)
- `any` types: 3,247 instances
- Type coverage: 64%

**After strict mode**:
- Runtime errors: 23 (6 months) - 97% reduction
- `any` types: 47 (mostly in migration layer)
- Type coverage: 97%

**Developer experience**:
- Autocomplete works everywhere
- Refactoring confidence: 10x
- Onboarding new devs: easier (types as documentation)

## When NOT to Use Strict Mode

- Prototyping/hackathons (speed > safety)
- Small scripts (< 100 lines)
- Pure JavaScript libraries with no types

## Bottom Line

Strict mode is TypeScript living up to its promise. The migration pain is temporary. The benefits are permanent.

Start new projects with `strict: true` from day one. Migrate legacy codebases gradually.

---

*TypeScript questions? We've seen it all!*
