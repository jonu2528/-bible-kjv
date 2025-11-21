# Part 11: Internationalization (i18n) & Localization (l10n)

## Table of Contents

1. [Overview](#1-overview)
2. [Architecture](#2-architecture)
3. [UI Translation System](#3-ui-translation-system)
4. [Bible Content Localization](#4-bible-content-localization)
5. [Date, Time & Number Formatting](#5-date-time--number-formatting)
6. [RTL Language Support](#6-rtl-language-support)
7. [Text-to-Speech Localization](#7-text-to-speech-localization)
8. [Translation Workflow](#8-translation-workflow)
9. [Implementation Guide](#9-implementation-guide)
10. [Testing i18n](#10-testing-i18n)

---

## 1. Overview

### 1.1 Supported Languages

| Language | Code | UI Support | Bible Versions | TTS Support | Direction |
|----------|------|------------|----------------|-------------|-----------|
| English | en | ✅ Primary | KJV, NIV, NKJV | ✅ | LTR |
| Thai | th | ✅ | Thai Standard | ✅ | LTR |
| Spanish | es | 🔄 Planned | RVR, NVI | ✅ | LTR |
| Chinese (Simplified) | zh-CN | 🔄 Planned | CUV | ✅ | LTR |
| Arabic | ar | 🔄 Planned | NAV | ✅ | RTL |
| Korean | ko | 🔄 Planned | KRV | ✅ | LTR |

### 1.2 i18n Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         i18n ARCHITECTURE                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Language Detection                                              │   │
│  │  ├── User preference (stored in profile)                         │   │
│  │  ├── Browser language (Accept-Language header)                   │   │
│  │  ├── URL parameter (?lang=th)                                    │   │
│  │  └── Default: English                                            │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              │                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Translation Resources                                           │   │
│  │  ├── /locales/en/common.json    (Common strings)                 │   │
│  │  ├── /locales/en/bible.json     (Bible-specific)                 │   │
│  │  ├── /locales/en/errors.json    (Error messages)                 │   │
│  │  └── /locales/th/...            (Thai translations)              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              │                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Runtime                                                         │   │
│  │  ├── next-i18next (Next.js integration)                          │   │
│  │  ├── react-intl / FormatJS (formatting)                          │   │
│  │  └── Lazy loading per namespace                                  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Architecture

### 2.1 Project Structure

```
frontend/
├── locales/
│   ├── en/
│   │   ├── common.json        # Common UI strings
│   │   ├── bible.json         # Bible navigation, book names
│   │   ├── study-tools.json   # Highlights, bookmarks, notes
│   │   ├── reading-plans.json # Reading plan UI
│   │   ├── auth.json          # Login, register, profile
│   │   ├── errors.json        # Error messages
│   │   └── accessibility.json # Screen reader text
│   ├── th/
│   │   ├── common.json
│   │   ├── bible.json
│   │   └── ...
│   └── es/
│       └── ...
├── src/
│   ├── i18n/
│   │   ├── config.ts          # i18n configuration
│   │   ├── languages.ts       # Language definitions
│   │   └── utils.ts           # Helper functions
│   └── hooks/
│       └── useTranslation.ts  # Translation hook
└── next-i18next.config.js
```

### 2.2 Configuration

```typescript
// next-i18next.config.js
module.exports = {
  i18n: {
    defaultLocale: 'en',
    locales: ['en', 'th', 'es', 'zh-CN', 'ar', 'ko'],
    localeDetection: true,
  },
  localePath: './locales',
  reloadOnPrerender: process.env.NODE_ENV === 'development',
  ns: ['common', 'bible', 'study-tools', 'reading-plans', 'auth', 'errors'],
  defaultNS: 'common',
  interpolation: {
    escapeValue: false, // React already escapes
  },
  react: {
    useSuspense: true,
  },
};
```

```typescript
// src/i18n/config.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import Backend from 'i18next-http-backend';
import LanguageDetector from 'i18next-browser-languagedetector';

export const supportedLanguages = [
  { code: 'en', name: 'English', nativeName: 'English', dir: 'ltr' },
  { code: 'th', name: 'Thai', nativeName: 'ไทย', dir: 'ltr' },
  { code: 'es', name: 'Spanish', nativeName: 'Español', dir: 'ltr' },
  { code: 'zh-CN', name: 'Chinese (Simplified)', nativeName: '简体中文', dir: 'ltr' },
  { code: 'ar', name: 'Arabic', nativeName: 'العربية', dir: 'rtl' },
  { code: 'ko', name: 'Korean', nativeName: '한국어', dir: 'ltr' },
];

i18n
  .use(Backend)
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    fallbackLng: 'en',
    supportedLngs: supportedLanguages.map(l => l.code),

    detection: {
      order: ['querystring', 'cookie', 'localStorage', 'navigator', 'htmlTag'],
      caches: ['cookie', 'localStorage'],
      lookupQuerystring: 'lang',
      lookupCookie: 'i18next',
      lookupLocalStorage: 'i18nextLng',
    },

    backend: {
      loadPath: '/locales/{{lng}}/{{ns}}.json',
    },

    interpolation: {
      escapeValue: false,
      formatSeparator: ',',
    },

    react: {
      useSuspense: true,
      bindI18n: 'languageChanged loaded',
      bindI18nStore: 'added removed',
      transEmptyNodeValue: '',
    },
  });

export default i18n;
```

---

## 3. UI Translation System

### 3.1 Translation Files Structure

```json
// locales/en/common.json
{
  "app": {
    "name": "Online Bible Reader",
    "tagline": "Read, Study, Grow"
  },
  "navigation": {
    "home": "Home",
    "bible": "Bible",
    "readingPlans": "Reading Plans",
    "search": "Search",
    "profile": "Profile",
    "settings": "Settings",
    "logout": "Logout"
  },
  "actions": {
    "save": "Save",
    "cancel": "Cancel",
    "delete": "Delete",
    "edit": "Edit",
    "share": "Share",
    "copy": "Copy",
    "loading": "Loading...",
    "retry": "Retry"
  },
  "messages": {
    "saved": "Changes saved successfully",
    "deleted": "Item deleted",
    "copied": "Copied to clipboard",
    "error": "Something went wrong. Please try again."
  },
  "footer": {
    "copyright": "© {{year}} Online Bible Reader. All rights reserved.",
    "privacy": "Privacy Policy",
    "terms": "Terms of Service"
  }
}
```

```json
// locales/th/common.json
{
  "app": {
    "name": "อ่านพระคัมภีร์ออนไลน์",
    "tagline": "อ่าน, ศึกษา, เติบโต"
  },
  "navigation": {
    "home": "หน้าแรก",
    "bible": "พระคัมภีร์",
    "readingPlans": "แผนการอ่าน",
    "search": "ค้นหา",
    "profile": "โปรไฟล์",
    "settings": "ตั้งค่า",
    "logout": "ออกจากระบบ"
  },
  "actions": {
    "save": "บันทึก",
    "cancel": "ยกเลิก",
    "delete": "ลบ",
    "edit": "แก้ไข",
    "share": "แชร์",
    "copy": "คัดลอก",
    "loading": "กำลังโหลด...",
    "retry": "ลองอีกครั้ง"
  },
  "messages": {
    "saved": "บันทึกการเปลี่ยนแปลงเรียบร้อยแล้ว",
    "deleted": "ลบรายการแล้ว",
    "copied": "คัดลอกไปยังคลิปบอร์ดแล้ว",
    "error": "เกิดข้อผิดพลาด กรุณาลองใหม่อีกครั้ง"
  },
  "footer": {
    "copyright": "© {{year}} อ่านพระคัมภีร์ออนไลน์ สงวนลิขสิทธิ์",
    "privacy": "นโยบายความเป็นส่วนตัว",
    "terms": "ข้อกำหนดการใช้งาน"
  }
}
```

```json
// locales/en/bible.json
{
  "testaments": {
    "old": "Old Testament",
    "new": "New Testament"
  },
  "books": {
    "genesis": "Genesis",
    "exodus": "Exodus",
    "leviticus": "Leviticus",
    "numbers": "Numbers",
    "deuteronomy": "Deuteronomy",
    "joshua": "Joshua",
    "judges": "Judges",
    "ruth": "Ruth",
    "1samuel": "1 Samuel",
    "2samuel": "2 Samuel",
    "matthew": "Matthew",
    "mark": "Mark",
    "luke": "Luke",
    "john": "John",
    "acts": "Acts",
    "romans": "Romans",
    "revelation": "Revelation"
  },
  "reader": {
    "chapter": "Chapter {{number}}",
    "verse": "Verse {{number}}",
    "previousChapter": "Previous Chapter",
    "nextChapter": "Next Chapter",
    "selectBook": "Select Book",
    "selectChapter": "Select Chapter",
    "parallelView": "Parallel View",
    "singleView": "Single View",
    "focusMode": "Focus Mode"
  },
  "versions": {
    "kjv": "King James Version",
    "niv": "New International Version",
    "nkjv": "New King James Version",
    "thai": "Thai Standard Version"
  },
  "navigation": {
    "goToVerse": "Go to verse",
    "bookList": "Book List",
    "chapterList": "Chapter List"
  }
}
```

```json
// locales/th/bible.json
{
  "testaments": {
    "old": "พันธสัญญาเดิม",
    "new": "พันธสัญญาใหม่"
  },
  "books": {
    "genesis": "ปฐมกาล",
    "exodus": "อพยพ",
    "leviticus": "เลวีนิติ",
    "numbers": "กันดารวิถี",
    "deuteronomy": "เฉลยธรรมบัญญัติ",
    "joshua": "โยชูวา",
    "judges": "ผู้วินิจฉัย",
    "ruth": "นางรูธ",
    "1samuel": "1 ซามูเอล",
    "2samuel": "2 ซามูเอล",
    "matthew": "มัทธิว",
    "mark": "มาระโก",
    "luke": "ลูกา",
    "john": "ยอห์น",
    "acts": "กิจการ",
    "romans": "โรม",
    "revelation": "วิวรณ์"
  },
  "reader": {
    "chapter": "บทที่ {{number}}",
    "verse": "ข้อ {{number}}",
    "previousChapter": "บทก่อนหน้า",
    "nextChapter": "บทถัดไป",
    "selectBook": "เลือกพระธรรม",
    "selectChapter": "เลือกบท",
    "parallelView": "มุมมองคู่ขนาน",
    "singleView": "มุมมองเดี่ยว",
    "focusMode": "โหมดโฟกัส"
  },
  "versions": {
    "kjv": "ฉบับ King James",
    "niv": "ฉบับ NIV",
    "nkjv": "ฉบับ New King James",
    "thai": "พระคริสตธรรมคัมภีร์ ฉบับมาตรฐาน"
  },
  "navigation": {
    "goToVerse": "ไปยังข้อ",
    "bookList": "รายชื่อพระธรรม",
    "chapterList": "รายการบท"
  }
}
```

### 3.2 Pluralization

```json
// locales/en/study-tools.json
{
  "highlights": {
    "title": "Highlights",
    "count": "{{count}} highlight",
    "count_plural": "{{count}} highlights",
    "noHighlights": "No highlights yet",
    "addHighlight": "Add Highlight",
    "removeHighlight": "Remove Highlight",
    "colors": {
      "yellow": "Yellow",
      "green": "Green",
      "blue": "Blue",
      "pink": "Pink",
      "orange": "Orange"
    }
  },
  "bookmarks": {
    "title": "Bookmarks",
    "count": "{{count}} bookmark",
    "count_plural": "{{count}} bookmarks",
    "noBookmarks": "No bookmarks yet",
    "addBookmark": "Add Bookmark",
    "removeBookmark": "Remove Bookmark",
    "enterTitle": "Enter bookmark title"
  },
  "notes": {
    "title": "Notes",
    "count": "{{count}} note",
    "count_plural": "{{count}} notes",
    "noNotes": "No notes yet",
    "addNote": "Add Note",
    "editNote": "Edit Note",
    "deleteNote": "Delete Note",
    "placeholder": "Write your note here...",
    "lastEdited": "Last edited {{date}}"
  }
}
```

```json
// locales/th/study-tools.json
{
  "highlights": {
    "title": "ไฮไลต์",
    "count": "{{count}} ไฮไลต์",
    "noHighlights": "ยังไม่มีไฮไลต์",
    "addHighlight": "เพิ่มไฮไลต์",
    "removeHighlight": "ลบไฮไลต์",
    "colors": {
      "yellow": "เหลือง",
      "green": "เขียว",
      "blue": "น้ำเงิน",
      "pink": "ชมพู",
      "orange": "ส้ม"
    }
  },
  "bookmarks": {
    "title": "บุ๊คมาร์ค",
    "count": "{{count}} บุ๊คมาร์ค",
    "noBookmarks": "ยังไม่มีบุ๊คมาร์ค",
    "addBookmark": "เพิ่มบุ๊คมาร์ค",
    "removeBookmark": "ลบบุ๊คมาร์ค",
    "enterTitle": "ใส่ชื่อบุ๊คมาร์ค"
  },
  "notes": {
    "title": "บันทึก",
    "count": "{{count}} บันทึก",
    "noNotes": "ยังไม่มีบันทึก",
    "addNote": "เพิ่มบันทึก",
    "editNote": "แก้ไขบันทึก",
    "deleteNote": "ลบบันทึก",
    "placeholder": "เขียนบันทึกของคุณที่นี่...",
    "lastEdited": "แก้ไขล่าสุด {{date}}"
  }
}
```

### 3.3 React Components

```tsx
// src/components/LocalizedText.tsx
import { useTranslation, Trans } from 'react-i18next';

interface LocalizedTextProps {
  i18nKey: string;
  ns?: string;
  values?: Record<string, any>;
  components?: Record<string, React.ReactElement>;
}

export function LocalizedText({ i18nKey, ns, values, components }: LocalizedTextProps) {
  const { t } = useTranslation(ns);

  if (components) {
    return <Trans i18nKey={i18nKey} values={values} components={components} />;
  }

  return <>{t(i18nKey, values)}</>;
}

// Usage examples
export function Examples() {
  const { t } = useTranslation('study-tools');

  return (
    <div>
      {/* Simple translation */}
      <h1>{t('highlights.title')}</h1>

      {/* With interpolation */}
      <p>{t('highlights.count', { count: 5 })}</p>

      {/* With pluralization (automatically selected) */}
      <p>{t('bookmarks.count', { count: 1 })}</p>
      <p>{t('bookmarks.count', { count: 10 })}</p>

      {/* With components */}
      <Trans
        i18nKey="common:messages.termsLink"
        components={{
          link: <a href="/terms" />,
        }}
      />
    </div>
  );
}
```

```tsx
// src/components/LanguageSwitcher.tsx
import { useTranslation } from 'react-i18next';
import { supportedLanguages } from '@/i18n/config';

export function LanguageSwitcher() {
  const { i18n, t } = useTranslation();
  const currentLang = supportedLanguages.find(l => l.code === i18n.language);

  const handleLanguageChange = async (langCode: string) => {
    await i18n.changeLanguage(langCode);

    // Update document direction for RTL languages
    const lang = supportedLanguages.find(l => l.code === langCode);
    document.documentElement.dir = lang?.dir || 'ltr';
    document.documentElement.lang = langCode;

    // Persist preference
    localStorage.setItem('i18nextLng', langCode);
  };

  return (
    <div className="relative">
      <button
        className="flex items-center gap-2 px-3 py-2 rounded-lg hover:bg-gray-100"
        aria-label={t('settings.selectLanguage')}
      >
        <span className="text-lg">{getLanguageFlag(i18n.language)}</span>
        <span>{currentLang?.nativeName}</span>
        <ChevronDownIcon className="w-4 h-4" />
      </button>

      <div className="absolute right-0 mt-2 w-48 bg-white rounded-lg shadow-lg border">
        {supportedLanguages.map(lang => (
          <button
            key={lang.code}
            onClick={() => handleLanguageChange(lang.code)}
            className={`
              w-full px-4 py-2 text-left hover:bg-gray-50
              ${lang.code === i18n.language ? 'bg-blue-50 text-blue-700' : ''}
            `}
          >
            <span className="mr-2">{getLanguageFlag(lang.code)}</span>
            <span>{lang.nativeName}</span>
            <span className="text-gray-500 text-sm ml-2">({lang.name})</span>
          </button>
        ))}
      </div>
    </div>
  );
}

function getLanguageFlag(langCode: string): string {
  const flags: Record<string, string> = {
    'en': '🇺🇸',
    'th': '🇹🇭',
    'es': '🇪🇸',
    'zh-CN': '🇨🇳',
    'ar': '🇸🇦',
    'ko': '🇰🇷',
  };
  return flags[langCode] || '🌐';
}
```

---

## 4. Bible Content Localization

### 4.1 Bible Versions Data Model

```typescript
// src/types/bible.ts
interface BibleVersion {
  id: string;
  code: string;           // 'kjv', 'thai', 'niv'
  name: string;           // Localized name
  language: string;       // ISO language code
  textDirection: 'ltr' | 'rtl';
  copyright: string;
  description: string;
  isDefault: boolean;
  features: {
    hasAudio: boolean;
    hasStrongs: boolean;
    hasFootnotes: boolean;
  };
}

interface LocalizedBookName {
  id: string;
  versionId: string;
  bookCode: string;       // 'gen', 'exo', 'mat', etc.
  name: string;           // Localized book name
  abbreviation: string;   // Localized abbreviation
  chapterCount: number;
}
```

### 4.2 Book Names Localization

```typescript
// src/data/book-names.ts
export const bookNames: Record<string, Record<string, BookNameInfo>> = {
  en: {
    gen: { name: 'Genesis', abbr: 'Gen', chapters: 50 },
    exo: { name: 'Exodus', abbr: 'Exod', chapters: 40 },
    lev: { name: 'Leviticus', abbr: 'Lev', chapters: 27 },
    num: { name: 'Numbers', abbr: 'Num', chapters: 36 },
    deu: { name: 'Deuteronomy', abbr: 'Deut', chapters: 34 },
    // ... all 66 books
    mat: { name: 'Matthew', abbr: 'Matt', chapters: 28 },
    mrk: { name: 'Mark', abbr: 'Mark', chapters: 16 },
    luk: { name: 'Luke', abbr: 'Luke', chapters: 24 },
    jhn: { name: 'John', abbr: 'John', chapters: 21 },
    // ...
    rev: { name: 'Revelation', abbr: 'Rev', chapters: 22 },
  },
  th: {
    gen: { name: 'ปฐมกาล', abbr: 'ปฐก', chapters: 50 },
    exo: { name: 'อพยพ', abbr: 'อพย', chapters: 40 },
    lev: { name: 'เลวีนิติ', abbr: 'ลนต', chapters: 27 },
    num: { name: 'กันดารวิถี', abbr: 'กดว', chapters: 36 },
    deu: { name: 'เฉลยธรรมบัญญัติ', abbr: 'ฉธบ', chapters: 34 },
    // ...
    mat: { name: 'มัทธิว', abbr: 'มธ', chapters: 28 },
    mrk: { name: 'มาระโก', abbr: 'มก', chapters: 16 },
    luk: { name: 'ลูกา', abbr: 'ลก', chapters: 24 },
    jhn: { name: 'ยอห์น', abbr: 'ยน', chapters: 21 },
    // ...
    rev: { name: 'วิวรณ์', abbr: 'วว', chapters: 22 },
  },
  es: {
    gen: { name: 'Génesis', abbr: 'Gén', chapters: 50 },
    exo: { name: 'Éxodo', abbr: 'Éxod', chapters: 40 },
    // ...
  },
};

export function getLocalizedBookName(bookCode: string, language: string): BookNameInfo {
  const langBooks = bookNames[language] || bookNames.en;
  return langBooks[bookCode] || bookNames.en[bookCode];
}
```

### 4.3 Bible Reference Formatting

```typescript
// src/utils/reference-formatter.ts
import { getLocalizedBookName } from './book-names';

interface ReferenceFormat {
  book: string;
  chapter: number;
  verse?: number;
  verseEnd?: number;
}

export function formatReference(
  ref: ReferenceFormat,
  language: string,
  options: { abbreviated?: boolean } = {}
): string {
  const bookInfo = getLocalizedBookName(ref.book, language);
  const bookName = options.abbreviated ? bookInfo.abbr : bookInfo.name;

  let result = `${bookName} ${ref.chapter}`;

  if (ref.verse !== undefined) {
    // Use localized separators
    const separator = getVerseSeparator(language);
    result += `${separator}${ref.verse}`;

    if (ref.verseEnd !== undefined && ref.verseEnd !== ref.verse) {
      const rangeSeparator = getRangeSeparator(language);
      result += `${rangeSeparator}${ref.verseEnd}`;
    }
  }

  return result;
}

function getVerseSeparator(language: string): string {
  const separators: Record<string, string> = {
    en: ':',
    th: ':',
    es: ':',
    'zh-CN': '：',
    ar: ':',
    ko: ':',
  };
  return separators[language] || ':';
}

function getRangeSeparator(language: string): string {
  const separators: Record<string, string> = {
    en: '-',
    th: '-',
    es: '-',
    'zh-CN': '－',
    ar: '-',
    ko: '-',
  };
  return separators[language] || '-';
}

// Usage
formatReference({ book: 'jhn', chapter: 3, verse: 16 }, 'en'); // "John 3:16"
formatReference({ book: 'jhn', chapter: 3, verse: 16 }, 'th'); // "ยอห์น 3:16"
formatReference({ book: 'jhn', chapter: 3, verse: 16, verseEnd: 18 }, 'en'); // "John 3:16-18"
```

---

## 5. Date, Time & Number Formatting

### 5.1 Date/Time Formatting

```typescript
// src/utils/date-formatter.ts
import { formatDistanceToNow, format, parseISO } from 'date-fns';
import { enUS, th, es, zhCN, ar, ko } from 'date-fns/locale';

const locales: Record<string, Locale> = {
  en: enUS,
  th: th,
  es: es,
  'zh-CN': zhCN,
  ar: ar,
  ko: ko,
};

export function formatDate(
  date: string | Date,
  language: string,
  formatStr: string = 'PPP'
): string {
  const dateObj = typeof date === 'string' ? parseISO(date) : date;
  const locale = locales[language] || enUS;

  return format(dateObj, formatStr, { locale });
}

export function formatRelativeTime(
  date: string | Date,
  language: string
): string {
  const dateObj = typeof date === 'string' ? parseISO(date) : date;
  const locale = locales[language] || enUS;

  return formatDistanceToNow(dateObj, {
    addSuffix: true,
    locale,
  });
}

// Predefined format options
export const dateFormats = {
  short: 'P',          // 04/29/2023
  medium: 'PP',        // Apr 29, 2023
  long: 'PPP',         // April 29th, 2023
  full: 'PPPP',        // Saturday, April 29th, 2023
  time: 'p',           // 12:00 AM
  dateTime: 'Pp',      // 04/29/2023, 12:00 AM
  dayMonth: 'MMM d',   // Apr 29
};

// Usage
formatDate('2025-01-14', 'en', dateFormats.long);     // "January 14th, 2025"
formatDate('2025-01-14', 'th', dateFormats.long);     // "14 มกราคม 2568" (Thai calendar)
formatRelativeTime('2025-01-10', 'en');               // "4 days ago"
formatRelativeTime('2025-01-10', 'th');               // "4 วันที่ผ่านมา"
```

### 5.2 Number Formatting

```typescript
// src/utils/number-formatter.ts
export function formatNumber(
  value: number,
  language: string,
  options?: Intl.NumberFormatOptions
): string {
  return new Intl.NumberFormat(language, options).format(value);
}

export function formatPercent(value: number, language: string): string {
  return new Intl.NumberFormat(language, {
    style: 'percent',
    minimumFractionDigits: 0,
    maximumFractionDigits: 1,
  }).format(value);
}

export function formatCompact(value: number, language: string): string {
  return new Intl.NumberFormat(language, {
    notation: 'compact',
    compactDisplay: 'short',
  }).format(value);
}

// Usage for reading progress
export function formatReadingProgress(
  completed: number,
  total: number,
  language: string
): string {
  const percent = completed / total;
  return formatPercent(percent, language);
}

// Usage
formatNumber(1234567, 'en');     // "1,234,567"
formatNumber(1234567, 'th');     // "1,234,567"
formatNumber(1234567, 'ar');     // "١٬٢٣٤٬٥٦٧" (Arabic numerals)
formatPercent(0.75, 'en');       // "75%"
formatPercent(0.75, 'th');       // "75%"
formatCompact(1234567, 'en');    // "1.2M"
formatCompact(1234567, 'th');    // "1.2 ล้าน"
```

---

## 6. RTL Language Support

### 6.1 CSS Configuration

```css
/* styles/rtl.css */
/* Base RTL styles */
[dir="rtl"] {
  /* Typography */
  text-align: right;

  /* Override directional properties */
  --direction: rtl;
  --start: right;
  --end: left;
}

/* Flip margins and paddings */
[dir="rtl"] .ml-4 { margin-left: 0; margin-right: 1rem; }
[dir="rtl"] .mr-4 { margin-right: 0; margin-left: 1rem; }
[dir="rtl"] .pl-4 { padding-left: 0; padding-right: 1rem; }
[dir="rtl"] .pr-4 { padding-right: 0; padding-left: 1rem; }

/* Flip borders */
[dir="rtl"] .border-l { border-left: none; border-right-width: 1px; }
[dir="rtl"] .border-r { border-right: none; border-left-width: 1px; }

/* Flip rounded corners */
[dir="rtl"] .rounded-l { border-radius: 0 0.25rem 0.25rem 0; }
[dir="rtl"] .rounded-r { border-radius: 0.25rem 0 0 0.25rem; }

/* Flip transforms */
[dir="rtl"] .translate-x-full { transform: translateX(-100%); }
[dir="rtl"] .-translate-x-full { transform: translateX(100%); }

/* Flip flex direction where needed */
[dir="rtl"] .flex-row-reverse-rtl { flex-direction: row-reverse; }

/* Icons that should flip */
[dir="rtl"] .icon-flip {
  transform: scaleX(-1);
}

/* Prevent number reversal */
[dir="rtl"] .ltr-number {
  direction: ltr;
  unicode-bidi: embed;
}
```

### 6.2 Tailwind RTL Configuration

```javascript
// tailwind.config.js
const plugin = require('tailwindcss/plugin');

module.exports = {
  plugins: [
    // RTL-aware utilities
    plugin(function({ addUtilities, addVariant }) {
      // Add RTL variant
      addVariant('rtl', '[dir="rtl"] &');
      addVariant('ltr', '[dir="ltr"] &');

      // Logical properties utilities
      addUtilities({
        '.ms-auto': { 'margin-inline-start': 'auto' },
        '.me-auto': { 'margin-inline-end': 'auto' },
        '.ps-4': { 'padding-inline-start': '1rem' },
        '.pe-4': { 'padding-inline-end': '1rem' },
        '.start-0': { 'inset-inline-start': '0' },
        '.end-0': { 'inset-inline-end': '0' },
        '.border-s': { 'border-inline-start-width': '1px' },
        '.border-e': { 'border-inline-end-width': '1px' },
        '.rounded-s': { 'border-start-start-radius': '0.25rem', 'border-end-start-radius': '0.25rem' },
        '.rounded-e': { 'border-start-end-radius': '0.25rem', 'border-end-end-radius': '0.25rem' },
        '.text-start': { 'text-align': 'start' },
        '.text-end': { 'text-align': 'end' },
      });
    }),
  ],
};
```

### 6.3 RTL-Aware Components

```tsx
// src/components/DirectionalIcon.tsx
import { useTranslation } from 'react-i18next';

interface DirectionalIconProps {
  icon: React.ComponentType<{ className?: string }>;
  className?: string;
  flip?: boolean;
}

export function DirectionalIcon({ icon: Icon, className = '', flip = true }: DirectionalIconProps) {
  const { i18n } = useTranslation();
  const isRTL = i18n.dir() === 'rtl';

  return (
    <Icon
      className={`${className} ${flip && isRTL ? 'transform scale-x-[-1]' : ''}`}
    />
  );
}

// Navigation arrows
export function NavigationArrows() {
  const { i18n, t } = useTranslation();
  const isRTL = i18n.dir() === 'rtl';

  // In RTL, "next" is visually left, "previous" is visually right
  const PrevIcon = isRTL ? ChevronRightIcon : ChevronLeftIcon;
  const NextIcon = isRTL ? ChevronLeftIcon : ChevronRightIcon;

  return (
    <div className="flex gap-2">
      <button aria-label={t('bible:reader.previousChapter')}>
        <PrevIcon className="w-6 h-6" />
      </button>
      <button aria-label={t('bible:reader.nextChapter')}>
        <NextIcon className="w-6 h-6" />
      </button>
    </div>
  );
}
```

```tsx
// src/components/BibleVerse.tsx
interface BibleVerseProps {
  verseNumber: number;
  text: string;
  textDirection: 'ltr' | 'rtl';
  language: string;
}

export function BibleVerse({ verseNumber, text, textDirection, language }: BibleVerseProps) {
  return (
    <div
      className="verse-container flex gap-2"
      dir={textDirection}
      lang={language}
    >
      {/* Verse number should always be in consistent position */}
      <span
        className="verse-number text-sm text-gray-500 font-medium ltr-number"
        dir="ltr"
      >
        {verseNumber}
      </span>
      <span className="verse-text flex-1">
        {text}
      </span>
    </div>
  );
}
```

---

## 7. Text-to-Speech Localization

### 7.1 TTS Language Configuration

```typescript
// src/config/tts-languages.ts
interface TTSVoiceConfig {
  language: string;
  voices: {
    name: string;
    lang: string;
    gender: 'male' | 'female';
    isDefault: boolean;
  }[];
  defaultRate: number;
  rateRange: { min: number; max: number };
}

export const ttsConfig: Record<string, TTSVoiceConfig> = {
  en: {
    language: 'en-US',
    voices: [
      { name: 'Google US English', lang: 'en-US', gender: 'female', isDefault: true },
      { name: 'Google UK English Female', lang: 'en-GB', gender: 'female', isDefault: false },
      { name: 'Google UK English Male', lang: 'en-GB', gender: 'male', isDefault: false },
    ],
    defaultRate: 1.0,
    rateRange: { min: 0.5, max: 2.0 },
  },
  th: {
    language: 'th-TH',
    voices: [
      { name: 'Google Thai', lang: 'th-TH', gender: 'female', isDefault: true },
    ],
    defaultRate: 0.9, // Thai may need slightly slower rate
    rateRange: { min: 0.5, max: 1.5 },
  },
  es: {
    language: 'es-ES',
    voices: [
      { name: 'Google Español', lang: 'es-ES', gender: 'female', isDefault: true },
      { name: 'Google Español de Estados Unidos', lang: 'es-US', gender: 'female', isDefault: false },
    ],
    defaultRate: 1.0,
    rateRange: { min: 0.5, max: 2.0 },
  },
  ar: {
    language: 'ar-SA',
    voices: [
      { name: 'Google العربية', lang: 'ar-SA', gender: 'female', isDefault: true },
    ],
    defaultRate: 0.9,
    rateRange: { min: 0.5, max: 1.5 },
  },
};
```

### 7.2 TTS Hook with Language Support

```typescript
// src/hooks/useTTS.ts
import { useState, useEffect, useCallback } from 'react';
import { useTranslation } from 'react-i18next';
import { ttsConfig } from '@/config/tts-languages';

export function useTTS(bibleLanguage?: string) {
  const { i18n } = useTranslation();
  const [voices, setVoices] = useState<SpeechSynthesisVoice[]>([]);
  const [selectedVoice, setSelectedVoice] = useState<SpeechSynthesisVoice | null>(null);
  const [isPlaying, setIsPlaying] = useState(false);
  const [rate, setRate] = useState(1.0);

  // Use Bible content language or UI language
  const targetLanguage = bibleLanguage || i18n.language;
  const config = ttsConfig[targetLanguage] || ttsConfig.en;

  useEffect(() => {
    const loadVoices = () => {
      const availableVoices = speechSynthesis.getVoices();
      const languageVoices = availableVoices.filter(
        voice => voice.lang.startsWith(config.language.split('-')[0])
      );
      setVoices(languageVoices);

      // Select default voice
      const defaultVoice = languageVoices.find(v =>
        config.voices.some(cv => cv.isDefault && v.name.includes(cv.name))
      ) || languageVoices[0];

      setSelectedVoice(defaultVoice);
      setRate(config.defaultRate);
    };

    loadVoices();
    speechSynthesis.onvoiceschanged = loadVoices;

    return () => {
      speechSynthesis.onvoiceschanged = null;
    };
  }, [targetLanguage, config]);

  const speak = useCallback((text: string) => {
    if (!selectedVoice) return;

    speechSynthesis.cancel();

    const utterance = new SpeechSynthesisUtterance(text);
    utterance.voice = selectedVoice;
    utterance.rate = rate;
    utterance.lang = config.language;

    utterance.onstart = () => setIsPlaying(true);
    utterance.onend = () => setIsPlaying(false);
    utterance.onerror = () => setIsPlaying(false);

    speechSynthesis.speak(utterance);
  }, [selectedVoice, rate, config.language]);

  const stop = useCallback(() => {
    speechSynthesis.cancel();
    setIsPlaying(false);
  }, []);

  return {
    voices,
    selectedVoice,
    setSelectedVoice,
    rate,
    setRate,
    rateRange: config.rateRange,
    isPlaying,
    speak,
    stop,
    isSupported: 'speechSynthesis' in window,
  };
}
```

---

## 8. Translation Workflow

### 8.1 Translation Management

```yaml
# .github/workflows/i18n.yml
name: i18n Workflow

on:
  push:
    paths:
      - 'locales/**'
  pull_request:
    paths:
      - 'locales/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Validate JSON syntax
        run: |
          for file in locales/**/*.json; do
            echo "Validating $file"
            node -e "JSON.parse(require('fs').readFileSync('$file'))"
          done

      - name: Check for missing translations
        run: npm run i18n:check

      - name: Check for unused translations
        run: npm run i18n:unused
```

### 8.2 Translation Scripts

```typescript
// scripts/i18n-check.ts
import fs from 'fs';
import path from 'path';
import glob from 'glob';

const LOCALES_DIR = './locales';
const SOURCE_LANG = 'en';

interface TranslationKeys {
  [key: string]: TranslationKeys | string;
}

function getKeys(obj: TranslationKeys, prefix = ''): string[] {
  const keys: string[] = [];
  for (const key of Object.keys(obj)) {
    const fullKey = prefix ? `${prefix}.${key}` : key;
    if (typeof obj[key] === 'object') {
      keys.push(...getKeys(obj[key] as TranslationKeys, fullKey));
    } else {
      keys.push(fullKey);
    }
  }
  return keys;
}

function checkMissingTranslations() {
  const languages = fs.readdirSync(LOCALES_DIR);
  const namespaces = fs.readdirSync(path.join(LOCALES_DIR, SOURCE_LANG))
    .map(f => f.replace('.json', ''));

  const missing: Record<string, string[]> = {};

  for (const ns of namespaces) {
    const sourceFile = path.join(LOCALES_DIR, SOURCE_LANG, `${ns}.json`);
    const sourceKeys = getKeys(JSON.parse(fs.readFileSync(sourceFile, 'utf-8')));

    for (const lang of languages) {
      if (lang === SOURCE_LANG) continue;

      const targetFile = path.join(LOCALES_DIR, lang, `${ns}.json`);
      if (!fs.existsSync(targetFile)) {
        missing[lang] = missing[lang] || [];
        missing[lang].push(`Missing file: ${ns}.json`);
        continue;
      }

      const targetKeys = getKeys(JSON.parse(fs.readFileSync(targetFile, 'utf-8')));
      const missingKeys = sourceKeys.filter(k => !targetKeys.includes(k));

      if (missingKeys.length > 0) {
        missing[lang] = missing[lang] || [];
        missing[lang].push(...missingKeys.map(k => `${ns}:${k}`));
      }
    }
  }

  if (Object.keys(missing).length > 0) {
    console.error('Missing translations:');
    for (const [lang, keys] of Object.entries(missing)) {
      console.error(`\n${lang}:`);
      keys.forEach(k => console.error(`  - ${k}`));
    }
    process.exit(1);
  }

  console.log('All translations are complete!');
}

checkMissingTranslations();
```

### 8.3 Translation Guidelines Document

```markdown
## Translation Guidelines

### General Rules
1. Maintain consistent terminology throughout
2. Use formal language (not colloquial)
3. Preserve placeholders exactly: {{variable}}
4. Keep HTML tags if present: <strong>, <em>
5. Don't translate brand names or technical terms

### Bible-Specific Rules
1. Use official Bible book names for each language
2. Maintain verse numbering format (e.g., "John 3:16")
3. Use traditional religious terminology where applicable
4. Preserve capitalization of sacred names

### Thai-Specific Guidelines
1. Use "พระคัมภีร์" for "Bible"
2. Use "ข้อ" for "verse"
3. Use "บท" for "chapter"
4. Use "พระธรรม" for "book" (of the Bible)
5. Use royal vocabulary when referring to God/Jesus

### Pluralization
- English: Use `_plural` suffix for plural forms
- Thai: Single form (no plural distinction)
- Arabic: May need `_zero`, `_one`, `_two`, `_few`, `_many`

### Context
Add `_context` suffix for translator notes:
```json
{
  "save": "Save",
  "save_context": "Button to save changes"
}
```
```

---

## 9. Implementation Guide

### 9.1 Server-Side Rendering

```typescript
// pages/_app.tsx
import { appWithTranslation } from 'next-i18next';
import { useRouter } from 'next/router';
import { useEffect } from 'react';

function MyApp({ Component, pageProps }) {
  const router = useRouter();

  // Set document direction based on locale
  useEffect(() => {
    const locale = router.locale || 'en';
    const dir = ['ar', 'he', 'fa'].includes(locale) ? 'rtl' : 'ltr';
    document.documentElement.dir = dir;
    document.documentElement.lang = locale;
  }, [router.locale]);

  return <Component {...pageProps} />;
}

export default appWithTranslation(MyApp);
```

```typescript
// pages/bible/[version]/[book]/[chapter].tsx
import { GetStaticProps, GetStaticPaths } from 'next';
import { serverSideTranslations } from 'next-i18next/serverSideTranslations';
import { useTranslation } from 'next-i18next';

export const getStaticProps: GetStaticProps = async ({ locale, params }) => {
  return {
    props: {
      ...(await serverSideTranslations(locale!, ['common', 'bible'])),
      version: params?.version,
      book: params?.book,
      chapter: params?.chapter,
    },
    revalidate: 3600, // Revalidate every hour
  };
};

export const getStaticPaths: GetStaticPaths = async ({ locales }) => {
  // Generate paths for all locales
  const paths = locales!.flatMap(locale => [
    { params: { version: 'kjv', book: 'john', chapter: '3' }, locale },
    // Add more common paths for pre-rendering
  ]);

  return {
    paths,
    fallback: 'blocking',
  };
};

export default function ChapterPage({ version, book, chapter }) {
  const { t } = useTranslation(['bible', 'common']);

  return (
    <div>
      <h1>{t('bible:reader.chapter', { number: chapter })}</h1>
      {/* Chapter content */}
    </div>
  );
}
```

### 9.2 API Localization

```typescript
// backend/src/middleware/i18n.ts
import { Request, Response, NextFunction } from 'express';

const supportedLocales = ['en', 'th', 'es', 'zh-CN', 'ar', 'ko'];
const defaultLocale = 'en';

export function i18nMiddleware(req: Request, res: Response, next: NextFunction) {
  // Get locale from various sources
  const locale =
    req.query.lang as string ||
    req.headers['accept-language']?.split(',')[0]?.split('-')[0] ||
    defaultLocale;

  // Validate and set locale
  req.locale = supportedLocales.includes(locale) ? locale : defaultLocale;

  next();
}

// Localized error messages
export function getLocalizedError(code: string, locale: string): string {
  const errors: Record<string, Record<string, string>> = {
    INVALID_CREDENTIALS: {
      en: 'Invalid email or password',
      th: 'อีเมลหรือรหัสผ่านไม่ถูกต้อง',
    },
    NOT_FOUND: {
      en: 'Resource not found',
      th: 'ไม่พบข้อมูล',
    },
    UNAUTHORIZED: {
      en: 'Please log in to continue',
      th: 'กรุณาเข้าสู่ระบบเพื่อดำเนินการต่อ',
    },
  };

  return errors[code]?.[locale] || errors[code]?.en || code;
}
```

---

## 10. Testing i18n

### 10.1 Unit Tests

```typescript
// src/__tests__/i18n.test.ts
import { describe, it, expect } from 'vitest';
import { formatReference, getLocalizedBookName } from '@/utils/bible';
import { formatDate, formatNumber } from '@/utils/formatters';

describe('i18n Utilities', () => {
  describe('getLocalizedBookName', () => {
    it('should return English book name', () => {
      expect(getLocalizedBookName('gen', 'en').name).toBe('Genesis');
      expect(getLocalizedBookName('mat', 'en').name).toBe('Matthew');
    });

    it('should return Thai book name', () => {
      expect(getLocalizedBookName('gen', 'th').name).toBe('ปฐมกาล');
      expect(getLocalizedBookName('mat', 'th').name).toBe('มัทธิว');
    });

    it('should fallback to English for unknown language', () => {
      expect(getLocalizedBookName('gen', 'xx').name).toBe('Genesis');
    });
  });

  describe('formatReference', () => {
    it('should format English reference', () => {
      expect(formatReference({ book: 'jhn', chapter: 3, verse: 16 }, 'en'))
        .toBe('John 3:16');
    });

    it('should format Thai reference', () => {
      expect(formatReference({ book: 'jhn', chapter: 3, verse: 16 }, 'th'))
        .toBe('ยอห์น 3:16');
    });

    it('should format verse range', () => {
      expect(formatReference({ book: 'jhn', chapter: 3, verse: 16, verseEnd: 18 }, 'en'))
        .toBe('John 3:16-18');
    });
  });

  describe('formatDate', () => {
    it('should format date in English', () => {
      const result = formatDate('2025-01-14', 'en', 'PPP');
      expect(result).toContain('January');
      expect(result).toContain('14');
      expect(result).toContain('2025');
    });

    it('should format date in Thai', () => {
      const result = formatDate('2025-01-14', 'th', 'PPP');
      expect(result).toContain('มกราคม');
    });
  });

  describe('formatNumber', () => {
    it('should format with English locale', () => {
      expect(formatNumber(1234567, 'en')).toBe('1,234,567');
    });

    it('should format with Arabic locale (Arabic numerals)', () => {
      const result = formatNumber(123, 'ar-SA');
      // Arabic numerals: ١٢٣
      expect(result).not.toBe('123');
    });
  });
});
```

### 10.2 E2E Tests

```typescript
// e2e/i18n.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Internationalization', () => {
  test('should display UI in English by default', async ({ page }) => {
    await page.goto('/');

    await expect(page.getByText('Online Bible Reader')).toBeVisible();
    await expect(page.getByText('Reading Plans')).toBeVisible();
  });

  test('should switch to Thai language', async ({ page }) => {
    await page.goto('/');

    // Open language switcher
    await page.click('[data-testid="language-switcher"]');

    // Select Thai
    await page.click('text=ไทย');

    // Verify Thai UI
    await expect(page.getByText('อ่านพระคัมภีร์ออนไลน์')).toBeVisible();
    await expect(page.getByText('แผนการอ่าน')).toBeVisible();
  });

  test('should display Thai Bible book names', async ({ page }) => {
    await page.goto('/th/bible');

    await expect(page.getByText('ปฐมกาล')).toBeVisible(); // Genesis
    await expect(page.getByText('มัทธิว')).toBeVisible();  // Matthew
  });

  test('should format dates according to locale', async ({ page }) => {
    await page.goto('/th/reading-plans');

    // Thai date format should be different from English
    const dateElement = page.locator('[data-testid="plan-date"]').first();
    const dateText = await dateElement.textContent();

    // Thai months contain Thai characters
    expect(dateText).toMatch(/[ก-๙]/);
  });

  test('should support RTL for Arabic', async ({ page }) => {
    await page.goto('/ar');

    // Check document direction
    const dir = await page.evaluate(() => document.documentElement.dir);
    expect(dir).toBe('rtl');

    // Check text alignment
    const textAlign = await page.locator('main').evaluate(
      el => getComputedStyle(el).textAlign
    );
    expect(textAlign).toBe('right');
  });

  test('should persist language preference', async ({ page }) => {
    await page.goto('/');

    // Switch to Thai
    await page.click('[data-testid="language-switcher"]');
    await page.click('text=ไทย');

    // Navigate away and back
    await page.goto('/reading-plans');
    await page.goto('/');

    // Should still be in Thai
    await expect(page.getByText('อ่านพระคัมภีร์ออนไลน์')).toBeVisible();
  });
});
```

---

## Summary

This internationalization guide provides comprehensive support for multiple languages in the Online Bible Reader application:

| Feature | Implementation |
|---------|---------------|
| UI Translation | next-i18next with JSON files |
| Bible Content | Localized book names, verse formatting |
| Date/Time | date-fns with locale support |
| Numbers | Intl.NumberFormat |
| RTL Support | CSS logical properties, Tailwind plugin |
| TTS | Language-specific voice configuration |
| Testing | Unit tests + E2E tests for all locales |

**Supported Languages:** English (primary), Thai, Spanish, Chinese, Arabic (RTL), Korean

---

**Document Version**: 1.0.0
**Last Updated**: 2025-01-14
**Author**: Localization Team
