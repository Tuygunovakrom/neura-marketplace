# Nonyovxona boshqaruv tizimi

Nonyovxonalar uchun boshqaruv tizimi: nonvoy, shofyor va admin rollari bilan.

- **Frontend:** React (Vite), react-router-dom, axios — mobil birinchi (mobile-first)
- **Backend:** Node.js, Express, MongoDB (Mongoose), JWT, bcryptjs

## Rollar

| Rol | Vazifasi |
|---|---|
| **admin** | foydalanuvchilar, do'konlar, kun nazorati, izohlar, audit, tuzatishlar |
| **nonvoy** | kunlik hisobot, shofyorga non berish, zakazlar, rasxod, kunni tugatish |
| **shofyor** | 3 nonvoyxonadan non qabul qilish, do'konlarga tarqatish, rasxod |

Login/parolni **faqat admin** yaratadi (ro'yxatdan o'tish yo'q). Har bir nonvoy bitta nonvoyxonaga biriktiriladi; nonvoyxonalar bir-birining ma'lumotini ko'ra olmaydi (server har so'rovda `req.user.bakery` bo'yicha filtrlaydi).

## Asosiy qoidalar

- Sana Toshkent vaqti (UTC+5) bo'yicha, `YYYY-MM-DD`.
- Nonvoy qoldigi: `yopilgan − sotilgan − yeyilgan − shofyorga berilgan` (manfiy bo'lishi serverda bloklanadi).
- Shofyor qoldigi: `qabul qilingan − tarqatilgan` (har nonvoyxona bo'yicha alohida).
- "Kunni tugatish" bosilgach nonvoy ham, shofyor ham o'sha kunni o'zgartira olmaydi; faqat admin qayta ochadi.
- Nonyov "berdim" va shofyor "oldim" mos kelmasa admin panelida farq **qizil** ko'rsatiladi.
- Nonyvoy/shofyor har bir yozuv yonidan adminga izoh yuboradi; admin uni "Hal qilindi" qiladi.
- Admin kiritgan har bir tuzatish AuditLog'ga yoziladi (kim, qachon, eski → yangi).

## O'rnatish

Talab: Node.js 18+, MongoDB (lokal yoki Atlas).

```bash
# 1. Paketlar
npm run setup          # server va client uchun npm install

# 2. .env fayli
# server/.env.example ni server/.env ga ko'chiring va JWT_SECRET ni ozgartiring
cp server/.env.example server/.env

# 3. Seed (admin + 3 nonvoyxona + namuna nonvoy/shofyor/do'konlar)
npm run seed
```

## Ishga tushirish

```bash
npm run dev            # server (5000) + client (5173) bir vaqtda
```

Yoki alohida: `npm run dev:server` va `npm run dev:client`.
Brauzerda: http://localhost:5173

## Vercelga joylash (deploy)

Loyiha Vercel serverless funksiya uchun tayyor: `vercel.json` API so'rovlarini
`server/api/index.js` ga yo'naltiradi, client `client/dist` dan xizmat qilinadi.
MongoDB lokal ishlamagani uchun **MongoDB Atlas** (bepul) kerak.

**1. MongoDB Atlas:**
- https://cloud.mongodb.com da cluster oching (bepul M0 yetarli).
- Database Access'dan foydalanuvchi yarating (login/parol).
- Network Access'da IP ro'yxatiga `0.0.0.0/0` qo'shing (Vercel IP'lari doimiy emas).
- Clusters → Connect → Drivers → Node.js: ulanish satrini oling, shunga o'xshaydi:
  `mongodb+srv://user:parol@cluster0.xxxxx.mongodb.net/nonvoyxona`

**2. Seed (bir marta, lokal kompyuterda):**
Atlas URI'sini `server/.env` dagi `MONGO_URI` ga yozing va seedni ishga tushiring —
admin va namuna foydalanuvchilar bulutdagi bazaga yoziladi:

```bash
# server/.env
MONGO_URI=mongodb+srv://user:parol@cluster0.xxxxx.mongodb.net/nonvoyxona
JWT_SECRET=istalgan_uzun_maxfiy_kalit
```
```bash
npm run seed
```

**3. Vercel:**
- Loyihani GitHub'ga yuklang (`vercel.json` ildizda qolishi shart).
- https://vercel.com → **Add New → Project** → repository'ni tanlang.
- **Environment Variables** bo'limiga ikkitasini qo'shing:
  - `MONGO_URI` — Atlas ulanish satri
  - `JWT_SECRET` — kuchli maxfiy kalit
- **Deploy** bosing. Tayyor: `https://loyiha-nomingiz.vercel.app`

Shunda sayt va `/api/*` bir domen orqali ishlaydi (CORS sozlamasi kerak emas).
Lokal ishga tushirish (`npm run dev`) avvalgidek o'zgarishsiz ishlaydi.

## Kirish ma'lumotlari (seed)

| Rol | Login | Parol |
|---|---|---|
| Admin | `admin` | `admin123` |
| Nonyov 1/2/3 | `nonvoy1`, `nonvoy2`, `nonvoy3` | `nonvoy123` |
| Shofyor 1/2/3 | `shofyor1`, `shofyor2`, `shofyor3` | `shofyor123` |

## Papka strukturasi

```
server/api/     index.js — Vercel serverless kirish nuqtasi
server/src/
  config/       db.js (ulanish keshlanadi — serverless uchun)
  utils/        date (UTC+5), ApiError, validate, asyncHandler
  middleware/   auth (JWT + rol), error (o'zbekcha xabarlar)
  models/       Bakery, User, Shop, DailyReport, Handoff, Delivery,
                Order, Expense, Comment, AuditLog
  services/     reportService, driverService, adminService
  controllers/  auth, user, bakery, shop, report, handoff, driver,
                order, expense, comment, admin
  routes/       auth, users, bakeries, shops, reports, handoffs,
                driver, deliveries, orders, expenses, comments, admin

client/src/
  api/          http.js (axios), index.js (endpointlar)
  context/      AuthContext
  hooks/        useAuth
  components/   Layout, ProtectedRoute, Modal, CommentButton, DateBar,
                ExpenseCard, AdminEditModal
  pages/        Login, baker/, driver/, admin/
```

## API (qisqacha)

| Metod va manzil | Rol | Tavsif |
|---|---|---|
| `POST /api/auth/login` | hammа | kirish |
| `GET/PUT /api/reports/mine`, `POST /api/reports/mine/close` | nonvoy | hisobot, kunni tugatish |
| `POST /api/handoffs` | nonvoy | shofyorga non berish |
| `PUT /api/handoffs/:id/confirm` | shofyor | "Qabul qildim" |
| `GET /api/driver/day`, `POST /api/driver/day/close` | shofyor | qabul, tarqatish, qoldiqlar; kunni tugatish |
| `POST /api/deliveries` | shofyor | do'konga tarqatish |
| `GET/POST /api/orders`, `PUT /api/orders/:id/done` | nonvoy | zakazlar |
| `GET/POST /api/expenses/mine` | nonvoy, shofyor | rasxod |
| `POST /api/comments`, `PUT /api/comments/:id/resolve` | nonvoy, shofyor / admin | izohlar |
| `GET /api/admin/day?date=` | admin | kun nazorati |
| `PUT /api/admin/{reports,handoffs,deliveries,expenses,orders}/:id` | admin | tuzatish (audit yoziladi) |
| `POST /api/admin/reports/:id/reopen` | admin | nonvoy kunini qayta ochish |
| `POST /api/admin/driverdays/:id/reopen` | admin | shofyor kunini qayta ochish |
| `GET /api/admin/audit` | admin | audit tarixi |
| `GET/POST/PUT /api/users` | admin | foydalanuvchilar |
| `GET/POST/PUT /api/shops` | hammа / admin | do'konlar va non narxi |
