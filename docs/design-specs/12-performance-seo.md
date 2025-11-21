# Part 12: Performance & SEO Optimization

## Table of Contents

1. [Performance Overview](#1-performance-overview)
2. [Core Web Vitals](#2-core-web-vitals)
3. [Frontend Performance](#3-frontend-performance)
4. [Backend Performance](#4-backend-performance)
5. [Database Optimization](#5-database-optimization)
6. [Caching Strategy](#6-caching-strategy)
7. [Image & Asset Optimization](#7-image--asset-optimization)
8. [SEO Fundamentals](#8-seo-fundamentals)
9. [Structured Data](#9-structured-data)
10. [Technical SEO](#10-technical-seo)
11. [Monitoring & Analytics](#11-monitoring--analytics)

---

## 1. Performance Overview

### 1.1 Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| First Contentful Paint (FCP) | < 1.8s | Lighthouse |
| Largest Contentful Paint (LCP) | < 2.5s | Lighthouse |
| First Input Delay (FID) | < 100ms | Real User Monitoring |
| Cumulative Layout Shift (CLS) | < 0.1 | Lighthouse |
| Time to Interactive (TTI) | < 3.8s | Lighthouse |
| Total Blocking Time (TBT) | < 200ms | Lighthouse |
| Speed Index | < 3.4s | Lighthouse |
| Lighthouse Score | > 90 | All categories |

### 1.2 Performance Budget

```javascript
// performance-budget.json
{
  "timings": [
    { "metric": "first-contentful-paint", "budget": 1800 },
    { "metric": "largest-contentful-paint", "budget": 2500 },
    { "metric": "cumulative-layout-shift", "budget": 0.1 },
    { "metric": "total-blocking-time", "budget": 200 },
    { "metric": "interactive", "budget": 3800 }
  ],
  "resourceSizes": [
    { "resourceType": "script", "budget": 300 },
    { "resourceType": "stylesheet", "budget": 100 },
    { "resourceType": "image", "budget": 500 },
    { "resourceType": "font", "budget": 100 },
    { "resourceType": "total", "budget": 1000 }
  ],
  "resourceCounts": [
    { "resourceType": "script", "budget": 15 },
    { "resourceType": "stylesheet", "budget": 5 },
    { "resourceType": "image", "budget": 20 },
    { "resourceType": "font", "budget": 4 }
  ]
}
```

---

## 2. Core Web Vitals

### 2.1 LCP Optimization

```tsx
// components/OptimizedHero.tsx
import Image from 'next/image';

export function HeroSection() {
  return (
    <section className="hero">
      {/* Preload critical hero image */}
      <Image
        src="/images/hero-bible.webp"
        alt="Open Bible"
        width={1200}
        height={600}
        priority // Preloads this image
        placeholder="blur"
        blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRg..."
        sizes="(max-width: 768px) 100vw, 1200px"
      />
      <h1 className="hero-title">Online Bible Reader</h1>
    </section>
  );
}
```

```html
<!-- Preload critical resources in _document.tsx -->
<Head>
  <!-- Preload LCP image -->
  <link
    rel="preload"
    as="image"
    href="/images/hero-bible.webp"
    type="image/webp"
  />

  <!-- Preload critical fonts -->
  <link
    rel="preload"
    as="font"
    type="font/woff2"
    href="/fonts/inter-var.woff2"
    crossOrigin="anonymous"
  />

  <!-- Preconnect to critical origins -->
  <link rel="preconnect" href="https://api.biblereader.app" />
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="dns-prefetch" href="https://www.google-analytics.com" />
</Head>
```

### 2.2 CLS Prevention

```tsx
// components/VerseContent.tsx
// Prevent layout shift with explicit dimensions

export function VerseContent({ verses, isLoading }) {
  return (
    <div
      className="verse-container"
      style={{
        // Reserve space to prevent CLS
        minHeight: isLoading ? '500px' : 'auto',
      }}
    >
      {isLoading ? (
        // Skeleton with exact dimensions
        <VerseSkeleton count={20} />
      ) : (
        verses.map(verse => (
          <Verse key={verse.id} {...verse} />
        ))
      )}
    </div>
  );
}

// Skeleton component
function VerseSkeleton({ count }) {
  return (
    <div className="space-y-3">
      {Array.from({ length: count }).map((_, i) => (
        <div
          key={i}
          className="animate-pulse flex gap-2"
          style={{ height: '24px' }} // Explicit height
        >
          <div className="w-8 h-5 bg-gray-200 rounded" />
          <div className="flex-1 h-5 bg-gray-200 rounded" />
        </div>
      ))}
    </div>
  );
}
```

```css
/* Prevent CLS with aspect-ratio */
.image-container {
  aspect-ratio: 16 / 9;
  width: 100%;
  background-color: #f3f4f6; /* Placeholder color */
}

/* Reserve space for ads */
.ad-container {
  min-height: 250px;
  width: 100%;
}

/* Font display swap to prevent FOIT */
@font-face {
  font-family: 'Bible Reader';
  src: url('/fonts/bible-reader.woff2') format('woff2');
  font-display: swap;
}
```

### 2.3 FID/INP Optimization

```tsx
// Defer non-critical JavaScript
// next.config.js
module.exports = {
  experimental: {
    optimizeCss: true,
  },
  compiler: {
    removeConsole: process.env.NODE_ENV === 'production',
  },
};

// Lazy load heavy components
import dynamic from 'next/dynamic';

const TTSPlayer = dynamic(() => import('@/components/TTSPlayer'), {
  loading: () => <TTSPlayerSkeleton />,
  ssr: false, // TTS requires browser APIs
});

const StudyToolsPanel = dynamic(() => import('@/components/StudyToolsPanel'), {
  loading: () => <StudyToolsSkeleton />,
});

// Use web workers for heavy computation
// workers/search.worker.ts
self.onmessage = (event) => {
  const { query, verses } = event.data;

  // Heavy search computation off main thread
  const results = verses.filter(verse =>
    verse.text.toLowerCase().includes(query.toLowerCase())
  );

  self.postMessage(results);
};
```

---

## 3. Frontend Performance

### 3.1 Code Splitting

```typescript
// next.config.js
module.exports = {
  // Optimize chunks
  webpack: (config, { isServer }) => {
    if (!isServer) {
      config.optimization.splitChunks = {
        chunks: 'all',
        minSize: 20000,
        maxSize: 244000,
        cacheGroups: {
          // Vendor chunks
          vendor: {
            test: /[\\/]node_modules[\\/]/,
            name: 'vendors',
            chunks: 'all',
          },
          // Bible data chunk
          bibleData: {
            test: /[\\/]data[\\/]bible[\\/]/,
            name: 'bible-data',
            chunks: 'all',
            priority: 10,
          },
          // Common components
          commons: {
            name: 'commons',
            chunks: 'initial',
            minChunks: 2,
          },
        },
      };
    }
    return config;
  },
};
```

### 3.2 Route-Based Code Splitting

```tsx
// pages/bible/[version]/[book]/[chapter].tsx
import dynamic from 'next/dynamic';

// Lazy load features not needed on initial render
const HighlightMenu = dynamic(() => import('@/components/HighlightMenu'));
const ShareDialog = dynamic(() => import('@/components/ShareDialog'));
const NotesPanel = dynamic(() => import('@/components/NotesPanel'));

export default function ChapterPage({ chapter }) {
  const [showNotes, setShowNotes] = useState(false);

  return (
    <div>
      {/* Critical content renders immediately */}
      <ChapterContent chapter={chapter} />

      {/* Non-critical features load on demand */}
      {showNotes && <NotesPanel />}
    </div>
  );
}
```

### 3.3 React Performance Optimizations

```tsx
// Use React.memo for expensive components
import { memo, useMemo, useCallback } from 'react';

export const VerseList = memo(function VerseList({ verses, onVerseClick }) {
  return (
    <div className="verse-list">
      {verses.map(verse => (
        <Verse
          key={verse.id}
          verse={verse}
          onClick={onVerseClick}
        />
      ))}
    </div>
  );
});

// Memoize expensive calculations
export function ReadingProgress({ readingPlan }) {
  const progress = useMemo(() => {
    return calculateProgress(readingPlan);
  }, [readingPlan.completedDays, readingPlan.totalDays]);

  return <ProgressBar value={progress} />;
}

// Stable callback references
export function BibleReader() {
  const [selectedVerse, setSelectedVerse] = useState(null);

  const handleVerseClick = useCallback((verse) => {
    setSelectedVerse(verse);
  }, []);

  return (
    <VerseList
      verses={verses}
      onVerseClick={handleVerseClick}
    />
  );
}
```

### 3.4 Virtual Scrolling for Long Lists

```tsx
// components/VirtualVerseList.tsx
import { useVirtualizer } from '@tanstack/react-virtual';
import { useRef } from 'react';

export function VirtualVerseList({ verses }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: verses.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 80, // Estimated row height
    overscan: 5, // Render 5 extra items above/below viewport
  });

  return (
    <div
      ref={parentRef}
      className="h-[600px] overflow-auto"
    >
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          width: '100%',
          position: 'relative',
        }}
      >
        {virtualizer.getVirtualItems().map(virtualRow => (
          <div
            key={virtualRow.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualRow.size}px`,
              transform: `translateY(${virtualRow.start}px)`,
            }}
          >
            <Verse verse={verses[virtualRow.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## 4. Backend Performance

### 4.1 Response Compression

```typescript
// backend/src/middleware/compression.ts
import compression from 'compression';

export const compressionMiddleware = compression({
  level: 6, // Balance between compression ratio and CPU usage
  threshold: 1024, // Only compress responses > 1KB
  filter: (req, res) => {
    // Don't compress if client doesn't accept
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  },
});

// Brotli compression for better ratios
import shrinkRay from 'shrink-ray-current';

export const brotliMiddleware = shrinkRay({
  brotli: {
    quality: 4, // Fast compression
  },
});
```

### 4.2 Response Streaming

```typescript
// backend/src/routes/bible.ts
import { pipeline } from 'stream/promises';

// Stream large responses
router.get('/bible/:version/full', async (req, res) => {
  const { version } = req.params;

  res.setHeader('Content-Type', 'application/json');
  res.setHeader('Transfer-Encoding', 'chunked');

  // Stream Bible content book by book
  res.write('{"books":[');

  const books = await getBooksList(version);
  let first = true;

  for (const book of books) {
    if (!first) res.write(',');
    first = false;

    const bookData = await getBookContent(version, book.code);
    res.write(JSON.stringify(bookData));
  }

  res.write(']}');
  res.end();
});
```

### 4.3 Query Optimization

```typescript
// backend/src/services/bible-service.ts
export class BibleService {
  // Use specific field selection
  async getChapter(version: string, book: string, chapter: number) {
    // Only select needed fields
    return db.query(`
      SELECT
        v.number,
        v.text,
        v.id
      FROM verses v
      JOIN chapters c ON v.chapter_id = c.id
      JOIN books b ON c.book_id = b.id
      JOIN bible_versions bv ON b.version_id = bv.id
      WHERE bv.code = $1
        AND b.code = $2
        AND c.number = $3
      ORDER BY v.number
    `, [version, book, chapter]);
  }

  // Batch queries for related data
  async getVerseWithUserData(verseIds: string[], userId: string) {
    // Single query with JOIN instead of N+1 queries
    return db.query(`
      SELECT
        v.*,
        h.color as highlight_color,
        b.id as bookmark_id,
        b.title as bookmark_title,
        n.content as note_content
      FROM verses v
      LEFT JOIN highlights h ON v.id = h.verse_id AND h.user_id = $2
      LEFT JOIN bookmarks b ON v.id = b.verse_id AND b.user_id = $2
      LEFT JOIN notes n ON v.id = n.verse_id AND n.user_id = $2
      WHERE v.id = ANY($1)
    `, [verseIds, userId]);
  }
}
```

---

## 5. Database Optimization

### 5.1 Index Strategy

```sql
-- Essential indexes for Bible queries
CREATE INDEX idx_verses_chapter ON verses(chapter_id);
CREATE INDEX idx_verses_version_book_chapter ON verses(version_id, book_code, chapter_number);

-- Full-text search index
CREATE INDEX idx_verses_text_search ON verses USING GIN(to_tsvector('english', text));

-- User data indexes
CREATE INDEX idx_highlights_user_verse ON highlights(user_id, verse_id);
CREATE INDEX idx_bookmarks_user ON bookmarks(user_id);
CREATE INDEX idx_notes_user ON notes(user_id);
CREATE INDEX idx_reading_progress_user_plan ON user_reading_progress(user_id, plan_id);

-- Composite indexes for common queries
CREATE INDEX idx_highlights_user_created ON highlights(user_id, created_at DESC);
CREATE INDEX idx_bookmarks_user_created ON bookmarks(user_id, created_at DESC);

-- Partial index for active users
CREATE INDEX idx_users_active ON users(last_login_at)
WHERE is_active = true;
```

### 5.2 Query Analysis

```sql
-- Analyze slow queries
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT v.*, h.color
FROM verses v
LEFT JOIN highlights h ON v.id = h.verse_id
WHERE v.chapter_id = 'chapter-john-3'
ORDER BY v.number;

-- Check index usage
SELECT
  schemaname,
  tablename,
  indexname,
  idx_scan,
  idx_tup_read,
  idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Find missing indexes
SELECT
  relname,
  seq_scan,
  seq_tup_read,
  idx_scan,
  idx_tup_fetch
FROM pg_stat_user_tables
WHERE seq_scan > idx_scan
ORDER BY seq_tup_read DESC;
```

### 5.3 Connection Pooling

```typescript
// backend/src/config/database.ts
import { Pool } from 'pg';

export const pool = new Pool({
  connectionString: process.env.DATABASE_URL,

  // Pool configuration
  max: 20,                    // Max connections
  min: 5,                     // Min connections
  idleTimeoutMillis: 30000,   // Close idle connections after 30s
  connectionTimeoutMillis: 5000, // Connection timeout

  // Statement timeout
  statement_timeout: 10000,   // 10 second query timeout
});

// Health check
pool.on('error', (err) => {
  console.error('Unexpected pool error:', err);
});
```

---

## 6. Caching Strategy

### 6.1 Multi-Layer Caching

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       CACHING ARCHITECTURE                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   Browser ────► Service Worker ────► CDN ────► API Server ────► DB     │
│                                                                         │
│   Layer 1: Browser Cache                                                │
│   ├── LocalStorage (user preferences)                                   │
│   ├── SessionStorage (temporary state)                                  │
│   └── HTTP Cache (static assets)                                        │
│                                                                         │
│   Layer 2: Service Worker Cache                                         │
│   ├── Bible content (cache-first)                                       │
│   ├── App shell (cache-first)                                           │
│   └── API responses (stale-while-revalidate)                            │
│                                                                         │
│   Layer 3: CDN Cache (CloudFlare)                                       │
│   ├── Static assets (1 year)                                            │
│   ├── Bible content API (1 week)                                        │
│   └── Dynamic pages (1 hour)                                            │
│                                                                         │
│   Layer 4: Application Cache (Redis)                                    │
│   ├── Database query results                                            │
│   ├── User sessions                                                     │
│   └── Rate limiting counters                                            │
│                                                                         │
│   Layer 5: Database Cache                                               │
│   ├── PostgreSQL shared_buffers                                         │
│   └── Query plan cache                                                  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Redis Caching Implementation

```typescript
// backend/src/cache/cache-service.ts
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

// Cache-aside pattern
export async function cacheAside<T>(
  key: string,
  fetchFn: () => Promise<T>,
  ttlSeconds: number = 3600
): Promise<T> {
  // Try cache first
  const cached = await redis.get(key);
  if (cached) {
    return JSON.parse(cached);
  }

  // Fetch from source
  const data = await fetchFn();

  // Store in cache
  await redis.setex(key, ttlSeconds, JSON.stringify(data));

  return data;
}

// Cache Bible content (long TTL)
export async function getBibleChapter(version: string, book: string, chapter: number) {
  const cacheKey = `bible:${version}:${book}:${chapter}`;

  return cacheAside(
    cacheKey,
    () => db.getChapter(version, book, chapter),
    86400 * 7 // 7 days - Bible content doesn't change
  );
}

// Cache user data (short TTL)
export async function getUserHighlights(userId: string) {
  const cacheKey = `user:${userId}:highlights`;

  return cacheAside(
    cacheKey,
    () => db.getUserHighlights(userId),
    300 // 5 minutes
  );
}

// Invalidate cache on update
export async function addHighlight(userId: string, data: HighlightInput) {
  const result = await db.createHighlight(userId, data);

  // Invalidate related caches
  await redis.del(`user:${userId}:highlights`);

  return result;
}
```

### 6.3 HTTP Caching Headers

```typescript
// backend/src/middleware/cache.ts
export function cacheControl(options: CacheOptions) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (options.public) {
      res.setHeader('Cache-Control', `public, max-age=${options.maxAge}, s-maxage=${options.sMaxAge || options.maxAge}`);
    } else {
      res.setHeader('Cache-Control', `private, max-age=${options.maxAge}`);
    }

    if (options.staleWhileRevalidate) {
      res.setHeader('Cache-Control',
        `${res.getHeader('Cache-Control')}, stale-while-revalidate=${options.staleWhileRevalidate}`
      );
    }

    next();
  };
}

// Usage
app.get('/api/bible/:version/:book/:chapter',
  cacheControl({
    public: true,
    maxAge: 86400,      // Browser: 1 day
    sMaxAge: 604800,    // CDN: 7 days
    staleWhileRevalidate: 86400,
  }),
  getBibleChapter
);

app.get('/api/user/highlights',
  cacheControl({
    public: false,
    maxAge: 60,         // 1 minute
  }),
  getUserHighlights
);
```

---

## 7. Image & Asset Optimization

### 7.1 Image Optimization Pipeline

```typescript
// next.config.js
module.exports = {
  images: {
    // Supported formats
    formats: ['image/avif', 'image/webp'],

    // Device sizes for responsive images
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],

    // Image sizes for next/image
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],

    // Image domains
    domains: ['storage.biblereader.app'],

    // Minimize image data
    minimumCacheTTL: 60 * 60 * 24 * 365, // 1 year
  },
};
```

### 7.2 Font Optimization

```tsx
// pages/_app.tsx
import { Inter, Noto_Serif } from 'next/font/google';

// Variable font for UI
const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
});

// Serif font for Bible text
const notoSerif = Noto_Serif({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-noto-serif',
  weight: ['400', '700'],
});

export default function App({ Component, pageProps }) {
  return (
    <main className={`${inter.variable} ${notoSerif.variable}`}>
      <Component {...pageProps} />
    </main>
  );
}
```

```css
/* styles/fonts.css */
/* Self-hosted fonts with subsetting */
@font-face {
  font-family: 'Bible Reader';
  src: url('/fonts/bible-reader-latin.woff2') format('woff2');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
  unicode-range: U+0000-00FF; /* Latin subset */
}

/* Thai font subset */
@font-face {
  font-family: 'Sarabun';
  src: url('/fonts/sarabun-thai.woff2') format('woff2');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
  unicode-range: U+0E00-0E7F; /* Thai subset */
}
```

### 7.3 Asset Loading Strategy

```tsx
// components/OptimizedAssets.tsx
import Script from 'next/script';

export function ThirdPartyScripts() {
  return (
    <>
      {/* Analytics - load after page is interactive */}
      <Script
        src="https://www.googletagmanager.com/gtag/js?id=GA_ID"
        strategy="afterInteractive"
      />

      {/* Non-critical scripts - load when idle */}
      <Script
        src="/scripts/chat-widget.js"
        strategy="lazyOnload"
      />

      {/* Critical scripts - load immediately */}
      <Script
        src="/scripts/critical.js"
        strategy="beforeInteractive"
      />
    </>
  );
}
```

---

## 8. SEO Fundamentals

### 8.1 Meta Tags Structure

```tsx
// components/SEO.tsx
import Head from 'next/head';
import { useRouter } from 'next/router';

interface SEOProps {
  title: string;
  description: string;
  image?: string;
  type?: 'website' | 'article';
  publishedTime?: string;
  modifiedTime?: string;
}

export function SEO({
  title,
  description,
  image = '/images/og-default.jpg',
  type = 'website',
  publishedTime,
  modifiedTime,
}: SEOProps) {
  const router = useRouter();
  const url = `https://biblereader.app${router.asPath}`;
  const fullTitle = `${title} | Online Bible Reader`;

  return (
    <Head>
      {/* Primary Meta Tags */}
      <title>{fullTitle}</title>
      <meta name="title" content={fullTitle} />
      <meta name="description" content={description} />
      <link rel="canonical" href={url} />

      {/* Open Graph / Facebook */}
      <meta property="og:type" content={type} />
      <meta property="og:url" content={url} />
      <meta property="og:title" content={fullTitle} />
      <meta property="og:description" content={description} />
      <meta property="og:image" content={image} />
      <meta property="og:site_name" content="Online Bible Reader" />
      <meta property="og:locale" content={router.locale} />

      {/* Twitter */}
      <meta name="twitter:card" content="summary_large_image" />
      <meta name="twitter:url" content={url} />
      <meta name="twitter:title" content={fullTitle} />
      <meta name="twitter:description" content={description} />
      <meta name="twitter:image" content={image} />

      {/* Article specific */}
      {publishedTime && (
        <meta property="article:published_time" content={publishedTime} />
      )}
      {modifiedTime && (
        <meta property="article:modified_time" content={modifiedTime} />
      )}

      {/* Alternate languages */}
      <link rel="alternate" hrefLang="en" href={`https://biblereader.app/en${router.asPath}`} />
      <link rel="alternate" hrefLang="th" href={`https://biblereader.app/th${router.asPath}`} />
      <link rel="alternate" hrefLang="x-default" href={url} />
    </Head>
  );
}
```

### 8.2 Dynamic Meta Tags for Bible Content

```tsx
// pages/bible/[version]/[book]/[chapter].tsx
import { SEO } from '@/components/SEO';
import { getBookName, getChapterVerseCount } from '@/utils/bible';

export default function ChapterPage({ version, book, chapter, verses }) {
  const bookName = getBookName(book);
  const verseCount = verses.length;

  // Generate meaningful title and description
  const title = `${bookName} ${chapter} - ${version.toUpperCase()}`;
  const description = `Read ${bookName} Chapter ${chapter} (${verseCount} verses) in the ${version.toUpperCase()} translation. "${verses[0]?.text.substring(0, 100)}..."`;

  return (
    <>
      <SEO
        title={title}
        description={description}
        type="article"
      />
      <ChapterContent verses={verses} />
    </>
  );
}

export async function getStaticProps({ params }) {
  const { version, book, chapter } = params;
  const verses = await getChapterVerses(version, book, chapter);

  return {
    props: {
      version,
      book,
      chapter: parseInt(chapter),
      verses,
    },
  };
}
```

---

## 9. Structured Data

### 9.1 JSON-LD Schema

```tsx
// components/StructuredData.tsx
import Head from 'next/head';

interface BibleChapterData {
  version: string;
  book: string;
  chapter: number;
  text: string;
}

export function BibleChapterStructuredData({ data }: { data: BibleChapterData }) {
  const structuredData = {
    '@context': 'https://schema.org',
    '@type': 'Book',
    name: `${data.book} ${data.chapter}`,
    bookEdition: data.version.toUpperCase(),
    inLanguage: getLanguageForVersion(data.version),
    isPartOf: {
      '@type': 'Book',
      name: 'The Holy Bible',
      bookEdition: data.version.toUpperCase(),
    },
    text: data.text.substring(0, 500), // First 500 chars
    url: `https://biblereader.app/bible/${data.version}/${data.book}/${data.chapter}`,
  };

  return (
    <Head>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(structuredData) }}
      />
    </Head>
  );
}

export function WebsiteStructuredData() {
  const structuredData = {
    '@context': 'https://schema.org',
    '@type': 'WebSite',
    name: 'Online Bible Reader',
    alternateName: 'Bible Reader',
    url: 'https://biblereader.app',
    potentialAction: {
      '@type': 'SearchAction',
      target: {
        '@type': 'EntryPoint',
        urlTemplate: 'https://biblereader.app/search?q={search_term_string}',
      },
      'query-input': 'required name=search_term_string',
    },
  };

  return (
    <Head>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(structuredData) }}
      />
    </Head>
  );
}

export function BreadcrumbStructuredData({ items }: { items: BreadcrumbItem[] }) {
  const structuredData = {
    '@context': 'https://schema.org',
    '@type': 'BreadcrumbList',
    itemListElement: items.map((item, index) => ({
      '@type': 'ListItem',
      position: index + 1,
      name: item.name,
      item: item.url,
    })),
  };

  return (
    <Head>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(structuredData) }}
      />
    </Head>
  );
}
```

### 9.2 Organization & FAQ Schema

```tsx
// components/OrganizationSchema.tsx
export function OrganizationStructuredData() {
  const structuredData = {
    '@context': 'https://schema.org',
    '@type': 'Organization',
    name: 'Online Bible Reader',
    url: 'https://biblereader.app',
    logo: 'https://biblereader.app/images/logo.png',
    sameAs: [
      'https://facebook.com/biblereaderapp',
      'https://twitter.com/biblereaderapp',
      'https://instagram.com/biblereaderapp',
    ],
    contactPoint: {
      '@type': 'ContactPoint',
      contactType: 'customer service',
      email: 'support@biblereader.app',
    },
  };

  return (
    <Head>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(structuredData) }}
      />
    </Head>
  );
}

// FAQ Schema for help pages
export function FAQStructuredData({ faqs }: { faqs: FAQ[] }) {
  const structuredData = {
    '@context': 'https://schema.org',
    '@type': 'FAQPage',
    mainEntity: faqs.map(faq => ({
      '@type': 'Question',
      name: faq.question,
      acceptedAnswer: {
        '@type': 'Answer',
        text: faq.answer,
      },
    })),
  };

  return (
    <Head>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(structuredData) }}
      />
    </Head>
  );
}
```

---

## 10. Technical SEO

### 10.1 Sitemap Generation

```typescript
// scripts/generate-sitemap.ts
import { writeFileSync } from 'fs';
import { getAllBiblePaths, getAllReadingPlanPaths } from './data';

async function generateSitemap() {
  const baseUrl = 'https://biblereader.app';

  // Static pages
  const staticPages = [
    '',
    '/about',
    '/reading-plans',
    '/search',
    '/privacy',
    '/terms',
  ];

  // Bible pages (all chapters)
  const biblePaths = await getAllBiblePaths();

  // Reading plan pages
  const planPaths = await getAllReadingPlanPaths();

  const allPaths = [
    ...staticPages,
    ...biblePaths.map(p => `/bible/${p.version}/${p.book}/${p.chapter}`),
    ...planPaths.map(p => `/reading-plans/${p.slug}`),
  ];

  const sitemap = `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:xhtml="http://www.w3.org/1999/xhtml">
${allPaths.map(path => `  <url>
    <loc>${baseUrl}${path}</loc>
    <lastmod>${new Date().toISOString()}</lastmod>
    <changefreq>${getChangeFreq(path)}</changefreq>
    <priority>${getPriority(path)}</priority>
    <xhtml:link rel="alternate" hreflang="en" href="${baseUrl}/en${path}" />
    <xhtml:link rel="alternate" hreflang="th" href="${baseUrl}/th${path}" />
  </url>`).join('\n')}
</urlset>`;

  writeFileSync('public/sitemap.xml', sitemap);
}

function getChangeFreq(path: string): string {
  if (path.startsWith('/bible/')) return 'monthly';
  if (path.startsWith('/reading-plans')) return 'weekly';
  return 'daily';
}

function getPriority(path: string): number {
  if (path === '') return 1.0;
  if (path.startsWith('/bible/')) return 0.8;
  if (path.startsWith('/reading-plans')) return 0.7;
  return 0.5;
}

generateSitemap();
```

### 10.2 Robots.txt

```
# public/robots.txt
User-agent: *
Allow: /

# Block user-specific pages
Disallow: /profile
Disallow: /settings
Disallow: /api/

# Block search results (prevent duplicate content)
Disallow: /search?

# Sitemap
Sitemap: https://biblereader.app/sitemap.xml

# Crawl delay for polite crawling
Crawl-delay: 1
```

### 10.3 URL Structure Best Practices

```typescript
// Canonical URL patterns
const urlPatterns = {
  // Bible reading
  chapter: '/bible/{version}/{book}/{chapter}',
  // Example: /bible/kjv/john/3

  verse: '/bible/{version}/{book}/{chapter}/{verse}',
  // Example: /bible/kjv/john/3/16

  // Search
  search: '/search?q={query}&version={version}',

  // Reading plans
  planList: '/reading-plans',
  planDetail: '/reading-plans/{slug}',
  // Example: /reading-plans/one-year-bible

  // User pages (noindex)
  profile: '/profile',
  settings: '/settings',
};

// URL generation helper
export function generateBibleUrl(version: string, book: string, chapter: number, verse?: number): string {
  const base = `/bible/${version.toLowerCase()}/${book.toLowerCase()}/${chapter}`;
  return verse ? `${base}/${verse}` : base;
}
```

---

## 11. Monitoring & Analytics

### 11.1 Performance Monitoring

```typescript
// src/lib/performance.ts
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

function sendToAnalytics(metric: Metric) {
  const body = JSON.stringify({
    name: metric.name,
    value: metric.value,
    id: metric.id,
    page: window.location.pathname,
  });

  // Use sendBeacon for reliability
  if (navigator.sendBeacon) {
    navigator.sendBeacon('/api/analytics/vitals', body);
  } else {
    fetch('/api/analytics/vitals', {
      body,
      method: 'POST',
      keepalive: true,
    });
  }
}

// Measure all Core Web Vitals
export function measureWebVitals() {
  getCLS(sendToAnalytics);
  getFID(sendToAnalytics);
  getFCP(sendToAnalytics);
  getLCP(sendToAnalytics);
  getTTFB(sendToAnalytics);
}

// Custom performance marks
export function measureCustom(name: string, fn: () => void) {
  performance.mark(`${name}-start`);
  fn();
  performance.mark(`${name}-end`);
  performance.measure(name, `${name}-start`, `${name}-end`);

  const measure = performance.getEntriesByName(name)[0];
  sendToAnalytics({
    name: `custom-${name}`,
    value: measure.duration,
    id: crypto.randomUUID(),
  });
}
```

### 11.2 Lighthouse CI Integration

```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Run Lighthouse
        uses: treosh/lighthouse-ci-action@v10
        with:
          configPath: ./lighthouserc.js
          uploadArtifacts: true
          temporaryPublicStorage: true

      - name: Post results
        uses: actions/github-script@v7
        if: github.event_name == 'pull_request'
        with:
          script: |
            const results = require('./lighthouse-results.json');
            const comment = formatLighthouseResults(results);
            github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: comment,
            });
```

### 11.3 Real User Monitoring (RUM)

```typescript
// src/lib/rum.ts
class RealUserMonitoring {
  private queue: PerformanceEntry[] = [];
  private flushInterval = 5000; // 5 seconds

  constructor() {
    this.observePerformance();
    this.startFlushInterval();
  }

  private observePerformance() {
    // Observe various performance metrics
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        this.queue.push(entry);
      }
    });

    // Observe different entry types
    observer.observe({ entryTypes: ['navigation', 'resource', 'longtask', 'paint'] });
  }

  private startFlushInterval() {
    setInterval(() => this.flush(), this.flushInterval);

    // Flush on page hide
    document.addEventListener('visibilitychange', () => {
      if (document.visibilityState === 'hidden') {
        this.flush();
      }
    });
  }

  private flush() {
    if (this.queue.length === 0) return;

    const entries = this.queue.splice(0);

    navigator.sendBeacon('/api/analytics/rum', JSON.stringify({
      entries: entries.map(e => ({
        name: e.name,
        type: e.entryType,
        duration: e.duration,
        startTime: e.startTime,
      })),
      url: window.location.href,
      timestamp: Date.now(),
    }));
  }
}

export const rum = new RealUserMonitoring();
```

---

## Summary

This performance and SEO optimization guide ensures the Online Bible Reader application delivers excellent user experience and search visibility:

### Performance Achievements

| Metric | Target | Strategy |
|--------|--------|----------|
| LCP | < 2.5s | Image optimization, preloading, CDN |
| FID | < 100ms | Code splitting, web workers |
| CLS | < 0.1 | Explicit dimensions, font loading |
| TTI | < 3.8s | Lazy loading, tree shaking |

### SEO Implementation

| Feature | Implementation |
|---------|---------------|
| Meta Tags | Dynamic per-page SEO component |
| Structured Data | JSON-LD for Bible content, breadcrumbs |
| Sitemap | Auto-generated with all Bible chapters |
| URL Structure | Clean, semantic URLs |
| i18n SEO | hreflang tags, localized content |

---

**Document Version**: 1.0.0
**Last Updated**: 2025-01-14
**Author**: Performance Team
