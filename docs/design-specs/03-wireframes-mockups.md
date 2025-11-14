# Part 3: Wireframes and Mockups

## 3.1 Design Principles และ Guidelines

### 3.1.1 Responsive Breakpoints
```
Mobile:   320px - 767px   (Primary: 375px, 414px)
Tablet:   768px - 1023px  (Primary: 768px, 834px)
Desktop:  1024px+          (Primary: 1440px, 1920px)
```

### 3.1.2 Spacing System
```
xs:  4px    (0.25rem)
sm:  8px    (0.5rem)
md:  16px   (1rem)
lg:  24px   (1.5rem)
xl:  32px   (2rem)
2xl: 48px   (3rem)
3xl: 64px   (4rem)
```

### 3.1.3 Typography Scale
```
Heading 1:  32px - 40px (2rem - 2.5rem)  - Page titles
Heading 2:  24px - 32px (1.5rem - 2rem)  - Section headers
Heading 3:  20px - 24px (1.25rem - 1.5rem) - Sub-sections
Body:       16px - 18px (1rem - 1.125rem) - Verse text
Small:      14px (0.875rem)               - Meta info
Tiny:       12px (0.75rem)                - Labels
```

### 3.1.4 Color Palette (Conceptual)

#### Light Theme:
```
Primary:      #2563eb (Blue)
Secondary:    #7c3aed (Purple)
Accent:       #f59e0b (Amber)
Background:   #ffffff (White)
Surface:      #f3f4f6 (Gray-100)
Text Primary: #111827 (Gray-900)
Text Muted:   #6b7280 (Gray-500)
Border:       #e5e7eb (Gray-200)
```

#### Dark Theme:
```
Primary:      #3b82f6 (Blue-lighter)
Secondary:    #8b5cf6 (Purple-lighter)
Accent:       #fbbf24 (Amber-lighter)
Background:   #111827 (Gray-900)
Surface:      #1f2937 (Gray-800)
Text Primary: #f9fafb (Gray-50)
Text Muted:   #9ca3af (Gray-400)
Border:       #374151 (Gray-700)
```

#### Sepia Theme:
```
Background:   #f4f1e8
Surface:      #ebe6d9
Text Primary: #5c4a3a
Text Muted:   #8c7d6b
```

### 3.1.5 Highlight Colors
```
Yellow:  #fef3c7 (background) / #f59e0b (border)
Green:   #d1fae5 (background) / #10b981 (border)
Blue:    #dbeafe (background) / #3b82f6 (border)
Pink:    #fce7f3 (background) / #ec4899 (border)
Orange:  #fed7aa (background) / #f97316 (border)
```

---

## 3.2 Home Page (หน้าหลัก)

### Desktop Layout (1440px)

```
┌─────────────────────────────────────────────────────────────────────┐
│ ┌─────┐  [Home] [Read] [Search] [Library] [Plans]    [⚙️] [👤]    │
│ │LOGO │                                                              │
│ └─────┘                                                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌────────────── QUOTE OF THE DAY ─────────────┐                    │
│  │                                               │                    │
│  │  "For God so loved the world, that he gave  │                    │
│  │   his only begotten Son, that whosoever     │                    │
│  │   believeth in him should not perish, but   │                    │
│  │   have everlasting life."                    │                    │
│  │                                               │                    │
│  │  — John 3:16 (KJV)                           │                    │
│  │                                               │                    │
│  │  [🔊 Listen]  [📖 Read in Context]  [Share]  │                    │
│  └───────────────────────────────────────────────┘                    │
│                                                                       │
│  ┌──── CONTINUE READING ────────────────────────┐                    │
│  │                                               │                    │
│  │  📖 Continue reading John Chapter 3          │                    │
│  │  "You were at verse 16"                      │                    │
│  │                                               │                    │
│  │  Progress: ▓▓▓▓▓▓▓▓▓░░░░░░  60%              │                    │
│  │                                               │                    │
│  │  [Continue Reading →]                        │                    │
│  └───────────────────────────────────────────────┘                    │
│                                                                       │
│  ┌──── QUICK ACCESS ────────────────────────────────────────┐        │
│  │                                                           │        │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐    │        │
│  │  │   📅    │  │   📚    │  │   🔖    │  │   📝    │    │        │
│  │  │ Today's │  │   My    │  │  Book-  │  │   My    │    │        │
│  │  │ Reading │  │ Library │  │  marks  │  │  Notes  │    │        │
│  │  │         │  │         │  │         │  │         │    │        │
│  │  │  Start  │  │  View   │  │   15    │  │   23    │    │        │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘    │        │
│  │                                                           │        │
│  └───────────────────────────────────────────────────────────┘        │
│                                                                       │
│  ┌──── YOUR READING STREAK ────────────────────┐                     │
│  │                                               │                    │
│  │  🔥 7 Day Streak!                            │                    │
│  │                                               │                    │
│  │  ▓▓▓▓▓▓▓░░░  Mon Tue Wed Thu Fri Sat Sun    │                    │
│  │                                               │                    │
│  │  📖 15 chapters this week                    │                    │
│  │  ⏱️ 2h 30m reading time                      │                    │
│  └───────────────────────────────────────────────┘                    │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘
```

### Mobile Layout (375px)

```
┌─────────────────────────────┐
│ ┌──┐ Online Bible Reader [⚙️]│
│ │🏠│                          │
│ └──┘                          │
├─────────────────────────────┤
│                              │
│ ┌────── QUOTE ─────────┐    │
│ │                       │    │
│ │ "For God so loved the │    │
│ │  world, that he gave  │    │
│ │  his only begotten    │    │
│ │  Son..."              │    │
│ │                       │    │
│ │ — John 3:16 (KJV)     │    │
│ │                       │    │
│ │ [🔊][📖][Share]       │    │
│ └───────────────────────┘    │
│                              │
│ ┌──── CONTINUE ────────┐    │
│ │                       │    │
│ │ 📖 John Chapter 3     │    │
│ │ At verse 16           │    │
│ │                       │    │
│ │ ▓▓▓▓▓▓░░░░ 60%        │    │
│ │                       │    │
│ │ [Continue Reading]    │    │
│ └───────────────────────┘    │
│                              │
│ ┌──── QUICK ACCESS ────┐    │
│ │ ┌────┐ ┌────┐        │    │
│ │ │ 📅 │ │ 📚 │        │    │
│ │ │Plan│ │Lib │        │    │
│ │ └────┘ └────┘        │    │
│ │ ┌────┐ ┌────┐        │    │
│ │ │ 🔖 │ │ 📝 │        │    │
│ │ │ 15 │ │ 23 │        │    │
│ │ └────┘ └────┘        │    │
│ └───────────────────────┘    │
│                              │
│ ┌──── STREAK ──────────┐    │
│ │                       │    │
│ │ 🔥 7 Day Streak!      │    │
│ │                       │    │
│ │ [M][T][W][T][F][S][S] │    │
│ │  ✓  ✓  ✓  ✓  ✓  ✓  ✓  │    │
│ │                       │    │
│ │ 📖 15 chapters        │    │
│ └───────────────────────┘    │
│                              │
├─────────────────────────────┤
│ [🏠][📖][🔍][📚][⋮]         │
└─────────────────────────────┘
```

---

## 3.3 Read Page - Single Version

### Desktop Layout (1440px)

```
┌────────────────────────────────────────────────────────────────────────────┐
│ Top Navigation                                                    [⚙️][👤] │
├───────────┬────────────────────────────────────────────────────────────────┤
│           │ ┌──────────────────────────────────────────────────────────┐  │
│ SIDEBAR   │ │ [KJV ▾] [Single ▾] [A-][A][A+] [☀️][🌙][📄] [Focus] [🔊]│  │
│ (240px)   │ └──────────────────────────────────────────────────────────┘  │
│           │                                                                │
│ Testament │  ┌────────────────────────────────────────────────────────┐  │
│ ┌───────┐ │  │ JOHN                                Chapter 3           │  │
│ │● Old  │ │  │                                            [← Prev][Next →]│
│ │  New  │ │  └────────────────────────────────────────────────────────┘  │
│ └───────┘ │                                                                │
│           │  ┌─ Verse Container ─────────────────────────────────────┐  │
│ Books     │  │                                                         │  │
│ ───────── │  │  ¹ There was a man of the Pharisees, named Nicodemus, │  │
│ Genesis   │  │    a ruler of the Jews:                                │  │
│ Exodus    │  │                                                         │  │
│ ...       │  │  ² The same came to Jesus by night, and said unto     │  │
│ Matthew   │  │    him, Rabbi, we know that thou art a teacher come   │  │
│ Mark      │  │    from God: for no man can do these miracles...      │  │
│ Luke      │  │                                                         │  │
│ ❯ John    │  │  ³ Jesus answered and said unto him, Verily, verily,  │  │
│   Ch 1    │  │    I say unto thee, Except a man be born again...     │  │
│   Ch 2    │  │                                                         │  │
│ ❯ Ch 3 ←  │  │  ...                                                   │  │
│   Ch 4    │  │                                                         │  │
│   ...     │  │  ¹⁶ For God so loved the world, that he gave his only │  │
│           │  │     begotten Son, that whosoever believeth in him     │  │
│ Chapters  │  │     should not perish, but have everlasting life.     │  │
│ ───────── │  │     [Highlighted in yellow]                            │  │
│           │  │     💡 My note: Most famous verse!                     │  │
│ [1] 2 3   │  │     [🎨][🔖][📝][🔗][📋][🔊]                            │  │
│  4  5 6   │  │                                                         │  │
│  7  8 ... │  │  ¹⁷ For God sent not his Son into the world to        │  │
│  21       │  │     condemn the world; but that the world through     │  │
│           │  │     him might be saved.                                │  │
│           │  │                                                         │  │
│           │  └─────────────────────────────────────────────────────────┘  │
│           │                                                                │
│           │  ┌─ TTS Player (when active) ──────────────────────────────┐ │
│           │  │ ▶️ || John 3:16 ▓▓▓▓▓▓▓░░░░░░░ 2:34/5:12  [1x ▾][🔊▾] │ │
│           │  └──────────────────────────────────────────────────────────┘ │
└───────────┴────────────────────────────────────────────────────────────────┘
```

### Mobile Layout (375px)

```
┌──────────────────────────────┐
│ [☰] John 3        [🔊][⋮]   │
├──────────────────────────────┤
│                               │
│ ┌──────────────────────────┐ │
│ │ [KJV▾]  [A▾]  [Theme▾]  │ │
│ └──────────────────────────┘ │
│                               │
│ ┌────────────────────────── │ │
│ │ JOHN Chapter 3            │ │
│ │                [←][→]     │ │
│ └────────────────────────── │ │
│                               │
│  ¹ There was a man of the    │
│    Pharisees, named           │
│    Nicodemus, a ruler of the  │
│    Jews:                      │
│                               │
│  ² The same came to Jesus by  │
│    night, and said unto him,  │
│    Rabbi, we know that thou   │
│    art a teacher come from    │
│    God: for no man can do...  │
│                               │
│  ³ Jesus answered and said    │
│    unto him, Verily, verily,  │
│    I say unto thee, Except a  │
│    man be born again, he...   │
│                               │
│  ...                          │
│                               │
│  ¹⁶ For God so loved the      │
│     world, that he gave his   │
│     only begotten Son, that   │
│     whosoever believeth in    │
│     him should not perish,    │
│     but have everlasting      │
│     life.                     │
│     [Highlighted in yellow]   │
│     💡 My note: Most famous!  │
│                               │
│  ¹⁷ For God sent not his Son  │
│     into the world to condemn │
│     the world; but that the   │
│     world through him might   │
│     be saved.                 │
│                               │
│         [+]  ← FAB            │
│                               │
├──────────────────────────────┤
│ ▶️ 3:16 ▓▓░░░ 2:34 [1x▾]    │
├──────────────────────────────┤
│ [🏠][📖][🔍][📚][⋮]         │
└──────────────────────────────┘
```

---

## 3.4 Read Page - Parallel View

### Desktop Layout (1440px)

```
┌────────────────────────────────────────────────────────────────────────────┐
│ Top Navigation                                                    [⚙️][👤] │
├───────────┬────────────────────────────────────────────────────────────────┤
│ SIDEBAR   │ ┌──────────────────────────────────────────────────────────┐  │
│ (240px)   │ │ [Parallel View ▾] [A-][A][A+] [☀️][🌙][📄] [Focus] [🔊]│  │
│           │ └──────────────────────────────────────────────────────────┘  │
│           │                                                                │
│           │  ┌────────────────────────────────────────────────────────┐  │
│           │  │ JOHN                                Chapter 3           │  │
│           │  │                                            [← Prev][Next →]│
│           │  └────────────────────────────────────────────────────────┘  │
│           │                                                                │
│           │  ┌─────────────────────┬─────────────────────────────────┐   │
│           │  │ [KJV - English ▾]  │ [Thai Bible ▾]              │   │
│           │  ├─────────────────────┼─────────────────────────────────┤   │
│           │  │                     │                             │   │
│           │  │  ¹ There was a man  │  ¹ มีคนหนึ่งในพวกฟาริสี    │   │
│           │  │    of the Pharisees,│    ชื่อนิโคเดมัส เป็นหัวหน้า│   │
│           │  │    named Nicodemus, │    ของยิว                   │   │
│           │  │    a ruler of the   │                             │   │
│           │  │    Jews:            │                             │   │
│           │  │                     │                             │   │
│           │  │  ² The same came to │  ² ท่านมาหาพระเยซูใน      │   │
│           │  │    Jesus by night,  │    เวลากลางคืน ทูลว่า     │   │
│           │  │    and said unto    │    "รับบี ข้าพเจ้าทราบว่า  │   │
│           │  │    him, Rabbi...    │    ท่านเป็นครู..."         │   │
│           │  │                     │                             │   │
│           │  │  ¹⁶ For God so loved│  ¹⁶ เพราะพระเจ้าทรงรัก    │   │
│           │  │     the world, that │     โลกมากถึงกับประทาน    │   │
│           │  │     he gave his only│     พระบุตรองค์เดียว...   │   │
│           │  │     begotten Son... │                             │   │
│           │  │     [Highlighted]   │     [Highlighted]           │   │
│           │  │                     │                             │   │
│           │  └─────────────────────┴─────────────────────────────────┘   │
│           │                                                                │
└───────────┴────────────────────────────────────────────────────────────────┘
```

### Mobile Layout - Tabs (375px)

```
┌──────────────────────────────┐
│ [☰] John 3      [Swap][⋮]   │
├──────────────────────────────┤
│ Parallel View - Swipe to     │
│ switch between versions      │
├──────────────────────────────┤
│ 📱 Tab Selector:              │
│ [● KJV-EN]  [○ Thai Bible]   │
├──────────────────────────────┤
│                               │
│ Showing: KJV - English        │
│ (Swipe left for Thai version) │
│                               │
│ ¹ There was a man of the      │
│   Pharisees, named Nicodemus, │
│   a ruler of the Jews:        │
│                               │
│ ² The same came to Jesus by   │
│   night, and said unto him... │
│                               │
│ ...                           │
│                               │
│ ¹⁶ For God so loved the world,│
│    that he gave his only      │
│    begotten Son, that         │
│    whosoever believeth in him │
│    should not perish, but have│
│    everlasting life.          │
│    [Highlighted]              │
│                               │
│ [Quick Switch Button]         │
│ [Switch to Thai Version]      │
│                               │
├──────────────────────────────┤
│ [🏠][📖][🔍][📚][⋮]         │
└──────────────────────────────┘
```

---

## 3.5 My Library - Highlights

### Desktop Layout (1440px)

```
┌────────────────────────────────────────────────────────────────────────────┐
│ Top Navigation                                                    [⚙️][👤] │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐ │
│  │ MY LIBRARY                                                           │ │
│  │                                                                       │ │
│  │ [Highlights] [Bookmarks] [Notes] [History]   ← Tabs                 │ │
│  └──────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌──── HIGHLIGHTS ────────────────────────────────────────────────────┐   │
│  │                                                                     │   │
│  │  ┌─ Filters & Stats ──────────────────────────────────────────┐   │   │
│  │  │ Filter by Color:                                           │   │   │
│  │  │ [All] [🟨 Yellow] [🟩 Green] [🟦 Blue] [🟪 Pink]         │   │   │
│  │  │                                                             │   │   │
│  │  │ Filter by Book: [All Books ▾]    Sort: [Date Added ▾]    │   │   │
│  │  │                                                             │   │   │
│  │  │ 📊 Stats: 47 total highlights  |  Most: Psalms (12)       │   │   │
│  │  └─────────────────────────────────────────────────────────────┘   │   │
│  │                                                                     │   │
│  │  ┌─ Highlight Card ────────────────────────────────────────────┐  │   │
│  │  │ │ [Yellow highlight indicator bar]                          │  │   │
│  │  │                                                               │  │   │
│  │  │  John 3:16                              📅 Added: Jan 15     │  │   │
│  │  │                                                               │  │   │
│  │  │  "For God so loved the world, that he gave his only         │  │   │
│  │  │   begotten Son, that whosoever believeth in him should      │  │   │
│  │  │   not perish, but have everlasting life."                   │  │   │
│  │  │                                                               │  │   │
│  │  │  💡 Note: Most famous verse!                                │  │   │
│  │  │                                                               │  │   │
│  │  │  [🎨 Change Color] [📖 Read in Context] [🗑️ Remove]         │  │   │
│  │  └───────────────────────────────────────────────────────────────┘  │   │
│  │                                                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

### Mobile Layout (375px)

```
┌──────────────────────────────┐
│ [☰] My Highlights  [Filter] │
├──────────────────────────────┤
│ [Highlights][Bookmarks]      │
│ [Notes][History]             │
├──────────────────────────────┤
│                               │
│ ┌─ Filters (collapsed) ────┐ │
│ │ [🎨 All Colors ▾]         │ │
│ │ [📚 All Books ▾]          │ │
│ └───────────────────────────┘ │
│                               │
│ 📊 47 highlights              │
│                               │
│ ┌─ Highlight Card ─────────┐ │
│ │ │ [Yellow bar]            │ │
│ │                           │ │
│ │ John 3:16   📅 Jan 15     │ │
│ │                           │ │
│ │ "For God so loved the     │ │
│ │  world, that he gave his  │ │
│ │  only begotten Son..."    │ │
│ │                           │ │
│ │ 💡 Note: Most famous!     │ │
│ │                           │ │
│ │ [Read] [⋮]                │ │
│ └───────────────────────────┘ │
│                               │
├──────────────────────────────┤
│ [🏠][📖][🔍][📚][⋮]         │
└──────────────────────────────┘
```

---

## 3.6 Settings Page

### Desktop Layout - Appearance Tab (1440px)

```
┌────────────────────────────────────────────────────────────────────────────┐
│ Top Navigation                                                    [⚙️][👤] │
├───────────┬────────────────────────────────────────────────────────────────┤
│ SETTINGS  │                                                                 │
│           │  SETTINGS - APPEARANCE                                          │
│ Sidebar   │                                                                 │
│ ─────────│                                                                 │
│           │  ┌──────────────────────────────────────────────────────────┐ │
│ ● Appear- │  │ APPEARANCE                                               │ │
│   ance    │  │                                                           │ │
│           │  │ ┌─ Theme ────────────────────────────────────────────┐  │ │
│   Language│  │ │                                                     │  │ │
│   & Ver.  │  │ │ Choose your preferred theme:                       │  │ │
│           │  │ │                                                     │  │ │
│   Audio/  │  │ │ ○ ☀️ Light Theme                                   │  │ │
│   TTS     │  │ │ ● 🌙 Dark Theme        ← Selected                  │  │ │
│           │  │ │ ○ 📄 Sepia Theme                                   │  │ │
│   Notific-│  │ │ ○ 🔄 Auto (Follow system preference)               │  │ │
│   ations  │  │ │                                                     │  │ │
│           │  │ │ [Preview: Sample verse text in selected theme]     │  │ │
│   Account │  │ └─────────────────────────────────────────────────────┘  │ │
│           │  │                                                           │ │
│   About   │  │ ┌─ Font Size ────────────────────────────────────────┐  │ │
│           │  │ │                                                     │  │ │
│           │  │ │ Adjust reading font size:                          │  │ │
│           │  │ │                                                     │  │ │
│           │  │ │ [A-] ━━━━●━━━━━━ [A+]      ← Slider               │  │ │
│           │  │ │                                                     │  │ │
│           │  │ │ Small    Medium    Large    X-Large                │  │ │
│           │  │ │                                                     │  │ │
│           │  │ │ Preview: "For God so loved the world..."           │  │ │
│           │  │ └─────────────────────────────────────────────────────┘  │ │
│           │  │                                                           │ │
│           │  │ [Reset to Defaults]                    [Save Changes]    │ │
│           │  └──────────────────────────────────────────────────────────┘ │
└───────────┴────────────────────────────────────────────────────────────────┘
```

### Mobile Layout (375px)

```
┌──────────────────────────────┐
│ [←] Appearance               │
├──────────────────────────────┤
│                               │
│ ┌─ THEME ──────────────────┐ │
│ │ ○ ☀️ Light               │ │
│ │ ● 🌙 Dark                │ │
│ │ ○ 📄 Sepia               │ │
│ │ ○ 🔄 Auto                │ │
│ └───────────────────────────┘ │
│                               │
│ ┌─ FONT SIZE ──────────────┐ │
│ │                           │ │
│ │ [A-] ━━●━━━ [A+]         │ │
│ │                           │ │
│ │ Preview:                  │ │
│ │ "For God so loved..."     │ │
│ └───────────────────────────┘ │
│                               │
│ ┌─ LINE SPACING ───────────┐ │
│ │ ○ Compact                 │ │
│ │ ● Normal                  │ │
│ │ ○ Relaxed                 │ │
│ └───────────────────────────┘ │
│                               │
│ [Save Changes]                │
└──────────────────────────────┘
```

---

## 3.7 Key Component Specifications

### Verse Component (Interactive)
```javascript
VerseComponent {
  States: normal | highlighted | selected | hover

  Elements:
  - VerseNumber: superscript, primary color, clickable
  - VerseText: 16-18px, line-height 1.8, serif font
  - Highlight: background color based on type
  - Actions: [Highlight][Bookmark][Note][CrossRef][Copy][Listen]
  - Note indicator: 💡 icon + preview text

  Interactions:
  - Click → Show context menu
  - Hover → Show action buttons (desktop)
  - Long-press → Show context menu (mobile)
}
```

### TTS Player Component
```javascript
TTSPlayer {
  Position: Fixed bottom bar
  Height: 80px (desktop) / 60px (mobile)

  Controls:
  - Play/Pause button (primary, 40px)
  - Progress bar (clickable, draggable)
  - Current verse display
  - Time display (current / total)
  - Speed selector (0.75x - 2x)
  - Volume control
  - Close button

  Behavior:
  - Auto-highlight current verse
  - Auto-scroll to follow
  - Persist state across page navigation
}
```

---

## สรุป Section 3

ส่วนที่ 3 นี้ครอบคลุม:

✅ **Design Principles:**
- Responsive breakpoints
- Spacing system
- Typography scale
- Color palette (Light/Dark/Sepia)

✅ **5 หน้าจอหลัก พร้อม wireframes:**
1. Home Page (Desktop + Mobile)
2. Read Page - Single Version (Desktop + Mobile)
3. Read Page - Parallel View (Desktop + Mobile)
4. My Library - Highlights (Desktop + Mobile)
5. Settings - Appearance (Desktop + Mobile)

✅ **Component Specifications:**
- Layout structures
- Interaction states
- Responsive variations
- Spacing and typography
- Color usage

✅ **UX Considerations:**
- Touch-friendly mobile interactions
- Keyboard navigation (desktop)
- Loading/Empty/Error states
- Accessibility features
