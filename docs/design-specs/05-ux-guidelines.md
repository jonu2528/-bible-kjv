# Part 5: UX Guidelines & Best Practices

## 5.1 ภาพรวม UX Philosophy

### 5.1.1 Core UX Principles

```
┌─────────────────────────────────────────────────────────────┐
│                    UX CORE PRINCIPLES                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. SIMPLICITY FIRST                                        │
│     └─ ใช้งานง่าย ไม่ซับซ้อน เข้าถึงได้ทันที              │
│                                                              │
│  2. CONTENT-FOCUSED                                          │
│     └─ เน้นเนื้อหาพระคัมภีร์เป็นหลัก UI เป็นรอง           │
│                                                              │
│  3. ACCESSIBILITY                                            │
│     └─ ทุกคนเข้าถึงได้ ไม่ว่าอุปกรณ์หรือความสามารถ         │
│                                                              │
│  4. PERFORMANCE                                              │
│     └─ เร็ว ลื่นไหล ไม่ค้าง responsive                     │
│                                                              │
│  5. CONSISTENCY                                              │
│     └─ UI/UX สม่ำเสมอทุกหน้า ทุกแพลตฟอร์ม                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 5.1.2 User Journey Focus

**Primary User Flows:**

1. **Quick Read** (80% of usage)
   - เปิดเว็บ → ไปยังตำแหน่งล่าสุด → อ่าน → ปิด
   - เวลา: < 30 วินาที จนถึงการเริ่มอ่าน

2. **Study Mode** (15% of usage)
   - ค้นหา → อ่าน → Highlight/Note → อ้างอิง → อ่านต่อ
   - เวลา: 10-30 นาที

3. **Audio Listen** (5% of usage)
   - เปิดเว็บ → เลือก chapter → กด Play → ฟังขณะทำอย่างอื่น
   - เวลา: 15-45 นาที

---

## 5.2 การอ่านที่สบายตา (Optimized Reading Experience)

### 5.2.1 Typography Best Practices

#### A. Font Selection

**Desktop:**
```css
font-family:
  /* Serif for Bible text (readable, traditional) */
  'Georgia', 'Times New Roman', 'Palatino', serif;

  /* Sans-serif for UI (modern, clean) */
  'Inter', 'SF Pro', 'Segoe UI', 'Roboto', sans-serif;
```

**Mobile:**
```css
font-family:
  /* System fonts for performance */
  -apple-system, BlinkMacSystemFont, 'Segoe UI',
  'Roboto', 'Helvetica Neue', Arial, sans-serif;
```

**Multi-language Considerations:**
```css
/* Thai text */
font-family: 'Sarabun', 'Noto Sans Thai', sans-serif;

/* Chinese text */
font-family: 'Noto Sans SC', 'PingFang SC', sans-serif;

/* Fallback */
font-family: system-ui, sans-serif;
```

#### B. Font Sizing Strategy

**Responsive Font Sizes:**
```
Base (Body text):
- Mobile:   16px (1rem)     - Minimum for readability
- Tablet:   17px (1.0625rem)
- Desktop:  18px (1.125rem) - Optimal for long reading

User-adjustable: 14px - 24px (0.875rem - 1.5rem)

Scale:
- Small:    14px
- Medium:   18px (default)
- Large:    21px
- X-Large:  24px
```

#### C. Line Height (Leading)

**Optimal for Reading:**
```css
/* Base line-height */
line-height: 1.8;  /* 180% - optimal for readability */

/* User options */
.compact  { line-height: 1.4; }  /* 140% - dense */
.normal   { line-height: 1.8; }  /* 180% - default */
.relaxed  { line-height: 2.2; }  /* 220% - spacious */
```

**Why 1.8?**
- Balance between compact and spacious
- Reduces eye strain during long reading
- Works well with multi-line verses
- Accessible for dyslexic readers

#### D. Line Length (Measure)

**Optimal Characters per Line:**
```
Desktop:  50-75 characters (optimal: 66)
Tablet:   45-60 characters
Mobile:   35-50 characters (full width acceptable)
```

**Implementation:**
```css
.verse-content {
  max-width: 65ch;  /* 65 characters */
  margin: 0 auto;

  /* Responsive */
  @media (max-width: 768px) {
    max-width: 100%;
    padding: 0 1rem;
  }
}
```

---

### 5.2.2 Color Contrast & Accessibility

#### A. WCAG 2.1 AA Compliance

**Minimum Contrast Ratios:**
- Normal text: 4.5:1
- Large text (18px+): 3:1
- UI components: 3:1

**Light Theme Contrast:**
```
Background: #ffffff (White)
Text:       #111827 (Gray-900) - Ratio: 16.1:1 ✓
Muted:      #6b7280 (Gray-500) - Ratio: 4.6:1 ✓
Primary:    #2563eb (Blue)     - Ratio: 4.8:1 ✓
```

**Dark Theme Contrast:**
```
Background: #111827 (Gray-900)
Text:       #f9fafb (Gray-50)  - Ratio: 16.1:1 ✓
Muted:      #9ca3af (Gray-400) - Ratio: 6.4:1 ✓
Primary:    #3b82f6 (Blue-400) - Ratio: 5.2:1 ✓
```

#### B. Highlight Colors - Accessible

**Light Mode:**
```css
.highlight-yellow {
  background: #fef3c7;
  border-left: 3px solid #f59e0b;
  /* Contrast with black text: 11.2:1 ✓ */
}

.highlight-green {
  background: #d1fae5;
  border-left: 3px solid #10b981;
  /* Contrast: 12.5:1 ✓ */
}

.highlight-blue {
  background: #dbeafe;
  border-left: 3px solid #3b82f6;
  /* Contrast: 10.8:1 ✓ */
}
```

**Dark Mode:**
```css
.highlight-yellow {
  background: rgba(251, 191, 36, 0.15);
  border-left: 3px solid #fbbf24;
}

.highlight-green {
  background: rgba(16, 185, 129, 0.15);
  border-left: 3px solid #34d399;
}
```

---

### 5.2.3 Whitespace & Breathing Room

#### A. Spacing Scale

```css
/* Base spacing unit: 4px (0.25rem) */
--space-xs:  4px;   /* 0.25rem */
--space-sm:  8px;   /* 0.5rem */
--space-md:  16px;  /* 1rem */
--space-lg:  24px;  /* 1.5rem */
--space-xl:  32px;  /* 2rem */
--space-2xl: 48px;  /* 3rem */
--space-3xl: 64px;  /* 4rem */
```

#### B. Content Margins

**Desktop:**
```css
.reading-container {
  padding: var(--space-2xl) var(--space-xl);
  max-width: 800px;
  margin: 0 auto;
}

.verse {
  margin-bottom: var(--space-lg);
}
```

**Mobile:**
```css
.reading-container {
  padding: var(--space-lg) var(--space-md);
}

.verse {
  margin-bottom: var(--space-md);
}
```

---

## 5.3 Text-to-Speech (TTS) UX

### 5.3.1 TTS Player Interface Design

#### A. Player Placement

**Desktop:**
```
┌────────────────────────────────────────┐
│ Main Content                           │
│ Reading text...                        │
├────────────────────────────────────────┤
│ [▶] John 3:16 ████░░░░ 2:34/5:12 [1x] │ ← Sticky bottom
└────────────────────────────────────────┘
```

**Mobile:**
```
┌──────────────────┐
│ Content          │
├──────────────────┤
│ ▶ 3:16 ██░ [1x] │ ← Compact bottom bar
├──────────────────┤
│ [🏠][📖][🔍][📚]│ ← Above bottom nav
└──────────────────┘
```

#### B. Essential Controls

**Priority Levels:**
```
P1 (Always visible):
  └─ Play/Pause button
  └─ Progress bar (seekable)
  └─ Current verse indicator

P2 (One tap away):
  └─ Speed control
  └─ Volume control
  └─ Close button

P3 (Settings):
  └─ Voice selection
  └─ Auto-scroll toggle
```

#### C. Visual Feedback

**Current Verse Highlight:**
```css
.verse[data-tts-active="true"] {
  background: var(--highlight-active);
  border-left: 4px solid var(--primary);
  animation: fadeIn 0.3s;
  scroll-margin-top: 100px;
}
```

---

### 5.3.2 Text Synchronization

#### A. Auto-Scroll Behavior

**Smooth Scrolling:**
```javascript
function scrollToActiveVerse(verseElement) {
  verseElement.scrollIntoView({
    behavior: 'smooth',
    block: 'center',
    inline: 'nearest'
  });
}
```

**User Override:**
```javascript
let userScrolling = false;
let scrollTimeout;

window.addEventListener('scroll', () => {
  userScrolling = true;
  clearTimeout(scrollTimeout);

  scrollTimeout = setTimeout(() => {
    userScrolling = false;
  }, 5000);  // Resume auto-scroll after 5s
});
```

---

### 5.3.3 Speed Control UX

#### A. Speed Options

**Standard Speeds:**
```
0.5x  - Very slow (learning)
0.75x - Slow (careful listening)
1.0x  - Normal (default)
1.25x - Slightly faster
1.5x  - Fast (time-saving)
2.0x  - Very fast
```

#### B. Speed Change Feedback

```javascript
function changeSpeed(newSpeed) {
  // Visual feedback
  speedButton.textContent = `${newSpeed}x`;

  // Haptic feedback (mobile)
  if ('vibrate' in navigator) {
    navigator.vibrate(10);
  }

  // Save preference
  saveUserPreference('ttsSpeed', newSpeed);
}
```

---

## 5.4 Multi-language Support UX

### 5.4.1 Language Switcher Design

#### A. Clear Separation

```
┌─────────────────────────────────────┐
│ Settings > Language                 │
├─────────────────────────────────────┤
│ 🌐 App Language (UI)                │
│ ┌─────────────────────────────────┐ │
│ │ English ▾                       │ │
│ └─────────────────────────────────┘ │
│ Controls menus, buttons, labels     │
│                                     │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                     │
│ 📖 Bible Language                   │
│ ┌─────────────────────────────────┐ │
│ │ KJV - English ▾                 │ │
│ └─────────────────────────────────┘ │
│ The text you're reading             │
└─────────────────────────────────────┘
```

---

### 5.4.2 Parallel View UX

#### A. Side-by-Side Layout

```
┌──────────────────┬──────────────────┐
│ KJV - English    │ Thai Bible       │
├──────────────────┼──────────────────┤
│ ¹ In the         │ ¹ ในปฐมกาล      │
│   beginning God  │   พระเจ้าทรง    │
│   created...     │   สร้าง...      │
└──────────────────┴──────────────────┘

[✓] Sync scrolling
```

#### B. Mobile Strategy

**Tabs (Recommended):**
```
┌──────────────────────────────┐
│ [ KJV ]  [ Thai Bible ]      │ ← Swipeable
├──────────────────────────────┤
│ ¹ In the beginning God       │
│   created...                  │
└──────────────────────────────┘

Swipe → to switch versions
```

---

## 5.5 Highlight, Bookmark, Note Management UX

### 5.5.1 Quick Action Menu

**Verse Click/Tap:**
```
┌──────────────────────────────────┐
│ 🎨 Highlight                     │
│    → Yellow, Green, Blue...      │
│ 🔖 Bookmark                       │
│ 📝 Add Note                       │
│ 🔗 Cross References              │
│ 📋 Copy Verse                    │
│ 🔊 Listen to Verse               │
│ 📤 Share                         │
└──────────────────────────────────┘
```

---

### 5.5.2 Note-Taking Experience

#### A. Note Editor Modal

```
┌────────────────────────────────────────┐
│ Add Note to John 3:16             [✕] │
├────────────────────────────────────────┤
│ "For God so loved the world..."       │
├────────────────────────────────────────┤
│ Your Note:                             │
│ ┌────────────────────────────────────┐ │
│ │ This is the most famous verse...  │ │
│ │                                    │ │
│ └────────────────────────────────────┘ │
│ 234 characters                         │
│ [ Cancel ]              [ Save Note ] │
└────────────────────────────────────────┘
```

#### B. Auto-Save

```javascript
let saveTimeout;
const AUTO_SAVE_DELAY = 2000;

noteEditor.addEventListener('input', (e) => {
  clearTimeout(saveTimeout);
  showSavingIndicator();

  saveTimeout = setTimeout(() => {
    saveNote(e.target.value);
    showSavedIndicator();
  }, AUTO_SAVE_DELAY);
});
```

**Visual Indicators:**
```
Typing:   "✏️ Editing..."
Saving:   "💾 Saving..."
Saved:    "✓ Saved"
Error:    "⚠️ Save failed [Retry]"
```

---

### 5.5.3 Library Organization

#### A. Filter & Sort

```
┌─────────────────────────────────────┐
│ My Highlights (47)                  │
├─────────────────────────────────────┤
│ Color:                              │
│ [All][🟨][🟩][🟦][🟪][🟧]          │
│                                     │
│ Book: [All Books ▾]                 │
│                                     │
│ Sort:                               │
│ ● Date Added (Newest)               │
│ ○ Book Order                        │
└─────────────────────────────────────┘
```

---

## 5.6 Performance & Loading States

### 5.6.1 Initial Load

**Progressive Loading:**
```
1. Shell (< 1s)
   └─ Basic HTML/CSS, navigation

2. Critical Content (< 2s)
   └─ Current chapter text

3. Enhanced Features (< 3s)
   └─ TTS, search, user data

4. Background (lazy)
   └─ Images, analytics, non-critical
```

---

### 5.6.2 Loading Indicators

**Skeleton Screens:**
```
┌────────────────────────────────┐
│ ████ ████ █                    │ ← Book name skeleton
│                                │
│ ░░░░░░░░░░░░░░░░░░░░░░░░░     │ ← Verse 1
│ ░░░░░░░░░░░░░░░░               │
│                                │
│ ░░░░░░░░░░░░░░░░░░░░░░░░░     │ ← Verse 2
│ ░░░░░░░░░░░░░░░░░░             │
└────────────────────────────────┘
```

---

### 5.6.3 Error States

**Network Error:**
```
┌────────────────────────────────┐
│         📡                     │
│                                │
│  Unable to load content        │
│  Check your internet connection│
│                                │
│  [Try Again]   [Go Offline]    │
└────────────────────────────────┘
```

**404 Chapter:**
```
┌────────────────────────────────┐
│         📖                     │
│                                │
│  Chapter not found             │
│  This book has only 21 chapters│
│                                │
│  [Go to Chapter 1]             │
└────────────────────────────────┘
```

---

## 5.7 Accessibility (A11y)

### 5.7.1 Keyboard Navigation

**Essential Shortcuts:**
```
[Space]     - Play/Pause TTS
[←][→]      - Previous/Next chapter
[↑][↓]      - Scroll
[H]         - Highlight selected verse
[B]         - Bookmark selected verse
[N]         - Add note
[/]         - Focus search
[Esc]       - Close modal/menu
```

---

### 5.7.2 Screen Reader Support

**ARIA Labels:**
```html
<button aria-label="Play chapter audio">
  <svg aria-hidden="true">...</svg>
</button>

<div role="article" aria-label="John Chapter 3">
  <div role="heading" aria-level="1">John</div>
  <div role="heading" aria-level="2">Chapter 3</div>

  <p id="verse-1" aria-label="Verse 1">
    <sup aria-hidden="true">1</sup>
    There was a man...
  </p>
</div>
```

---

### 5.7.3 Focus Management

**Focus Trap in Modals:**
```javascript
function trapFocus(modal) {
  const focusable = modal.querySelectorAll(
    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
  );

  const first = focusable[0];
  const last = focusable[focusable.length - 1];

  modal.addEventListener('keydown', (e) => {
    if (e.key === 'Tab') {
      if (e.shiftKey && document.activeElement === first) {
        e.preventDefault();
        last.focus();
      } else if (!e.shiftKey && document.activeElement === last) {
        e.preventDefault();
        first.focus();
      }
    }
  });
}
```

---

## 5.8 Mobile-Specific UX

### 5.8.1 Touch Gestures

**Supported Gestures:**
```
Swipe Left/Right:  Navigate chapters
Long Press:        Show verse actions
Pinch to Zoom:     Adjust font size (optional)
Pull to Refresh:   Reload chapter
```

---

### 5.8.2 Mobile Safari Considerations

**Address Bar Hiding:**
```css
/* Account for Safari address bar */
.fullscreen-content {
  height: 100vh;
  height: -webkit-fill-available;
}
```

**Prevent Zoom on Input:**
```html
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
```

---

## สรุป Section 5

ส่วนที่ 5 นี้ครอบคลุม:

✅ **Reading Experience:**
- Typography best practices
- Color contrast & accessibility
- Whitespace & visual hierarchy

✅ **TTS UX:**
- Player interface design
- Text synchronization
- Speed control & voice selection

✅ **Multi-language:**
- Language switcher design
- Parallel view strategies
- RTL support (future)

✅ **Study Tools:**
- Highlight/bookmark/note workflows
- Library organization
- Export & backup

✅ **Performance:**
- Loading states
- Error handling
- Progressive enhancement

✅ **Accessibility:**
- Keyboard navigation
- Screen reader support
- Focus management

✅ **Mobile UX:**
- Touch gestures
- Platform-specific optimizations
