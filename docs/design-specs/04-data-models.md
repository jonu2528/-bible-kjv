# Part 4: Data Models (Conceptual Level)

## 4.1 ภาพรวมสถาปัตยกรรมข้อมูล (Data Architecture Overview)

### 4.1.1 Database Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│                    APPLICATION LAYER                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────┐      ┌─────────────────────────┐         │
│  │  BIBLE CONTENT   │      │   USER DATA             │         │
│  │  DATABASE        │      │   DATABASE              │         │
│  │                  │      │                         │         │
│  │  • Versions      │      │  • Users                │         │
│  │  • Books         │      │  • Highlights           │         │
│  │  • Chapters      │      │  • Bookmarks            │         │
│  │  • Verses        │      │  • Notes                │         │
│  │  • Cross Refs    │      │  • Reading Plans        │         │
│  │                  │      │  • Progress             │         │
│  │  (Read-mostly)   │      │  (Read-write)           │         │
│  └──────────────────┘      └─────────────────────────┘         │
│           │                           │                         │
│           └───────────┬───────────────┘                         │
│                       │                                         │
│              ┌────────▼─────────┐                               │
│              │  CACHE LAYER     │                               │
│              │  (Redis/Memory)  │                               │
│              └──────────────────┘                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 4.1.2 Storage Considerations

| Data Type | Storage Strategy | Reason |
|-----------|-----------------|--------|
| **Bible Content** | Relational DB (PostgreSQL/MySQL) | Structured, normalized, complex queries |
| **User Data** | Relational DB + JSON fields | Mix of structured and flexible data |
| **User Sessions** | Redis/Memory Cache | Fast access, temporary |
| **Offline Cache** | IndexedDB (Client-side) | PWA offline support |
| **Static Assets** | CDN | Fast delivery globally |

---

## 4.2 Bible Content Models

### 4.2.1 Bible Versions (bible_versions)

เก็บข้อมูลเวอร์ชัน/ภาษาต่างๆ ของ Bible

```sql
Table: bible_versions
─────────────────────────────────────────────────────
id                  UUID/INT         PRIMARY KEY
version_code        VARCHAR(20)      UNIQUE, NOT NULL  -- "KJV", "NIV", "THAI1"
version_name        VARCHAR(100)     NOT NULL          -- "King James Version"
language_code       VARCHAR(10)      NOT NULL          -- "en", "th", "es"
language_name       VARCHAR(50)      NOT NULL          -- "English", "Thai"
description         TEXT             NULL
is_primary          BOOLEAN          DEFAULT FALSE     -- KJV is primary
is_active           BOOLEAN          DEFAULT TRUE
has_tts_support     BOOLEAN          DEFAULT FALSE
copyright_info      TEXT             NULL
publisher           VARCHAR(200)     NULL
publication_year    INT              NULL
sort_order          INT              DEFAULT 0
created_at          TIMESTAMP        DEFAULT NOW()
updated_at          TIMESTAMP        DEFAULT NOW()

INDEXES:
  - idx_version_code ON (version_code)
  - idx_language_code ON (language_code)
  - idx_is_active ON (is_active)
```

**Sample Data:**
```sql
(1, 'KJV', 'King James Version', 'en', 'English',
 'The King James Version...', TRUE, TRUE, TRUE,
 'Public Domain', NULL, 1611, 1)

(2, 'THAI1', 'Thai Bible Standard Version', 'th', 'Thai',
 'ฉบับมาตรฐาน พระคริสตธรรมคัมภีร์ภาษาไทย',
 FALSE, TRUE, TRUE, '...', 'Thai Bible Society', 2011, 2)
```

---

### 4.2.2 Bible Books (bible_books)

เก็บข้อมูลหนังสือทั้งหมดใน Bible (66 เล่ม)

```sql
Table: bible_books
─────────────────────────────────────────────────────
id                  INT              PRIMARY KEY AUTO_INCREMENT
book_number         INT              UNIQUE, NOT NULL  -- 1-66
book_code           VARCHAR(10)      UNIQUE, NOT NULL  -- "GEN", "EXO", "MAT"
book_name_en        VARCHAR(50)      NOT NULL          -- "Genesis", "Matthew"
book_name_abbr      VARCHAR(10)      NOT NULL          -- "Gen", "Matt"
testament           ENUM             NOT NULL          -- 'OLD', 'NEW'
total_chapters      INT              NOT NULL
sort_order          INT              NOT NULL
book_group          VARCHAR(50)      NULL              -- "Pentateuch", "Gospels"
created_at          TIMESTAMP        DEFAULT NOW()

INDEXES:
  - idx_book_number ON (book_number)
  - idx_book_code ON (book_code)
  - idx_testament ON (testament)
```

**Sample Data:**
```sql
(1,  1,  'GEN', 'Genesis',   'Gen',  'OLD', 50,  1,  'Pentateuch'),
(40, 40, 'MAT', 'Matthew',   'Matt', 'NEW', 28,  40, 'Gospels'),
(43, 43, 'JOH', 'John',      'John', 'NEW', 21,  43, 'Gospels'),
(66, 66, 'REV', 'Revelation','Rev',  'NEW', 22,  66, 'Prophecy')
```

---

### 4.2.3 Bible Book Names (bible_book_names)

เก็บชื่อหนังสือในหลายภาษา (for i18n)

```sql
Table: bible_book_names
─────────────────────────────────────────────────────
id                  INT              PRIMARY KEY AUTO_INCREMENT
book_id             INT              NOT NULL, FK -> bible_books.id
language_code       VARCHAR(10)      NOT NULL          -- "en", "th"
book_name           VARCHAR(100)     NOT NULL          -- Translated name
book_name_abbr      VARCHAR(20)      NULL

UNIQUE INDEX: (book_id, language_code)
```

**Sample Data:**
```sql
(1, 1, 'en', 'Genesis', 'Gen'),
(2, 1, 'th', 'ปฐมกาล', 'ปฐก.'),
(3, 43, 'en', 'John', 'John'),
(4, 43, 'th', 'ยอห์น', 'ยน.')
```

---

### 4.2.4 Bible Verses (bible_verses)

เก็บข้อความทุกข้อของ Bible แต่ละเวอร์ชัน

```sql
Table: bible_verses
─────────────────────────────────────────────────────
id                  BIGINT           PRIMARY KEY AUTO_INCREMENT
version_id          INT              NOT NULL, FK -> bible_versions.id
book_id             INT              NOT NULL, FK -> bible_books.id
chapter             INT              NOT NULL          -- 1-150 (Psalms)
verse               INT              NOT NULL          -- 1-176 (Psalm 119)
verse_text          TEXT             NOT NULL
verse_text_search   TEXT             NULL              -- Normalized for search
created_at          TIMESTAMP        DEFAULT NOW()
updated_at          TIMESTAMP        DEFAULT NOW()

UNIQUE INDEX: (version_id, book_id, chapter, verse)

INDEXES:
  - idx_version_book_chapter ON (version_id, book_id, chapter)
  - idx_book_chapter_verse ON (book_id, chapter, verse)
  - idx_verse_text_search ON (verse_text_search) FULLTEXT
```

**Sample Data:**
```sql
-- KJV (version_id=1)
(1, 1, 1, 1, 1,
 'In the beginning God created the heaven and the earth.',
 'in the beginning god created the heaven and the earth'),

(2, 1, 43, 3, 16,
 'For God so loved the world, that he gave his only begotten Son...',
 'for god so loved the world that he gave his only begotten son...'),

-- Thai (version_id=2)
(100001, 2, 1, 1, 1,
 'ในปฐมกาลพระเจ้าทรงสร้างฟ้าและแผ่นดิน',
 'ในปฐมกาลพระเจ้าทรงสร้างฟ้าและแผ่นดิน')
```

**Storage Estimates:**
- KJV: ~31,102 verses × avg 100 chars = ~3.1 MB per version
- With 10 versions: ~31 MB
- With indexes: ~100-200 MB total

---

### 4.2.5 Cross References (cross_references)

เก็บข้อมูลการอ้างอิงระหว่าง verse

```sql
Table: cross_references
─────────────────────────────────────────────────────
id                  BIGINT           PRIMARY KEY AUTO_INCREMENT
from_book_id        INT              NOT NULL, FK -> bible_books.id
from_chapter        INT              NOT NULL
from_verse          INT              NOT NULL
to_book_id          INT              NOT NULL, FK -> bible_books.id
to_chapter          INT              NOT NULL
to_verse            INT              NOT NULL
reference_type      VARCHAR(50)      NULL              -- "parallel", "quotation"
notes               TEXT             NULL
created_at          TIMESTAMP        DEFAULT NOW()

INDEXES:
  - idx_from_verse ON (from_book_id, from_chapter, from_verse)
  - idx_to_verse ON (to_book_id, to_chapter, to_verse)
```

---

## 4.3 User Models

### 4.3.1 Users (users)

เก็บข้อมูลผู้ใช้หลัก

```sql
Table: users
─────────────────────────────────────────────────────
id                  UUID/BIGINT      PRIMARY KEY
username            VARCHAR(50)      UNIQUE, NULL
email               VARCHAR(255)     UNIQUE, NOT NULL
email_verified      BOOLEAN          DEFAULT FALSE
password_hash       VARCHAR(255)     NULL              -- NULL if social login
full_name           VARCHAR(100)     NULL
avatar_url          VARCHAR(500)     NULL
timezone            VARCHAR(50)      DEFAULT 'UTC'
locale              VARCHAR(10)      DEFAULT 'en'
is_active           BOOLEAN          DEFAULT TRUE
is_premium          BOOLEAN          DEFAULT FALSE
last_login_at       TIMESTAMP        NULL
created_at          TIMESTAMP        DEFAULT NOW()
updated_at          TIMESTAMP        DEFAULT NOW()

INDEXES:
  - idx_email ON (email)
  - idx_username ON (username)
```

---

### 4.3.2 User Settings (user_settings)

เก็บการตั้งค่าของผู้ใช้

```sql
Table: user_settings
─────────────────────────────────────────────────────
id                  BIGINT           PRIMARY KEY AUTO_INCREMENT
user_id             UUID/BIGINT      UNIQUE, NOT NULL, FK -> users.id

-- Appearance
theme               VARCHAR(20)      DEFAULT 'light'   -- 'light','dark','sepia','auto'
font_size           VARCHAR(20)      DEFAULT 'medium'  -- 'small','medium','large'
line_spacing        VARCHAR(20)      DEFAULT 'normal'  -- 'compact','normal','relaxed'
text_align          VARCHAR(20)      DEFAULT 'left'    -- 'left','justified'

-- Language & Version
ui_language         VARCHAR(10)      DEFAULT 'en'
default_version_id  INT              NULL, FK -> bible_versions.id
parallel_left_id    INT              NULL, FK -> bible_versions.id
parallel_right_id   INT              NULL, FK -> bible_versions.id
sync_parallel       BOOLEAN          DEFAULT TRUE

-- Audio/TTS
tts_speed           DECIMAL(3,2)     DEFAULT 1.00      -- 0.50 - 2.00
tts_voice           VARCHAR(50)      NULL
auto_scroll         BOOLEAN          DEFAULT TRUE
highlight_reading   BOOLEAN          DEFAULT TRUE
auto_next_chapter   BOOLEAN          DEFAULT FALSE

-- Notifications
notifications_enabled   BOOLEAN      DEFAULT FALSE
quote_time          TIME             DEFAULT '07:00:00'
quote_enabled       BOOLEAN          DEFAULT FALSE
plan_time           TIME             DEFAULT '20:00:00'
plan_enabled        BOOLEAN          DEFAULT FALSE

created_at          TIMESTAMP        DEFAULT NOW()
updated_at          TIMESTAMP        DEFAULT NOW()
```

---

### 4.3.3 User Reading Position (user_reading_position)

เก็บตำแหน่งการอ่านล่าสุด (Continue Reading)

```sql
Table: user_reading_position
─────────────────────────────────────────────────────
id                  BIGINT           PRIMARY KEY AUTO_INCREMENT
user_id             UUID/BIGINT      NOT NULL, FK -> users.id
version_id          INT              NOT NULL, FK -> bible_versions.id
book_id             INT              NOT NULL, FK -> bible_books.id
chapter             INT              NOT NULL
verse               INT              NOT NULL
scroll_position     INT              DEFAULT 0
reading_mode        VARCHAR(20)      DEFAULT 'single'  -- 'single','parallel','focus'
updated_at          TIMESTAMP        DEFAULT NOW()

UNIQUE INDEX: (user_id, version_id)
```

---

## 4.4 User Study Tools Models

### 4.4.1 Highlights (user_highlights)

เก็บการ highlight ของผู้ใช้

```sql
Table: user_highlights
─────────────────────────────────────────────────────
id                  BIGINT           PRIMARY KEY AUTO_INCREMENT
user_id             UUID/BIGINT      NOT NULL, FK -> users.id
book_id             INT              NOT NULL, FK -> bible_books.id
chapter             INT              NOT NULL
verse               INT              NOT NULL
color               VARCHAR(20)      NOT NULL          -- 'yellow','green','blue'
created_at          TIMESTAMP        DEFAULT NOW()
updated_at          TIMESTAMP        DEFAULT NOW()

-- Not tied to specific version - applies across all versions
UNIQUE INDEX: (user_id, book_id, chapter, verse)

INDEXES:
  - idx_user_id ON (user_id)
  - idx_color ON (color)
  - idx_verse_reference ON (book_id, chapter, verse)
```

---

### 4.4.2 Bookmarks (user_bookmarks)

เก็บ bookmark ของผู้ใช้

```sql
Table: user_bookmarks
─────────────────────────────────────────────────────
id                  BIGINT           PRIMARY KEY AUTO_INCREMENT
user_id             UUID/BIGINT      NOT NULL, FK -> users.id
book_id             INT              NOT NULL, FK -> bible_books.id
chapter             INT              NOT NULL
verse               INT              NOT NULL
title               VARCHAR(200)     NULL              -- Custom title
notes               TEXT             NULL
created_at          TIMESTAMP        DEFAULT NOW()
updated_at          TIMESTAMP        DEFAULT NOW()

UNIQUE INDEX: (user_id, book_id, chapter, verse)

INDEXES:
  - idx_user_id ON (user_id)
  - idx_created_at ON (created_at)
```

---

### 4.4.3 Notes (user_notes)

เก็บบันทึกส่วนตัวของผู้ใช้

```sql
Table: user_notes
─────────────────────────────────────────────────────
id                  BIGINT           PRIMARY KEY AUTO_INCREMENT
user_id             UUID/BIGINT      NOT NULL, FK -> users.id
book_id             INT              NOT NULL, FK -> bible_books.id
chapter             INT              NOT NULL
verse               INT              NOT NULL
note_text           TEXT             NOT NULL
is_private          BOOLEAN          DEFAULT TRUE
created_at          TIMESTAMP        DEFAULT NOW()
updated_at          TIMESTAMP        DEFAULT NOW()

UNIQUE INDEX: (user_id, book_id, chapter, verse)

INDEXES:
  - idx_user_id ON (user_id)
  - idx_verse_reference ON (book_id, chapter, verse)
  - idx_note_text ON (note_text) FULLTEXT
```

---

### 4.4.4 Reading History (user_reading_history)

เก็บประวัติการอ่าน

```sql
Table: user_reading_history
─────────────────────────────────────────────────────
id                  BIGINT           PRIMARY KEY AUTO_INCREMENT
user_id             UUID/BIGINT      NOT NULL, FK -> users.id
version_id          INT              NOT NULL, FK -> bible_versions.id
book_id             INT              NOT NULL, FK -> bible_books.id
chapter             INT              NOT NULL
read_date           DATE             NOT NULL
read_duration       INT              NULL              -- Seconds
completed           BOOLEAN          DEFAULT FALSE
created_at          TIMESTAMP        DEFAULT NOW()

UNIQUE INDEX: (user_id, book_id, chapter, read_date)

INDEXES:
  - idx_user_id ON (user_id)
  - idx_read_date ON (read_date)
```

---

## 4.5 Reading Plans Models

### 4.5.1 Reading Plans (reading_plans)

แผนการอ่านที่มีให้เลือก

```sql
Table: reading_plans
─────────────────────────────────────────────────────
id                  INT              PRIMARY KEY AUTO_INCREMENT
plan_code           VARCHAR(50)      UNIQUE, NOT NULL  -- "bible-1-year"
plan_name           VARCHAR(100)     NOT NULL
description         TEXT             NULL
total_days          INT              NOT NULL
difficulty          VARCHAR(20)      DEFAULT 'medium'  -- 'easy','medium','hard'
category            VARCHAR(50)      NULL              -- 'complete','testament'
is_active           BOOLEAN          DEFAULT TRUE
sort_order          INT              DEFAULT 0
created_at          TIMESTAMP        DEFAULT NOW()
updated_at          TIMESTAMP        DEFAULT NOW()

INDEXES:
  - idx_plan_code ON (plan_code)
  - idx_category ON (category)
```

**Sample Data:**
```sql
(1, 'bible-1-year', 'Read the Bible in One Year',
 'Complete Bible reading plan spread over 365 days',
 365, 'medium', 'complete', TRUE, 1),

(2, 'nt-90-days', 'New Testament in 90 Days',
 'Read through the entire New Testament in 90 days',
 90, 'hard', 'testament', TRUE, 2)
```

---

### 4.5.2 Reading Plan Days (reading_plan_days)

กำหนดการอ่านแต่ละวันของแต่ละแผน

```sql
Table: reading_plan_days
─────────────────────────────────────────────────────
id                  BIGINT           PRIMARY KEY AUTO_INCREMENT
plan_id             INT              NOT NULL, FK -> reading_plans.id
day_number          INT              NOT NULL          -- 1-365
book_id             INT              NOT NULL, FK -> bible_books.id
start_chapter       INT              NOT NULL
end_chapter         INT              NOT NULL
start_verse         INT              NULL
end_verse           INT              NULL
reading_order       INT              DEFAULT 1

UNIQUE INDEX: (plan_id, day_number, reading_order)

INDEXES:
  - idx_plan_id ON (plan_id)
  - idx_day_number ON (day_number)
```

---

### 4.5.3 User Reading Plans (user_reading_plans)

แผนการอ่านที่ผู้ใช้เลือกใช้

```sql
Table: user_reading_plans
─────────────────────────────────────────────────────
id                  BIGINT           PRIMARY KEY AUTO_INCREMENT
user_id             UUID/BIGINT      NOT NULL, FK -> users.id
plan_id             INT              NOT NULL, FK -> reading_plans.id
start_date          DATE             NOT NULL
current_day         INT              DEFAULT 1
status              VARCHAR(20)      DEFAULT 'active'  -- 'active','paused','completed'
completed_at        TIMESTAMP        NULL
notes               TEXT             NULL
created_at          TIMESTAMP        DEFAULT NOW()
updated_at          TIMESTAMP        DEFAULT NOW()

INDEXES:
  - idx_user_id ON (user_id)
  - idx_status ON (status)
```

---

### 4.5.4 User Reading Plan Progress (user_reading_plan_progress)

ติดตามความคืบหน้าแต่ละวัน

```sql
Table: user_reading_plan_progress
─────────────────────────────────────────────────────
id                  BIGINT           PRIMARY KEY AUTO_INCREMENT
user_plan_id        BIGINT           NOT NULL, FK -> user_reading_plans.id
day_number          INT              NOT NULL
completed           BOOLEAN          DEFAULT FALSE
completed_at        TIMESTAMP        NULL
skipped             BOOLEAN          DEFAULT FALSE
notes               TEXT             NULL

UNIQUE INDEX: (user_plan_id, day_number)

INDEXES:
  - idx_user_plan_id ON (user_plan_id)
  - idx_completed ON (completed)
```

---

## 4.6 Quote of the Day Models

### 4.6.1 Quote Pool (quote_pool)

รายการ verse ที่ใช้เป็น Quote of the Day

```sql
Table: quote_pool
─────────────────────────────────────────────────────
id                  INT              PRIMARY KEY AUTO_INCREMENT
book_id             INT              NOT NULL, FK -> bible_books.id
chapter             INT              NOT NULL
verse               INT              NOT NULL
category            VARCHAR(50)      NULL              -- 'faith','love','hope'
weight              INT              DEFAULT 1
is_active           BOOLEAN          DEFAULT TRUE
added_by            VARCHAR(50)      DEFAULT 'system'
created_at          TIMESTAMP        DEFAULT NOW()

INDEXES:
  - idx_verse_reference ON (book_id, chapter, verse)
  - idx_category ON (category)
```

**Sample Data:**
```sql
(1, 43, 3, 16, 'love', 10, TRUE, 'curated'),      -- John 3:16
(2, 19, 23, 1, 'comfort', 8, TRUE, 'curated'),    -- Psalm 23:1
(3, 50, 4, 13, 'strength', 7, TRUE, 'curated')    -- Phil 4:13
```

---

### 4.6.2 Quote Schedule (quote_schedule)

กำหนดการ quote แต่ละวัน

```sql
Table: quote_schedule
─────────────────────────────────────────────────────
id                  BIGINT           PRIMARY KEY AUTO_INCREMENT
quote_date          DATE             UNIQUE, NOT NULL
book_id             INT              NOT NULL, FK -> bible_books.id
chapter             INT              NOT NULL
verse               INT              NOT NULL
selected_by         VARCHAR(50)      DEFAULT 'auto'    -- 'auto','manual'
created_at          TIMESTAMP        DEFAULT NOW()

INDEXES:
  - idx_quote_date ON (quote_date)
```

---

## 4.7 Analytics & Statistics Models

### 4.7.1 User Statistics (user_statistics)

สถิติการอ่านของผู้ใช้

```sql
Table: user_statistics
─────────────────────────────────────────────────────
id                  BIGINT           PRIMARY KEY AUTO_INCREMENT
user_id             UUID/BIGINT      UNIQUE, NOT NULL, FK -> users.id

-- Reading Stats
total_chapters_read     INT          DEFAULT 0
total_verses_read       INT          DEFAULT 0
total_reading_time      INT          DEFAULT 0         -- Seconds
books_completed         INT          DEFAULT 0

-- Study Stats
total_highlights        INT          DEFAULT 0
total_bookmarks         INT          DEFAULT 0
total_notes             INT          DEFAULT 0

-- Engagement Stats
current_streak          INT          DEFAULT 0         -- Days
longest_streak          INT          DEFAULT 0
last_read_date          DATE         NULL
total_reading_days      INT          DEFAULT 0

-- Plan Stats
plans_completed         INT          DEFAULT 0
plans_active            INT          DEFAULT 0

updated_at          TIMESTAMP        DEFAULT NOW()

INDEXES:
  - idx_user_id ON (user_id)
  - idx_current_streak ON (current_streak)
```

---

## 4.8 PWA & Offline Support Models

### 4.8.1 Push Subscriptions (push_subscriptions)

เก็บ push notification subscriptions

```sql
Table: push_subscriptions
─────────────────────────────────────────────────────
id                  BIGINT           PRIMARY KEY AUTO_INCREMENT
user_id             UUID/BIGINT      NOT NULL, FK -> users.id
endpoint            TEXT             NOT NULL
p256dh_key          TEXT             NOT NULL
auth_key            TEXT             NOT NULL
device_type         VARCHAR(20)      NULL              -- 'desktop','mobile'
browser             VARCHAR(50)      NULL              -- 'chrome','firefox'
is_active           BOOLEAN          DEFAULT TRUE
created_at          TIMESTAMP        DEFAULT NOW()
last_used_at        TIMESTAMP        NULL

INDEXES:
  - idx_user_id ON (user_id)
  - idx_is_active ON (is_active)
```

---

## 4.9 Data Relationships Diagram

```
BIBLE CONTENT
├── bible_versions ──→ bible_verses
├── bible_books ─────→ bible_verses
│   └─→ bible_book_names
└── cross_references

USER SYSTEM
├── users ──┬─→ user_settings
│           ├─→ user_reading_position
│           ├─→ user_highlights ────→ bible_books
│           ├─→ user_bookmarks ─────→ bible_books
│           ├─→ user_notes ─────────→ bible_books
│           ├─→ user_reading_history
│           ├─→ user_reading_plans ──→ reading_plans
│           ├─→ user_statistics
│           └─→ push_subscriptions

READING PLANS
└── reading_plans ──→ reading_plan_days ──→ bible_books

QUOTE SYSTEM
├── quote_pool ──→ bible_books
└── quote_schedule ──→ bible_books
```

---

## 4.10 Key Design Decisions

### 4.10.1 Verse Reference Independence

**Decision:** Highlights, bookmarks, notes ไม่ผูกกับ `version_id` แต่ผูกกับ `(book_id, chapter, verse)`

**Rationale:**
- ผู้ใช้สามารถเปลี่ยนภาษา/version แล้วยังเห็น highlights/bookmarks/notes เดิม
- Verse reference เป็น universal key ที่ใช้ได้กับทุก version
- รองรับ parallel view โดยแสดง highlight ได้ทั้ง 2 ด้าน

---

### 4.10.2 Denormalized Statistics

**Decision:** เก็บ statistics ใน `user_statistics` แบบ denormalized

**Rationale:**
- Performance: การคำนวณ real-time จาก history table ช้าเกินไป
- ใช้ triggers/background jobs update stats เมื่อมีการเปลี่ยนแปลง
- Trade-off: ใช้ storage เพิ่ม แต่ได้ performance

---

### 4.10.3 Quote Pool Strategy

**Decision:** ใช้ `quote_pool` + `quote_schedule` แทนการสุ่มทุกวัน

**Rationale:**
- Consistency: Quote เดียวกันทั่ว global ในวันเดียวกัน
- Control: Admin สามารถกำหนด quote ล่วงหน้าได้
- Performance: ไม่ต้องสุ่มทุกครั้งที่มีคน request

---

## 4.11 Data Volume Estimates

### Bible Content (Per Version):
- Books: 66 rows
- Verses: ~31,102 rows
- Storage per version: ~5-10 MB
- With 10 versions: 50-100 MB

### User Data (Per 10,000 Active Users):
- Users: 10,000 rows (~1 MB)
- Highlights: ~100,000 rows (~10 MB)
- Bookmarks: ~30,000 rows (~3 MB)
- Notes: ~50,000 rows (~50 MB with text)
- Reading history: ~500,000 rows (~50 MB)
- Total: ~115 MB per 10K users

---

## สรุป Section 4

ส่วนที่ 4 นี้ครอบคลุม:

✅ **15+ Data Models:**
- Bible Content (5 tables)
- User System (3 tables)
- Study Tools (4 tables)
- Reading Plans (4 tables)
- Quote System (2 tables)
- Analytics (1 table)
- PWA (1 table)

✅ **Database Design Principles:**
- Normalization (3NF)
- Proper indexing strategy
- Foreign key relationships
- Data integrity constraints

✅ **Scalability Considerations:**
- Partitioning strategies
- Caching layers
- Archive policies

✅ **Flexibility:**
- Multi-language support
- Multi-version support
- Extensible design
- Future-proof structure
