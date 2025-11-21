# Part 13: Analytics & Tracking Implementation

## Table of Contents

1. [Overview](#1-overview)
2. [Analytics Architecture](#2-analytics-architecture)
3. [Event Tracking Schema](#3-event-tracking-schema)
4. [Google Analytics 4 Implementation](#4-google-analytics-4-implementation)
5. [Custom Analytics System](#5-custom-analytics-system)
6. [User Behavior Tracking](#6-user-behavior-tracking)
7. [Reading Analytics](#7-reading-analytics)
8. [Dashboard & Reporting](#8-dashboard--reporting)
9. [Privacy & Compliance](#9-privacy--compliance)
10. [A/B Testing](#10-ab-testing)

---

## 1. Overview

### 1.1 Analytics Goals

- **User Engagement**: Track reading habits, session duration, return rates
- **Feature Usage**: Understand which features users value most
- **Content Performance**: Identify popular Bible passages and reading plans
- **Conversion Tracking**: Monitor user registration, premium upgrades
- **Technical Performance**: Monitor page load times, errors, crashes
- **Business Intelligence**: Support data-driven product decisions

### 1.2 Key Metrics (KPIs)

| Category | Metric | Target |
|----------|--------|--------|
| Engagement | Daily Active Users (DAU) | Growth 5% MoM |
| Engagement | Session Duration | > 5 minutes |
| Engagement | Pages per Session | > 10 |
| Retention | Day 1 Retention | > 40% |
| Retention | Day 7 Retention | > 25% |
| Retention | Day 30 Retention | > 15% |
| Reading | Chapters Read/User/Day | > 2 |
| Reading | Reading Plan Completion | > 30% |
| Features | TTS Usage Rate | > 20% |
| Features | Highlight Creation Rate | > 10% |
| Technical | Lighthouse Score | > 90 |
| Technical | Error Rate | < 0.1% |

---

## 2. Analytics Architecture

### 2.1 Data Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      ANALYTICS DATA FLOW                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   User Actions                                                          │
│       │                                                                 │
│       ▼                                                                 │
│   ┌─────────────────┐                                                   │
│   │ Event Collector │ ──► Queue (in-memory)                             │
│   └────────┬────────┘                                                   │
│            │                                                            │
│            ├──────────────────────┬─────────────────────┐               │
│            │                      │                     │               │
│            ▼                      ▼                     ▼               │
│   ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐       │
│   │  Google         │   │  Custom         │   │   Error         │       │
│   │  Analytics 4    │   │  Analytics API  │   │   Tracking      │       │
│   │  (gtag.js)      │   │  (Internal)     │   │   (Sentry)      │       │
│   └────────┬────────┘   └────────┬────────┘   └────────┬────────┘       │
│            │                     │                     │                │
│            ▼                     ▼                     ▼                │
│   ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐       │
│   │  BigQuery       │   │   PostgreSQL    │   │    Sentry       │       │
│   │  (Storage)      │   │   (Storage)     │   │    Dashboard    │       │
│   └────────┬────────┘   └────────┬────────┘   └─────────────────┘       │
│            │                     │                                      │
│            └──────────┬──────────┘                                      │
│                       │                                                 │
│                       ▼                                                 │
│              ┌─────────────────┐                                        │
│              │   Metabase /    │                                        │
│              │   Looker Studio │                                        │
│              │   (Dashboard)   │                                        │
│              └─────────────────┘                                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Technology Stack

```typescript
// Analytics technology stack
const analyticsStack = {
  // Primary Analytics
  ga4: {
    name: 'Google Analytics 4',
    purpose: 'User behavior, acquisition, demographics',
    dataRetention: '14 months',
  },

  // Custom Analytics
  custom: {
    name: 'Internal Analytics API',
    database: 'PostgreSQL + TimescaleDB',
    purpose: 'Detailed reading analytics, custom metrics',
    dataRetention: '24 months',
  },

  // Error Tracking
  errorTracking: {
    name: 'Sentry',
    purpose: 'Error monitoring, performance issues',
    dataRetention: '90 days',
  },

  // Product Analytics
  productAnalytics: {
    name: 'Mixpanel / Amplitude',
    purpose: 'Funnel analysis, cohort analysis',
    dataRetention: '12 months',
  },

  // Visualization
  dashboard: {
    name: 'Metabase / Looker Studio',
    purpose: 'Reporting, visualization',
  },
};
```

---

## 3. Event Tracking Schema

### 3.1 Event Naming Convention

```typescript
// Event naming: category_action_detail
// Examples:
// - bible_chapter_view
// - study_highlight_create
// - reading_plan_start
// - auth_login_success

interface AnalyticsEvent {
  event_name: string;           // snake_case event name
  event_category: EventCategory;
  event_action: string;
  event_label?: string;
  event_value?: number;
  timestamp: string;            // ISO 8601
  session_id: string;
  user_id?: string;             // If authenticated
  device_id: string;            // Anonymous device ID
  properties: Record<string, any>;
}

type EventCategory =
  | 'bible'           // Bible reading events
  | 'study'           // Study tools (highlights, notes, bookmarks)
  | 'reading_plan'    // Reading plan events
  | 'search'          // Search events
  | 'tts'             // Text-to-speech events
  | 'auth'            // Authentication events
  | 'user'            // User profile events
  | 'navigation'      // Navigation events
  | 'engagement'      // Engagement events
  | 'error'           // Error events
  | 'performance';    // Performance events
```

### 3.2 Event Definitions

```typescript
// events/bible-events.ts
export const bibleEvents = {
  // Chapter viewing
  CHAPTER_VIEW: {
    name: 'bible_chapter_view',
    category: 'bible',
    properties: {
      version: 'string',      // 'kjv', 'niv', etc.
      book: 'string',         // 'john', 'genesis', etc.
      chapter: 'number',      // 1-150
      view_mode: 'string',    // 'single', 'parallel'
      referrer: 'string',     // How user got here
    },
  },

  // Verse interaction
  VERSE_SELECT: {
    name: 'bible_verse_select',
    category: 'bible',
    properties: {
      version: 'string',
      book: 'string',
      chapter: 'number',
      verse: 'number',
      action: 'string',       // 'highlight', 'bookmark', 'note', 'copy', 'share'
    },
  },

  // Version change
  VERSION_CHANGE: {
    name: 'bible_version_change',
    category: 'bible',
    properties: {
      from_version: 'string',
      to_version: 'string',
    },
  },

  // Navigation
  NAVIGATION: {
    name: 'bible_navigation',
    category: 'bible',
    properties: {
      from_book: 'string',
      from_chapter: 'number',
      to_book: 'string',
      to_chapter: 'number',
      method: 'string',       // 'next', 'prev', 'menu', 'search', 'link'
    },
  },
};

// events/study-events.ts
export const studyEvents = {
  // Highlight events
  HIGHLIGHT_CREATE: {
    name: 'study_highlight_create',
    category: 'study',
    properties: {
      verse_reference: 'string',  // 'john:3:16'
      color: 'string',
    },
  },

  HIGHLIGHT_DELETE: {
    name: 'study_highlight_delete',
    category: 'study',
    properties: {
      verse_reference: 'string',
    },
  },

  // Bookmark events
  BOOKMARK_CREATE: {
    name: 'study_bookmark_create',
    category: 'study',
    properties: {
      verse_reference: 'string',
      has_title: 'boolean',
    },
  },

  // Note events
  NOTE_CREATE: {
    name: 'study_note_create',
    category: 'study',
    properties: {
      verse_reference: 'string',
      content_length: 'number',
    },
  },

  NOTE_EDIT: {
    name: 'study_note_edit',
    category: 'study',
    properties: {
      verse_reference: 'string',
      content_length: 'number',
    },
  },
};

// events/reading-plan-events.ts
export const readingPlanEvents = {
  PLAN_VIEW: {
    name: 'reading_plan_view',
    category: 'reading_plan',
    properties: {
      plan_id: 'string',
      plan_name: 'string',
    },
  },

  PLAN_START: {
    name: 'reading_plan_start',
    category: 'reading_plan',
    properties: {
      plan_id: 'string',
      plan_name: 'string',
      plan_duration_days: 'number',
    },
  },

  PLAN_DAY_COMPLETE: {
    name: 'reading_plan_day_complete',
    category: 'reading_plan',
    properties: {
      plan_id: 'string',
      day_number: 'number',
      total_days: 'number',
      progress_percent: 'number',
    },
  },

  PLAN_COMPLETE: {
    name: 'reading_plan_complete',
    category: 'reading_plan',
    properties: {
      plan_id: 'string',
      plan_name: 'string',
      days_to_complete: 'number',
      started_at: 'string',
    },
  },

  PLAN_ABANDON: {
    name: 'reading_plan_abandon',
    category: 'reading_plan',
    properties: {
      plan_id: 'string',
      progress_percent: 'number',
      days_active: 'number',
    },
  },
};

// events/tts-events.ts
export const ttsEvents = {
  TTS_START: {
    name: 'tts_playback_start',
    category: 'tts',
    properties: {
      verse_reference: 'string',
      voice: 'string',
      speed: 'number',
    },
  },

  TTS_PAUSE: {
    name: 'tts_playback_pause',
    category: 'tts',
    properties: {
      duration_played: 'number',
      verses_played: 'number',
    },
  },

  TTS_COMPLETE: {
    name: 'tts_playback_complete',
    category: 'tts',
    properties: {
      duration_total: 'number',
      verses_played: 'number',
      chapter: 'string',
    },
  },

  TTS_SPEED_CHANGE: {
    name: 'tts_speed_change',
    category: 'tts',
    properties: {
      from_speed: 'number',
      to_speed: 'number',
    },
  },
};

// events/search-events.ts
export const searchEvents = {
  SEARCH_PERFORM: {
    name: 'search_perform',
    category: 'search',
    properties: {
      query: 'string',
      query_length: 'number',
      filters: 'object',
      results_count: 'number',
    },
  },

  SEARCH_RESULT_CLICK: {
    name: 'search_result_click',
    category: 'search',
    properties: {
      query: 'string',
      result_position: 'number',
      verse_reference: 'string',
    },
  },

  SEARCH_NO_RESULTS: {
    name: 'search_no_results',
    category: 'search',
    properties: {
      query: 'string',
      filters: 'object',
    },
  },
};
```

---

## 4. Google Analytics 4 Implementation

### 4.1 GA4 Setup

```typescript
// lib/analytics/ga4.ts
declare global {
  interface Window {
    dataLayer: any[];
    gtag: (...args: any[]) => void;
  }
}

const GA_MEASUREMENT_ID = process.env.NEXT_PUBLIC_GA_ID;

// Initialize GA4
export function initGA() {
  if (typeof window === 'undefined' || !GA_MEASUREMENT_ID) return;

  window.dataLayer = window.dataLayer || [];
  window.gtag = function gtag() {
    window.dataLayer.push(arguments);
  };

  window.gtag('js', new Date());
  window.gtag('config', GA_MEASUREMENT_ID, {
    page_path: window.location.pathname,
    send_page_view: false, // We'll send manually for SPA
    cookie_flags: 'SameSite=None;Secure',
  });
}

// Track page view
export function trackPageView(url: string, title?: string) {
  if (!window.gtag) return;

  window.gtag('event', 'page_view', {
    page_path: url,
    page_title: title,
    page_location: window.location.href,
  });
}

// Track custom event
export function trackEvent(
  eventName: string,
  eventParams: Record<string, any> = {}
) {
  if (!window.gtag) return;

  window.gtag('event', eventName, {
    ...eventParams,
    timestamp: new Date().toISOString(),
  });
}

// Set user properties
export function setUserProperties(properties: Record<string, any>) {
  if (!window.gtag) return;

  window.gtag('set', 'user_properties', properties);
}

// Set user ID for logged-in users
export function setUserId(userId: string | null) {
  if (!window.gtag) return;

  window.gtag('config', GA_MEASUREMENT_ID, {
    user_id: userId,
  });
}
```

### 4.2 GA4 React Integration

```tsx
// components/GoogleAnalytics.tsx
import Script from 'next/script';
import { useRouter } from 'next/router';
import { useEffect } from 'react';
import { initGA, trackPageView } from '@/lib/analytics/ga4';

const GA_MEASUREMENT_ID = process.env.NEXT_PUBLIC_GA_ID;

export function GoogleAnalytics() {
  const router = useRouter();

  useEffect(() => {
    initGA();
  }, []);

  useEffect(() => {
    const handleRouteChange = (url: string) => {
      trackPageView(url);
    };

    router.events.on('routeChangeComplete', handleRouteChange);
    return () => {
      router.events.off('routeChangeComplete', handleRouteChange);
    };
  }, [router.events]);

  if (!GA_MEASUREMENT_ID) return null;

  return (
    <>
      <Script
        src={`https://www.googletagmanager.com/gtag/js?id=${GA_MEASUREMENT_ID}`}
        strategy="afterInteractive"
      />
      <Script id="ga4-init" strategy="afterInteractive">
        {`
          window.dataLayer = window.dataLayer || [];
          function gtag(){dataLayer.push(arguments);}
          gtag('js', new Date());
          gtag('config', '${GA_MEASUREMENT_ID}', {
            send_page_view: false
          });
        `}
      </Script>
    </>
  );
}
```

### 4.3 GA4 Custom Dimensions

```typescript
// GA4 custom dimensions configuration
const customDimensions = {
  // User-scoped dimensions
  user_type: 'user',              // 'free', 'premium'
  preferred_version: 'string',    // 'kjv', 'niv'
  ui_language: 'string',          // 'en', 'th'
  theme_preference: 'string',     // 'light', 'dark', 'system'

  // Event-scoped dimensions
  bible_version: 'string',
  bible_book: 'string',
  bible_testament: 'string',      // 'old', 'new'
  reading_plan_name: 'string',
  tts_voice: 'string',
  highlight_color: 'string',
  search_filters: 'string',

  // Session-scoped dimensions
  entry_page_type: 'string',      // 'home', 'bible', 'plan', 'search'
  referrer_type: 'string',        // 'organic', 'direct', 'social', 'email'
};

// Set custom dimensions
export function setCustomDimensions(dimensions: Partial<typeof customDimensions>) {
  trackEvent('set_custom_dimensions', dimensions);
}
```

---

## 5. Custom Analytics System

### 5.1 Analytics API

```typescript
// backend/src/analytics/analytics-service.ts
import { Pool } from 'pg';

interface AnalyticsEvent {
  event_name: string;
  event_category: string;
  user_id?: string;
  session_id: string;
  device_id: string;
  properties: Record<string, any>;
  timestamp: string;
  user_agent: string;
  ip_hash: string; // Hashed for privacy
}

export class AnalyticsService {
  constructor(private pool: Pool) {}

  async trackEvent(event: AnalyticsEvent): Promise<void> {
    await this.pool.query(`
      INSERT INTO analytics_events (
        event_name, event_category, user_id, session_id,
        device_id, properties, timestamp, user_agent, ip_hash
      ) VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9)
    `, [
      event.event_name,
      event.event_category,
      event.user_id,
      event.session_id,
      event.device_id,
      JSON.stringify(event.properties),
      event.timestamp,
      event.user_agent,
      event.ip_hash,
    ]);
  }

  async batchTrackEvents(events: AnalyticsEvent[]): Promise<void> {
    const values = events.map((e, i) => {
      const offset = i * 9;
      return `($${offset + 1}, $${offset + 2}, $${offset + 3}, $${offset + 4}, $${offset + 5}, $${offset + 6}, $${offset + 7}, $${offset + 8}, $${offset + 9})`;
    }).join(', ');

    const params = events.flatMap(e => [
      e.event_name, e.event_category, e.user_id, e.session_id,
      e.device_id, JSON.stringify(e.properties), e.timestamp,
      e.user_agent, e.ip_hash,
    ]);

    await this.pool.query(`
      INSERT INTO analytics_events (
        event_name, event_category, user_id, session_id,
        device_id, properties, timestamp, user_agent, ip_hash
      ) VALUES ${values}
    `, params);
  }

  // Aggregate queries
  async getDailyActiveUsers(startDate: Date, endDate: Date): Promise<number[]> {
    const result = await this.pool.query(`
      SELECT DATE(timestamp) as date, COUNT(DISTINCT COALESCE(user_id, device_id)) as dau
      FROM analytics_events
      WHERE timestamp BETWEEN $1 AND $2
      GROUP BY DATE(timestamp)
      ORDER BY date
    `, [startDate, endDate]);

    return result.rows;
  }

  async getPopularChapters(limit: number = 10): Promise<any[]> {
    const result = await this.pool.query(`
      SELECT
        properties->>'book' as book,
        (properties->>'chapter')::int as chapter,
        COUNT(*) as view_count
      FROM analytics_events
      WHERE event_name = 'bible_chapter_view'
        AND timestamp > NOW() - INTERVAL '30 days'
      GROUP BY properties->>'book', properties->>'chapter'
      ORDER BY view_count DESC
      LIMIT $1
    `, [limit]);

    return result.rows;
  }
}
```

### 5.2 Database Schema for Analytics

```sql
-- Create analytics database schema
CREATE EXTENSION IF NOT EXISTS timescaledb;

-- Main events table (TimescaleDB hypertable for time-series)
CREATE TABLE analytics_events (
  id BIGSERIAL,
  event_name VARCHAR(100) NOT NULL,
  event_category VARCHAR(50) NOT NULL,
  user_id UUID,
  session_id VARCHAR(100) NOT NULL,
  device_id VARCHAR(100) NOT NULL,
  properties JSONB DEFAULT '{}',
  timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  user_agent TEXT,
  ip_hash VARCHAR(64),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Convert to hypertable for efficient time-series queries
SELECT create_hypertable('analytics_events', 'timestamp');

-- Create indexes
CREATE INDEX idx_events_name ON analytics_events (event_name);
CREATE INDEX idx_events_category ON analytics_events (event_category);
CREATE INDEX idx_events_user ON analytics_events (user_id) WHERE user_id IS NOT NULL;
CREATE INDEX idx_events_session ON analytics_events (session_id);
CREATE INDEX idx_events_timestamp ON analytics_events (timestamp DESC);
CREATE INDEX idx_events_properties ON analytics_events USING GIN (properties);

-- Continuous aggregates for common queries
CREATE MATERIALIZED VIEW daily_active_users
WITH (timescaledb.continuous) AS
SELECT
  time_bucket('1 day', timestamp) AS day,
  COUNT(DISTINCT COALESCE(user_id::text, device_id)) AS dau,
  COUNT(*) AS total_events
FROM analytics_events
GROUP BY time_bucket('1 day', timestamp);

-- Refresh policy
SELECT add_continuous_aggregate_policy('daily_active_users',
  start_offset => INTERVAL '3 days',
  end_offset => INTERVAL '1 hour',
  schedule_interval => INTERVAL '1 hour');

-- Reading analytics aggregation
CREATE MATERIALIZED VIEW reading_stats_daily
WITH (timescaledb.continuous) AS
SELECT
  time_bucket('1 day', timestamp) AS day,
  properties->>'version' AS bible_version,
  properties->>'book' AS book,
  COUNT(*) AS chapter_views,
  COUNT(DISTINCT COALESCE(user_id::text, device_id)) AS unique_readers
FROM analytics_events
WHERE event_name = 'bible_chapter_view'
GROUP BY
  time_bucket('1 day', timestamp),
  properties->>'version',
  properties->>'book';
```

### 5.3 Analytics API Endpoints

```typescript
// backend/src/routes/analytics.ts
import { Router } from 'express';
import { AnalyticsService } from '../analytics/analytics-service';

const router = Router();
const analytics = new AnalyticsService(pool);

// Batch event tracking endpoint
router.post('/events', async (req, res) => {
  try {
    const { events } = req.body;

    // Validate and sanitize events
    const validEvents = events.map((e: any) => ({
      ...e,
      timestamp: e.timestamp || new Date().toISOString(),
      user_agent: req.headers['user-agent'],
      ip_hash: hashIP(req.ip),
    }));

    await analytics.batchTrackEvents(validEvents);
    res.status(202).json({ received: events.length });
  } catch (error) {
    console.error('Analytics error:', error);
    res.status(500).json({ error: 'Failed to track events' });
  }
});

// Web Vitals endpoint
router.post('/vitals', async (req, res) => {
  try {
    const { name, value, id, page } = req.body;

    await analytics.trackEvent({
      event_name: `performance_${name.toLowerCase()}`,
      event_category: 'performance',
      session_id: req.headers['x-session-id'] as string,
      device_id: req.headers['x-device-id'] as string,
      properties: { metric_name: name, metric_value: value, page },
      timestamp: new Date().toISOString(),
      user_agent: req.headers['user-agent'] || '',
      ip_hash: hashIP(req.ip),
    });

    res.status(202).json({ received: true });
  } catch (error) {
    res.status(500).json({ error: 'Failed to track vitals' });
  }
});

export default router;
```

---

## 6. User Behavior Tracking

### 6.1 Session Tracking

```typescript
// lib/analytics/session.ts
import { v4 as uuidv4 } from 'uuid';

const SESSION_KEY = 'analytics_session';
const SESSION_TIMEOUT = 30 * 60 * 1000; // 30 minutes

interface Session {
  id: string;
  startedAt: number;
  lastActiveAt: number;
  pageViews: number;
  events: number;
}

export function getOrCreateSession(): Session {
  const stored = sessionStorage.getItem(SESSION_KEY);

  if (stored) {
    const session = JSON.parse(stored) as Session;
    const now = Date.now();

    // Check if session expired
    if (now - session.lastActiveAt > SESSION_TIMEOUT) {
      return createNewSession();
    }

    // Update last active
    session.lastActiveAt = now;
    sessionStorage.setItem(SESSION_KEY, JSON.stringify(session));
    return session;
  }

  return createNewSession();
}

function createNewSession(): Session {
  const session: Session = {
    id: uuidv4(),
    startedAt: Date.now(),
    lastActiveAt: Date.now(),
    pageViews: 0,
    events: 0,
  };

  sessionStorage.setItem(SESSION_KEY, JSON.stringify(session));
  return session;
}

export function incrementPageViews(): void {
  const session = getOrCreateSession();
  session.pageViews++;
  sessionStorage.setItem(SESSION_KEY, JSON.stringify(session));
}

export function getDeviceId(): string {
  const DEVICE_KEY = 'analytics_device';
  let deviceId = localStorage.getItem(DEVICE_KEY);

  if (!deviceId) {
    deviceId = uuidv4();
    localStorage.setItem(DEVICE_KEY, deviceId);
  }

  return deviceId;
}
```

### 6.2 Engagement Tracking

```typescript
// lib/analytics/engagement.ts
export class EngagementTracker {
  private startTime: number = 0;
  private totalTime: number = 0;
  private isActive: boolean = false;
  private scrollDepth: number = 0;
  private maxScrollDepth: number = 0;

  constructor() {
    this.setupListeners();
  }

  private setupListeners() {
    // Track active time
    document.addEventListener('visibilitychange', () => {
      if (document.visibilityState === 'visible') {
        this.startTracking();
      } else {
        this.pauseTracking();
      }
    });

    // Track scroll depth
    window.addEventListener('scroll', this.handleScroll.bind(this), { passive: true });

    // Track before unload
    window.addEventListener('beforeunload', this.sendEngagementData.bind(this));
  }

  startTracking() {
    this.isActive = true;
    this.startTime = Date.now();
  }

  pauseTracking() {
    if (this.isActive) {
      this.totalTime += Date.now() - this.startTime;
      this.isActive = false;
    }
  }

  private handleScroll() {
    const scrollTop = window.scrollY;
    const docHeight = document.documentElement.scrollHeight - window.innerHeight;
    this.scrollDepth = Math.round((scrollTop / docHeight) * 100);
    this.maxScrollDepth = Math.max(this.maxScrollDepth, this.scrollDepth);
  }

  getEngagementData() {
    if (this.isActive) {
      this.totalTime += Date.now() - this.startTime;
      this.startTime = Date.now();
    }

    return {
      timeOnPage: Math.round(this.totalTime / 1000), // seconds
      maxScrollDepth: this.maxScrollDepth,
    };
  }

  private sendEngagementData() {
    const data = this.getEngagementData();

    // Use sendBeacon for reliability
    navigator.sendBeacon('/api/analytics/events', JSON.stringify({
      events: [{
        event_name: 'page_engagement',
        event_category: 'engagement',
        properties: data,
        timestamp: new Date().toISOString(),
      }],
    }));
  }
}
```

### 6.3 Click & Interaction Tracking

```tsx
// hooks/useTrackClick.ts
import { useCallback } from 'react';
import { trackEvent } from '@/lib/analytics';

export function useTrackClick(
  eventName: string,
  properties: Record<string, any> = {}
) {
  return useCallback((e: React.MouseEvent) => {
    const target = e.currentTarget as HTMLElement;

    trackEvent(eventName, {
      ...properties,
      element_text: target.textContent?.substring(0, 100),
      element_id: target.id,
      element_class: target.className,
    });
  }, [eventName, properties]);
}

// Usage
function BibleNavigation() {
  const trackChapterClick = useTrackClick('bible_navigation', {
    method: 'chapter_list',
  });

  return (
    <div className="chapters">
      {chapters.map(ch => (
        <button
          key={ch}
          onClick={(e) => {
            trackChapterClick(e);
            navigateToChapter(ch);
          }}
        >
          Chapter {ch}
        </button>
      ))}
    </div>
  );
}
```

---

## 7. Reading Analytics

### 7.1 Reading Progress Tracking

```typescript
// lib/analytics/reading-tracker.ts
export class ReadingTracker {
  private currentChapter: string | null = null;
  private readStartTime: number = 0;
  private versesRead: Set<number> = new Set();
  private scrollPositions: number[] = [];

  startReading(book: string, chapter: number, version: string) {
    // Send previous reading data
    if (this.currentChapter) {
      this.endReading();
    }

    this.currentChapter = `${version}:${book}:${chapter}`;
    this.readStartTime = Date.now();
    this.versesRead.clear();
    this.scrollPositions = [];

    trackEvent('bible_reading_start', {
      book,
      chapter,
      version,
    });
  }

  markVerseRead(verseNumber: number) {
    this.versesRead.add(verseNumber);
  }

  recordScrollPosition(position: number) {
    this.scrollPositions.push(position);
  }

  endReading() {
    if (!this.currentChapter) return;

    const [version, book, chapter] = this.currentChapter.split(':');
    const readingDuration = Math.round((Date.now() - this.readStartTime) / 1000);

    trackEvent('bible_reading_end', {
      book,
      chapter: parseInt(chapter),
      version,
      duration_seconds: readingDuration,
      verses_read_count: this.versesRead.size,
      scroll_depth: Math.max(...this.scrollPositions, 0),
      average_verse_time: this.versesRead.size > 0
        ? Math.round(readingDuration / this.versesRead.size)
        : 0,
    });

    this.currentChapter = null;
  }
}
```

### 7.2 Reading Analytics Dashboard Queries

```sql
-- Most read chapters this week
SELECT
  properties->>'book' AS book,
  (properties->>'chapter')::int AS chapter,
  COUNT(*) AS read_count,
  COUNT(DISTINCT COALESCE(user_id::text, device_id)) AS unique_readers,
  AVG((properties->>'duration_seconds')::numeric) AS avg_duration
FROM analytics_events
WHERE event_name = 'bible_reading_end'
  AND timestamp > NOW() - INTERVAL '7 days'
GROUP BY properties->>'book', properties->>'chapter'
ORDER BY read_count DESC
LIMIT 20;

-- Reading habits by day of week
SELECT
  EXTRACT(DOW FROM timestamp) AS day_of_week,
  COUNT(*) AS chapter_reads,
  COUNT(DISTINCT COALESCE(user_id::text, device_id)) AS unique_readers
FROM analytics_events
WHERE event_name = 'bible_reading_end'
  AND timestamp > NOW() - INTERVAL '30 days'
GROUP BY EXTRACT(DOW FROM timestamp)
ORDER BY day_of_week;

-- Reading plan completion funnel
WITH plan_starts AS (
  SELECT
    properties->>'plan_id' AS plan_id,
    COUNT(*) AS starts
  FROM analytics_events
  WHERE event_name = 'reading_plan_start'
    AND timestamp > NOW() - INTERVAL '90 days'
  GROUP BY properties->>'plan_id'
),
plan_completions AS (
  SELECT
    properties->>'plan_id' AS plan_id,
    COUNT(*) AS completions
  FROM analytics_events
  WHERE event_name = 'reading_plan_complete'
    AND timestamp > NOW() - INTERVAL '90 days'
  GROUP BY properties->>'plan_id'
)
SELECT
  s.plan_id,
  s.starts,
  COALESCE(c.completions, 0) AS completions,
  ROUND(COALESCE(c.completions, 0)::numeric / s.starts * 100, 2) AS completion_rate
FROM plan_starts s
LEFT JOIN plan_completions c ON s.plan_id = c.plan_id
ORDER BY s.starts DESC;
```

---

## 8. Dashboard & Reporting

### 8.1 Key Dashboards

```typescript
// Analytics dashboard structure
const dashboards = {
  executive: {
    name: 'Executive Overview',
    metrics: [
      'daily_active_users',
      'weekly_active_users',
      'monthly_active_users',
      'new_user_signups',
      'premium_conversions',
      'revenue',
    ],
    charts: [
      { type: 'line', metric: 'dau_trend', period: '30d' },
      { type: 'funnel', metric: 'registration_funnel' },
      { type: 'pie', metric: 'user_segments' },
    ],
  },

  engagement: {
    name: 'User Engagement',
    metrics: [
      'session_duration_avg',
      'pages_per_session',
      'chapters_read_per_user',
      'return_rate',
      'feature_adoption',
    ],
    charts: [
      { type: 'line', metric: 'engagement_trend', period: '30d' },
      { type: 'heatmap', metric: 'usage_by_hour' },
      { type: 'bar', metric: 'feature_usage' },
    ],
  },

  content: {
    name: 'Content Performance',
    metrics: [
      'most_read_books',
      'most_read_chapters',
      'search_trends',
      'popular_reading_plans',
    ],
    charts: [
      { type: 'treemap', metric: 'bible_book_reads' },
      { type: 'bar', metric: 'top_chapters' },
      { type: 'wordcloud', metric: 'search_terms' },
    ],
  },

  technical: {
    name: 'Technical Performance',
    metrics: [
      'lcp_p75',
      'fid_p75',
      'cls_p75',
      'error_rate',
      'api_latency_p95',
    ],
    charts: [
      { type: 'line', metric: 'web_vitals_trend' },
      { type: 'bar', metric: 'errors_by_type' },
      { type: 'line', metric: 'api_performance' },
    ],
  },
};
```

### 8.2 Automated Reports

```typescript
// backend/src/analytics/reports.ts
import { CronJob } from 'cron';
import { sendEmail } from '../email/email-service';

// Daily report
const dailyReport = new CronJob('0 9 * * *', async () => {
  const report = await generateDailyReport();
  await sendEmail({
    to: 'team@biblereader.app',
    subject: `Daily Analytics Report - ${new Date().toDateString()}`,
    html: formatReportAsHtml(report),
  });
});

// Weekly report
const weeklyReport = new CronJob('0 9 * * 1', async () => {
  const report = await generateWeeklyReport();
  await sendEmail({
    to: 'leadership@biblereader.app',
    subject: `Weekly Analytics Report - Week ${getWeekNumber()}`,
    html: formatReportAsHtml(report),
  });
});

async function generateDailyReport(): Promise<DailyReport> {
  const yesterday = new Date();
  yesterday.setDate(yesterday.getDate() - 1);

  return {
    date: yesterday,
    dau: await analytics.getDailyActiveUsers(yesterday),
    newUsers: await analytics.getNewUsers(yesterday),
    topChapters: await analytics.getTopChapters(yesterday, 10),
    featureUsage: await analytics.getFeatureUsage(yesterday),
    errors: await analytics.getErrorCount(yesterday),
    webVitals: await analytics.getWebVitals(yesterday),
  };
}
```

---

## 9. Privacy & Compliance

### 9.1 Consent Management

```tsx
// components/CookieConsent.tsx
import { useState, useEffect } from 'react';
import { initGA } from '@/lib/analytics/ga4';

type ConsentLevel = 'none' | 'essential' | 'analytics' | 'all';

export function CookieConsent() {
  const [consent, setConsent] = useState<ConsentLevel | null>(null);
  const [showBanner, setShowBanner] = useState(false);

  useEffect(() => {
    const storedConsent = localStorage.getItem('cookie_consent') as ConsentLevel;
    if (storedConsent) {
      setConsent(storedConsent);
      if (storedConsent === 'analytics' || storedConsent === 'all') {
        initGA();
      }
    } else {
      setShowBanner(true);
    }
  }, []);

  const handleConsent = (level: ConsentLevel) => {
    setConsent(level);
    localStorage.setItem('cookie_consent', level);
    setShowBanner(false);

    // Initialize analytics if consented
    if (level === 'analytics' || level === 'all') {
      initGA();
    }

    // Update GA consent mode
    if (window.gtag) {
      window.gtag('consent', 'update', {
        analytics_storage: level === 'analytics' || level === 'all' ? 'granted' : 'denied',
        ad_storage: level === 'all' ? 'granted' : 'denied',
      });
    }
  };

  if (!showBanner) return null;

  return (
    <div className="fixed bottom-0 left-0 right-0 bg-white shadow-lg p-4 border-t">
      <div className="max-w-4xl mx-auto flex flex-col md:flex-row items-center gap-4">
        <p className="text-sm text-gray-600 flex-1">
          We use cookies to improve your experience and analyze site usage.
          Read our <a href="/privacy" className="text-blue-600">Privacy Policy</a>.
        </p>
        <div className="flex gap-2">
          <button
            onClick={() => handleConsent('essential')}
            className="px-4 py-2 text-sm border rounded hover:bg-gray-50"
          >
            Essential Only
          </button>
          <button
            onClick={() => handleConsent('analytics')}
            className="px-4 py-2 text-sm bg-blue-600 text-white rounded hover:bg-blue-700"
          >
            Accept Analytics
          </button>
        </div>
      </div>
    </div>
  );
}
```

### 9.2 Data Anonymization

```typescript
// lib/analytics/anonymize.ts
import crypto from 'crypto';

// Hash IP address for privacy
export function hashIP(ip: string): string {
  // Add daily salt for extra privacy
  const salt = new Date().toDateString();
  return crypto
    .createHash('sha256')
    .update(ip + salt)
    .digest('hex')
    .substring(0, 16);
}

// Anonymize user data for analytics
export function anonymizeUser(user: User): AnonymizedUser {
  return {
    id: hashUserId(user.id),
    accountAge: calculateAccountAge(user.createdAt),
    tier: user.isPremium ? 'premium' : 'free',
    // Don't include email, name, or other PII
  };
}

// Truncate search queries for privacy
export function sanitizeSearchQuery(query: string): string {
  // Remove potential PII patterns
  const sanitized = query
    .replace(/\b[\w._%+-]+@[\w.-]+\.[a-zA-Z]{2,}\b/g, '[email]') // Email
    .replace(/\b\d{10,}\b/g, '[phone]') // Phone numbers
    .replace(/\b\d{3}[-.]?\d{3}[-.]?\d{4}\b/g, '[phone]'); // US phone

  // Truncate long queries
  return sanitized.substring(0, 100);
}
```

### 9.3 Data Retention Policies

```typescript
// Data retention configuration
const retentionPolicies = {
  // Raw analytics events
  analyticsEvents: {
    retention: '90 days',
    action: 'delete',
    schedule: 'daily at 2 AM UTC',
  },

  // Aggregated data
  aggregatedMetrics: {
    retention: '24 months',
    action: 'archive',
    schedule: 'monthly',
  },

  // User behavior data
  userBehavior: {
    retention: '12 months',
    action: 'anonymize',
    schedule: 'weekly',
  },

  // Error logs
  errorLogs: {
    retention: '30 days',
    action: 'delete',
    schedule: 'daily',
  },
};

// Cleanup job
async function runDataRetention() {
  // Delete old analytics events
  await db.query(`
    DELETE FROM analytics_events
    WHERE timestamp < NOW() - INTERVAL '90 days'
  `);

  // Anonymize old user behavior
  await db.query(`
    UPDATE analytics_events
    SET user_id = NULL, ip_hash = 'anonymized'
    WHERE timestamp < NOW() - INTERVAL '12 months'
      AND user_id IS NOT NULL
  `);
}
```

---

## 10. A/B Testing

### 10.1 A/B Testing Framework

```typescript
// lib/ab-testing/experiment.ts
import { trackEvent } from '../analytics';

interface Experiment {
  id: string;
  name: string;
  variants: {
    id: string;
    name: string;
    weight: number; // 0-100
  }[];
  targetAudience?: {
    newUsers?: boolean;
    userTier?: 'free' | 'premium';
    language?: string[];
  };
}

class ABTestingService {
  private experiments: Map<string, Experiment> = new Map();
  private userAssignments: Map<string, Map<string, string>> = new Map();

  async getVariant(experimentId: string, userId: string): Promise<string> {
    // Check for existing assignment
    const existing = this.getUserAssignment(userId, experimentId);
    if (existing) return existing;

    const experiment = this.experiments.get(experimentId);
    if (!experiment) return 'control';

    // Assign variant based on weights
    const variant = this.assignVariant(experiment, userId);

    // Store assignment
    this.setUserAssignment(userId, experimentId, variant);

    // Track exposure
    trackEvent('experiment_exposure', {
      experiment_id: experimentId,
      experiment_name: experiment.name,
      variant_id: variant,
    });

    return variant;
  }

  private assignVariant(experiment: Experiment, userId: string): string {
    // Deterministic assignment based on user ID
    const hash = this.hashString(`${experiment.id}:${userId}`);
    const bucket = hash % 100;

    let cumWeight = 0;
    for (const variant of experiment.variants) {
      cumWeight += variant.weight;
      if (bucket < cumWeight) {
        return variant.id;
      }
    }

    return experiment.variants[0].id;
  }

  trackConversion(experimentId: string, userId: string, metric: string, value?: number) {
    const variant = this.getUserAssignment(userId, experimentId);
    if (!variant) return;

    trackEvent('experiment_conversion', {
      experiment_id: experimentId,
      variant_id: variant,
      metric,
      value,
    });
  }

  private hashString(str: string): number {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash;
    }
    return Math.abs(hash);
  }
}

export const abTesting = new ABTestingService();
```

### 10.2 A/B Test React Hook

```tsx
// hooks/useExperiment.ts
import { useState, useEffect } from 'react';
import { abTesting } from '@/lib/ab-testing';
import { useAuth } from './useAuth';

export function useExperiment(experimentId: string) {
  const { user } = useAuth();
  const [variant, setVariant] = useState<string>('control');
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const userId = user?.id || getAnonymousId();

    abTesting.getVariant(experimentId, userId).then(v => {
      setVariant(v);
      setIsLoading(false);
    });
  }, [experimentId, user?.id]);

  const trackConversion = (metric: string, value?: number) => {
    const userId = user?.id || getAnonymousId();
    abTesting.trackConversion(experimentId, userId, metric, value);
  };

  return { variant, isLoading, trackConversion };
}

// Usage example
function TTSButton() {
  const { variant, trackConversion } = useExperiment('tts_button_design');

  const handleClick = () => {
    trackConversion('tts_click');
    startTTS();
  };

  if (variant === 'large_button') {
    return <LargeTTSButton onClick={handleClick} />;
  }

  return <StandardTTSButton onClick={handleClick} />;
}
```

---

## Summary

This analytics and tracking implementation provides comprehensive insights into user behavior and application performance:

### Analytics Coverage

| Area | Implementation | Tools |
|------|---------------|-------|
| Page Views | Automatic SPA tracking | GA4 + Custom |
| User Events | Structured event schema | GA4 + Custom |
| Reading Analytics | Chapter/verse tracking | Custom |
| Feature Usage | Click & interaction tracking | GA4 + Custom |
| Performance | Core Web Vitals | web-vitals + Custom |
| Errors | Exception tracking | Sentry |
| A/B Testing | Experiment framework | Custom |

### Key Features

- Privacy-compliant analytics with consent management
- Real-time and aggregated metrics
- Custom dashboards for different stakeholders
- Automated reporting
- GDPR-compliant data handling

---

**Document Version**: 1.0.0
**Last Updated**: 2025-01-14
**Author**: Analytics Team
