# exact-stack-angular — Angular / TypeScript / Jest

Stack reference for the EXACT workflow. Import this alongside the methodology guides.

Works with: Claude Code, OpenAI Codex, OpenCode/pi.dev, and other AI-assisted development tools.

## Technical Stack

- **Language:** TypeScript (strict mode, no `any`)
- **Framework:** Angular 17+ (standalone components, signals where appropriate)
- **Testing:** Jest + Angular Testing Library (or TestBed for component tests)
- **Assertions:** Jest matchers (`expect(...).toBe(...)`) or `@testing-library/jest-dom`
- **Linting:** ESLint + Prettier
- **No Barrel files** — import directly; no magic re-exports that obscure dependencies

## Project Structure

```
src/
├── app/
│   ├── app.component.ts
│   ├── app.routes.ts
│   ├── features/
│   │   └── beer/
│   │       ├── beer.component.ts
│   │       ├── beer.component.html
│   │       ├── beer.component.spec.ts
│   │       ├── beer.service.ts
│   │       └── beer.service.spec.ts
│   └── shared/
│       └── models/
│           └── beer.model.ts
├── environments/
└── main.ts
```

## Common Commands

```bash
# Run all tests
npx jest              # or: npm test

# Run single test file
npx jest beer.service.spec.ts

# Run tests matching a name
npx jest --testNamePattern "should return IPAs"

# Run with watch mode (during development)
npx jest --watch

# Run with coverage
npx jest --coverage

# Start dev server → http://localhost:4200
ng serve              # or: npm start

# Build
ng build

# Generate component
ng generate component features/beer/beer

# Generate service
ng generate service features/beer/beer
```

## RED Phase: Skeleton Before Test

Before writing a test for a class that doesn't exist yet, create a minimal skeleton so TypeScript compiles. Use `throw new Error('not implemented')` — this gives a clear test failure, not a compile error.

```typescript
export class BeerValidator {
    static isValid(name: string | null): boolean {
        throw new Error('not implemented');
    }
}
```

```typescript
@Injectable({ providedIn: 'root' })
export class BeerService {
    constructor(private http: HttpClient) {}

    getBeers(): Observable<Beer[]> {
        throw new Error('not implemented');
    }
}
```

Then write the test, run `npx jest` → RED, then implement in GREEN.

## Test Code Patterns

### Service Unit Test (pure function)
```typescript
import { BeerValidator } from './beer-validator';

describe('BeerValidator', () => {
  it('should return true when name is valid', () => {
    const result = BeerValidator.isValid('IPA');
    expect(result).toBe(true);
  });

  it('should return false when name is empty', () => {
    const result = BeerValidator.isValid('');
    expect(result).toBe(false);
  });

  it('should throw when name is null', () => {
    expect(() => BeerValidator.isValid(null as any)).toThrow('Name required');
  });
});
```

### Angular Service Test (with HttpClientTestingModule)
```typescript
import { TestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
import { BeerService } from './beer.service';

describe('BeerService', () => {
  let service: BeerService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [BeerService],
    });
    service = TestBed.inject(BeerService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => httpMock.verify());

  it('should return beers when GET /beers succeeds', () => {
    const mockBeers = [{ name: 'IPA', abv: 6.5, style: 'American' }];

    service.getBeers().subscribe(beers => {
      expect(beers).toHaveLength(1);
      expect(beers[0].name).toBe('IPA');
    });

    const req = httpMock.expectOne('/api/beers');
    expect(req.request.method).toBe('GET');
    req.flush(mockBeers);
  });
});
```

### Angular Component Test (with Testing Library)
```typescript
import { render, screen } from '@testing-library/angular';
import { BeerListComponent } from './beer-list.component';
import { BeerService } from './beer.service';
import { of } from 'rxjs';

describe('BeerListComponent', () => {
  it('should display beer names', async () => {
    const mockService = { getBeers: () => of([{ name: 'IPA', abv: 6.5, style: 'American' }]) };

    await render(BeerListComponent, {
      providers: [{ provide: BeerService, useValue: mockService }],
    });

    expect(screen.getByText('IPA')).toBeInTheDocument();
  });

  it('should show empty state when no beers', async () => {
    const mockService = { getBeers: () => of([]) };

    await render(BeerListComponent, {
      providers: [{ provide: BeerService, useValue: mockService }],
    });

    expect(screen.getByText('No beers available')).toBeInTheDocument();
  });
});
```

### Component Test (with TestBed, no library)
```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { BeerListComponent } from './beer-list.component';

describe('BeerListComponent', () => {
  let fixture: ComponentFixture<BeerListComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [BeerListComponent],
    }).compileComponents();

    fixture = TestBed.createComponent(BeerListComponent);
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(fixture.componentInstance).toBeTruthy();
  });
});
```

## Production Code Patterns

### Model / Interface
```typescript
export interface Beer {
  name: string;
  abv: number;
  style: string;
}
```

### Service
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { Beer } from '../models/beer.model';

@Injectable({ providedIn: 'root' })
export class BeerService {
  constructor(private http: HttpClient) {}

  getBeers(): Observable<Beer[]> {
    return this.http.get<Beer[]>('/api/beers');
  }
}
```

### Standalone Component
```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { BeerService } from './beer.service';
import { Beer } from '../models/beer.model';

@Component({
  selector: 'app-beer-list',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './beer-list.component.html',
})
export class BeerListComponent implements OnInit {
  beers: Beer[] = [];

  constructor(private beerService: BeerService) {}

  ngOnInit(): void {
    this.beerService.getBeers().subscribe(beers => {
      this.beers = beers;
    });
  }
}
```

### Pure Validator (utility function)
```typescript
export class BeerValidator {
  static isValid(name: string | null | undefined): boolean {
    if (name == null) throw new Error('Name required');
    return name.length > 0;
  }
}
```

## TypeScript-Specific Tips for EXACT

- **No `any`** — always type explicitly; use `unknown` + type guard if input type is unclear
- **Strict null checks** — use `string | null` not `string` when null is a valid input
- **Readonly** — prefer `readonly` on arrays and properties in models:
  ```typescript
  export interface Beer {
    readonly name: string;
    readonly abv: number;
  }
  ```
- **Avoid implicit returns** — be explicit in functions that could return `undefined`
- **Use `jest.fn()`** for stubs/mocks instead of full mock services where possible

## Jest Quick Reference

```typescript
// Equality
expect(value).toBe(5);
expect(value).toEqual({ name: 'IPA' });

// Truthy/Falsy
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeUndefined();

// Strings
expect(name).toBe('IPA');
expect(name).toContain('IPA');
expect(name).toMatch(/^IPA/);

// Arrays
expect(beers).toHaveLength(3);
expect(beers).toContainEqual({ name: 'IPA', abv: 6.5, style: 'American' });

// Exceptions
expect(() => service.method()).toThrow('Name required');
expect(() => service.method()).toThrow(Error);

// Async
await expect(promise).resolves.toBe('IPA');
await expect(promise).rejects.toThrow('Error message');
```

## Test File Naming

```
Source: src/app/features/beer/beer.service.ts
Test:   src/app/features/beer/beer.service.spec.ts

Source: src/app/features/beer/beer.component.ts
Test:   src/app/features/beer/beer.component.spec.ts
```
