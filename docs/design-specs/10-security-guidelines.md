# Part 10: Security Guidelines

## Table of Contents

1. [Overview](#1-overview)
2. [Authentication Security](#2-authentication-security)
3. [Authorization & Access Control](#3-authorization--access-control)
4. [Data Protection](#4-data-protection)
5. [API Security](#5-api-security)
6. [Frontend Security](#6-frontend-security)
7. [Infrastructure Security](#7-infrastructure-security)
8. [OWASP Top 10 Mitigations](#8-owasp-top-10-mitigations)
9. [Security Headers](#9-security-headers)
10. [Logging & Monitoring](#10-logging--monitoring)
11. [Incident Response](#11-incident-response)
12. [Compliance & Privacy](#12-compliance--privacy)

---

## 1. Overview

### 1.1 Security Principles

- **Defense in Depth**: Multiple layers of security controls
- **Least Privilege**: Minimum access necessary for functionality
- **Secure by Default**: Security enabled out of the box
- **Fail Secure**: System fails to a secure state
- **Zero Trust**: Never trust, always verify

### 1.2 Security Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         SECURITY LAYERS                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Layer 1: Network Security                                       │   │
│  │  • WAF (Web Application Firewall)                                │   │
│  │  • DDoS Protection                                               │   │
│  │  • TLS 1.3 Encryption                                            │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              │                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Layer 2: Application Security                                   │   │
│  │  • Authentication (JWT + OAuth)                                  │   │
│  │  • Authorization (RBAC)                                          │   │
│  │  • Input Validation                                              │   │
│  │  • Output Encoding                                               │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              │                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Layer 3: Data Security                                          │   │
│  │  • Encryption at Rest                                            │   │
│  │  • Encryption in Transit                                         │   │
│  │  • Data Masking                                                  │   │
│  │  • Secure Key Management                                         │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Authentication Security

### 2.1 Password Policy

```typescript
// backend/src/config/security.ts
export const passwordPolicy = {
  minLength: 8,
  maxLength: 128,
  requireUppercase: true,
  requireLowercase: true,
  requireNumbers: true,
  requireSpecialChars: false, // Optional but encouraged
  preventCommon: true, // Check against common password list
  preventUserInfo: true, // Prevent email/name in password
  historyCount: 5, // Prevent reuse of last 5 passwords
  maxAge: 90, // Days before password change recommended
};

// Password validation
export function validatePassword(password: string, userInfo?: UserInfo): ValidationResult {
  const errors: string[] = [];

  if (password.length < passwordPolicy.minLength) {
    errors.push(`Password must be at least ${passwordPolicy.minLength} characters`);
  }

  if (password.length > passwordPolicy.maxLength) {
    errors.push(`Password must not exceed ${passwordPolicy.maxLength} characters`);
  }

  if (passwordPolicy.requireUppercase && !/[A-Z]/.test(password)) {
    errors.push('Password must contain at least one uppercase letter');
  }

  if (passwordPolicy.requireLowercase && !/[a-z]/.test(password)) {
    errors.push('Password must contain at least one lowercase letter');
  }

  if (passwordPolicy.requireNumbers && !/[0-9]/.test(password)) {
    errors.push('Password must contain at least one number');
  }

  if (passwordPolicy.preventCommon && isCommonPassword(password)) {
    errors.push('Password is too common. Please choose a stronger password');
  }

  if (passwordPolicy.preventUserInfo && userInfo) {
    if (password.toLowerCase().includes(userInfo.email.toLowerCase().split('@')[0])) {
      errors.push('Password cannot contain your email address');
    }
  }

  return {
    valid: errors.length === 0,
    errors,
  };
}
```

### 2.2 Password Hashing

```typescript
// backend/src/utils/password.ts
import argon2 from 'argon2';

const hashingConfig = {
  type: argon2.argon2id,
  memoryCost: 65536, // 64 MB
  timeCost: 3, // 3 iterations
  parallelism: 4, // 4 threads
  hashLength: 32,
};

export async function hashPassword(password: string): Promise<string> {
  return argon2.hash(password, hashingConfig);
}

export async function verifyPassword(password: string, hash: string): Promise<boolean> {
  try {
    return await argon2.verify(hash, password);
  } catch (error) {
    return false;
  }
}

// Check if password hash needs rehashing (config changed)
export async function needsRehash(hash: string): Promise<boolean> {
  return argon2.needsRehash(hash, hashingConfig);
}
```

### 2.3 JWT Token Security

```typescript
// backend/src/auth/jwt.ts
import jwt from 'jsonwebtoken';
import { v4 as uuidv4 } from 'uuid';

interface TokenPayload {
  sub: string; // User ID
  email: string;
  jti: string; // JWT ID for revocation
  iat: number;
  exp: number;
}

const accessTokenConfig = {
  expiresIn: '15m',
  algorithm: 'RS256' as const,
};

const refreshTokenConfig = {
  expiresIn: '7d',
  algorithm: 'RS256' as const,
};

export function generateAccessToken(user: User): string {
  const payload: Partial<TokenPayload> = {
    sub: user.id,
    email: user.email,
    jti: uuidv4(),
  };

  return jwt.sign(payload, process.env.JWT_PRIVATE_KEY!, {
    expiresIn: accessTokenConfig.expiresIn,
    algorithm: accessTokenConfig.algorithm,
    issuer: 'bible-reader-api',
    audience: 'bible-reader-app',
  });
}

export function generateRefreshToken(user: User): string {
  const payload: Partial<TokenPayload> = {
    sub: user.id,
    jti: uuidv4(),
  };

  return jwt.sign(payload, process.env.JWT_REFRESH_PRIVATE_KEY!, {
    expiresIn: refreshTokenConfig.expiresIn,
    algorithm: refreshTokenConfig.algorithm,
    issuer: 'bible-reader-api',
    audience: 'bible-reader-app',
  });
}

export function verifyAccessToken(token: string): TokenPayload {
  return jwt.verify(token, process.env.JWT_PUBLIC_KEY!, {
    algorithms: [accessTokenConfig.algorithm],
    issuer: 'bible-reader-api',
    audience: 'bible-reader-app',
  }) as TokenPayload;
}

// Token blacklist for logout/revocation
export class TokenBlacklist {
  private redis: RedisClient;

  async add(jti: string, expiresAt: number): Promise<void> {
    const ttl = expiresAt - Math.floor(Date.now() / 1000);
    if (ttl > 0) {
      await this.redis.setex(`blacklist:${jti}`, ttl, '1');
    }
  }

  async isBlacklisted(jti: string): Promise<boolean> {
    const result = await this.redis.get(`blacklist:${jti}`);
    return result !== null;
  }
}
```

### 2.4 OAuth Security

```typescript
// backend/src/auth/oauth.ts
import { OAuth2Client } from 'google-auth-library';

const googleClient = new OAuth2Client({
  clientId: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  redirectUri: `${process.env.APP_URL}/auth/google/callback`,
});

export async function verifyGoogleToken(idToken: string): Promise<GoogleUser | null> {
  try {
    const ticket = await googleClient.verifyIdToken({
      idToken,
      audience: process.env.GOOGLE_CLIENT_ID,
    });

    const payload = ticket.getPayload();
    if (!payload) return null;

    // Verify email is verified
    if (!payload.email_verified) {
      throw new Error('Email not verified by Google');
    }

    return {
      id: payload.sub,
      email: payload.email!,
      name: payload.name || '',
      picture: payload.picture || '',
    };
  } catch (error) {
    console.error('Google token verification failed:', error);
    return null;
  }
}

// State parameter for CSRF protection
export function generateOAuthState(): string {
  return crypto.randomBytes(32).toString('hex');
}

export function verifyOAuthState(state: string, storedState: string): boolean {
  return crypto.timingSafeEqual(
    Buffer.from(state),
    Buffer.from(storedState)
  );
}
```

### 2.5 Multi-Factor Authentication (MFA)

```typescript
// backend/src/auth/mfa.ts
import speakeasy from 'speakeasy';
import QRCode from 'qrcode';

export function generateMFASecret(email: string): MFASetup {
  const secret = speakeasy.generateSecret({
    name: `BibleReader:${email}`,
    issuer: 'Bible Reader',
    length: 32,
  });

  return {
    secret: secret.base32,
    otpauthUrl: secret.otpauth_url!,
  };
}

export async function generateMFAQRCode(otpauthUrl: string): Promise<string> {
  return QRCode.toDataURL(otpauthUrl);
}

export function verifyMFAToken(secret: string, token: string): boolean {
  return speakeasy.totp.verify({
    secret,
    encoding: 'base32',
    token,
    window: 1, // Allow 1 step tolerance (30 seconds)
  });
}

// Backup codes
export function generateBackupCodes(count: number = 10): string[] {
  const codes: string[] = [];
  for (let i = 0; i < count; i++) {
    const code = crypto.randomBytes(4).toString('hex').toUpperCase();
    codes.push(`${code.slice(0, 4)}-${code.slice(4)}`);
  }
  return codes;
}
```

### 2.6 Account Lockout

```typescript
// backend/src/auth/lockout.ts
export class AccountLockout {
  private redis: RedisClient;

  private readonly maxAttempts = 5;
  private readonly lockoutDuration = 15 * 60; // 15 minutes
  private readonly attemptWindow = 5 * 60; // 5 minutes

  async recordFailedAttempt(identifier: string): Promise<LockoutStatus> {
    const key = `lockout:${identifier}`;
    const attempts = await this.redis.incr(key);

    if (attempts === 1) {
      await this.redis.expire(key, this.attemptWindow);
    }

    if (attempts >= this.maxAttempts) {
      await this.redis.setex(`locked:${identifier}`, this.lockoutDuration, '1');
      return {
        locked: true,
        remainingAttempts: 0,
        lockoutEndsAt: Date.now() + this.lockoutDuration * 1000,
      };
    }

    return {
      locked: false,
      remainingAttempts: this.maxAttempts - attempts,
    };
  }

  async isLocked(identifier: string): Promise<boolean> {
    const locked = await this.redis.get(`locked:${identifier}`);
    return locked !== null;
  }

  async resetAttempts(identifier: string): Promise<void> {
    await this.redis.del(`lockout:${identifier}`);
    await this.redis.del(`locked:${identifier}`);
  }
}
```

---

## 3. Authorization & Access Control

### 3.1 Role-Based Access Control (RBAC)

```typescript
// backend/src/auth/rbac.ts
export enum Role {
  USER = 'user',
  PREMIUM = 'premium',
  ADMIN = 'admin',
  SUPER_ADMIN = 'super_admin',
}

export enum Permission {
  // Bible
  READ_BIBLE = 'bible:read',
  READ_ALL_VERSIONS = 'bible:read:all',

  // Study Tools
  CREATE_HIGHLIGHT = 'highlight:create',
  READ_HIGHLIGHT = 'highlight:read',
  DELETE_HIGHLIGHT = 'highlight:delete',

  CREATE_BOOKMARK = 'bookmark:create',
  READ_BOOKMARK = 'bookmark:read',
  DELETE_BOOKMARK = 'bookmark:delete',

  CREATE_NOTE = 'note:create',
  READ_NOTE = 'note:read',
  UPDATE_NOTE = 'note:update',
  DELETE_NOTE = 'note:delete',

  // Reading Plans
  READ_PLANS = 'plan:read',
  CREATE_PLAN = 'plan:create', // Custom plans
  JOIN_PLAN = 'plan:join',

  // Admin
  MANAGE_USERS = 'user:manage',
  MANAGE_CONTENT = 'content:manage',
  VIEW_ANALYTICS = 'analytics:view',
}

const rolePermissions: Record<Role, Permission[]> = {
  [Role.USER]: [
    Permission.READ_BIBLE,
    Permission.CREATE_HIGHLIGHT,
    Permission.READ_HIGHLIGHT,
    Permission.DELETE_HIGHLIGHT,
    Permission.CREATE_BOOKMARK,
    Permission.READ_BOOKMARK,
    Permission.DELETE_BOOKMARK,
    Permission.CREATE_NOTE,
    Permission.READ_NOTE,
    Permission.UPDATE_NOTE,
    Permission.DELETE_NOTE,
    Permission.READ_PLANS,
    Permission.JOIN_PLAN,
  ],
  [Role.PREMIUM]: [
    // All USER permissions plus:
    Permission.READ_ALL_VERSIONS,
    Permission.CREATE_PLAN,
  ],
  [Role.ADMIN]: [
    // All PREMIUM permissions plus:
    Permission.MANAGE_CONTENT,
    Permission.VIEW_ANALYTICS,
  ],
  [Role.SUPER_ADMIN]: [
    // All permissions
    ...Object.values(Permission),
  ],
};

export function hasPermission(userRole: Role, permission: Permission): boolean {
  const permissions = rolePermissions[userRole] || [];
  return permissions.includes(permission);
}

// Middleware
export function requirePermission(permission: Permission) {
  return (req: Request, res: Response, next: NextFunction) => {
    const user = req.user;

    if (!user) {
      return res.status(401).json({ error: 'Unauthorized' });
    }

    if (!hasPermission(user.role, permission)) {
      return res.status(403).json({ error: 'Forbidden' });
    }

    next();
  };
}
```

### 3.2 Resource-Based Access Control

```typescript
// backend/src/auth/resource-access.ts
export class ResourceAccessControl {
  // Check if user owns the resource
  async canAccess(userId: string, resourceType: string, resourceId: string): Promise<boolean> {
    switch (resourceType) {
      case 'highlight':
        return this.checkHighlightOwnership(userId, resourceId);
      case 'bookmark':
        return this.checkBookmarkOwnership(userId, resourceId);
      case 'note':
        return this.checkNoteOwnership(userId, resourceId);
      default:
        return false;
    }
  }

  private async checkHighlightOwnership(userId: string, highlightId: string): Promise<boolean> {
    const result = await db.query(
      'SELECT user_id FROM highlights WHERE id = $1',
      [highlightId]
    );
    return result.rows[0]?.user_id === userId;
  }

  // Similar methods for other resources...
}

// Middleware
export function requireResourceOwnership(resourceType: string) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const userId = req.user?.id;
    const resourceId = req.params.id;

    if (!userId) {
      return res.status(401).json({ error: 'Unauthorized' });
    }

    const rac = new ResourceAccessControl();
    const canAccess = await rac.canAccess(userId, resourceType, resourceId);

    if (!canAccess) {
      return res.status(403).json({ error: 'Access denied to this resource' });
    }

    next();
  };
}
```

---

## 4. Data Protection

### 4.1 Encryption at Rest

```typescript
// backend/src/security/encryption.ts
import crypto from 'crypto';

const ALGORITHM = 'aes-256-gcm';
const IV_LENGTH = 12;
const AUTH_TAG_LENGTH = 16;
const SALT_LENGTH = 32;

export class DataEncryption {
  private masterKey: Buffer;

  constructor() {
    // Master key from environment (should be from KMS in production)
    this.masterKey = Buffer.from(process.env.ENCRYPTION_KEY!, 'base64');
  }

  // Derive a key for specific data types
  private deriveKey(salt: Buffer, context: string): Buffer {
    return crypto.pbkdf2Sync(
      this.masterKey,
      Buffer.concat([salt, Buffer.from(context)]),
      100000,
      32,
      'sha256'
    );
  }

  encrypt(plaintext: string, context: string): string {
    const salt = crypto.randomBytes(SALT_LENGTH);
    const key = this.deriveKey(salt, context);
    const iv = crypto.randomBytes(IV_LENGTH);

    const cipher = crypto.createCipheriv(ALGORITHM, key, iv);
    let encrypted = cipher.update(plaintext, 'utf8', 'base64');
    encrypted += cipher.final('base64');

    const authTag = cipher.getAuthTag();

    // Combine: salt + iv + authTag + encrypted
    return Buffer.concat([
      salt,
      iv,
      authTag,
      Buffer.from(encrypted, 'base64'),
    ]).toString('base64');
  }

  decrypt(ciphertext: string, context: string): string {
    const data = Buffer.from(ciphertext, 'base64');

    const salt = data.subarray(0, SALT_LENGTH);
    const iv = data.subarray(SALT_LENGTH, SALT_LENGTH + IV_LENGTH);
    const authTag = data.subarray(
      SALT_LENGTH + IV_LENGTH,
      SALT_LENGTH + IV_LENGTH + AUTH_TAG_LENGTH
    );
    const encrypted = data.subarray(SALT_LENGTH + IV_LENGTH + AUTH_TAG_LENGTH);

    const key = this.deriveKey(salt, context);

    const decipher = crypto.createDecipheriv(ALGORITHM, key, iv);
    decipher.setAuthTag(authTag);

    let decrypted = decipher.update(encrypted);
    decrypted = Buffer.concat([decrypted, decipher.final()]);

    return decrypted.toString('utf8');
  }
}

// Usage for sensitive user data
export function encryptUserNote(userId: string, content: string): string {
  const encryption = new DataEncryption();
  return encryption.encrypt(content, `user:${userId}:note`);
}
```

### 4.2 Data Masking

```typescript
// backend/src/security/masking.ts
export function maskEmail(email: string): string {
  const [local, domain] = email.split('@');
  if (local.length <= 2) {
    return `${local[0]}***@${domain}`;
  }
  return `${local[0]}***${local[local.length - 1]}@${domain}`;
}

export function maskPhoneNumber(phone: string): string {
  // Show only last 4 digits
  return phone.replace(/\d(?=\d{4})/g, '*');
}

// Sanitize user object for API responses
export function sanitizeUser(user: User): SanitizedUser {
  return {
    id: user.id,
    displayName: user.displayName,
    email: maskEmail(user.email),
    role: user.role,
    createdAt: user.createdAt,
    // Exclude: passwordHash, mfaSecret, etc.
  };
}

// Sanitize for logging (prevent sensitive data in logs)
export function sanitizeForLogging(data: Record<string, any>): Record<string, any> {
  const sensitiveFields = ['password', 'token', 'secret', 'key', 'authorization'];
  const sanitized = { ...data };

  for (const field of sensitiveFields) {
    if (field in sanitized) {
      sanitized[field] = '[REDACTED]';
    }
  }

  return sanitized;
}
```

### 4.3 Secure Data Deletion

```typescript
// backend/src/security/data-deletion.ts
export class SecureDataDeletion {
  async deleteUserData(userId: string): Promise<void> {
    // Use transaction for atomicity
    const client = await pool.connect();

    try {
      await client.query('BEGIN');

      // Delete in correct order (foreign key constraints)
      await client.query('DELETE FROM notes WHERE user_id = $1', [userId]);
      await client.query('DELETE FROM highlights WHERE user_id = $1', [userId]);
      await client.query('DELETE FROM bookmarks WHERE user_id = $1', [userId]);
      await client.query('DELETE FROM user_reading_progress WHERE user_id = $1', [userId]);
      await client.query('DELETE FROM user_reading_plans WHERE user_id = $1', [userId]);
      await client.query('DELETE FROM push_subscriptions WHERE user_id = $1', [userId]);
      await client.query('DELETE FROM refresh_tokens WHERE user_id = $1', [userId]);
      await client.query('DELETE FROM users WHERE id = $1', [userId]);

      await client.query('COMMIT');

      // Clear from cache
      await redis.del(`user:${userId}`);
      await redis.del(`user:${userId}:*`);

      // Log deletion for compliance
      logger.info('User data deleted', { userId, timestamp: new Date().toISOString() });

    } catch (error) {
      await client.query('ROLLBACK');
      throw error;
    } finally {
      client.release();
    }
  }

  // GDPR data export
  async exportUserData(userId: string): Promise<UserDataExport> {
    const user = await this.getUser(userId);
    const highlights = await this.getUserHighlights(userId);
    const bookmarks = await this.getUserBookmarks(userId);
    const notes = await this.getUserNotes(userId);
    const readingProgress = await this.getUserReadingProgress(userId);

    return {
      exportDate: new Date().toISOString(),
      user: {
        email: user.email,
        displayName: user.displayName,
        createdAt: user.createdAt,
      },
      highlights,
      bookmarks,
      notes,
      readingProgress,
    };
  }
}
```

---

## 5. API Security

### 5.1 Rate Limiting

```typescript
// backend/src/middleware/rate-limit.ts
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';

// General API rate limit
export const apiLimiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rl:api:',
  }),
  windowMs: 60 * 1000, // 1 minute
  max: 100, // 100 requests per minute
  standardHeaders: true,
  legacyHeaders: false,
  keyGenerator: (req) => req.user?.id || req.ip,
  handler: (req, res) => {
    res.status(429).json({
      error: 'Too Many Requests',
      message: 'Rate limit exceeded. Please try again later.',
      retryAfter: res.getHeader('Retry-After'),
    });
  },
});

// Stricter limit for authentication endpoints
export const authLimiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rl:auth:',
  }),
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 10, // 10 attempts per 15 minutes
  keyGenerator: (req) => req.ip,
  skipSuccessfulRequests: true, // Only count failed attempts
});

// Search endpoint limiter
export const searchLimiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rl:search:',
  }),
  windowMs: 60 * 1000,
  max: 30, // 30 searches per minute
});

// Expensive operations limiter
export const expensiveLimiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rl:expensive:',
  }),
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 10, // 10 per hour
});
```

### 5.2 Input Validation

```typescript
// backend/src/validation/schemas.ts
import { z } from 'zod';
import xss from 'xss';

// Custom XSS sanitizer
const sanitizeString = (val: string) => xss(val.trim(), {
  whiteList: {},
  stripIgnoreTag: true,
  stripIgnoreTagBody: ['script', 'style'],
});

// User registration schema
export const registerSchema = z.object({
  email: z.string()
    .email('Invalid email format')
    .max(255, 'Email too long')
    .transform(val => val.toLowerCase().trim()),

  password: z.string()
    .min(8, 'Password must be at least 8 characters')
    .max(128, 'Password too long')
    .regex(/[A-Z]/, 'Password must contain uppercase letter')
    .regex(/[a-z]/, 'Password must contain lowercase letter')
    .regex(/[0-9]/, 'Password must contain a number'),

  displayName: z.string()
    .min(1, 'Display name required')
    .max(100, 'Display name too long')
    .transform(sanitizeString),
});

// Note creation schema
export const createNoteSchema = z.object({
  verseId: z.string()
    .uuid('Invalid verse ID format'),

  content: z.string()
    .min(1, 'Note content required')
    .max(10000, 'Note too long')
    .transform(sanitizeString),
});

// Search query schema
export const searchSchema = z.object({
  q: z.string()
    .min(2, 'Search query too short')
    .max(200, 'Search query too long')
    .transform(sanitizeString),

  version: z.string().uuid().optional(),

  limit: z.coerce.number()
    .int()
    .min(1)
    .max(100)
    .default(20),

  offset: z.coerce.number()
    .int()
    .min(0)
    .default(0),
});

// Validation middleware
export function validate<T extends z.ZodSchema>(schema: T) {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      const validated = await schema.parseAsync({
        ...req.body,
        ...req.query,
        ...req.params,
      });
      req.validated = validated;
      next();
    } catch (error) {
      if (error instanceof z.ZodError) {
        return res.status(400).json({
          error: 'Validation Error',
          details: error.errors.map(e => ({
            field: e.path.join('.'),
            message: e.message,
          })),
        });
      }
      next(error);
    }
  };
}
```

### 5.3 SQL Injection Prevention

```typescript
// backend/src/database/safe-query.ts
import { Pool, QueryConfig, QueryResult } from 'pg';

// ALWAYS use parameterized queries
export async function safeQuery<T>(
  pool: Pool,
  text: string,
  params: any[] = []
): Promise<QueryResult<T>> {
  // Validate that dynamic parts use parameters
  const paramCount = (text.match(/\$\d+/g) || []).length;
  if (paramCount !== params.length) {
    throw new Error('Parameter count mismatch - possible SQL injection attempt');
  }

  return pool.query<T>({ text, values: params });
}

// Query builder with safe escaping for dynamic table/column names
export function safeIdentifier(identifier: string): string {
  // Only allow alphanumeric and underscore
  if (!/^[a-zA-Z_][a-zA-Z0-9_]*$/.test(identifier)) {
    throw new Error('Invalid identifier');
  }
  return `"${identifier}"`;
}

// Example: Safe dynamic ORDER BY
export function buildOrderClause(
  column: string,
  direction: 'ASC' | 'DESC',
  allowedColumns: string[]
): string {
  if (!allowedColumns.includes(column)) {
    throw new Error('Invalid sort column');
  }
  return `ORDER BY ${safeIdentifier(column)} ${direction}`;
}
```

---

## 6. Frontend Security

### 6.1 XSS Prevention

```typescript
// frontend/src/utils/sanitize.ts
import DOMPurify from 'dompurify';

// Configure DOMPurify
const purifyConfig = {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'br', 'p'],
  ALLOWED_ATTR: [],
  KEEP_CONTENT: true,
};

export function sanitizeHTML(dirty: string): string {
  return DOMPurify.sanitize(dirty, purifyConfig);
}

// For rendering user-generated content
export function SafeContent({ html }: { html: string }) {
  return (
    <div
      dangerouslySetInnerHTML={{ __html: sanitizeHTML(html) }}
    />
  );
}

// Escape for text content (no HTML allowed)
export function escapeText(text: string): string {
  const div = document.createElement('div');
  div.textContent = text;
  return div.innerHTML;
}
```

### 6.2 CSRF Protection

```typescript
// backend/src/middleware/csrf.ts
import csrf from 'csurf';
import cookieParser from 'cookie-parser';

// CSRF token configuration
export const csrfProtection = csrf({
  cookie: {
    key: '_csrf',
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 3600, // 1 hour
  },
});

// Frontend: Include CSRF token in requests
// frontend/src/lib/api.ts
export async function apiRequest(url: string, options: RequestInit = {}) {
  const csrfToken = document.querySelector('meta[name="csrf-token"]')?.getAttribute('content');

  return fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'X-CSRF-Token': csrfToken || '',
    },
    credentials: 'include',
  });
}
```

### 6.3 Secure Storage

```typescript
// frontend/src/utils/storage.ts

// NEVER store sensitive data in localStorage
// Use httpOnly cookies for tokens

// For non-sensitive preferences only
export const secureStorage = {
  set(key: string, value: any): void {
    try {
      // Prefix with app name to avoid conflicts
      const prefixedKey = `bible_reader_${key}`;
      sessionStorage.setItem(prefixedKey, JSON.stringify(value));
    } catch (e) {
      console.error('Storage error:', e);
    }
  },

  get<T>(key: string): T | null {
    try {
      const prefixedKey = `bible_reader_${key}`;
      const item = sessionStorage.getItem(prefixedKey);
      return item ? JSON.parse(item) : null;
    } catch (e) {
      return null;
    }
  },

  remove(key: string): void {
    const prefixedKey = `bible_reader_${key}`;
    sessionStorage.removeItem(prefixedKey);
  },

  clear(): void {
    // Only clear our app's data
    Object.keys(sessionStorage)
      .filter(key => key.startsWith('bible_reader_'))
      .forEach(key => sessionStorage.removeItem(key));
  },
};

// Sensitive data handling (tokens should be in httpOnly cookies)
export function clearSensitiveData(): void {
  secureStorage.clear();
  // Call logout API to clear server-side session
}
```

---

## 7. Infrastructure Security

### 7.1 Network Security

```hcl
# terraform/security-groups.tf

# Application Load Balancer Security Group
resource "aws_security_group" "alb" {
  name        = "bible-alb-sg"
  description = "Security group for ALB"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "HTTPS from anywhere"
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "HTTP for redirect to HTTPS"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "bible-alb-sg"
  }
}

# ECS Tasks Security Group
resource "aws_security_group" "ecs_tasks" {
  name        = "bible-ecs-sg"
  description = "Security group for ECS tasks"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 4000
    to_port         = 4000
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
    description     = "API port from ALB only"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "bible-ecs-sg"
  }
}

# Database Security Group
resource "aws_security_group" "rds" {
  name        = "bible-rds-sg"
  description = "Security group for RDS"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.ecs_tasks.id]
    description     = "PostgreSQL from ECS only"
  }

  # No egress needed for RDS

  tags = {
    Name = "bible-rds-sg"
  }
}
```

### 7.2 WAF Configuration

```hcl
# terraform/waf.tf

resource "aws_wafv2_web_acl" "bible_waf" {
  name        = "bible-waf"
  description = "WAF for Bible Reader application"
  scope       = "REGIONAL"

  default_action {
    allow {}
  }

  # AWS Managed Rules - Common Rule Set
  rule {
    name     = "AWSManagedRulesCommonRuleSet"
    priority = 1

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesCommonRuleSet"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWSManagedRulesCommonRuleSetMetric"
      sampled_requests_enabled   = true
    }
  }

  # AWS Managed Rules - SQL Injection
  rule {
    name     = "AWSManagedRulesSQLiRuleSet"
    priority = 2

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesSQLiRuleSet"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWSManagedRulesSQLiRuleSetMetric"
      sampled_requests_enabled   = true
    }
  }

  # Rate limiting rule
  rule {
    name     = "RateLimitRule"
    priority = 3

    action {
      block {}
    }

    statement {
      rate_based_statement {
        limit              = 2000
        aggregate_key_type = "IP"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "RateLimitRuleMetric"
      sampled_requests_enabled   = true
    }
  }

  # Block bad bots
  rule {
    name     = "AWSManagedRulesBotControlRuleSet"
    priority = 4

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesBotControlRuleSet"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWSManagedRulesBotControlRuleSetMetric"
      sampled_requests_enabled   = true
    }
  }

  visibility_config {
    cloudwatch_metrics_enabled = true
    metric_name                = "bibleWafMetric"
    sampled_requests_enabled   = true
  }
}
```

---

## 8. OWASP Top 10 Mitigations

### 8.1 Summary Table

| # | Vulnerability | Mitigation |
|---|--------------|------------|
| A01 | Broken Access Control | RBAC, resource ownership checks, JWT validation |
| A02 | Cryptographic Failures | TLS 1.3, AES-256-GCM encryption, Argon2id hashing |
| A03 | Injection | Parameterized queries, input validation, Zod schemas |
| A04 | Insecure Design | Security requirements, threat modeling, secure defaults |
| A05 | Security Misconfiguration | Security headers, least privilege, hardened containers |
| A06 | Vulnerable Components | npm audit, Dependabot, regular updates |
| A07 | Auth Failures | MFA, account lockout, secure password policy |
| A08 | Data Integrity Failures | Input validation, signed JWTs, integrity checks |
| A09 | Security Logging | Structured logging, audit trails, monitoring |
| A10 | SSRF | URL validation, allowlist, network segmentation |

### 8.2 Dependency Security

```yaml
# .github/workflows/security.yml
name: Security Scan

on:
  push:
    branches: [main, develop]
  schedule:
    - cron: '0 0 * * *' # Daily

jobs:
  npm-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm audit --audit-level=high

  snyk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  trivy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          severity: 'CRITICAL,HIGH'
```

---

## 9. Security Headers

### 9.1 Complete Headers Configuration

```typescript
// backend/src/middleware/security-headers.ts
import helmet from 'helmet';

export const securityHeaders = helmet({
  // Content Security Policy
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: [
        "'self'",
        "'unsafe-inline'", // Required for Next.js
        "https://www.googletagmanager.com",
        "https://www.google-analytics.com",
      ],
      styleSrc: [
        "'self'",
        "'unsafe-inline'",
        "https://fonts.googleapis.com",
      ],
      imgSrc: [
        "'self'",
        "data:",
        "https:",
        "blob:",
      ],
      fontSrc: [
        "'self'",
        "https://fonts.gstatic.com",
      ],
      connectSrc: [
        "'self'",
        "https://api.biblereader.app",
        "https://www.google-analytics.com",
        "wss://api.biblereader.app", // WebSocket
      ],
      mediaSrc: ["'self'"],
      objectSrc: ["'none'"],
      frameSrc: ["'none'"],
      frameAncestors: ["'none'"],
      formAction: ["'self'"],
      baseUri: ["'self'"],
      upgradeInsecureRequests: [],
    },
  },

  // Cross-Origin policies
  crossOriginEmbedderPolicy: false, // Disable for external resources
  crossOriginOpenerPolicy: { policy: 'same-origin-allow-popups' },
  crossOriginResourcePolicy: { policy: 'cross-origin' },

  // DNS Prefetch Control
  dnsPrefetchControl: { allow: false },

  // Expect-CT (deprecated but still useful)
  expectCt: {
    maxAge: 86400,
    enforce: true,
  },

  // Frameguard (X-Frame-Options)
  frameguard: { action: 'deny' },

  // HSTS
  hsts: {
    maxAge: 31536000, // 1 year
    includeSubDomains: true,
    preload: true,
  },

  // IE No Open
  ieNoOpen: true,

  // No Sniff (X-Content-Type-Options)
  noSniff: true,

  // Origin-Agent-Cluster
  originAgentCluster: true,

  // Permitted Cross-Domain Policies
  permittedCrossDomainPolicies: { permittedPolicies: 'none' },

  // Referrer Policy
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },

  // XSS Filter (X-XSS-Protection)
  xssFilter: true,
});

// Additional custom headers
export function customSecurityHeaders(req: Request, res: Response, next: NextFunction) {
  // Permissions Policy (formerly Feature-Policy)
  res.setHeader('Permissions-Policy',
    'accelerometer=(), camera=(), geolocation=(), gyroscope=(), magnetometer=(), microphone=(), payment=(), usb=()'
  );

  // Cache-Control for sensitive pages
  if (req.path.startsWith('/api/user') || req.path.startsWith('/api/auth')) {
    res.setHeader('Cache-Control', 'no-store, no-cache, must-revalidate, proxy-revalidate');
    res.setHeader('Pragma', 'no-cache');
    res.setHeader('Expires', '0');
  }

  next();
}
```

---

## 10. Logging & Monitoring

### 10.1 Security Event Logging

```typescript
// backend/src/security/audit-log.ts
import { logger } from '../utils/logger';

export enum SecurityEventType {
  LOGIN_SUCCESS = 'auth.login.success',
  LOGIN_FAILURE = 'auth.login.failure',
  LOGOUT = 'auth.logout',
  PASSWORD_CHANGE = 'auth.password.change',
  PASSWORD_RESET_REQUEST = 'auth.password.reset.request',
  MFA_ENABLED = 'auth.mfa.enabled',
  MFA_DISABLED = 'auth.mfa.disabled',
  ACCOUNT_LOCKED = 'auth.account.locked',
  ACCOUNT_UNLOCKED = 'auth.account.unlocked',
  TOKEN_REFRESH = 'auth.token.refresh',
  PERMISSION_DENIED = 'authz.permission.denied',
  RATE_LIMIT_EXCEEDED = 'security.rate_limit.exceeded',
  SUSPICIOUS_ACTIVITY = 'security.suspicious_activity',
  DATA_EXPORT = 'data.export',
  DATA_DELETE = 'data.delete',
}

interface SecurityEvent {
  type: SecurityEventType;
  userId?: string;
  ip: string;
  userAgent: string;
  metadata?: Record<string, any>;
}

export function logSecurityEvent(event: SecurityEvent): void {
  logger.info({
    category: 'security',
    event: event.type,
    userId: event.userId,
    ip: event.ip,
    userAgent: event.userAgent,
    timestamp: new Date().toISOString(),
    ...event.metadata,
  });

  // Critical events trigger alerts
  const criticalEvents = [
    SecurityEventType.ACCOUNT_LOCKED,
    SecurityEventType.SUSPICIOUS_ACTIVITY,
    SecurityEventType.RATE_LIMIT_EXCEEDED,
  ];

  if (criticalEvents.includes(event.type)) {
    sendSecurityAlert(event);
  }
}

// Middleware to log security events
export function securityLogger(req: Request, res: Response, next: NextFunction) {
  res.on('finish', () => {
    // Log authentication failures
    if (req.path.startsWith('/api/auth') && res.statusCode === 401) {
      logSecurityEvent({
        type: SecurityEventType.LOGIN_FAILURE,
        ip: req.ip,
        userAgent: req.headers['user-agent'] || 'unknown',
        metadata: { path: req.path },
      });
    }

    // Log permission denied
    if (res.statusCode === 403) {
      logSecurityEvent({
        type: SecurityEventType.PERMISSION_DENIED,
        userId: req.user?.id,
        ip: req.ip,
        userAgent: req.headers['user-agent'] || 'unknown',
        metadata: { path: req.path, method: req.method },
      });
    }
  });

  next();
}
```

### 10.2 Intrusion Detection

```typescript
// backend/src/security/intrusion-detection.ts
export class IntrusionDetection {
  private redis: RedisClient;

  // Detect brute force attacks
  async detectBruteForce(ip: string): Promise<boolean> {
    const key = `ids:bruteforce:${ip}`;
    const attempts = await this.redis.incr(key);

    if (attempts === 1) {
      await this.redis.expire(key, 300); // 5 minute window
    }

    return attempts > 20; // More than 20 failed attempts in 5 minutes
  }

  // Detect credential stuffing
  async detectCredentialStuffing(ip: string): Promise<boolean> {
    const key = `ids:credstuff:${ip}`;
    const uniqueEmails = await this.redis.scard(key);

    return uniqueEmails > 10; // More than 10 different emails from same IP
  }

  // Record failed login attempt
  async recordFailedLogin(ip: string, email: string): Promise<void> {
    const key = `ids:credstuff:${ip}`;
    await this.redis.sadd(key, email);
    await this.redis.expire(key, 3600); // 1 hour window
  }

  // Detect anomalous behavior
  async detectAnomaly(userId: string, action: string): Promise<boolean> {
    // Check for unusual patterns
    const recentActions = await this.getRecentActions(userId);

    // Example: Too many exports in short time
    if (action === 'data_export') {
      const exportCount = recentActions.filter(a => a === 'data_export').length;
      return exportCount > 3;
    }

    return false;
  }
}
```

---

## 11. Incident Response

### 11.1 Response Plan

```markdown
## Security Incident Response Plan

### Phase 1: Detection & Analysis (0-15 minutes)
1. Identify the incident type and severity
2. Gather initial information (logs, alerts, affected systems)
3. Notify incident response team lead

### Phase 2: Containment (15-60 minutes)
1. Isolate affected systems if necessary
2. Block malicious IPs/users
3. Preserve evidence (logs, memory dumps)
4. Implement temporary mitigations

### Phase 3: Eradication (1-4 hours)
1. Identify root cause
2. Remove malicious content/access
3. Patch vulnerabilities
4. Reset compromised credentials

### Phase 4: Recovery (4-24 hours)
1. Restore systems from clean backups
2. Verify system integrity
3. Monitor for recurrence
4. Gradually restore services

### Phase 5: Post-Incident (24-72 hours)
1. Document the incident
2. Conduct post-mortem
3. Update security controls
4. Communicate with stakeholders (if required)

### Severity Levels

| Level | Description | Response Time | Notification |
|-------|-------------|---------------|--------------|
| Critical | Data breach, full compromise | Immediate | Executive team, legal |
| High | Active attack, service down | 15 minutes | Engineering lead |
| Medium | Suspicious activity, vulnerability found | 1 hour | Security team |
| Low | Policy violation, minor issue | 24 hours | Security team |
```

### 11.2 Emergency Procedures

```typescript
// backend/src/security/emergency.ts
export class EmergencyProcedures {
  // Emergency: Revoke all tokens for a user
  async revokeAllUserSessions(userId: string): Promise<void> {
    // Increment token version to invalidate all existing tokens
    await db.query(
      'UPDATE users SET token_version = token_version + 1 WHERE id = $1',
      [userId]
    );

    // Clear all refresh tokens
    await db.query('DELETE FROM refresh_tokens WHERE user_id = $1', [userId]);

    // Clear cached sessions
    await redis.del(`sessions:${userId}:*`);

    logSecurityEvent({
      type: SecurityEventType.SUSPICIOUS_ACTIVITY,
      userId,
      ip: 'system',
      userAgent: 'system',
      metadata: { action: 'revoke_all_sessions' },
    });
  }

  // Emergency: Lock user account
  async lockAccount(userId: string, reason: string): Promise<void> {
    await db.query(
      'UPDATE users SET is_locked = true, locked_at = NOW(), lock_reason = $2 WHERE id = $1',
      [userId, reason]
    );

    await this.revokeAllUserSessions(userId);

    logSecurityEvent({
      type: SecurityEventType.ACCOUNT_LOCKED,
      userId,
      ip: 'system',
      userAgent: 'system',
      metadata: { reason },
    });
  }

  // Emergency: Block IP address
  async blockIP(ip: string, duration: number, reason: string): Promise<void> {
    await redis.setex(`blocked:ip:${ip}`, duration, reason);

    logSecurityEvent({
      type: SecurityEventType.SUSPICIOUS_ACTIVITY,
      ip,
      userAgent: 'system',
      metadata: { action: 'block_ip', duration, reason },
    });
  }

  // Emergency: Enable maintenance mode
  async enableMaintenanceMode(): Promise<void> {
    await redis.set('maintenance_mode', 'true');
    // This will be checked by middleware to block all requests
  }
}
```

---

## 12. Compliance & Privacy

### 12.1 GDPR Compliance

```typescript
// backend/src/compliance/gdpr.ts
export class GDPRCompliance {
  // Right to access
  async exportUserData(userId: string): Promise<UserDataExport> {
    const user = await this.getUser(userId);
    const data = {
      personalInfo: {
        email: user.email,
        displayName: user.displayName,
        createdAt: user.createdAt,
      },
      preferences: await this.getUserPreferences(userId),
      highlights: await this.getUserHighlights(userId),
      bookmarks: await this.getUserBookmarks(userId),
      notes: await this.getUserNotes(userId),
      readingProgress: await this.getUserReadingProgress(userId),
    };

    logSecurityEvent({
      type: SecurityEventType.DATA_EXPORT,
      userId,
      ip: 'api',
      userAgent: 'api',
    });

    return data;
  }

  // Right to erasure
  async deleteUserData(userId: string): Promise<void> {
    // Verify user identity before deletion
    const deletion = new SecureDataDeletion();
    await deletion.deleteUserData(userId);

    logSecurityEvent({
      type: SecurityEventType.DATA_DELETE,
      userId,
      ip: 'api',
      userAgent: 'api',
    });
  }

  // Right to rectification
  async updateUserData(userId: string, updates: Partial<UserData>): Promise<void> {
    // Validate and sanitize updates
    const sanitized = sanitizeUserUpdates(updates);
    await this.applyUpdates(userId, sanitized);
  }

  // Consent management
  async recordConsent(userId: string, consentType: string, granted: boolean): Promise<void> {
    await db.query(`
      INSERT INTO user_consents (user_id, consent_type, granted, recorded_at)
      VALUES ($1, $2, $3, NOW())
      ON CONFLICT (user_id, consent_type)
      DO UPDATE SET granted = $3, recorded_at = NOW()
    `, [userId, consentType, granted]);
  }
}
```

### 12.2 Privacy Policy Integration

```typescript
// backend/src/compliance/privacy.ts
export const dataRetentionPolicies = {
  // User data retention
  userAccount: {
    activeUsers: 'indefinite',
    inactiveUsers: '2 years after last activity',
    deletedUsers: 'immediate deletion',
  },

  // Logs retention
  logs: {
    accessLogs: '90 days',
    securityLogs: '1 year',
    errorLogs: '30 days',
  },

  // Analytics retention
  analytics: {
    aggregatedData: 'indefinite',
    individualData: '90 days',
  },
};

// Automated data cleanup
export async function runDataRetentionCleanup(): Promise<void> {
  // Delete inactive users (no login in 2 years)
  await db.query(`
    DELETE FROM users
    WHERE last_login_at < NOW() - INTERVAL '2 years'
    AND is_active = false
  `);

  // Delete old logs
  await db.query(`
    DELETE FROM access_logs
    WHERE created_at < NOW() - INTERVAL '90 days'
  `);

  // Delete old security logs
  await db.query(`
    DELETE FROM security_logs
    WHERE created_at < NOW() - INTERVAL '1 year'
  `);
}
```

---

## Summary

This security guidelines document covers comprehensive security measures for the Online Bible Reader application:

| Area | Key Measures |
|------|-------------|
| Authentication | Argon2id hashing, JWT with RS256, OAuth 2.0, MFA |
| Authorization | RBAC, resource ownership, least privilege |
| Data Protection | AES-256-GCM encryption, data masking, secure deletion |
| API Security | Rate limiting, input validation, parameterized queries |
| Frontend | XSS prevention, CSRF protection, secure storage |
| Infrastructure | WAF, TLS 1.3, security groups, network segmentation |
| Monitoring | Security event logging, intrusion detection, alerting |
| Compliance | GDPR data export/deletion, consent management |

---

**Document Version**: 1.0.0
**Last Updated**: 2025-01-14
**Author**: Security Team
