# Part 7: API Specifications

## 7.1 API Overview & Architecture

### 7.1.1 API Design Principles

```
┌─────────────────────────────────────────────────────────────┐
│                    API DESIGN PRINCIPLES                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. RESTful Design                                          │
│     └─ Resource-based URLs                                  │
│     └─ HTTP verbs (GET, POST, PUT, DELETE)                 │
│     └─ Stateless communication                              │
│                                                              │
│  2. Consistent Response Format                              │
│     └─ JSON responses                                       │
│     └─ Standard error structure                             │
│     └─ Pagination for lists                                 │
│                                                              │
│  3. Security First                                          │
│     └─ HTTPS only                                           │
│     └─ JWT authentication                                   │
│     └─ Rate limiting                                        │
│                                                              │
│  4. Versioning                                              │
│     └─ URL-based versioning (/api/v1/...)                  │
│     └─ Backward compatibility                               │
│                                                              │
│  5. Documentation                                           │
│     └─ OpenAPI/Swagger specification                        │
│     └─ Interactive API explorer                             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 7.1.2 Base URL Structure

```
Production:  https://api.biblereader.com/v1
Staging:     https://api-staging.biblereader.com/v1
Development: http://localhost:3000/api/v1
```

### 7.1.3 Standard Response Format

**Success Response:**
```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "timestamp": "2025-01-14T10:30:00Z",
    "requestId": "req_abc123"
  }
}
```

**Paginated Response:**
```json
{
  "success": true,
  "data": [ ... ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": false
  },
  "meta": {
    "timestamp": "2025-01-14T10:30:00Z",
    "requestId": "req_abc123"
  }
}
```

**Error Response:**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request parameters",
    "details": [
      {
        "field": "chapter",
        "message": "Chapter must be a positive integer"
      }
    ]
  },
  "meta": {
    "timestamp": "2025-01-14T10:30:00Z",
    "requestId": "req_abc123"
  }
}
```

### 7.1.4 HTTP Status Codes

| Code | Meaning | Usage |
|------|---------|-------|
| **200** | OK | Successful GET, PUT |
| **201** | Created | Successful POST (resource created) |
| **204** | No Content | Successful DELETE |
| **400** | Bad Request | Invalid request parameters |
| **401** | Unauthorized | Missing or invalid authentication |
| **403** | Forbidden | Authenticated but not authorized |
| **404** | Not Found | Resource doesn't exist |
| **409** | Conflict | Resource conflict (e.g., duplicate) |
| **422** | Unprocessable Entity | Validation errors |
| **429** | Too Many Requests | Rate limit exceeded |
| **500** | Internal Server Error | Server-side error |

---

## 7.2 Authentication & Authorization

### 7.2.1 Authentication Flow

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Client    │      │   Server    │      │   Database  │
└──────┬──────┘      └──────┬──────┘      └──────┬──────┘
       │                    │                    │
       │ POST /auth/login   │                    │
       │ {email, password}  │                    │
       │───────────────────>│                    │
       │                    │ Verify credentials │
       │                    │───────────────────>│
       │                    │<───────────────────│
       │                    │                    │
       │ {accessToken,      │                    │
       │  refreshToken}     │                    │
       │<───────────────────│                    │
       │                    │                    │
       │ GET /api/v1/...    │                    │
       │ Authorization:     │                    │
       │ Bearer <token>     │                    │
       │───────────────────>│                    │
       │                    │ Validate JWT       │
       │                    │───────────────────>│
       │                    │<───────────────────│
       │ {data}             │                    │
       │<───────────────────│                    │
       │                    │                    │
```

### 7.2.2 Authentication Endpoints

#### **POST /auth/register**
Register a new user.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecureP@ss123",
  "fullName": "John Doe",
  "locale": "en"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "usr_abc123",
      "email": "user@example.com",
      "fullName": "John Doe",
      "locale": "en",
      "emailVerified": false,
      "createdAt": "2025-01-14T10:30:00Z"
    },
    "tokens": {
      "accessToken": "eyJhbGciOiJIUzI1NiIs...",
      "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
      "expiresIn": 3600
    }
  }
}
```

**Validation Rules:**
- `email`: Valid email format, unique
- `password`: Min 8 chars, 1 uppercase, 1 number, 1 special char
- `fullName`: 2-100 characters
- `locale`: Valid locale code (en, th, etc.)

---

#### **POST /auth/login**
Authenticate user and get tokens.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecureP@ss123"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "usr_abc123",
      "email": "user@example.com",
      "fullName": "John Doe",
      "locale": "en",
      "emailVerified": true,
      "lastLoginAt": "2025-01-14T10:30:00Z"
    },
    "tokens": {
      "accessToken": "eyJhbGciOiJIUzI1NiIs...",
      "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
      "expiresIn": 3600
    }
  }
}
```

**Error Response (401 Unauthorized):**
```json
{
  "success": false,
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Email or password is incorrect"
  }
}
```

---

#### **POST /auth/refresh**
Refresh access token.

**Request:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "expiresIn": 3600
  }
}
```

---

#### **POST /auth/logout**
Invalidate tokens.

**Headers:**
```
Authorization: Bearer <accessToken>
```

**Response (204 No Content)**

---

#### **POST /auth/forgot-password**
Request password reset.

**Request:**
```json
{
  "email": "user@example.com"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "message": "Password reset instructions sent to your email"
  }
}
```

---

#### **POST /auth/reset-password**
Reset password with token.

**Request:**
```json
{
  "token": "reset_token_abc123",
  "password": "NewSecureP@ss456"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "message": "Password has been reset successfully"
  }
}
```

---

#### **POST /auth/social**
Social login (Google, Facebook).

**Request:**
```json
{
  "provider": "google",
  "token": "oauth_token_from_provider"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "user": { ... },
    "tokens": { ... },
    "isNewUser": false
  }
}
```

---

### 7.2.3 JWT Token Structure

**Access Token Payload:**
```json
{
  "sub": "usr_abc123",
  "email": "user@example.com",
  "iat": 1705225800,
  "exp": 1705229400,
  "type": "access"
}
```

**Refresh Token Payload:**
```json
{
  "sub": "usr_abc123",
  "iat": 1705225800,
  "exp": 1705830600,
  "type": "refresh",
  "jti": "token_unique_id"
}
```

**Token Lifetimes:**
- Access Token: 1 hour (3600 seconds)
- Refresh Token: 7 days

---

## 7.3 Bible Content Endpoints

### 7.3.1 Bible Versions

#### **GET /bible/versions**
Get all available Bible versions.

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "code": "KJV",
      "name": "King James Version",
      "languageCode": "en",
      "languageName": "English",
      "description": "The King James Version...",
      "isPrimary": true,
      "hasTTSSupport": true,
      "copyright": "Public Domain",
      "totalBooks": 66,
      "totalChapters": 1189,
      "totalVerses": 31102
    },
    {
      "id": 2,
      "code": "THAI1",
      "name": "Thai Bible Standard Version",
      "languageCode": "th",
      "languageName": "Thai",
      "description": "ฉบับมาตรฐาน...",
      "isPrimary": false,
      "hasTTSSupport": true,
      "copyright": "...",
      "totalBooks": 66,
      "totalChapters": 1189,
      "totalVerses": 31102
    }
  ]
}
```

---

### 7.3.2 Bible Books

#### **GET /bible/books**
Get all books of the Bible.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `testament` | string | No | Filter: "OLD" or "NEW" |
| `lang` | string | No | Language code for book names (default: en) |

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "oldTestament": [
      {
        "id": 1,
        "number": 1,
        "code": "GEN",
        "name": "Genesis",
        "nameAbbr": "Gen",
        "totalChapters": 50,
        "group": "Pentateuch"
      },
      {
        "id": 2,
        "number": 2,
        "code": "EXO",
        "name": "Exodus",
        "nameAbbr": "Exo",
        "totalChapters": 40,
        "group": "Pentateuch"
      }
    ],
    "newTestament": [
      {
        "id": 40,
        "number": 40,
        "code": "MAT",
        "name": "Matthew",
        "nameAbbr": "Matt",
        "totalChapters": 28,
        "group": "Gospels"
      }
    ]
  }
}
```

---

#### **GET /bible/books/:bookCode**
Get details of a specific book.

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": 43,
    "number": 43,
    "code": "JOH",
    "name": "John",
    "nameAbbr": "John",
    "testament": "NEW",
    "totalChapters": 21,
    "group": "Gospels",
    "chapters": [
      { "chapter": 1, "verseCount": 51 },
      { "chapter": 2, "verseCount": 25 },
      { "chapter": 3, "verseCount": 36 }
    ]
  }
}
```

---

### 7.3.3 Bible Chapters & Verses

#### **GET /bible/verses**
Get verses for a specific chapter.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `version` | string | No | Version code (default: KJV) |
| `book` | string | Yes | Book code (e.g., "JOH") |
| `chapter` | number | Yes | Chapter number |

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "version": {
      "code": "KJV",
      "name": "King James Version",
      "languageCode": "en"
    },
    "book": {
      "code": "JOH",
      "name": "John",
      "testament": "NEW"
    },
    "chapter": 3,
    "totalVerses": 36,
    "navigation": {
      "prevChapter": { "book": "JOH", "chapter": 2 },
      "nextChapter": { "book": "JOH", "chapter": 4 }
    },
    "verses": [
      {
        "verse": 1,
        "text": "There was a man of the Pharisees, named Nicodemus, a ruler of the Jews:"
      },
      {
        "verse": 2,
        "text": "The same came to Jesus by night, and said unto him, Rabbi, we know that thou art a teacher come from God: for no man can do these miracles that thou doest, except God be with him."
      },
      {
        "verse": 16,
        "text": "For God so loved the world, that he gave his only begotten Son, that whosoever believeth in him should not perish, but have everlasting life."
      }
    ]
  }
}
```

---

#### **GET /bible/verse**
Get a specific verse.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `version` | string | No | Version code (default: KJV) |
| `book` | string | Yes | Book code |
| `chapter` | number | Yes | Chapter number |
| `verse` | number | Yes | Verse number |

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "version": "KJV",
    "book": "JOH",
    "bookName": "John",
    "chapter": 3,
    "verse": 16,
    "text": "For God so loved the world, that he gave his only begotten Son, that whosoever believeth in him should not perish, but have everlasting life.",
    "reference": "John 3:16"
  }
}
```

---

#### **GET /bible/parallel**
Get parallel verses (multiple versions).

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `versions` | string | Yes | Comma-separated version codes |
| `book` | string | Yes | Book code |
| `chapter` | number | Yes | Chapter number |

**Example:** `/bible/parallel?versions=KJV,THAI1&book=JOH&chapter=3`

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "book": "JOH",
    "bookName": "John",
    "chapter": 3,
    "versions": ["KJV", "THAI1"],
    "verses": [
      {
        "verse": 1,
        "texts": {
          "KJV": "There was a man of the Pharisees, named Nicodemus...",
          "THAI1": "มีคนหนึ่งในพวกฟาริสีชื่อนิโคเดมัส..."
        }
      },
      {
        "verse": 16,
        "texts": {
          "KJV": "For God so loved the world...",
          "THAI1": "เพราะพระเจ้าทรงรักโลกมาก..."
        }
      }
    ]
  }
}
```

---

### 7.3.4 Cross References

#### **GET /bible/cross-references**
Get cross references for a verse.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `book` | string | Yes | Book code |
| `chapter` | number | Yes | Chapter number |
| `verse` | number | Yes | Verse number |

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "source": {
      "book": "JOH",
      "chapter": 3,
      "verse": 16,
      "reference": "John 3:16"
    },
    "crossReferences": [
      {
        "book": "ROM",
        "bookName": "Romans",
        "chapter": 8,
        "verse": 32,
        "reference": "Romans 8:32",
        "text": "He that spared not his own Son...",
        "type": "theme"
      },
      {
        "book": "1JO",
        "bookName": "1 John",
        "chapter": 4,
        "verse": 9,
        "reference": "1 John 4:9",
        "text": "In this was manifested the love of God...",
        "type": "theme"
      }
    ]
  }
}
```

---

## 7.4 Search Endpoints

### 7.4.1 Full-Text Search

#### **GET /search**
Search Bible text.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | string | Yes | Search query |
| `version` | string | No | Version code (default: KJV) |
| `testament` | string | No | "OLD" or "NEW" |
| `books` | string | No | Comma-separated book codes |
| `page` | number | No | Page number (default: 1) |
| `limit` | number | No | Results per page (default: 20, max: 100) |

**Example:** `/search?q=love&version=KJV&testament=NEW&limit=10`

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "query": "love",
    "version": "KJV",
    "filters": {
      "testament": "NEW",
      "books": []
    },
    "results": [
      {
        "book": "JOH",
        "bookName": "John",
        "chapter": 3,
        "verse": 16,
        "text": "For God so <mark>love</mark>d the world, that he gave his only begotten Son...",
        "reference": "John 3:16",
        "relevanceScore": 0.95
      },
      {
        "book": "1JO",
        "bookName": "1 John",
        "chapter": 4,
        "verse": 8,
        "text": "He that <mark>love</mark>th not knoweth not God; for God is <mark>love</mark>.",
        "reference": "1 John 4:8",
        "relevanceScore": 0.92
      }
    ]
  },
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 348,
    "totalPages": 35
  }
}
```

---

## 7.5 User Data Endpoints

### 7.5.1 User Profile

#### **GET /users/me**
Get current user profile.

**Headers:**
```
Authorization: Bearer <accessToken>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "usr_abc123",
    "email": "user@example.com",
    "fullName": "John Doe",
    "avatarUrl": "https://...",
    "locale": "en",
    "timezone": "America/New_York",
    "emailVerified": true,
    "isPremium": false,
    "createdAt": "2025-01-01T00:00:00Z",
    "lastLoginAt": "2025-01-14T10:30:00Z"
  }
}
```

---

#### **PUT /users/me**
Update current user profile.

**Request:**
```json
{
  "fullName": "John Smith",
  "locale": "th",
  "timezone": "Asia/Bangkok"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "usr_abc123",
    "fullName": "John Smith",
    "locale": "th",
    "timezone": "Asia/Bangkok",
    "updatedAt": "2025-01-14T10:35:00Z"
  }
}
```

---

### 7.5.2 User Settings

#### **GET /users/me/settings**
Get user settings.

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "appearance": {
      "theme": "dark",
      "fontSize": "medium",
      "lineSpacing": "normal",
      "textAlign": "left"
    },
    "language": {
      "uiLanguage": "en",
      "defaultBibleVersion": "KJV",
      "parallelLeftVersion": "KJV",
      "parallelRightVersion": "THAI1",
      "syncParallel": true
    },
    "audio": {
      "ttsSpeed": 1.0,
      "ttsVoice": "en-US-male",
      "autoScroll": true,
      "highlightReading": true,
      "autoNextChapter": false
    },
    "notifications": {
      "enabled": true,
      "quoteEnabled": true,
      "quoteTime": "07:00",
      "planEnabled": true,
      "planTime": "20:00"
    }
  }
}
```

---

#### **PUT /users/me/settings**
Update user settings.

**Request:**
```json
{
  "appearance": {
    "theme": "light",
    "fontSize": "large"
  },
  "audio": {
    "ttsSpeed": 1.25
  }
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "message": "Settings updated successfully",
    "updatedAt": "2025-01-14T10:35:00Z"
  }
}
```

---

### 7.5.3 Reading Position

#### **GET /users/me/reading-position**
Get user's last reading position.

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "version": "KJV",
    "book": "JOH",
    "bookName": "John",
    "chapter": 3,
    "verse": 16,
    "readingMode": "single",
    "updatedAt": "2025-01-14T10:30:00Z"
  }
}
```

---

#### **PUT /users/me/reading-position**
Update reading position.

**Request:**
```json
{
  "version": "KJV",
  "book": "JOH",
  "chapter": 4,
  "verse": 1,
  "readingMode": "single"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "message": "Reading position updated"
  }
}
```

---

## 7.6 Study Tools Endpoints

### 7.6.1 Highlights

#### **GET /users/me/highlights**
Get all user highlights.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `book` | string | No | Filter by book code |
| `color` | string | No | Filter by color |
| `page` | number | No | Page number |
| `limit` | number | No | Items per page |
| `sort` | string | No | "date" or "book" |

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "hl_abc123",
      "book": "JOH",
      "bookName": "John",
      "chapter": 3,
      "verse": 16,
      "reference": "John 3:16",
      "text": "For God so loved the world...",
      "color": "yellow",
      "createdAt": "2025-01-10T08:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 47
  }
}
```

---

#### **POST /users/me/highlights**
Create a highlight.

**Request:**
```json
{
  "book": "JOH",
  "chapter": 3,
  "verse": 16,
  "color": "yellow"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "hl_abc123",
    "book": "JOH",
    "chapter": 3,
    "verse": 16,
    "color": "yellow",
    "createdAt": "2025-01-14T10:30:00Z"
  }
}
```

---

#### **PUT /users/me/highlights/:id**
Update a highlight.

**Request:**
```json
{
  "color": "green"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "hl_abc123",
    "color": "green",
    "updatedAt": "2025-01-14T10:35:00Z"
  }
}
```

---

#### **DELETE /users/me/highlights/:id**
Delete a highlight.

**Response (204 No Content)**

---

### 7.6.2 Bookmarks

#### **GET /users/me/bookmarks**
Get all user bookmarks.

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "bm_abc123",
      "book": "JOH",
      "bookName": "John",
      "chapter": 3,
      "verse": 16,
      "reference": "John 3:16",
      "title": "My Favorite Verse",
      "text": "For God so loved the world...",
      "createdAt": "2025-01-10T08:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 15
  }
}
```

---

#### **POST /users/me/bookmarks**
Create a bookmark.

**Request:**
```json
{
  "book": "JOH",
  "chapter": 3,
  "verse": 16,
  "title": "My Favorite Verse"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "bm_abc123",
    "book": "JOH",
    "chapter": 3,
    "verse": 16,
    "title": "My Favorite Verse",
    "createdAt": "2025-01-14T10:30:00Z"
  }
}
```

---

#### **DELETE /users/me/bookmarks/:id**
Delete a bookmark.

**Response (204 No Content)**

---

### 7.6.3 Notes

#### **GET /users/me/notes**
Get all user notes.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `book` | string | No | Filter by book |
| `search` | string | No | Search within notes |
| `page` | number | No | Page number |
| `limit` | number | No | Items per page |

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "note_abc123",
      "book": "JOH",
      "bookName": "John",
      "chapter": 3,
      "verse": 16,
      "reference": "John 3:16",
      "verseText": "For God so loved the world...",
      "noteText": "This is the most famous verse...",
      "createdAt": "2025-01-10T08:00:00Z",
      "updatedAt": "2025-01-12T09:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 23
  }
}
```

---

#### **POST /users/me/notes**
Create a note.

**Request:**
```json
{
  "book": "JOH",
  "chapter": 3,
  "verse": 16,
  "noteText": "This is the most famous verse in the Bible..."
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "note_abc123",
    "book": "JOH",
    "chapter": 3,
    "verse": 16,
    "noteText": "This is the most famous verse...",
    "createdAt": "2025-01-14T10:30:00Z"
  }
}
```

---

#### **PUT /users/me/notes/:id**
Update a note.

**Request:**
```json
{
  "noteText": "Updated note content..."
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "note_abc123",
    "noteText": "Updated note content...",
    "updatedAt": "2025-01-14T10:35:00Z"
  }
}
```

---

#### **DELETE /users/me/notes/:id**
Delete a note.

**Response (204 No Content)**

---

## 7.7 Reading Plans Endpoints

### 7.7.1 Available Plans

#### **GET /reading-plans**
Get all available reading plans.

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "code": "bible-1-year",
      "name": "Read the Bible in One Year",
      "description": "Complete Bible reading...",
      "totalDays": 365,
      "difficulty": "medium",
      "category": "complete"
    },
    {
      "id": 2,
      "code": "nt-90-days",
      "name": "New Testament in 90 Days",
      "description": "Read through the entire...",
      "totalDays": 90,
      "difficulty": "hard",
      "category": "testament"
    }
  ]
}
```

---

#### **GET /reading-plans/:code**
Get plan details with schedule.

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "code": "bible-1-year",
    "name": "Read the Bible in One Year",
    "description": "...",
    "totalDays": 365,
    "schedule": [
      {
        "day": 1,
        "readings": [
          { "book": "GEN", "startChapter": 1, "endChapter": 3 },
          { "book": "MAT", "startChapter": 1, "endChapter": 1 }
        ]
      },
      {
        "day": 2,
        "readings": [
          { "book": "GEN", "startChapter": 4, "endChapter": 6 },
          { "book": "MAT", "startChapter": 2, "endChapter": 2 }
        ]
      }
    ]
  }
}
```

---

### 7.7.2 User Plans

#### **GET /users/me/reading-plans**
Get user's active reading plans.

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "up_abc123",
      "plan": {
        "code": "bible-1-year",
        "name": "Read the Bible in One Year"
      },
      "startDate": "2025-01-01",
      "currentDay": 14,
      "totalDays": 365,
      "status": "active",
      "progressPercent": 3.8,
      "streak": 7,
      "todayReading": {
        "day": 14,
        "readings": [
          { "book": "GEN", "chapters": "40-42" },
          { "book": "MAT", "chapters": "14" }
        ],
        "completed": false
      }
    }
  ]
}
```

---

#### **POST /users/me/reading-plans**
Start a new reading plan.

**Request:**
```json
{
  "planCode": "bible-1-year",
  "startDate": "2025-01-15"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "up_xyz789",
    "planCode": "bible-1-year",
    "startDate": "2025-01-15",
    "currentDay": 1,
    "status": "active"
  }
}
```

---

#### **PUT /users/me/reading-plans/:id/progress**
Update reading plan progress.

**Request:**
```json
{
  "day": 14,
  "completed": true
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "day": 14,
    "completed": true,
    "completedAt": "2025-01-14T10:30:00Z",
    "currentDay": 15,
    "streak": 8
  }
}
```

---

## 7.8 Quote of the Day Endpoints

#### **GET /quote-of-the-day**
Get today's quote.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `version` | string | No | Bible version (default: user's default or KJV) |
| `date` | string | No | Specific date (YYYY-MM-DD) |

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "date": "2025-01-14",
    "book": "JOH",
    "bookName": "John",
    "chapter": 3,
    "verse": 16,
    "reference": "John 3:16",
    "text": "For God so loved the world, that he gave his only begotten Son, that whosoever believeth in him should not perish, but have everlasting life.",
    "version": "KJV",
    "category": "love"
  }
}
```

---

## 7.9 Push Notification Endpoints

#### **GET /push/vapid-key**
Get VAPID public key.

**Response (200 OK):**
```
BEl62iUYgUivxIkv69yViEuiBIa-Ib9-SkvMeAtA3LFgDzkrxZJjSgSnfckjBJuBkr3qBUYIHBQFLXYp5Nksh8U
```

---

#### **POST /push/subscribe**
Subscribe to push notifications.

**Request:**
```json
{
  "endpoint": "https://fcm.googleapis.com/fcm/send/...",
  "keys": {
    "p256dh": "BNcRdreALRFXTkOOUHK1EtK2wtaz5Ry4YfYCA_0QTpQtUbVlUls0VJXg7A8u-Ts1XbjhazAkj7I99e8QcYP7DkM",
    "auth": "tBHItJI5svbpez7KI4CCXg"
  },
  "deviceType": "mobile",
  "browser": "chrome"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "push_abc123",
    "message": "Successfully subscribed to push notifications"
  }
}
```

---

#### **DELETE /push/subscribe**
Unsubscribe from push notifications.

**Request:**
```json
{
  "endpoint": "https://fcm.googleapis.com/fcm/send/..."
}
```

**Response (204 No Content)**

---

## 7.10 Statistics Endpoints

#### **GET /users/me/statistics**
Get user reading statistics.

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "reading": {
      "totalChaptersRead": 156,
      "totalVersesRead": 4521,
      "totalReadingTime": 28800,
      "booksCompleted": 3,
      "otChaptersRead": 89,
      "ntChaptersRead": 67
    },
    "study": {
      "totalHighlights": 47,
      "totalBookmarks": 15,
      "totalNotes": 23
    },
    "engagement": {
      "currentStreak": 7,
      "longestStreak": 14,
      "lastReadDate": "2025-01-14",
      "totalReadingDays": 45
    },
    "plans": {
      "plansCompleted": 0,
      "plansActive": 1
    }
  }
}
```

---

## 7.11 Sync Endpoints

#### **POST /sync**
Sync offline data.

**Request:**
```json
{
  "lastSyncTimestamp": "2025-01-14T00:00:00Z",
  "changes": {
    "highlights": {
      "created": [
        { "book": "PSA", "chapter": 23, "verse": 1, "color": "green", "clientId": "temp_1" }
      ],
      "updated": [],
      "deleted": ["hl_old123"]
    },
    "bookmarks": {
      "created": [],
      "updated": [],
      "deleted": []
    },
    "notes": {
      "created": [
        { "book": "ROM", "chapter": 8, "verse": 28, "noteText": "...", "clientId": "temp_2" }
      ],
      "updated": [],
      "deleted": []
    },
    "readingHistory": [
      { "book": "JOH", "chapter": 3, "date": "2025-01-14", "completed": true }
    ]
  }
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "syncTimestamp": "2025-01-14T10:35:00Z",
    "idMappings": {
      "temp_1": "hl_new456",
      "temp_2": "note_new789"
    },
    "serverChanges": {
      "highlights": [],
      "bookmarks": [],
      "notes": [],
      "settings": {
        "theme": "dark"
      }
    },
    "conflicts": []
  }
}
```

---

## 7.12 Rate Limiting

### 7.12.1 Rate Limit Headers

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1705226400
```

### 7.12.2 Rate Limits by Endpoint

| Endpoint Category | Limit | Window |
|-------------------|-------|--------|
| Authentication | 10 requests | 15 minutes |
| Bible Content (read) | 1000 requests | 1 hour |
| Search | 100 requests | 1 hour |
| User Data (write) | 100 requests | 1 hour |
| Push Subscription | 10 requests | 1 hour |

### 7.12.3 Rate Limit Exceeded Response

**Response (429 Too Many Requests):**
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again later.",
    "retryAfter": 3600
  }
}
```

---

## 7.13 Error Codes

| Code | Description |
|------|-------------|
| `VALIDATION_ERROR` | Invalid request parameters |
| `INVALID_CREDENTIALS` | Wrong email or password |
| `TOKEN_EXPIRED` | JWT token has expired |
| `TOKEN_INVALID` | JWT token is invalid |
| `UNAUTHORIZED` | Authentication required |
| `FORBIDDEN` | Not authorized for this action |
| `RESOURCE_NOT_FOUND` | Requested resource doesn't exist |
| `RESOURCE_CONFLICT` | Resource already exists |
| `RATE_LIMIT_EXCEEDED` | Too many requests |
| `INTERNAL_ERROR` | Server-side error |

---

## สรุป Section 7

Part 7 นี้ครอบคลุม:

✅ **API Architecture:**
- RESTful design principles
- Standard response formats
- HTTP status codes

✅ **Authentication:**
- Register, Login, Logout
- JWT tokens (access + refresh)
- Password reset
- Social login

✅ **Bible Content:**
- Versions, Books, Chapters, Verses
- Parallel view
- Cross references

✅ **Search:**
- Full-text search with filters
- Pagination

✅ **User Data:**
- Profile & Settings
- Reading position

✅ **Study Tools:**
- Highlights (CRUD)
- Bookmarks (CRUD)
- Notes (CRUD)

✅ **Reading Plans:**
- Available plans
- User plans & progress

✅ **Additional:**
- Quote of the Day
- Push notifications
- Statistics
- Sync
- Rate limiting
- Error codes

Ready for implementation! 🚀
