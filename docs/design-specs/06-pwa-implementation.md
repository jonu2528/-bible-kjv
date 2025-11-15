# Part 6: PWA Implementation Details

## 6.1 PWA Overview & Requirements

### 6.1.1 Why PWA for Bible Reader?

**Key Benefits:**
```
1. Offline Access
   └─ อ่านพระคัมภีร์ได้แม้ไม่มีอินเทอร์เน็ต
   └─ Essential for spiritual reading anytime, anywhere

2. App-like Experience
   └─ ติดตั้งบน home screen
   └─ Full-screen mode (no browser UI)
   └─ Fast, smooth performance

3. Push Notifications
   └─ Quote of the Day reminders
   └─ Reading plan notifications
   └─ Re-engagement

4. Lower Barrier to Entry
   └─ No app store approval
   └─ Instant updates
   └─ Cross-platform (one codebase)

5. Data Efficiency
   └─ Cache static resources
   └─ Reduce bandwidth usage
   └─ Faster subsequent loads
```

### 6.1.2 PWA Checklist

**Baseline Requirements:**
```
✅ HTTPS (required for service workers)
✅ Web App Manifest (manifest.json)
✅ Service Worker registered
✅ Responsive design (mobile-first)
✅ Fast load time (< 3s on 3G)
✅ Works offline (at least basic functionality)
```

**Enhanced Requirements:**
```
✅ Add to Home Screen prompt
✅ Full-screen/standalone mode
✅ Push notifications
✅ Background sync
✅ App-like navigation (no browser chrome)
✅ Lighthouse PWA score > 90
```

---

## 6.2 Web App Manifest

### 6.2.1 Manifest File Structure

**File: `/public/manifest.json`**

```json
{
  "name": "Online Bible Reader: KJV and Multi-language",
  "short_name": "Bible Reader",
  "description": "Read the King James Version Bible online with multi-language support, text-to-speech, and study tools",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#2563eb",
  "orientation": "any",
  "scope": "/",

  "icons": [
    {
      "src": "/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    }
  ],

  "screenshots": [
    {
      "src": "/screenshots/home-mobile.png",
      "sizes": "540x720",
      "type": "image/png",
      "form_factor": "narrow"
    },
    {
      "src": "/screenshots/read-mobile.png",
      "sizes": "540x720",
      "type": "image/png",
      "form_factor": "narrow"
    },
    {
      "src": "/screenshots/home-desktop.png",
      "sizes": "1280x720",
      "type": "image/png",
      "form_factor": "wide"
    }
  ],

  "categories": ["books", "education", "lifestyle"],
  "lang": "en-US",

  "shortcuts": [
    {
      "name": "Continue Reading",
      "short_name": "Continue",
      "description": "Resume your last reading position",
      "url": "/read?continue=true",
      "icons": [
        {
          "src": "/icons/shortcut-continue.png",
          "sizes": "96x96"
        }
      ]
    },
    {
      "name": "Quote of the Day",
      "short_name": "Quote",
      "description": "View today's inspirational verse",
      "url": "/?quote=today",
      "icons": [
        {
          "src": "/icons/shortcut-quote.png",
          "sizes": "96x96"
        }
      ]
    },
    {
      "name": "Search Bible",
      "short_name": "Search",
      "description": "Search for verses and topics",
      "url": "/search",
      "icons": [
        {
          "src": "/icons/shortcut-search.png",
          "sizes": "96x96"
        }
      ]
    }
  ],

  "related_applications": [],
  "prefer_related_applications": false
}
```

### 6.2.2 Icon Requirements

**Icon Sizes Needed:**
```
Required:
- 192x192 (Android)
- 512x512 (Android splash screen)

Recommended:
- 72x72, 96x96, 128x128, 144x144 (various Android devices)
- 152x152 (iPad)
- 180x180 (iPhone)
- 384x384 (high-res devices)

Format:
- PNG with transparency
- Purpose: "any maskable" (adaptive icons for Android)
```

**Maskable Icon Guidelines:**
```
Safe Zone:
┌─────────────────────────────┐
│        (40px padding)       │
│   ┌─────────────────────┐   │
│   │                     │   │
│   │   Icon content      │   │ ← 80% of total size
│   │   (safe zone)       │   │   (e.g., 410x410 in 512x512)
│   │                     │   │
│   └─────────────────────┘   │
│                             │
└─────────────────────────────┘

Why?
- Android may crop icons to circles/rounded squares
- Safe zone ensures icon isn't cut off
```

### 6.2.3 HTML Integration

**In `<head>` section:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <!-- PWA Manifest -->
  <link rel="manifest" href="/manifest.json">

  <!-- Theme Color -->
  <meta name="theme-color" content="#2563eb">
  <meta name="theme-color" media="(prefers-color-scheme: dark)" content="#1e40af">

  <!-- iOS Support -->
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="default">
  <meta name="apple-mobile-web-app-title" content="Bible Reader">
  <link rel="apple-touch-icon" href="/icons/icon-180x180.png">

  <!-- Windows Support -->
  <meta name="msapplication-TileImage" content="/icons/icon-144x144.png">
  <meta name="msapplication-TileColor" content="#2563eb">

  <!-- Favicon -->
  <link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png">
  <link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png">

  <title>Online Bible Reader - KJV and Multi-language</title>
</head>
<body>
  <!-- App content -->
</body>
</html>
```

---

## 6.3 Service Worker Implementation

### 6.3.1 Service Worker Architecture

**File Structure:**
```
/public/
├── sw.js                    ← Main service worker
├── manifest.json
└── offline.html             ← Offline fallback page

/src/
└── service-worker/
    ├── register.js          ← Registration logic
    ├── config.js            ← Cache names, versions
    └── strategies/
        ├── cache-first.js
        ├── network-first.js
        └── stale-while-revalidate.js
```

### 6.3.2 Service Worker Registration

**File: `/src/service-worker/register.js`**

```javascript
/**
 * Register Service Worker
 * Call this in your main app entry point
 */
export function registerServiceWorker() {
  // Check if service workers are supported
  if ('serviceWorker' in navigator) {
    // Wait for page load
    window.addEventListener('load', () => {
      navigator.serviceWorker
        .register('/sw.js')
        .then(registration => {
          console.log('✅ Service Worker registered:', registration.scope);

          // Check for updates
          checkForUpdates(registration);

          // Listen for new service worker
          registration.addEventListener('updatefound', () => {
            const newWorker = registration.installing;

            newWorker.addEventListener('statechange', () => {
              if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
                // New service worker available
                showUpdateNotification();
              }
            });
          });
        })
        .catch(error => {
          console.error('❌ Service Worker registration failed:', error);
        });

      // Listen for service worker messages
      navigator.serviceWorker.addEventListener('message', handleSWMessage);
    });
  } else {
    console.warn('Service Workers not supported in this browser');
  }
}

/**
 * Check for updates every hour
 */
function checkForUpdates(registration) {
  setInterval(() => {
    registration.update();
  }, 60 * 60 * 1000); // 1 hour
}

/**
 * Show update notification to user
 */
function showUpdateNotification() {
  const notification = document.createElement('div');
  notification.className = 'update-notification';
  notification.innerHTML = `
    <p>A new version is available!</p>
    <button onclick="window.location.reload()">Update Now</button>
    <button onclick="this.parentElement.remove()">Later</button>
  `;
  document.body.appendChild(notification);
}

/**
 * Handle messages from service worker
 */
function handleSWMessage(event) {
  const { type, payload } = event.data;

  switch (type) {
    case 'CACHE_UPDATED':
      console.log('Cache updated:', payload);
      break;

    case 'OFFLINE_MODE':
      showOfflineIndicator();
      break;

    case 'ONLINE_MODE':
      hideOfflineIndicator();
      break;
  }
}
```

### 6.3.3 Service Worker Main File

**File: `/public/sw.js`**

```javascript
// Service Worker Version
const SW_VERSION = 'v1.0.0';

// Cache Names
const CACHE_NAMES = {
  static: `bible-reader-static-${SW_VERSION}`,
  dynamic: `bible-reader-dynamic-${SW_VERSION}`,
  bible: `bible-reader-content-${SW_VERSION}`,
  images: `bible-reader-images-${SW_VERSION}`,
};

// Static Assets to Pre-cache
const STATIC_ASSETS = [
  '/',
  '/offline.html',
  '/manifest.json',
  '/css/main.css',
  '/js/app.js',
  '/icons/icon-192x192.png',
  '/icons/icon-512x512.png',
];

// API Endpoints
const API_BASE = '/api/v1';

/**
 * Install Event - Pre-cache static assets
 */
self.addEventListener('install', event => {
  console.log('[SW] Installing version:', SW_VERSION);

  event.waitUntil(
    caches.open(CACHE_NAMES.static)
      .then(cache => {
        console.log('[SW] Pre-caching static assets');
        return cache.addAll(STATIC_ASSETS);
      })
      .then(() => {
        // Skip waiting to activate immediately
        return self.skipWaiting();
      })
  );
});

/**
 * Activate Event - Clean up old caches
 */
self.addEventListener('activate', event => {
  console.log('[SW] Activating version:', SW_VERSION);

  event.waitUntil(
    caches.keys()
      .then(cacheNames => {
        return Promise.all(
          cacheNames
            .filter(cacheName => {
              // Delete old caches
              return Object.values(CACHE_NAMES).indexOf(cacheName) === -1;
            })
            .map(cacheName => {
              console.log('[SW] Deleting old cache:', cacheName);
              return caches.delete(cacheName);
            })
        );
      })
      .then(() => {
        // Take control of all clients
        return self.clients.claim();
      })
  );
});

/**
 * Fetch Event - Intercept network requests
 */
self.addEventListener('fetch', event => {
  const { request } = event;
  const url = new URL(request.url);

  // Skip non-GET requests
  if (request.method !== 'GET') {
    return;
  }

  // Route requests to appropriate strategy
  if (url.pathname.startsWith('/api/bible/')) {
    // Bible content - Cache First
    event.respondWith(cacheFirst(request, CACHE_NAMES.bible));
  }
  else if (url.pathname.startsWith('/api/')) {
    // Other API - Network First
    event.respondWith(networkFirst(request, CACHE_NAMES.dynamic));
  }
  else if (request.destination === 'image') {
    // Images - Cache First
    event.respondWith(cacheFirst(request, CACHE_NAMES.images));
  }
  else if (STATIC_ASSETS.includes(url.pathname)) {
    // Static assets - Cache First
    event.respondWith(cacheFirst(request, CACHE_NAMES.static));
  }
  else {
    // HTML pages - Network First with fallback
    event.respondWith(networkFirstWithOffline(request));
  }
});

/**
 * Cache First Strategy
 * Use for: Static assets, Bible content, Images
 */
async function cacheFirst(request, cacheName) {
  const cache = await caches.open(cacheName);
  const cached = await cache.match(request);

  if (cached) {
    // Return cached version
    return cached;
  }

  // Not in cache, fetch from network
  try {
    const response = await fetch(request);

    // Cache successful response
    if (response.ok) {
      cache.put(request, response.clone());
    }

    return response;
  } catch (error) {
    console.error('[SW] Fetch failed:', error);

    // Return offline page for navigation requests
    if (request.mode === 'navigate') {
      return caches.match('/offline.html');
    }

    throw error;
  }
}

/**
 * Network First Strategy
 * Use for: API calls, dynamic content
 */
async function networkFirst(request, cacheName) {
  const cache = await caches.open(cacheName);

  try {
    // Try network first
    const response = await fetch(request);

    // Cache successful response
    if (response.ok) {
      cache.put(request, response.clone());
    }

    return response;
  } catch (error) {
    // Network failed, try cache
    const cached = await cache.match(request);

    if (cached) {
      // Notify client about offline mode
      self.clients.matchAll().then(clients => {
        clients.forEach(client => {
          client.postMessage({
            type: 'OFFLINE_MODE',
            payload: { url: request.url }
          });
        });
      });

      return cached;
    }

    throw error;
  }
}

/**
 * Network First with Offline Fallback
 * Use for: HTML pages
 */
async function networkFirstWithOffline(request) {
  try {
    const response = await fetch(request);

    // Cache HTML pages
    if (response.ok && request.mode === 'navigate') {
      const cache = await caches.open(CACHE_NAMES.dynamic);
      cache.put(request, response.clone());
    }

    return response;
  } catch (error) {
    // Try cache
    const cached = await caches.match(request);
    if (cached) {
      return cached;
    }

    // Return offline page
    if (request.mode === 'navigate') {
      return caches.match('/offline.html');
    }

    throw error;
  }
}

/**
 * Stale While Revalidate Strategy (Alternative)
 * Use for: Assets that can be slightly outdated
 */
async function staleWhileRevalidate(request, cacheName) {
  const cache = await caches.open(cacheName);
  const cached = await cache.match(request);

  // Fetch in background
  const fetchPromise = fetch(request).then(response => {
    if (response.ok) {
      cache.put(request, response.clone());
    }
    return response;
  });

  // Return cached immediately, or wait for network
  return cached || fetchPromise;
}

/**
 * Message Handler
 */
self.addEventListener('message', event => {
  const { type, payload } = event.data;

  switch (type) {
    case 'SKIP_WAITING':
      self.skipWaiting();
      break;

    case 'CACHE_CHAPTER':
      cacheChapter(payload.book, payload.chapter);
      break;

    case 'CLEAR_CACHE':
      clearAllCaches();
      break;
  }
});

/**
 * Cache a specific chapter for offline reading
 */
async function cacheChapter(book, chapter) {
  const cache = await caches.open(CACHE_NAMES.bible);
  const url = `${API_BASE}/bible/verses?book=${book}&chapter=${chapter}`;

  try {
    const response = await fetch(url);
    if (response.ok) {
      await cache.put(url, response);

      // Notify client
      self.clients.matchAll().then(clients => {
        clients.forEach(client => {
          client.postMessage({
            type: 'CHAPTER_CACHED',
            payload: { book, chapter }
          });
        });
      });
    }
  } catch (error) {
    console.error('[SW] Failed to cache chapter:', error);
  }
}

/**
 * Clear all caches
 */
async function clearAllCaches() {
  const cacheNames = await caches.keys();
  await Promise.all(
    cacheNames.map(cacheName => caches.delete(cacheName))
  );

  console.log('[SW] All caches cleared');
}
```

---

## 6.4 Caching Strategies

### 6.4.1 Strategy Decision Matrix

| Resource Type | Strategy | Rationale |
|---------------|----------|-----------|
| **Bible Text** | Cache First | Core content, rarely changes, essential offline |
| **Static Assets** (CSS/JS) | Cache First | Versioned files, immutable |
| **Images/Icons** | Cache First | Rarely change, save bandwidth |
| **User Data API** | Network First | Need fresh data, cache as backup |
| **HTML Pages** | Network First + Offline | Fresh content preferred, offline fallback |
| **Search Results** | Network Only | Always need fresh results |
| **Analytics** | Network Only | Non-critical, skip if offline |

### 6.4.2 Cache Size Management

**Quota Monitoring:**

```javascript
/**
 * Check cache storage quota
 */
async function checkStorageQuota() {
  if ('storage' in navigator && 'estimate' in navigator.storage) {
    const estimate = await navigator.storage.estimate();
    const percent = (estimate.usage / estimate.quota * 100).toFixed(2);

    console.log(`Storage: ${formatBytes(estimate.usage)} / ${formatBytes(estimate.quota)} (${percent}%)`);

    // Warn if approaching limit
    if (percent > 80) {
      console.warn('⚠️ Cache storage approaching limit');
      // Trigger cleanup
      await cleanupOldCaches();
    }

    return estimate;
  }
}

function formatBytes(bytes) {
  if (bytes < 1024) return bytes + ' B';
  if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(2) + ' KB';
  return (bytes / (1024 * 1024)).toFixed(2) + ' MB';
}
```

**Cache Cleanup Strategy:**

```javascript
/**
 * LRU (Least Recently Used) Cache Cleanup
 */
async function cleanupOldCaches() {
  const cache = await caches.open(CACHE_NAMES.bible);
  const requests = await cache.keys();

  // Get cache entries with metadata
  const entries = await Promise.all(
    requests.map(async request => {
      const response = await cache.match(request);
      const dateHeader = response.headers.get('date');
      const lastUsed = dateHeader ? new Date(dateHeader) : new Date(0);

      return { request, lastUsed };
    })
  );

  // Sort by last used (oldest first)
  entries.sort((a, b) => a.lastUsed - b.lastUsed);

  // Delete oldest 20% of entries
  const deleteCount = Math.floor(entries.length * 0.2);
  const toDelete = entries.slice(0, deleteCount);

  await Promise.all(
    toDelete.map(({ request }) => cache.delete(request))
  );

  console.log(`[SW] Cleaned up ${deleteCount} old cache entries`);
}
```

---

## 6.5 Offline Support

### 6.5.1 Offline Page

**File: `/public/offline.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Offline - Bible Reader</title>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      display: flex;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
      text-align: center;
      padding: 2rem;
    }

    .container {
      max-width: 500px;
    }

    .icon {
      font-size: 80px;
      margin-bottom: 1rem;
    }

    h1 {
      font-size: 2rem;
      margin-bottom: 1rem;
    }

    p {
      font-size: 1.1rem;
      line-height: 1.6;
      margin-bottom: 2rem;
      opacity: 0.9;
    }

    .button {
      display: inline-block;
      padding: 1rem 2rem;
      background: white;
      color: #667eea;
      text-decoration: none;
      border-radius: 8px;
      font-weight: 600;
      transition: transform 0.2s;
      cursor: pointer;
      border: none;
      font-size: 1rem;
    }

    .button:hover {
      transform: translateY(-2px);
    }

    .cached-list {
      margin-top: 2rem;
      text-align: left;
      background: rgba(255, 255, 255, 0.1);
      padding: 1.5rem;
      border-radius: 8px;
    }

    .cached-list h2 {
      font-size: 1.2rem;
      margin-bottom: 1rem;
    }

    .cached-list ul {
      list-style: none;
    }

    .cached-list li {
      padding: 0.5rem 0;
      border-bottom: 1px solid rgba(255, 255, 255, 0.2);
    }

    .cached-list li:last-child {
      border-bottom: none;
    }

    .cached-list a {
      color: white;
      text-decoration: none;
    }

    .cached-list a:hover {
      text-decoration: underline;
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="icon">📡</div>
    <h1>You're Offline</h1>
    <p>
      It looks like you've lost your internet connection.
      Don't worry, you can still access cached Bible chapters below.
    </p>

    <button class="button" onclick="window.location.reload()">
      Try Again
    </button>

    <div class="cached-list" id="cachedContent">
      <h2>Available Offline:</h2>
      <ul id="cachedChaptersList">
        <li>Loading cached content...</li>
      </ul>
    </div>
  </div>

  <script>
    // Load cached chapters list
    async function loadCachedChapters() {
      try {
        const cache = await caches.open('bible-reader-content-v1.0.0');
        const requests = await cache.keys();

        const chapters = requests
          .map(req => {
            const url = new URL(req.url);
            const params = new URLSearchParams(url.search);
            return {
              book: params.get('book'),
              chapter: params.get('chapter'),
              url: url.pathname + url.search
            };
          })
          .filter(c => c.book && c.chapter);

        const list = document.getElementById('cachedChaptersList');

        if (chapters.length === 0) {
          list.innerHTML = '<li>No cached chapters available</li>';
        } else {
          list.innerHTML = chapters
            .map(c => `
              <li>
                <a href="/read/${c.book}/${c.chapter}">
                  ${c.book} Chapter ${c.chapter}
                </a>
              </li>
            `)
            .join('');
        }
      } catch (error) {
        console.error('Failed to load cached chapters:', error);
      }
    }

    loadCachedChapters();
  </script>
</body>
</html>
```

### 6.5.2 Offline Detection

```javascript
/**
 * Network status detection
 */
class NetworkStatus {
  constructor() {
    this.isOnline = navigator.onLine;
    this.listeners = [];

    window.addEventListener('online', () => this.handleOnline());
    window.addEventListener('offline', () => this.handleOffline());
  }

  handleOnline() {
    this.isOnline = true;
    this.showNotification('You are back online! 🎉', 'success');
    this.notifyListeners('online');

    // Sync pending changes
    this.syncPendingData();
  }

  handleOffline() {
    this.isOnline = false;
    this.showNotification('You are offline. Cached content is available.', 'warning');
    this.notifyListeners('offline');
  }

  onChange(callback) {
    this.listeners.push(callback);
  }

  notifyListeners(status) {
    this.listeners.forEach(callback => callback(status));
  }

  showNotification(message, type) {
    // Implementation depends on your UI framework
    const banner = document.createElement('div');
    banner.className = `network-status ${type}`;
    banner.textContent = message;
    document.body.appendChild(banner);

    setTimeout(() => banner.remove(), 5000);
  }

  async syncPendingData() {
    // Sync highlights, bookmarks, notes
    if ('sync' in registration) {
      await registration.sync.register('sync-user-data');
    }
  }
}

// Initialize
const networkStatus = new NetworkStatus();

// Usage
networkStatus.onChange(status => {
  console.log('Network status changed:', status);

  if (status === 'online') {
    // Re-enable features that require network
    enableSearchFeature();
    syncUserData();
  } else {
    // Disable network-dependent features
    disableSearchFeature();
    showOfflineMode();
  }
});
```

---

## 6.6 Push Notifications

### 6.6.1 Push Notification Setup

**Request Permission:**

```javascript
/**
 * Request notification permission
 */
async function requestNotificationPermission() {
  if (!('Notification' in window)) {
    console.warn('This browser does not support notifications');
    return false;
  }

  if (Notification.permission === 'granted') {
    return true;
  }

  if (Notification.permission !== 'denied') {
    const permission = await Notification.requestPermission();
    return permission === 'granted';
  }

  return false;
}

/**
 * Subscribe to push notifications
 */
async function subscribeToPush() {
  try {
    const permission = await requestNotificationPermission();

    if (!permission) {
      throw new Error('Notification permission denied');
    }

    const registration = await navigator.serviceWorker.ready;

    // Get VAPID public key from server
    const vapidPublicKey = await fetch('/api/push/vapid-key')
      .then(res => res.text());

    // Subscribe
    const subscription = await registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: urlBase64ToUint8Array(vapidPublicKey)
    });

    // Send subscription to server
    await fetch('/api/push/subscribe', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(subscription)
    });

    console.log('✅ Subscribed to push notifications');
    return subscription;

  } catch (error) {
    console.error('❌ Push subscription failed:', error);
    throw error;
  }
}

/**
 * Convert VAPID key
 */
function urlBase64ToUint8Array(base64String) {
  const padding = '='.repeat((4 - base64String.length % 4) % 4);
  const base64 = (base64String + padding)
    .replace(/\-/g, '+')
    .replace(/_/g, '/');

  const rawData = window.atob(base64);
  const outputArray = new Uint8Array(rawData.length);

  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i);
  }

  return outputArray;
}
```

### 6.6.2 Handle Push Events (Service Worker)

**Add to `/public/sw.js`:**

```javascript
/**
 * Push Event Handler
 */
self.addEventListener('push', event => {
  console.log('[SW] Push received:', event);

  let data = {
    title: 'Bible Reader',
    body: 'You have a new notification',
    icon: '/icons/icon-192x192.png',
    badge: '/icons/badge-72x72.png',
    tag: 'default',
    requireInteraction: false,
    data: {}
  };

  if (event.data) {
    data = { ...data, ...event.data.json() };
  }

  const options = {
    body: data.body,
    icon: data.icon,
    badge: data.badge,
    tag: data.tag,
    requireInteraction: data.requireInteraction,
    data: data.data,
    actions: data.actions || [],
    vibrate: [200, 100, 200]
  };

  event.waitUntil(
    self.registration.showNotification(data.title, options)
  );
});

/**
 * Notification Click Handler
 */
self.addEventListener('notificationclick', event => {
  console.log('[SW] Notification clicked:', event);

  event.notification.close();

  const { action, notification } = event;
  const data = notification.data || {};

  // Handle action buttons
  if (action === 'read') {
    // Open reading page
    event.waitUntil(
      clients.openWindow(data.url || '/read')
    );
  } else if (action === 'dismiss') {
    // Just close
    return;
  } else {
    // Default: open app
    event.waitUntil(
      clients.matchAll({ type: 'window', includeUncontrolled: true })
        .then(clientList => {
          // Focus existing window if available
          for (let client of clientList) {
            if (client.url === '/' && 'focus' in client) {
              return client.focus();
            }
          }

          // Open new window
          if (clients.openWindow) {
            return clients.openWindow(data.url || '/');
          }
        })
    );
  }
});
```

### 6.6.3 Notification Types

**Quote of the Day:**

```javascript
// Server-side (Node.js example)
const webpush = require('web-push');

async function sendQuoteNotification(subscription, quote) {
  const payload = JSON.stringify({
    title: '📖 Quote of the Day',
    body: `"${quote.text}" — ${quote.reference}`,
    icon: '/icons/icon-192x192.png',
    badge: '/icons/badge-72x72.png',
    tag: 'quote-daily',
    data: {
      type: 'quote',
      url: `/?quote=${quote.date}`,
      quoteId: quote.id
    },
    actions: [
      {
        action: 'read',
        title: 'Read in Context'
      },
      {
        action: 'dismiss',
        title: 'Dismiss'
      }
    ]
  });

  try {
    await webpush.sendNotification(subscription, payload);
    console.log('✅ Quote notification sent');
  } catch (error) {
    console.error('❌ Failed to send notification:', error);

    // Remove invalid subscriptions
    if (error.statusCode === 410) {
      await removeSubscription(subscription);
    }
  }
}
```

**Reading Plan Reminder:**

```javascript
async function sendReadingPlanReminder(subscription, plan) {
  const payload = JSON.stringify({
    title: '📅 Reading Plan Reminder',
    body: `Today's reading: ${plan.todayReading}`,
    icon: '/icons/icon-192x192.png',
    badge: '/icons/badge-72x72.png',
    tag: 'reading-plan',
    requireInteraction: true,
    data: {
      type: 'reading-plan',
      url: `/plans/active?day=${plan.currentDay}`,
      planId: plan.id
    },
    actions: [
      {
        action: 'read',
        title: 'Start Reading'
      },
      {
        action: 'dismiss',
        title: 'Later'
      }
    ]
  });

  await webpush.sendNotification(subscription, payload);
}
```

---

## 6.7 Background Sync

### 6.7.1 Background Sync Setup

**Register Sync (Client-side):**

```javascript
/**
 * Queue data for background sync
 */
async function queueForSync(action, data) {
  // Store in IndexedDB
  await saveToIndexedDB('sync-queue', {
    id: Date.now(),
    action,
    data,
    timestamp: new Date().toISOString(),
    synced: false
  });

  // Register sync
  if ('sync' in registration) {
    try {
      await registration.sync.register(`sync-${action}`);
      console.log('✅ Sync registered:', action);
    } catch (error) {
      console.error('❌ Sync registration failed:', error);
      // Fallback: try sync immediately
      await syncData(action, data);
    }
  } else {
    // Background Sync not supported
    // Try immediate sync if online
    if (navigator.onLine) {
      await syncData(action, data);
    }
  }
}

// Usage
async function saveHighlight(verse, color) {
  // Save locally first
  await saveToLocalStorage('highlights', { verse, color });

  // Queue for sync
  await queueForSync('save-highlight', { verse, color });
}
```

**Handle Sync (Service Worker):**

```javascript
/**
 * Background Sync Event Handler
 * Add to /public/sw.js
 */
self.addEventListener('sync', event => {
  console.log('[SW] Background sync triggered:', event.tag);

  if (event.tag.startsWith('sync-')) {
    const action = event.tag.replace('sync-', '');

    event.waitUntil(
      syncUserData(action)
    );
  }
});

/**
 * Sync user data to server
 */
async function syncUserData(action) {
  try {
    // Get pending items from IndexedDB
    const db = await openDB();
    const items = await db.getAll('sync-queue');
    const pending = items.filter(item =>
      !item.synced && item.action === action
    );

    if (pending.length === 0) {
      console.log('[SW] No pending items to sync');
      return;
    }

    // Sync each item
    for (const item of pending) {
      try {
        const response = await fetch(`/api/sync/${item.action}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(item.data)
        });

        if (response.ok) {
          // Mark as synced
          item.synced = true;
          await db.put('sync-queue', item);

          console.log('[SW] Synced:', item.action, item.id);
        }
      } catch (error) {
        console.error('[SW] Sync failed for item:', item.id, error);
        // Keep in queue for next sync
      }
    }

    // Notify client
    const clients = await self.clients.matchAll();
    clients.forEach(client => {
      client.postMessage({
        type: 'SYNC_COMPLETE',
        payload: { action, count: pending.length }
      });
    });

  } catch (error) {
    console.error('[SW] Background sync failed:', error);
    throw error; // Retry
  }
}

/**
 * IndexedDB helper
 */
async function openDB() {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open('bible-reader-db', 1);

    request.onerror = () => reject(request.error);
    request.onsuccess = () => resolve(request.result);

    request.onupgradeneeded = (event) => {
      const db = event.target.result;

      if (!db.objectStoreNames.contains('sync-queue')) {
        db.createObjectStore('sync-queue', { keyPath: 'id' });
      }
    };
  });
}
```

---

## 6.8 Installation & Update Flow

### 6.8.1 App Install Prompt

```javascript
/**
 * Handle beforeinstallprompt event
 */
let deferredPrompt;

window.addEventListener('beforeinstallprompt', (e) => {
  console.log('[PWA] Install prompt ready');

  // Prevent default prompt
  e.preventDefault();

  // Save for later
  deferredPrompt = e;

  // Show custom install button
  showInstallButton();
});

/**
 * Show custom install UI
 */
function showInstallButton() {
  const installButton = document.getElementById('install-button');

  if (installButton) {
    installButton.style.display = 'block';

    installButton.addEventListener('click', async () => {
      if (!deferredPrompt) return;

      // Show install prompt
      deferredPrompt.prompt();

      // Wait for user response
      const { outcome } = await deferredPrompt.userChoice;

      console.log(`User ${outcome} the install prompt`);

      // Clear prompt
      deferredPrompt = null;

      // Hide button
      installButton.style.display = 'none';

      // Track analytics
      trackEvent('pwa_install', { outcome });
    });
  }
}

/**
 * Detect if app is installed
 */
window.addEventListener('appinstalled', (e) => {
  console.log('✅ PWA installed successfully');

  // Hide install button
  hideInstallButton();

  // Show welcome message
  showWelcomeMessage();

  // Track analytics
  trackEvent('pwa_installed');
});

/**
 * Check if running as installed PWA
 */
function isInstalledPWA() {
  // Check display mode
  if (window.matchMedia('(display-mode: standalone)').matches) {
    return true;
  }

  // Check navigator
  if (window.navigator.standalone === true) {
    return true; // iOS
  }

  return false;
}

// Usage
if (isInstalledPWA()) {
  console.log('Running as installed PWA');
  // Hide install prompts
  // Enable PWA-specific features
} else {
  console.log('Running in browser');
  // Show install prompts
}
```

### 6.8.2 Update Notification

```javascript
/**
 * Notify user of available update
 */
function showUpdateNotification() {
  const notification = document.createElement('div');
  notification.className = 'update-banner';
  notification.innerHTML = `
    <div class="update-content">
      <span class="update-icon">🎉</span>
      <div class="update-text">
        <strong>New version available!</strong>
        <p>Update now to get the latest features and improvements.</p>
      </div>
      <div class="update-actions">
        <button class="btn-update" onclick="updateApp()">
          Update Now
        </button>
        <button class="btn-dismiss" onclick="this.closest('.update-banner').remove()">
          Later
        </button>
      </div>
    </div>
  `;

  document.body.appendChild(notification);
}

/**
 * Update app to new version
 */
async function updateApp() {
  if (!navigator.serviceWorker.controller) return;

  const registration = await navigator.serviceWorker.ready;

  // Tell waiting SW to skip waiting
  registration.waiting?.postMessage({ type: 'SKIP_WAITING' });

  // Reload page when new SW activates
  navigator.serviceWorker.addEventListener('controllerchange', () => {
    window.location.reload();
  });
}
```

---

## 6.9 Performance Optimization

### 6.9.1 Resource Hints

```html
<!-- Preconnect to API -->
<link rel="preconnect" href="https://api.biblereader.com">
<link rel="dns-prefetch" href="https://api.biblereader.com">

<!-- Preload critical resources -->
<link rel="preload" href="/css/main.css" as="style">
<link rel="preload" href="/js/app.js" as="script">
<link rel="preload" href="/fonts/bible-serif.woff2" as="font" type="font/woff2" crossorigin>

<!-- Prefetch likely next pages -->
<link rel="prefetch" href="/api/bible/verses?book=john&chapter=3">
```

### 6.9.2 Lazy Loading

```javascript
/**
 * Lazy load images
 */
if ('IntersectionObserver' in window) {
  const imageObserver = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const img = entry.target;
        img.src = img.dataset.src;
        img.classList.remove('lazy');
        imageObserver.unobserve(img);
      }
    });
  });

  document.querySelectorAll('img.lazy').forEach(img => {
    imageObserver.observe(img);
  });
}
```

### 6.9.3 Code Splitting

```javascript
// Dynamic import for heavy features
async function loadTTSModule() {
  const { TTSPlayer } = await import('./modules/tts-player.js');
  return new TTSPlayer();
}

// Load on demand
document.getElementById('tts-button').addEventListener('click', async () => {
  const tts = await loadTTSModule();
  tts.play();
});
```

---

## 6.10 Testing & Debugging

### 6.10.1 Lighthouse Audit

**Run Lighthouse:**
```bash
# Chrome DevTools > Lighthouse tab
# Or CLI:
npm install -g lighthouse
lighthouse https://your-app.com --view
```

**Target Scores:**
```
Performance:    > 90
Accessibility:  > 90
Best Practices: > 90
SEO:           > 90
PWA:           100 ✅
```

### 6.10.2 Service Worker Debugging

**Chrome DevTools:**
```
Application Tab > Service Workers
- View registered service workers
- Update/Unregister
- Bypass for network (testing)
- View cache storage
```

**Useful Commands:**
```javascript
// In Console

// Check registration
navigator.serviceWorker.getRegistration()

// Unregister
navigator.serviceWorker.getRegistration().then(r => r.unregister())

// Clear all caches
caches.keys().then(keys => Promise.all(keys.map(k => caches.delete(k))))

// Check cache contents
caches.open('bible-reader-static-v1.0.0').then(c => c.keys())
```

### 6.10.3 Testing Offline Mode

**Manual Testing:**
```
1. Open Chrome DevTools
2. Network tab > Throttling dropdown
3. Select "Offline"
4. Reload page
5. Verify offline functionality
```

**Automated Testing:**
```javascript
// Using Playwright/Puppeteer
await page.setOfflineMode(true);
await page.goto('https://your-app.com');
// Assert offline page or cached content loads
```

---

## สรุป Section 6

Part 6 นี้ครอบคลุม:

✅ **PWA Fundamentals:**
- Why PWA for Bible Reader
- Complete checklist
- Web App Manifest (manifest.json)

✅ **Service Worker:**
- Registration & lifecycle
- Complete sw.js implementation
- Multiple caching strategies

✅ **Caching:**
- Cache-first, Network-first strategies
- Cache size management
- LRU cleanup

✅ **Offline Support:**
- Offline page design
- Network status detection
- Cached content access

✅ **Push Notifications:**
- Permission request
- Subscription setup
- Notification types (Quote, Reading Plan)
- Event handlers

✅ **Background Sync:**
- Queue system
- Sync implementation
- IndexedDB integration

✅ **Installation:**
- Install prompt handling
- Update notifications
- PWA detection

✅ **Performance:**
- Resource hints
- Lazy loading
- Code splitting

✅ **Testing:**
- Lighthouse audit
- Debugging tools
- Offline testing

Ready for implementation! 🚀
