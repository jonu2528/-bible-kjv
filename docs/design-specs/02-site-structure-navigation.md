# Part 2: Site Structure and Navigation

## 2.1 โครงสร้างการนำทางหลัก (Main Navigation Structure)

ระบบใช้ **Hybrid Navigation Pattern** ที่ปรับตามขนาดหน้าจอ:

### Desktop Navigation
- Top Navigation Bar (sticky)
- Left Sidebar สำหรับ Bible navigation (collapsible)
- Main Content Area

### Mobile/Tablet Navigation
- Top Header (minimal, collapsible)
- Bottom Tab Navigation (primary)
- Drawer Menu (secondary)

---

## 2.2 Site Map แบบภาพรวม

```
Online Bible Reader
│
├── 🏠 Home (หน้าหลัก)
│
├── 📖 Read (หน้าอ่านพระคัมภีร์)
│   ├── Single Version View
│   ├── Parallel View (2 versions)
│   └── Focus Mode
│
├── 🔍 Search (ค้นหา)
│   ├── Search Results
│   └── Advanced Search Filters
│
├── 📚 My Library (ห้องสมุดส่วนตัว)
│   ├── My Highlights
│   ├── My Bookmarks
│   ├── My Notes
│   └── Continue Reading
│
├── 📅 Reading Plans (แผนการอ่าน)
│   ├── Available Plans
│   ├── My Active Plans
│   └── Progress & Statistics
│
├── ⚙️ Settings (ตั้งค่า)
│   ├── Appearance
│   ├── Language & Version
│   ├── Audio/TTS Settings
│   ├── Notifications
│   └── Account Settings
│
└── 👤 User Account
    ├── Login / Sign Up
    ├── Profile
    └── Sync Status
```

---

## 2.3 รายละเอียดหน้าแต่ละหน้า

### 2.3.1 Home Page (หน้าหลัก)

**URL:** `/` หรือ `/home`

**วัตถุประสงค์:** เป็นหน้าแรกที่ผู้ใช้เห็นเมื่อเข้าเว็บ เป็นศูนย์กลางในการเริ่มต้นการอ่านและดู activity ส่วนตัว

**Components หลัก:**

1. **Hero Section - Quote of the Day Card**
   - แสดง verse ประจำวัน
   - Bible version ตามที่ user ตั้งค่าไว้
   - ปุ่ม "Read in Context" → jump ไปยัง chapter นั้น
   - ปุ่ม "Share" และ "Listen" (TTS)

2. **Continue Reading Section**
   - แสดง verse/chapter ล่าสุดที่กำลังอ่านอยู่
   - Format: "Continue reading **John 3:16**"
   - ปุ่ม "Continue" → กลับไปยังตำแหน่งที่อ่านค้างไว้
   - Progress indicator (ถ้ามี reading plan)

3. **Quick Access Cards**
   - Reading Plans, My Library, Search
   - แสดงจำนวน highlights, bookmarks, notes

4. **Reading Streak & Stats** (ถ้า login)
   - แสดงจำนวนวันที่อ่านติดต่อกัน
   - Chapters read this week/month
   - Visual progress indicator

---

### 2.3.2 Read Page (หน้าอ่านพระคัมภีร์)

**URL:** `/read/:book/:chapter` หรือ `/read/:book/:chapter/:verse`

**วัตถุประสงค์:** หน้าหลักสำหรับอ่านพระคัมภีร์ เน้นความสบายตาและ functionality

**Components หลัก:**

1. **Bible Navigation Panel** (Sidebar บน Desktop / Drawer บน Mobile)
   - Testament Selector: Old Testament / New Testament
   - Book Selector: Grid หรือ List view, group by Testament
   - Chapter Selector: Number grid, Quick jump input
   - Verse Selector (Optional)

2. **Reading Toolbar** (Top bar)
   - Left: Version/Language selector, View mode (Single/Parallel)
   - Center: Current location display "John 3:16", Breadcrumb
   - Right: Font size controls, Theme selector, Focus Mode toggle, TTS button

3. **Main Reading Area**
   - Chapter Header: Book name + Chapter number, Navigation arrows
   - Verse Display: แต่ละ verse แสดงชัดเจนพร้อม verse number
   - Interactive Features per Verse:
     - Highlight (เลือกสี)
     - Bookmark
     - Add Note
     - Cross References
     - Copy
     - Listen (TTS)

4. **Parallel View Mode**
   - แบ่งหน้าจอเป็น 2 columns: Version A | Version B
   - Verse แต่ละข้อ align กัน
   - Sync scroll between columns

5. **Audio/TTS Control Bar** (Bottom bar)
   - Play/Pause/Stop controls
   - Progress bar แสดงตำแหน่งปัจจุบัน
   - Verse ที่กำลังอ่านจะ highlight
   - Speed control: 0.75x, 1x, 1.25x, 1.5x
   - Voice selection

6. **Focus Mode**
   - Full-screen reading mode
   - ซ่อน navigation, toolbar
   - เหลือแค่ Text content + Minimal controls

---

### 2.3.3 Search Page (ค้นหา)

**URL:** `/search?q={query}`

**วัตถุประสงค์:** ค้นหาคำ, วลี, หรือหัวข้อในพระคัมภีร์

**Components หลัก:**

1. **Search Input Section**
   - Large search box พร้อม icon
   - Auto-suggestion ขณะพิมพ์
   - Voice search (optional)

2. **Search Filters** (Sidebar/Expandable)
   - Bible Version
   - Testament: All / Old / New
   - Books: Select All / Individual
   - Search Type: Exact phrase / Any word / All words

3. **Search Results**
   - Results Header: "Found X results for 'query' in KJV"
   - Result Items: Book + Chapter:Verse, Snippet with highlighted keyword
   - Pagination: 20-50 results per page

4. **Quick Filters**
   - Popular searches: "Faith", "Love", "Peace"
   - Recent searches

---

### 2.3.4 My Library (ห้องสมุดส่วนตัว)

**URL:** `/library`

**วัตถุประสงค์:** จัดการ highlights, bookmarks, และ notes

#### Sub-pages:

**A. My Highlights** (`/library/highlights`)
- Filter by color: All / Yellow / Green / Blue / Pink
- Filter by book, Sort by date/book order
- Card layout: Color indicator, Book+Chapter:Verse, Verse text, Actions

**B. My Bookmarks** (`/library/bookmarks`)
- Custom title (editable)
- Filter and sort options
- Card layout: Bookmark icon, Title, Reference, Verse snippet, Actions

**C. My Notes** (`/library/notes`)
- Search within notes
- Filter by book
- Card layout: Reference, Verse quote, Note content (expandable), Actions

**D. Continue Reading** (`/library/history`)
- Recent reading history
- Reading progress summary
- Quick resume buttons

---

### 2.3.5 Reading Plans (แผนการอ่าน)

**URL:** `/plans`

**วัตถุประสงค์:** เลือกและติดตาม reading plans

#### Sub-pages:

**A. Available Plans** (`/plans/browse`)
- Plan Categories: Complete Bible, NT only, OT only, Topical
- Plan Cards: Name, Description, Duration, Difficulty
- Start button to activate plan

**B. My Active Plans** (`/plans/active`)
- Active Plan Dashboard: Progress bar, Days completed
- Today's Reading: List of chapters/verses, Checkboxes
- Calendar View: Visual calendar with completed days
- Catch Up Section: Missed readings

**C. Progress & Statistics** (`/plans/stats`)
- Overall Stats: Total chapters, Current streak, Longest streak
- Charts: Reading activity heatmap, Chapters per week
- Achievements/Badges (optional gamification)

---

### 2.3.6 Settings (ตั้งค่า)

**URL:** `/settings`

**วัตถุประสงค์:** ตั้งค่าทุกอย่างเกี่ยวกับการใช้งาน

#### Settings Categories:

**A. Appearance** (`/settings/appearance`)
- Theme: Light / Dark / Sepia / Auto
- Font Size: Slider (Small → Large)
- Line Spacing: Compact / Normal / Relaxed
- Text Alignment: Left / Justified

**B. Language & Version** (`/settings/language`)
- UI Language: English, ไทย, etc.
- Default Bible Version: KJV, Thai Bible, etc.
- Parallel View Defaults: Left pane version, Right pane version
- Synchronize scrolling toggle

**C. Audio/TTS Settings** (`/settings/audio`)
- Default Playback Speed: 0.5x - 2x slider
- Voice Selection: Male/Female, Preview
- Auto-scroll toggle
- Auto-play next chapter toggle
- Delay between verses

**D. Notifications** (`/settings/notifications`)
- Enable Notifications toggle + browser permission
- Quote of the Day: Time picker, Days selector
- Reading Plan Reminders: Time, Frequency
- Notification Preview and Test button

**E. Account Settings** (`/settings/account`)
- Profile Information: Name, Email, Avatar
- Password: Change password
- Connected Accounts: Google, Facebook (Connect/Disconnect)
- Data & Privacy: Export data, Delete account
- Sync Status: Last synced timestamp, Sync now button

**F. About** (`/settings/about`)
- App version, Bible version info, Credits
- Terms of Service, Privacy Policy, Contact/Support

---

### 2.3.7 User Account Pages

**A. Login Page** (`/login`)
- Email + Password fields
- Social login buttons (Google, Facebook)
- "Forgot password?" link
- "Sign up" link

**B. Sign Up Page** (`/signup`)
- Name, Email, Password fields
- Password strength indicator
- Terms agreement checkbox
- Social sign up options

**C. Profile Page** (`/profile`)
- Profile header (avatar, name, join date)
- Quick stats: Chapters read, Streak, Highlights/Bookmarks/Notes count
- Recent activity feed
- Settings link

---

## 2.4 Navigation Flow Diagram

```
[Home] ←→ [Read] ←→ [Search]
  ↓          ↓         ↓
[Library] ←→ [Plans] ←→ [Settings]
  ↓
[Profile/Account]
```

**Cross-navigation:**
- Anywhere → Verse detail modal
- Anywhere → Settings (via menu)
- Anywhere → Search (via quick search)

---

## 2.5 Main Navigation Menu Structure

### Desktop Top Navigation:
```
[Logo] [Home] [Read] [Search] [Library] [Plans] | [Settings] [Profile]
```

### Mobile Bottom Tab Navigation:
```
[Home] [Read] [Search] [Library] [More]
                                    └─→ [Plans]
                                    └─→ [Settings]
                                    └─→ [Profile]
```

---

## 2.6 URL Structure Summary

| Page | URL Pattern | Notes |
|------|-------------|-------|
| Home | `/` | Landing page |
| Read | `/read/:book/:chapter?/:verse?` | Dynamic routing |
| Search | `/search?q={query}&filter={...}` | Query params |
| Library Overview | `/library` | Hub page |
| Highlights | `/library/highlights` | Sub-section |
| Bookmarks | `/library/bookmarks` | Sub-section |
| Notes | `/library/notes` | Sub-section |
| Reading History | `/library/history` | Sub-section |
| Plans Browse | `/plans/browse` | Available plans |
| Active Plans | `/plans/active` | User's plans |
| Plan Stats | `/plans/stats` | Progress tracking |
| Settings | `/settings/:section` | Nested routes |
| Login | `/login` | Auth page |
| Sign Up | `/signup` | Auth page |
| Profile | `/profile` | User profile |

---

## 2.7 Breadcrumb Examples

- Home
- Home > Read > Genesis > Chapter 1
- Home > Library > My Highlights
- Home > Plans > Active Plans > Bible in 1 Year
- Home > Settings > Appearance

---

## สรุป

โครงสร้าง site map นี้ครอบคลุม:
- ✅ **8 main pages** + sub-pages
- ✅ **Clear navigation hierarchy**
- ✅ **Responsive navigation patterns** (desktop vs mobile)
- ✅ **Logical information architecture**
- ✅ **User-friendly URL structure**
- ✅ **Cross-linking between related features**

ทุกหน้าออกแบบให้:
1. **มีวัตถุประสงค์ชัดเจน**
2. **เข้าถึงได้ภายใน 2-3 clicks**
3. **Responsive บนทุก device**
4. **รองรับ PWA routing**
