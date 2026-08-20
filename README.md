# Telegram Ecard & Wallet Management Bot

بک‌اند واقعی و production-ready برای مدیریت درخواست‌های ایکارت، کیف پول USDT، واریزی‌ها، دفترکل مالی و پشتیبانی در Telegram. پروژه برای GitHub و Railway طراحی شده و از Node.js، TypeScript، grammY، PostgreSQL، Prisma، Fastify، Zod و QR Code استفاده می‌کند.

## معماری

- Node.js 22 + TypeScript
- grammY برای Telegram Bot API
- PostgreSQL + Prisma ORM
- Fastify برای `/health`
- Zod برای validation
- AES-256-GCM برای رمزنگاری credentialهای حساس در حالت ذخیره‌شده
- Pino برای structured logging
- QRCode برای تولید QR آدرس دریافت
- Dockerfile مناسب Railway
- Migration اولیه Prisma
- تست‌های unit و اسکلت integration برای PostgreSQL

## قابلیت‌های اصلی

### منوی کاربر

- 🔗 اتصال اتاق کار
- 💳 چنچ ایکارت
- 🪪 تهیه ایکارت
- 💰 واریز به کیف پول
- 👛 موجودی کیف پول
- 📋 پیگیری درخواست‌ها
- 💬 پشتیبانی

### امنیت ادمین

Telegram numeric ID ادمین‌ها فقط از `ADMIN_TELEGRAM_IDS` خوانده می‌شود. هیچ ID ادمینی در source code hard-code نشده است. تمام callbackهای مدیریتی با `isAdmin()` محافظت می‌شوند.

### credentialهای حساس

رمز اتاق کار، رمز Q Account، رمز ایمیل و رمز ایکارت در PostgreSQL به صورت AES-256-GCM ذخیره می‌شوند. رمزها در log یا پیام عادی ادمین نمایش داده نمی‌شوند. کلید رمزنگاری فقط از `ENCRYPTION_KEY` خوانده می‌شود.

### کیف پول

برای هر کاربر و هر شبکه فقط یک assignment ممکن است. محدودیت در سطح دیتابیس با:

```text
UNIQUE(userId, networkId)
UNIQUE(walletId)
```

و همزمان با transaction با isolation سطح `SERIALIZABLE` اجرا می‌شود. اگر کاربر قبلاً برای شبکه‌ای آدرس داشته باشد همان آدرس برگردانده می‌شود.

### دفترکل

موجودی از روی `LedgerTransaction` محاسبه می‌شود و balance در یک فیلد قابل ویرایش ذخیره نمی‌شود. تأیید واریزی در transaction انجام شده و یک ledger entry غیرقابل‌overwrite برای آن ایجاد می‌کند.

## متغیرهای محیطی

فایل `.env.example` را کپی کنید و مقدارهای واقعی را فقط در محیط محلی یا Railway وارد کنید:

```env
NODE_ENV=production
BOT_TOKEN=
ADMIN_TELEGRAM_IDS=
DATABASE_URL=
ENCRYPTION_KEY=
LOG_LEVEL=info
APP_URL=
PORT=3000
```

### BOT_TOKEN را از کجا بگیریم؟

1. در Telegram، `@BotFather` را باز کنید.
2. `/newbot` را اجرا کنید.
3. نام و username ربات را انتخاب کنید.
4. BotFather یک token می‌دهد.
5. token را فقط در Railway Variables یا `.env` محلی قرار دهید.
6. token را در GitHub commit نکنید.

### Telegram numeric ID ادمین را چطور پیدا کنیم؟

می‌توانید از یک bot معتبر نمایش‌دهنده Telegram ID استفاده کنید یا از ابزار/ربات داخلی مورد اعتماد خودتان استفاده کنید. مقدار باید numeric Telegram user ID باشد، نه username.

مثال چند ادمین:

```env
ADMIN_TELEGRAM_IDS=123456789,987654321
```

در کد هیچ ID واقعی قرار نمی‌گیرد.

### ENCRYPTION_KEY

یک secret تصادفی طولانی بسازید. حداقل 32 کاراکتر لازم است. برای محیط production بهتر است مقدار تصادفی و غیرقابل‌حدس استفاده شود.

مثلاً با Node.js:

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('base64url'))"
```

کلید را در GitHub commit نکنید و آن را عوض نکنید مگر اینکه برنامه‌ای برای re-encrypt کردن credentialهای قبلی داشته باشید.

## اجرای محلی

پیش‌نیازها:

- Node.js 20 یا بالاتر
- PostgreSQL 14 یا بالاتر

نصب:

```bash
npm ci
npx prisma generate
npx prisma migrate deploy
npm run build
npm start
```

در development:

```bash
npm run dev
```

Health check:

```text
GET /health
```

پاسخ موفق:

```json
{"status":"ok"}
```

## Railway + PostgreSQL

1. یک repository در GitHub بسازید.
2. تمام فایل‌های این پروژه را در repository قرار دهید.
3. در Railway یک Project بسازید.
4. PostgreSQL service اضافه کنید.
5. یک service از GitHub repository ایجاد کنید.
6. در Variables ربات، مقدار `BOT_TOKEN` را قرار دهید.
7. `ADMIN_TELEGRAM_IDS` را وارد کنید.
8. `DATABASE_URL` را با connection string PostgreSQL تنظیم کنید. اگر Railway آن را به صورت reference variable ارائه می‌کند، همان reference را استفاده کنید.
9. `ENCRYPTION_KEY`، `LOG_LEVEL` و در صورت نیاز `APP_URL` را تنظیم کنید.
10. `PORT` را معمولاً خالی بگذارید تا Railway مقدار آن را inject کند؛ برنامه از `process.env.PORT` استفاده می‌کند.
11. Deploy را اجرا کنید.
12. Container ابتدا `prisma migrate deploy` را اجرا و سپس bot را start می‌کند.

Docker command نهایی پروژه:

```text
npx prisma migrate deploy && node dist/src/index.js
```

## GitHub

فایل‌های زیر باید در repository باشند:

```text
Dockerfile
README.md
.env.example
.gitignore
package.json
tsconfig.json
src/
prisma/
tests/
.github/workflows/ci.yml
```

این موارد نباید commit شوند:

```text
.env
BOT_TOKEN واقعی
ADMIN_TELEGRAM_IDS واقعی در فایل source
DATABASE_URL واقعی
DATABASE password
ENCRYPTION_KEY واقعی
private key
```

## Migration

Migration اولیه در:

```text
prisma/migrations/20260820165900_init/migration.sql
```

در Railway:

```bash
npx prisma migrate deploy
```

در حالت local بعد از تغییر schema، migration جدید بسازید:

```bash
npx prisma migrate dev --name your_change
```

سپس migration را commit کنید.

## تنظیم شبکه‌ها

در اولین startup، اگر هیچ شبکه‌ای وجود نداشته باشد، این شبکه‌های نمونه ساخته می‌شوند:

- USDT / TRC20
- USDT / ERC20
- USDT / BEP20
- USDT / POLYGON
- USDT / TON

ادمین از پنل Telegram می‌تواند شبکه را فعال/غیرفعال یا نام نمایشی آن را تغییر دهد و شبکه جدید اضافه کند.

## افزودن کیف پول

از پنل مدیریت:

```text
⚙️ پنل مدیریت
→ 🏦 مدیریت کیف پول‌ها
→ ➕ افزودن آدرس
```

فقط public receiving address وارد کنید. private key هرگز در این پروژه ذخیره نمی‌شود.

برای هر network می‌توانید چند receiving address وارد کنید. هنگام اولین assignment، یک آدرس آزاد انتخاب می‌شود و سپس assignment در PostgreSQL دائمی می‌ماند.

ادمین می‌تواند آدرس را فعال/غیرفعال کند و در موارد مناسب assignment را لغو کند.

## تیم‌ها

از:

```text
⚙️ پنل مدیریت
→ 👥 تیم‌ها
```

می‌توانید تیم جدید اضافه کنید، فعال/غیرفعال کنید و نام نمایشی تیم را تغییر دهید. فرم‌های کاربر فقط تیم‌های active را نشان می‌دهند.

## درخواست تهیه ایکارت

محدوده مبلغ:

```text
MINIMUM = 200 USDT
MAXIMUM = 10000 USDT
```

بعد از ثبت درخواست، ربات شبکه انتخابی را ثبت می‌کند، آدرس اختصاصی همان کاربر را دریافت می‌کند و آدرس + QR را ارسال می‌کند.

## کدهای پیگیری

- اتصال اتاق کار: `WR-XXXXXX`
- چنچ ایکارت: `EC-XXXXXX`
- تهیه ایکارت: `CARD-XXXXXX`
- واریزی: `DEP-XXXXXX`

همه tracking codeها در جدول مربوطه unique هستند.

## پشتیبانی

کاربر می‌تواند گفت‌وگو را باز کند. پیام‌ها در PostgreSQL ذخیره می‌شوند. ادمین می‌تواند گفت‌وگو را فعال، پیام ارسال یا آن را ببندد. Telegram ID ادمین برای کاربر ارسال نمی‌شود.

## تست

تست unit:

```bash
npm test
```

تست‌های integration PostgreSQL زمانی فعال می‌شوند که `DATABASE_URL` و `TEST_DATABASE_URL` تنظیم شده باشند. برای integration از یک database کاملاً جدا از production استفاده کنید.

## بررسی build و Prisma

```bash
npm run prisma:validate
npx prisma generate
npm run build
npm test
```

برای بررسی migration روی PostgreSQL واقعی:

```bash
npx prisma migrate deploy
```

## Docker

Build:

```bash
docker build -t telegram-ecard-wallet-bot .
```

Run:

```bash
docker run --env-file .env -p 3000:3000 telegram-ecard-wallet-bot
```

## عیب‌یابی

### Bot شروع نمی‌شود

بررسی کنید:

- `BOT_TOKEN` خالی نباشد.
- `DATABASE_URL` معتبر باشد.
- `ENCRYPTION_KEY` حداقل 32 کاراکتر باشد.
- PostgreSQL از Railway قابل دسترس باشد.

### خطای migration

```bash
npx prisma validate
npx prisma migrate deploy
```

اگر migration قبلاً روی database اجرا نشده، لاگ Railway را بررسی کنید. migrationها را در production حذف یا دستی دستکاری نکنید.

### آدرس کیف پول پیدا نمی‌شود

در پنل:

```text
🏦 مدیریت کیف پول‌ها
```

بررسی کنید برای network موردنظر wallet فعال و آزاد وجود داشته باشد.

### کاربر همان آدرس قبلی را نمی‌گیرد

assignment باید در `WalletAssignment` وجود داشته باشد. محدودیت database روی `(userId, networkId)` اجازه ساخت assignment دوم را نمی‌دهد.

### موجودی اشتباه است

موجودی فقط از `LedgerTransaction` محاسبه می‌شود. برای واریزی تأییدشده باید یک ledger entry با type `DEPOSIT` وجود داشته باشد.

## Health check در Railway

مسیر:

```text
/health
```

پاسخ:

```json
{
  "status": "ok"
}
```

## نکات production

- `.env` را commit نکنید.
- secretها را فقط در Railway Variables نگه دارید.
- private key را وارد bot نکنید.
- `ENCRYPTION_KEY` را backup امن کنید.
- database backup را برای PostgreSQL فعال نگه دارید.
- برای integration test از database جدا استفاده کنید.
- لاگ‌های production را برای credential و secret بررسی کنید.
