# Part 9: Testing Strategy

## Table of Contents

1. [Overview](#1-overview)
2. [Testing Pyramid](#2-testing-pyramid)
3. [Unit Testing](#3-unit-testing)
4. [Integration Testing](#4-integration-testing)
5. [End-to-End Testing](#5-end-to-end-testing)
6. [API Testing](#6-api-testing)
7. [Performance Testing](#7-performance-testing)
8. [Accessibility Testing](#8-accessibility-testing)
9. [Visual Regression Testing](#9-visual-regression-testing)
10. [Mobile Testing](#10-mobile-testing)
11. [Test Data Management](#11-test-data-management)
12. [CI/CD Integration](#12-cicd-integration)

---

## 1. Overview

### 1.1 Testing Goals

- **Reliability**: Ensure application functions correctly under all conditions
- **Quality**: Maintain high code quality and prevent regressions
- **Coverage**: Achieve minimum 80% code coverage for critical paths
- **Speed**: Fast feedback loop for developers
- **Confidence**: Enable safe refactoring and feature additions

### 1.2 Testing Tools Stack

```
┌─────────────────────────────────────────────────────────────────┐
│                      TESTING TOOLS STACK                        │
├─────────────────────────────────────────────────────────────────┤
│  Unit Testing:        Jest / Vitest                             │
│  Component Testing:   React Testing Library / Vue Test Utils    │
│  E2E Testing:         Playwright / Cypress                      │
│  API Testing:         Supertest / Postman/Newman                │
│  Performance:         k6 / Lighthouse CI                        │
│  Accessibility:       axe-core / pa11y                          │
│  Visual Regression:   Percy / Chromatic                         │
│  Mocking:             MSW (Mock Service Worker)                 │
│  Coverage:            Istanbul / c8                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Testing Pyramid

```
                          ┌─────────┐
                         /   E2E    \           ~10%
                        /   Tests    \          (Critical flows)
                       ───────────────
                      /  Integration  \         ~20%
                     /     Tests       \        (API, Components)
                    ─────────────────────
                   /     Unit Tests      \      ~70%
                  /   (Functions, Utils)  \     (Fast, isolated)
                 ─────────────────────────────

Speed:     Fast ◄─────────────────────────────► Slow
Cost:      Low  ◄─────────────────────────────► High
Confidence: Low ◄─────────────────────────────► High
```

### 2.1 Coverage Targets

| Test Type | Coverage Target | Run Frequency |
|-----------|-----------------|---------------|
| Unit Tests | 80%+ | Every commit |
| Integration Tests | 60%+ | Every PR |
| E2E Tests | Critical paths 100% | Pre-deployment |
| Performance Tests | Key metrics | Weekly + Pre-release |
| Accessibility Tests | WCAG 2.1 AA | Every PR |

---

## 3. Unit Testing

### 3.1 Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    include: ['src/**/*.{test,spec}.{js,ts,tsx}'],
    exclude: ['node_modules', 'dist', 'e2e'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      exclude: [
        'node_modules/',
        'src/test/',
        '**/*.d.ts',
        '**/*.config.*',
        '**/types/*',
      ],
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 75,
        statements: 80,
      },
    },
    mockReset: true,
    restoreMocks: true,
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

### 3.2 Test Setup

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom';
import { cleanup } from '@testing-library/react';
import { afterEach, beforeAll, afterAll } from 'vitest';
import { server } from './mocks/server';

// Start MSW server
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => {
  cleanup();
  server.resetHandlers();
});
afterAll(() => server.close());

// Mock window.matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: (query: string) => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: () => {},
    removeListener: () => {},
    addEventListener: () => {},
    removeEventListener: () => {},
    dispatchEvent: () => {},
  }),
});

// Mock IntersectionObserver
class MockIntersectionObserver {
  observe = () => null;
  disconnect = () => null;
  unobserve = () => null;
}
window.IntersectionObserver = MockIntersectionObserver as any;

// Mock Speech Synthesis
window.speechSynthesis = {
  speak: () => {},
  cancel: () => {},
  pause: () => {},
  resume: () => {},
  getVoices: () => [],
  speaking: false,
  paused: false,
  pending: false,
  onvoiceschanged: null,
  addEventListener: () => {},
  removeEventListener: () => {},
  dispatchEvent: () => true,
};
```

### 3.3 Unit Test Examples

```typescript
// src/utils/bible.test.ts
import { describe, it, expect } from 'vitest';
import {
  parseVerseReference,
  formatVerseReference,
  getBookAbbreviation,
  calculateReadingProgress,
} from './bible';

describe('Bible Utilities', () => {
  describe('parseVerseReference', () => {
    it('should parse simple reference', () => {
      expect(parseVerseReference('John 3:16')).toEqual({
        book: 'John',
        chapter: 3,
        verse: 16,
      });
    });

    it('should parse reference with verse range', () => {
      expect(parseVerseReference('Romans 8:28-30')).toEqual({
        book: 'Romans',
        chapter: 8,
        verseStart: 28,
        verseEnd: 30,
      });
    });

    it('should parse books with numbers', () => {
      expect(parseVerseReference('1 Corinthians 13:4')).toEqual({
        book: '1 Corinthians',
        chapter: 13,
        verse: 4,
      });
    });

    it('should throw error for invalid reference', () => {
      expect(() => parseVerseReference('Invalid')).toThrow('Invalid verse reference');
    });
  });

  describe('formatVerseReference', () => {
    it('should format single verse', () => {
      expect(formatVerseReference('John', 3, 16)).toBe('John 3:16');
    });

    it('should format verse range', () => {
      expect(formatVerseReference('Romans', 8, 28, 30)).toBe('Romans 8:28-30');
    });
  });

  describe('getBookAbbreviation', () => {
    it('should return correct abbreviation', () => {
      expect(getBookAbbreviation('Genesis')).toBe('Gen');
      expect(getBookAbbreviation('Matthew')).toBe('Matt');
      expect(getBookAbbreviation('1 Corinthians')).toBe('1 Cor');
    });
  });

  describe('calculateReadingProgress', () => {
    it('should calculate correct percentage', () => {
      expect(calculateReadingProgress(50, 100)).toBe(50);
      expect(calculateReadingProgress(75, 300)).toBe(25);
    });

    it('should return 0 for empty reading', () => {
      expect(calculateReadingProgress(0, 100)).toBe(0);
    });

    it('should cap at 100%', () => {
      expect(calculateReadingProgress(150, 100)).toBe(100);
    });
  });
});
```

```typescript
// src/hooks/useBibleReader.test.ts
import { renderHook, act, waitFor } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { useBibleReader } from './useBibleReader';
import { QueryClientProvider, QueryClient } from '@tanstack/react-query';

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
};

describe('useBibleReader', () => {
  it('should load chapter content', async () => {
    const { result } = renderHook(
      () => useBibleReader('kjv', 'john', 3),
      { wrapper: createWrapper() }
    );

    expect(result.current.isLoading).toBe(true);

    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
    });

    expect(result.current.chapter).toBeDefined();
    expect(result.current.verses).toHaveLength(36);
  });

  it('should navigate to next chapter', async () => {
    const { result } = renderHook(
      () => useBibleReader('kjv', 'john', 3),
      { wrapper: createWrapper() }
    );

    await waitFor(() => !result.current.isLoading);

    act(() => {
      result.current.goToNextChapter();
    });

    await waitFor(() => {
      expect(result.current.currentChapter).toBe(4);
    });
  });

  it('should handle verse selection', async () => {
    const { result } = renderHook(
      () => useBibleReader('kjv', 'john', 3),
      { wrapper: createWrapper() }
    );

    await waitFor(() => !result.current.isLoading);

    act(() => {
      result.current.selectVerse(16);
    });

    expect(result.current.selectedVerse).toBe(16);
  });
});
```

### 3.4 Component Testing

```typescript
// src/components/VerseDisplay.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { VerseDisplay } from './VerseDisplay';

const mockVerse = {
  id: 'verse-john-3-16',
  number: 16,
  text: 'For God so loved the world, that he gave his only begotten Son...',
  highlights: [],
  bookmarks: [],
  notes: [],
};

describe('VerseDisplay', () => {
  it('should render verse number and text', () => {
    render(<VerseDisplay verse={mockVerse} />);

    expect(screen.getByText('16')).toBeInTheDocument();
    expect(screen.getByText(/For God so loved/)).toBeInTheDocument();
  });

  it('should call onSelect when verse is clicked', () => {
    const onSelect = vi.fn();
    render(<VerseDisplay verse={mockVerse} onSelect={onSelect} />);

    fireEvent.click(screen.getByText(/For God so loved/));

    expect(onSelect).toHaveBeenCalledWith(mockVerse);
  });

  it('should show highlight indicator when verse is highlighted', () => {
    const highlightedVerse = {
      ...mockVerse,
      highlights: [{ color: 'yellow' }],
    };

    render(<VerseDisplay verse={highlightedVerse} />);

    expect(screen.getByTestId('highlight-indicator')).toHaveClass('bg-yellow-200');
  });

  it('should show bookmark icon when verse is bookmarked', () => {
    const bookmarkedVerse = {
      ...mockVerse,
      bookmarks: [{ id: 'bm-1', title: 'Favorite' }],
    };

    render(<VerseDisplay verse={bookmarkedVerse} />);

    expect(screen.getByTestId('bookmark-icon')).toBeInTheDocument();
  });

  it('should be accessible', () => {
    render(<VerseDisplay verse={mockVerse} />);

    const verseElement = screen.getByRole('article');
    expect(verseElement).toHaveAttribute('aria-label', 'Verse 16');
  });
});
```

---

## 4. Integration Testing

### 4.1 API Integration Tests

```typescript
// src/api/__tests__/bible-api.integration.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { createTestServer } from '@/test/utils/test-server';
import { BibleAPI } from '../bible-api';

describe('Bible API Integration', () => {
  let server: ReturnType<typeof createTestServer>;
  let api: BibleAPI;

  beforeAll(async () => {
    server = createTestServer();
    await server.start();
    api = new BibleAPI(server.url);
  });

  afterAll(async () => {
    await server.stop();
  });

  describe('GET /bible/versions', () => {
    it('should return available Bible versions', async () => {
      const versions = await api.getVersions();

      expect(versions).toBeInstanceOf(Array);
      expect(versions.length).toBeGreaterThan(0);
      expect(versions[0]).toHaveProperty('id');
      expect(versions[0]).toHaveProperty('name');
      expect(versions[0]).toHaveProperty('language');
    });
  });

  describe('GET /bible/:version/:book/:chapter', () => {
    it('should return chapter content', async () => {
      const chapter = await api.getChapter('kjv', 'john', 3);

      expect(chapter).toHaveProperty('book', 'John');
      expect(chapter).toHaveProperty('chapter', 3);
      expect(chapter.verses).toHaveLength(36);
    });

    it('should return 404 for invalid book', async () => {
      await expect(api.getChapter('kjv', 'invalid', 1)).rejects.toThrow('Not Found');
    });

    it('should return 404 for invalid chapter', async () => {
      await expect(api.getChapter('kjv', 'john', 999)).rejects.toThrow('Not Found');
    });
  });

  describe('GET /search', () => {
    it('should return search results', async () => {
      const results = await api.search('love', { limit: 10 });

      expect(results.items).toBeInstanceOf(Array);
      expect(results.total).toBeGreaterThan(0);
      expect(results.items[0]).toHaveProperty('verse');
      expect(results.items[0]).toHaveProperty('text');
    });

    it('should respect limit parameter', async () => {
      const results = await api.search('faith', { limit: 5 });

      expect(results.items).toHaveLength(5);
    });

    it('should filter by version', async () => {
      const results = await api.search('grace', { version: 'kjv' });

      results.items.forEach(item => {
        expect(item.version).toBe('kjv');
      });
    });
  });
});
```

### 4.2 Database Integration Tests

```typescript
// backend/src/__tests__/database.integration.test.ts
import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest';
import { Pool } from 'pg';
import { UserRepository } from '../repositories/user-repository';
import { HighlightRepository } from '../repositories/highlight-repository';
import { createTestDatabase, dropTestDatabase } from './utils/test-db';

describe('Database Integration', () => {
  let pool: Pool;
  let userRepo: UserRepository;
  let highlightRepo: HighlightRepository;
  let testUserId: string;

  beforeAll(async () => {
    pool = await createTestDatabase();
    userRepo = new UserRepository(pool);
    highlightRepo = new HighlightRepository(pool);
  });

  afterAll(async () => {
    await dropTestDatabase(pool);
    await pool.end();
  });

  beforeEach(async () => {
    // Clean up and create test user
    await pool.query('TRUNCATE users, highlights, bookmarks, notes CASCADE');
    const user = await userRepo.create({
      email: 'test@example.com',
      displayName: 'Test User',
      passwordHash: 'hashedpassword',
    });
    testUserId = user.id;
  });

  describe('UserRepository', () => {
    it('should create user', async () => {
      const user = await userRepo.create({
        email: 'new@example.com',
        displayName: 'New User',
        passwordHash: 'hash',
      });

      expect(user.id).toBeDefined();
      expect(user.email).toBe('new@example.com');
    });

    it('should find user by email', async () => {
      const user = await userRepo.findByEmail('test@example.com');

      expect(user).toBeDefined();
      expect(user?.displayName).toBe('Test User');
    });

    it('should return null for non-existent email', async () => {
      const user = await userRepo.findByEmail('nonexistent@example.com');

      expect(user).toBeNull();
    });
  });

  describe('HighlightRepository', () => {
    it('should create highlight', async () => {
      const highlight = await highlightRepo.create({
        userId: testUserId,
        verseId: 'verse-john-3-16',
        color: 'yellow',
      });

      expect(highlight.id).toBeDefined();
      expect(highlight.color).toBe('yellow');
    });

    it('should get highlights by user', async () => {
      await highlightRepo.create({
        userId: testUserId,
        verseId: 'verse-john-3-16',
        color: 'yellow',
      });
      await highlightRepo.create({
        userId: testUserId,
        verseId: 'verse-romans-8-28',
        color: 'green',
      });

      const highlights = await highlightRepo.findByUser(testUserId);

      expect(highlights).toHaveLength(2);
    });

    it('should delete highlight', async () => {
      const highlight = await highlightRepo.create({
        userId: testUserId,
        verseId: 'verse-john-3-16',
        color: 'yellow',
      });

      await highlightRepo.delete(highlight.id, testUserId);

      const found = await highlightRepo.findById(highlight.id);
      expect(found).toBeNull();
    });
  });
});
```

---

## 5. End-to-End Testing

### 5.1 Playwright Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html', { open: 'never' }],
    ['json', { outputFile: 'test-results/results.json' }],
    ['junit', { outputFile: 'test-results/junit.xml' }],
  ],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'on-first-retry',
  },
  projects: [
    // Desktop browsers
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    // Mobile browsers
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 13'] },
    },
    // Tablet
    {
      name: 'iPad',
      use: { ...devices['iPad Pro 11'] },
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120000,
  },
});
```

### 5.2 E2E Test Examples

```typescript
// e2e/bible-reading.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Bible Reading', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('should display home page with book selection', async ({ page }) => {
    await expect(page.getByRole('heading', { name: /Online Bible Reader/i })).toBeVisible();
    await expect(page.getByText('Genesis')).toBeVisible();
    await expect(page.getByText('Matthew')).toBeVisible();
  });

  test('should navigate to specific chapter', async ({ page }) => {
    // Select book
    await page.click('text=John');

    // Select chapter
    await page.click('text=Chapter 3');

    // Verify navigation
    await expect(page).toHaveURL(/\/bible\/kjv\/john\/3/);
    await expect(page.getByRole('heading', { name: 'John 3' })).toBeVisible();
  });

  test('should display verses with numbers', async ({ page }) => {
    await page.goto('/bible/kjv/john/3');

    // Check verse numbers
    const verses = page.locator('[data-testid="verse"]');
    await expect(verses).toHaveCount(36);

    // Check specific verse
    await expect(page.getByText(/For God so loved the world/)).toBeVisible();
  });

  test('should navigate between chapters', async ({ page }) => {
    await page.goto('/bible/kjv/john/3');

    // Go to next chapter
    await page.click('[aria-label="Next chapter"]');
    await expect(page).toHaveURL(/\/bible\/kjv\/john\/4/);

    // Go to previous chapter
    await page.click('[aria-label="Previous chapter"]');
    await expect(page).toHaveURL(/\/bible\/kjv\/john\/3/);
  });

  test('should support parallel view', async ({ page }) => {
    await page.goto('/bible/kjv/john/3');

    // Enable parallel view
    await page.click('[aria-label="Parallel view"]');

    // Select second version
    await page.selectOption('[data-testid="second-version"]', 'thai');

    // Verify both versions displayed
    await expect(page.locator('[data-testid="primary-text"]')).toBeVisible();
    await expect(page.locator('[data-testid="secondary-text"]')).toBeVisible();
  });
});
```

```typescript
// e2e/user-authentication.spec.ts
import { test, expect } from '@playwright/test';

test.describe('User Authentication', () => {
  test('should register new user', async ({ page }) => {
    await page.goto('/register');

    await page.fill('[name="email"]', 'newuser@example.com');
    await page.fill('[name="displayName"]', 'New User');
    await page.fill('[name="password"]', 'SecurePass123!');
    await page.fill('[name="confirmPassword"]', 'SecurePass123!');

    await page.click('button[type="submit"]');

    // Should redirect to home or verification page
    await expect(page).toHaveURL(/\/(home|verify-email)/);
  });

  test('should login existing user', async ({ page }) => {
    await page.goto('/login');

    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'TestPass123!');

    await page.click('button[type="submit"]');

    await expect(page).toHaveURL('/');
    await expect(page.getByText('Welcome back')).toBeVisible();
  });

  test('should show error for invalid credentials', async ({ page }) => {
    await page.goto('/login');

    await page.fill('[name="email"]', 'wrong@example.com');
    await page.fill('[name="password"]', 'wrongpassword');

    await page.click('button[type="submit"]');

    await expect(page.getByText(/invalid email or password/i)).toBeVisible();
  });

  test('should logout user', async ({ page }) => {
    // Login first
    await page.goto('/login');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'TestPass123!');
    await page.click('button[type="submit"]');

    // Logout
    await page.click('[aria-label="User menu"]');
    await page.click('text=Logout');

    await expect(page.getByText('Login')).toBeVisible();
  });
});
```

```typescript
// e2e/study-tools.spec.ts
import { test, expect } from '@playwright/test';
import { loginAsTestUser } from './utils/auth';

test.describe('Study Tools', () => {
  test.beforeEach(async ({ page }) => {
    await loginAsTestUser(page);
    await page.goto('/bible/kjv/john/3');
  });

  test('should highlight verse', async ({ page }) => {
    // Select verse
    await page.click('[data-verse="16"]');

    // Open highlight menu
    await page.click('[aria-label="Highlight"]');

    // Select color
    await page.click('[data-color="yellow"]');

    // Verify highlight applied
    await expect(page.locator('[data-verse="16"]')).toHaveClass(/bg-yellow/);
  });

  test('should add bookmark', async ({ page }) => {
    // Select verse
    await page.click('[data-verse="16"]');

    // Click bookmark button
    await page.click('[aria-label="Bookmark"]');

    // Add title
    await page.fill('[name="bookmarkTitle"]', 'Favorite Verse');
    await page.click('button:text("Save")');

    // Verify bookmark icon visible
    await expect(page.locator('[data-verse="16"] [data-testid="bookmark-icon"]')).toBeVisible();
  });

  test('should add note to verse', async ({ page }) => {
    // Select verse
    await page.click('[data-verse="16"]');

    // Click note button
    await page.click('[aria-label="Add note"]');

    // Write note
    await page.fill('[name="noteContent"]', 'This verse speaks about God\'s love for humanity.');
    await page.click('button:text("Save")');

    // Verify note indicator
    await expect(page.locator('[data-verse="16"] [data-testid="note-icon"]')).toBeVisible();

    // Open and verify note content
    await page.click('[data-verse="16"] [data-testid="note-icon"]');
    await expect(page.getByText(/God's love for humanity/)).toBeVisible();
  });

  test('should search Bible', async ({ page }) => {
    // Open search
    await page.click('[aria-label="Search"]');

    // Enter search term
    await page.fill('[name="searchQuery"]', 'love');
    await page.press('[name="searchQuery"]', 'Enter');

    // Verify results
    await expect(page.getByText(/search results/i)).toBeVisible();
    await expect(page.locator('[data-testid="search-result"]').first()).toBeVisible();
  });
});
```

```typescript
// e2e/reading-plans.spec.ts
import { test, expect } from '@playwright/test';
import { loginAsTestUser } from './utils/auth';

test.describe('Reading Plans', () => {
  test.beforeEach(async ({ page }) => {
    await loginAsTestUser(page);
  });

  test('should display available reading plans', async ({ page }) => {
    await page.goto('/reading-plans');

    await expect(page.getByText('One Year Bible')).toBeVisible();
    await expect(page.getByText('90 Days New Testament')).toBeVisible();
  });

  test('should start new reading plan', async ({ page }) => {
    await page.goto('/reading-plans');

    // Select plan
    await page.click('text=One Year Bible');

    // Start plan
    await page.click('button:text("Start Plan")');

    // Verify plan started
    await expect(page.getByText('Day 1')).toBeVisible();
    await expect(page.getByText('Genesis 1-2')).toBeVisible();
  });

  test('should mark reading as complete', async ({ page }) => {
    await page.goto('/reading-plans/my-plans');

    // Click on today's reading
    await page.click('[data-testid="today-reading"]');

    // Read content
    await page.goto('/bible/kjv/genesis/1');

    // Mark as complete
    await page.click('button:text("Mark Complete")');

    // Verify progress updated
    await page.goto('/reading-plans/my-plans');
    await expect(page.locator('[data-testid="progress-bar"]')).toHaveAttribute('aria-valuenow', '1');
  });

  test('should show reading streak', async ({ page }) => {
    await page.goto('/profile');

    await expect(page.getByText(/reading streak/i)).toBeVisible();
    await expect(page.locator('[data-testid="streak-count"]')).toBeVisible();
  });
});
```

### 5.3 E2E Test Utilities

```typescript
// e2e/utils/auth.ts
import { Page } from '@playwright/test';

export async function loginAsTestUser(page: Page) {
  await page.goto('/login');
  await page.fill('[name="email"]', 'e2e-test@example.com');
  await page.fill('[name="password"]', 'E2ETestPass123!');
  await page.click('button[type="submit"]');
  await page.waitForURL('/');
}

export async function createTestUser(page: Page, email: string) {
  await page.goto('/register');
  await page.fill('[name="email"]', email);
  await page.fill('[name="displayName"]', 'E2E Test User');
  await page.fill('[name="password"]', 'E2ETestPass123!');
  await page.fill('[name="confirmPassword"]', 'E2ETestPass123!');
  await page.click('button[type="submit"]');
  await page.waitForURL(/\/(home|verify-email)/);
}
```

```typescript
// e2e/utils/fixtures.ts
import { test as base } from '@playwright/test';

type TestFixtures = {
  authenticatedPage: Page;
};

export const test = base.extend<TestFixtures>({
  authenticatedPage: async ({ page }, use) => {
    // Login before test
    await page.goto('/login');
    await page.fill('[name="email"]', 'e2e-test@example.com');
    await page.fill('[name="password"]', 'E2ETestPass123!');
    await page.click('button[type="submit"]');
    await page.waitForURL('/');

    // Use the authenticated page
    await use(page);

    // Logout after test
    await page.click('[aria-label="User menu"]');
    await page.click('text=Logout');
  },
});

export { expect } from '@playwright/test';
```

---

## 6. API Testing

### 6.1 Supertest Configuration

```typescript
// backend/src/__tests__/api/setup.ts
import { createServer } from '../../server';
import { Pool } from 'pg';
import supertest from 'supertest';

export async function setupTestApp() {
  const pool = new Pool({
    connectionString: process.env.TEST_DATABASE_URL,
  });

  const app = await createServer({ pool });
  const request = supertest(app);

  return { app, request, pool };
}

export async function cleanupTestApp(pool: Pool) {
  await pool.query('TRUNCATE users, highlights, bookmarks, notes CASCADE');
  await pool.end();
}

export async function getAuthToken(request: supertest.SuperTest<supertest.Test>) {
  const response = await request
    .post('/api/v1/auth/login')
    .send({ email: 'test@example.com', password: 'TestPass123!' });

  return response.body.accessToken;
}
```

### 6.2 API Test Examples

```typescript
// backend/src/__tests__/api/bible.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { setupTestApp, cleanupTestApp } from './setup';

describe('Bible API', () => {
  let request: any;
  let pool: any;

  beforeAll(async () => {
    const setup = await setupTestApp();
    request = setup.request;
    pool = setup.pool;
  });

  afterAll(async () => {
    await cleanupTestApp(pool);
  });

  describe('GET /api/v1/bible/versions', () => {
    it('should return list of Bible versions', async () => {
      const response = await request.get('/api/v1/bible/versions');

      expect(response.status).toBe(200);
      expect(response.body).toHaveProperty('data');
      expect(Array.isArray(response.body.data)).toBe(true);
    });
  });

  describe('GET /api/v1/bible/:version/books', () => {
    it('should return list of books', async () => {
      const response = await request.get('/api/v1/bible/kjv/books');

      expect(response.status).toBe(200);
      expect(response.body.data).toHaveLength(66);
    });

    it('should return 404 for invalid version', async () => {
      const response = await request.get('/api/v1/bible/invalid/books');

      expect(response.status).toBe(404);
    });
  });

  describe('GET /api/v1/bible/:version/:book/:chapter', () => {
    it('should return chapter content', async () => {
      const response = await request.get('/api/v1/bible/kjv/john/3');

      expect(response.status).toBe(200);
      expect(response.body.data).toHaveProperty('book', 'John');
      expect(response.body.data).toHaveProperty('chapter', 3);
      expect(response.body.data.verses).toHaveLength(36);
    });

    it('should include verse text', async () => {
      const response = await request.get('/api/v1/bible/kjv/john/3');

      const verse16 = response.body.data.verses.find((v: any) => v.number === 16);
      expect(verse16.text).toContain('For God so loved');
    });
  });

  describe('GET /api/v1/search', () => {
    it('should search verses', async () => {
      const response = await request
        .get('/api/v1/search')
        .query({ q: 'love', limit: 10 });

      expect(response.status).toBe(200);
      expect(response.body.data.items.length).toBeLessThanOrEqual(10);
    });

    it('should require search query', async () => {
      const response = await request.get('/api/v1/search');

      expect(response.status).toBe(400);
    });
  });
});
```

```typescript
// backend/src/__tests__/api/user.test.ts
import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest';
import { setupTestApp, cleanupTestApp, getAuthToken } from './setup';

describe('User API', () => {
  let request: any;
  let pool: any;
  let authToken: string;

  beforeAll(async () => {
    const setup = await setupTestApp();
    request = setup.request;
    pool = setup.pool;
  });

  afterAll(async () => {
    await cleanupTestApp(pool);
  });

  beforeEach(async () => {
    authToken = await getAuthToken(request);
  });

  describe('GET /api/v1/user/highlights', () => {
    it('should return user highlights', async () => {
      const response = await request
        .get('/api/v1/user/highlights')
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(200);
      expect(Array.isArray(response.body.data)).toBe(true);
    });

    it('should return 401 without auth', async () => {
      const response = await request.get('/api/v1/user/highlights');

      expect(response.status).toBe(401);
    });
  });

  describe('POST /api/v1/user/highlights', () => {
    it('should create highlight', async () => {
      const response = await request
        .post('/api/v1/user/highlights')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          verseId: 'verse-john-3-16',
          color: 'yellow',
        });

      expect(response.status).toBe(201);
      expect(response.body.data).toHaveProperty('id');
      expect(response.body.data.color).toBe('yellow');
    });

    it('should validate color', async () => {
      const response = await request
        .post('/api/v1/user/highlights')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          verseId: 'verse-john-3-16',
          color: 'invalid-color',
        });

      expect(response.status).toBe(400);
    });
  });

  describe('DELETE /api/v1/user/highlights/:id', () => {
    it('should delete highlight', async () => {
      // Create highlight first
      const createResponse = await request
        .post('/api/v1/user/highlights')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ verseId: 'verse-john-3-16', color: 'yellow' });

      const highlightId = createResponse.body.data.id;

      // Delete it
      const response = await request
        .delete(`/api/v1/user/highlights/${highlightId}`)
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(204);
    });

    it('should return 404 for non-existent highlight', async () => {
      const response = await request
        .delete('/api/v1/user/highlights/non-existent-id')
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(404);
    });
  });
});
```

---

## 7. Performance Testing

### 7.1 k6 Load Tests

```javascript
// k6/load-test.js
import http from 'k6/http';
import { check, sleep, group } from 'k6';
import { Rate, Trend } from 'k6/metrics';

// Custom metrics
const errorRate = new Rate('errors');
const chapterLoadTime = new Trend('chapter_load_time');
const searchTime = new Trend('search_time');

export const options = {
  stages: [
    { duration: '1m', target: 50 },   // Ramp up
    { duration: '5m', target: 100 },  // Stay at 100
    { duration: '2m', target: 200 },  // Spike
    { duration: '5m', target: 100 },  // Back to normal
    { duration: '1m', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1000'],
    errors: ['rate<0.01'],
    chapter_load_time: ['p(95)<300'],
    search_time: ['p(95)<500'],
  },
};

const BASE_URL = __ENV.BASE_URL || 'http://localhost:4000';

const books = ['genesis', 'exodus', 'psalms', 'matthew', 'john', 'romans'];

export default function () {
  group('Bible Reading', () => {
    const book = books[Math.floor(Math.random() * books.length)];
    const chapter = Math.floor(Math.random() * 20) + 1;

    const start = Date.now();
    const response = http.get(`${BASE_URL}/api/v1/bible/kjv/${book}/${chapter}`);
    chapterLoadTime.add(Date.now() - start);

    check(response, {
      'chapter loaded': (r) => r.status === 200,
      'has verses': (r) => JSON.parse(r.body).data.verses.length > 0,
    });
    errorRate.add(response.status !== 200);
  });

  sleep(Math.random() * 2 + 1);

  group('Search', () => {
    const queries = ['love', 'faith', 'hope', 'grace', 'peace'];
    const query = queries[Math.floor(Math.random() * queries.length)];

    const start = Date.now();
    const response = http.get(`${BASE_URL}/api/v1/search?q=${query}&limit=20`);
    searchTime.add(Date.now() - start);

    check(response, {
      'search completed': (r) => r.status === 200,
      'has results': (r) => JSON.parse(r.body).data.items.length > 0,
    });
    errorRate.add(response.status !== 200);
  });

  sleep(Math.random() * 3 + 1);
}
```

### 7.2 Lighthouse CI Configuration

```javascript
// lighthouserc.js
module.exports = {
  ci: {
    collect: {
      url: [
        'http://localhost:3000/',
        'http://localhost:3000/bible/kjv/john/3',
        'http://localhost:3000/search',
        'http://localhost:3000/reading-plans',
      ],
      numberOfRuns: 3,
      settings: {
        preset: 'desktop',
      },
    },
    assert: {
      assertions: {
        'categories:performance': ['error', { minScore: 0.9 }],
        'categories:accessibility': ['error', { minScore: 0.9 }],
        'categories:best-practices': ['error', { minScore: 0.9 }],
        'categories:seo': ['error', { minScore: 0.9 }],
        'first-contentful-paint': ['error', { maxNumericValue: 2000 }],
        'largest-contentful-paint': ['error', { maxNumericValue: 2500 }],
        'cumulative-layout-shift': ['error', { maxNumericValue: 0.1 }],
        'total-blocking-time': ['error', { maxNumericValue: 300 }],
      },
    },
    upload: {
      target: 'temporary-public-storage',
    },
  },
};
```

---

## 8. Accessibility Testing

### 8.1 axe-core Integration

```typescript
// src/test/accessibility.test.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('Accessibility', () => {
  test('home page should have no accessibility violations', async ({ page }) => {
    await page.goto('/');

    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21aa'])
      .analyze();

    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test('bible reader should have no accessibility violations', async ({ page }) => {
    await page.goto('/bible/kjv/john/3');

    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21aa'])
      .exclude('[data-testid="tts-player"]') // Exclude known issues
      .analyze();

    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test('search page should have no accessibility violations', async ({ page }) => {
    await page.goto('/search');

    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21aa'])
      .analyze();

    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test('login form should be accessible', async ({ page }) => {
    await page.goto('/login');

    // Check form labels
    const emailInput = page.locator('[name="email"]');
    const passwordInput = page.locator('[name="password"]');

    await expect(emailInput).toHaveAttribute('aria-label');
    await expect(passwordInput).toHaveAttribute('aria-label');

    // Check button accessibility
    const submitButton = page.locator('button[type="submit"]');
    await expect(submitButton).toHaveAttribute('aria-label');
  });
});
```

### 8.2 Manual Accessibility Checklist

```markdown
## Accessibility Testing Checklist

### Keyboard Navigation
- [ ] All interactive elements are keyboard accessible
- [ ] Focus order follows logical reading order
- [ ] Focus indicators are visible
- [ ] No keyboard traps
- [ ] Skip links are available

### Screen Reader
- [ ] All images have alt text
- [ ] Form inputs have associated labels
- [ ] ARIA landmarks are properly used
- [ ] Dynamic content updates are announced
- [ ] Error messages are announced

### Visual
- [ ] Color contrast meets WCAG AA (4.5:1 for text)
- [ ] Information not conveyed by color alone
- [ ] Text can be resized to 200% without loss
- [ ] No content requires horizontal scrolling at 320px

### Multimedia
- [ ] Audio controls are available
- [ ] TTS controls are accessible
- [ ] Pause/stop controls for auto-playing content

### Forms
- [ ] All form fields have labels
- [ ] Required fields are indicated
- [ ] Error messages are clear and associated
- [ ] Form validation is accessible
```

---

## 9. Visual Regression Testing

### 9.1 Percy Configuration

```yaml
# .percy.yml
version: 2
snapshot:
  widths:
    - 375   # Mobile
    - 768   # Tablet
    - 1280  # Desktop
  min-height: 1024
  percy-css: |
    /* Hide dynamic content */
    [data-testid="current-time"] { visibility: hidden; }
    [data-testid="streak-count"] { visibility: hidden; }
```

### 9.2 Visual Regression Tests

```typescript
// e2e/visual-regression.spec.ts
import { test, expect } from '@playwright/test';
import percySnapshot from '@percy/playwright';

test.describe('Visual Regression', () => {
  test('home page', async ({ page }) => {
    await page.goto('/');
    await page.waitForLoadState('networkidle');
    await percySnapshot(page, 'Home Page');
  });

  test('bible reader - light theme', async ({ page }) => {
    await page.goto('/bible/kjv/john/3');
    await page.waitForLoadState('networkidle');
    await percySnapshot(page, 'Bible Reader - Light Theme');
  });

  test('bible reader - dark theme', async ({ page }) => {
    await page.goto('/bible/kjv/john/3');
    await page.click('[aria-label="Toggle dark mode"]');
    await page.waitForLoadState('networkidle');
    await percySnapshot(page, 'Bible Reader - Dark Theme');
  });

  test('search results', async ({ page }) => {
    await page.goto('/search?q=love');
    await page.waitForLoadState('networkidle');
    await percySnapshot(page, 'Search Results');
  });

  test('reading plans', async ({ page }) => {
    await page.goto('/reading-plans');
    await page.waitForLoadState('networkidle');
    await percySnapshot(page, 'Reading Plans');
  });

  test('verse with highlight', async ({ page }) => {
    // Login and add highlight
    await page.goto('/bible/kjv/john/3');
    // ... setup highlighted verse
    await percySnapshot(page, 'Verse with Highlight');
  });
});
```

---

## 10. Mobile Testing

### 10.1 Device-Specific Tests

```typescript
// e2e/mobile.spec.ts
import { test, expect, devices } from '@playwright/test';

const iPhone = devices['iPhone 13'];
const android = devices['Pixel 5'];

test.describe('Mobile - iPhone', () => {
  test.use({ ...iPhone });

  test('should show mobile navigation', async ({ page }) => {
    await page.goto('/');

    // Desktop nav should be hidden
    await expect(page.locator('[data-testid="desktop-nav"]')).toBeHidden();

    // Mobile menu button should be visible
    await expect(page.locator('[aria-label="Open menu"]')).toBeVisible();
  });

  test('should open mobile menu', async ({ page }) => {
    await page.goto('/');

    await page.click('[aria-label="Open menu"]');

    await expect(page.locator('[data-testid="mobile-menu"]')).toBeVisible();
    await expect(page.getByText('Home')).toBeVisible();
    await expect(page.getByText('Reading Plans')).toBeVisible();
  });

  test('should support swipe gestures', async ({ page }) => {
    await page.goto('/bible/kjv/john/3');

    // Simulate swipe left (next chapter)
    await page.locator('[data-testid="chapter-content"]').evaluate((el) => {
      el.dispatchEvent(new TouchEvent('touchstart', {
        touches: [{ clientX: 300, clientY: 200 }],
      }));
      el.dispatchEvent(new TouchEvent('touchend', {
        changedTouches: [{ clientX: 50, clientY: 200 }],
      }));
    });

    await expect(page).toHaveURL(/\/bible\/kjv\/john\/4/);
  });

  test('should have touch-friendly buttons', async ({ page }) => {
    await page.goto('/bible/kjv/john/3');

    const buttons = page.locator('button');
    const count = await buttons.count();

    for (let i = 0; i < count; i++) {
      const button = buttons.nth(i);
      const box = await button.boundingBox();

      if (box) {
        // Minimum touch target size: 44x44px
        expect(box.width).toBeGreaterThanOrEqual(44);
        expect(box.height).toBeGreaterThanOrEqual(44);
      }
    }
  });
});

test.describe('Mobile - Android', () => {
  test.use({ ...android });

  test('should work on Android', async ({ page }) => {
    await page.goto('/');
    await expect(page.getByRole('heading', { name: /Bible Reader/i })).toBeVisible();
  });
});
```

### 10.2 PWA Testing

```typescript
// e2e/pwa.spec.ts
import { test, expect } from '@playwright/test';

test.describe('PWA Features', () => {
  test('should have valid manifest', async ({ page }) => {
    await page.goto('/');

    const manifestLink = page.locator('link[rel="manifest"]');
    await expect(manifestLink).toHaveAttribute('href', '/manifest.json');

    const response = await page.request.get('/manifest.json');
    expect(response.status()).toBe(200);

    const manifest = await response.json();
    expect(manifest.name).toBe('Online Bible Reader');
    expect(manifest.start_url).toBe('/');
    expect(manifest.display).toBe('standalone');
  });

  test('should register service worker', async ({ page }) => {
    await page.goto('/');

    // Wait for service worker to register
    await page.waitForFunction(() => {
      return navigator.serviceWorker.controller !== null;
    });

    const swRegistration = await page.evaluate(async () => {
      const registration = await navigator.serviceWorker.getRegistration();
      return registration !== undefined;
    });

    expect(swRegistration).toBe(true);
  });

  test('should work offline after caching', async ({ page, context }) => {
    // First visit to cache
    await page.goto('/bible/kjv/john/3');
    await page.waitForLoadState('networkidle');

    // Go offline
    await context.setOffline(true);

    // Navigate to cached page
    await page.goto('/bible/kjv/john/3');

    // Should still work
    await expect(page.getByText('John 3')).toBeVisible();
  });
});
```

---

## 11. Test Data Management

### 11.1 Test Fixtures

```typescript
// src/test/fixtures/bible.ts
export const testVerses = {
  john316: {
    id: 'verse-john-3-16',
    book: 'John',
    chapter: 3,
    number: 16,
    text: 'For God so loved the world, that he gave his only begotten Son, that whosoever believeth in him should not perish, but have everlasting life.',
  },
  romans828: {
    id: 'verse-romans-8-28',
    book: 'Romans',
    chapter: 8,
    number: 28,
    text: 'And we know that all things work together for good to them that love God, to them who are the called according to his purpose.',
  },
};

export const testChapter = {
  version: 'kjv',
  book: 'John',
  chapter: 3,
  verses: [
    { number: 1, text: 'There was a man of the Pharisees...' },
    { number: 2, text: 'The same came to Jesus by night...' },
    // ... more verses
    { number: 16, text: testVerses.john316.text },
    // ... more verses
  ],
};

export const testUser = {
  id: 'user-test-123',
  email: 'test@example.com',
  displayName: 'Test User',
  preferences: {
    theme: 'light',
    fontSize: 'medium',
    defaultVersion: 'kjv',
  },
};
```

### 11.2 Database Seeding

```typescript
// backend/src/test/seed.ts
import { Pool } from 'pg';
import { hashPassword } from '../utils/auth';

export async function seedTestDatabase(pool: Pool) {
  // Create test user
  const passwordHash = await hashPassword('TestPass123!');
  const { rows: [user] } = await pool.query(`
    INSERT INTO users (email, display_name, password_hash)
    VALUES ($1, $2, $3)
    RETURNING id
  `, ['test@example.com', 'Test User', passwordHash]);

  // Create test highlights
  await pool.query(`
    INSERT INTO highlights (user_id, verse_id, color)
    VALUES ($1, $2, $3)
  `, [user.id, 'verse-john-3-16', 'yellow']);

  // Create test bookmark
  await pool.query(`
    INSERT INTO bookmarks (user_id, verse_id, title)
    VALUES ($1, $2, $3)
  `, [user.id, 'verse-john-3-16', 'Favorite Verse']);

  // Create test note
  await pool.query(`
    INSERT INTO notes (user_id, verse_id, content)
    VALUES ($1, $2, $3)
  `, [user.id, 'verse-john-3-16', 'This is my favorite verse']);

  return { userId: user.id };
}

export async function cleanTestDatabase(pool: Pool) {
  await pool.query(`
    TRUNCATE users, highlights, bookmarks, notes,
             reading_plans, user_reading_progress CASCADE
  `);
}
```

---

## 12. CI/CD Integration

### 12.1 GitHub Actions Test Workflow

```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm run test:unit -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  integration-tests:
    name: Integration Tests
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run migrations
        run: npm run db:migrate
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test

      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test
          REDIS_URL: redis://localhost:6379

  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: [unit-tests, integration-tests]

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright
        run: npx playwright install --with-deps

      - name: Build application
        run: npm run build

      - name: Run E2E tests
        run: npm run test:e2e

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/

  accessibility-tests:
    name: Accessibility Tests
    runs-on: ubuntu-latest
    needs: [unit-tests]

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright
        run: npx playwright install --with-deps

      - name: Build application
        run: npm run build

      - name: Run accessibility tests
        run: npm run test:a11y

  lighthouse:
    name: Lighthouse CI
    runs-on: ubuntu-latest
    needs: [unit-tests]

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npm run build

      - name: Run Lighthouse CI
        run: |
          npm install -g @lhci/cli
          lhci autorun
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}
```

### 12.2 Test Scripts (package.json)

```json
{
  "scripts": {
    "test": "npm run test:unit && npm run test:integration",
    "test:unit": "vitest run",
    "test:unit:watch": "vitest",
    "test:unit:coverage": "vitest run --coverage",
    "test:integration": "vitest run --config vitest.integration.config.ts",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:debug": "playwright test --debug",
    "test:a11y": "playwright test --project=accessibility",
    "test:visual": "percy exec -- playwright test --project=visual",
    "test:api": "vitest run --config vitest.api.config.ts",
    "test:perf": "k6 run k6/load-test.js",
    "test:lighthouse": "lhci autorun"
  }
}
```

---

## Summary

This testing strategy ensures comprehensive coverage of the Online Bible Reader application:

| Test Type | Tools | Coverage Target | Frequency |
|-----------|-------|-----------------|-----------|
| Unit | Vitest, RTL | 80%+ | Every commit |
| Integration | Supertest, Test DB | 60%+ | Every PR |
| E2E | Playwright | Critical paths | Pre-deploy |
| API | Supertest | All endpoints | Every PR |
| Performance | k6, Lighthouse | Key metrics | Weekly |
| Accessibility | axe-core | WCAG 2.1 AA | Every PR |
| Visual | Percy | All pages | Every PR |
| Mobile | Playwright | All viewports | Every PR |

---

**Document Version**: 1.0.0
**Last Updated**: 2025-01-14
**Author**: QA Team
