### MVP Build Prompt — Freetown Connect

#### Project Title
Freetown Connect — Web-based service marketplace for Freetown, Sierra Leone.

#### Objective
Build a production-quality MVP that connects Service Seekers with Freelancers/Service Providers. Focus on trust, responsiveness, and core marketplace flows: discovery, booking, messaging, completion, and review. Include an Admin dashboard for moderation.

---

### 1) Branding & UX
- **Brand Name**: Freetown Connect
- **Logo**: Use provided PNG
- **Colors**:
  - Navy Blue `#002D72` (primary)
  - Gold `#F4B400` (accent/CTA)
  - Light Gray `#F5F5F5` (backgrounds)
- **Typography**: Headings: Montserrat; Body: Open Sans (fallback Arial)
- **Design Style**: Clean, minimal, modern, accessible; mobile-first
- **UI Notes**:
  - Consistent spacing, 8px grid
  - Clear CTAs (gold), high-contrast text
  - Empty states and helpful errors
  - Use icons sparingly (services categories)

---

### 2) Roles
- **Service Seeker (SEEKER)**: Finds providers, books, pays externally (MVP), reviews
- **Service Provider (PROVIDER)**: Lists services, accepts/rejects bookings, completes jobs
- **Admin (ADMIN)**: Full moderation over users, services, bookings, reviews

---

### 3) Core MVP Features
- **Auth & Onboarding**
  - Email/password registration & login (JWT-based)
  - Role selection at sign-up: SEEKER or PROVIDER
  - Profile creation: name, phone, location (Freetown default), bio, avatar
- **Service Listings**
  - Create/edit/delete (Provider)
  - Fields: category, title, description, price, location, tags, images
  - Search & filter: q (text), category, price range, minRating, location
  - Pagination & sorting (recent, rating, price)
- **Booking System**
  - Seeker requests booking for a service
  - Provider accepts/rejects
  - Status: Pending → Accepted → Completed (or Canceled)
- **In-App Messaging (Basic)**
  - Conversation auto-created on booking acceptance
  - Real-time updates via Socket.io
- **Ratings & Reviews**
  - Seeker can rate provider after completion (1–5 stars + comment)
  - Provider profile shows average rating & count
- **Admin Panel**
  - Manage users, services, bookings, reviews
  - Suspend users, deactivate listings
  - Basic stats (totals by entity + daily counts)

---

### 4) Tech Stack
- **Frontend**: React (Next.js 14 App Router recommended), TailwindCSS, TypeScript
- **Backend**: Node.js 20 + Express, TypeScript, Socket.io
- **Database**: MongoDB Atlas + Mongoose
- **Auth**: JWT (access + refresh), bcrypt for password hashing
- **File Storage**: Cloudinary (images)
- **Hosting**: Frontend on Vercel, Backend on Render/Railway, DB on MongoDB Atlas
- **Validation**: Zod or Joi; sanitize inputs
- **Utilities**: Winston/Pino logging, Helmet, CORS, Rate limiting

---

### 5) Folder Structure
```
/client
  /app (if Next.js App Router)
  /components
  /hooks
  /lib
  /styles
  /public
  package.json
  next.config.js

/server
  /src
    /config
    /controllers
    /middlewares
    /models
    /routes
    /services
    /sockets
    /utils
    server.ts
  package.json

/.env.example
/README.md
```

---

### 6) Environment (.env.example)
```env
# Server
PORT=4000
NODE_ENV=development
MONGODB_URI=mongodb+srv://<user>:<pass>@cluster/dbname
JWT_ACCESS_SECRET=replace_with_secure_random
JWT_REFRESH_SECRET=replace_with_secure_random
JWT_ACCESS_EXPIRES=15m
JWT_REFRESH_EXPIRES=7d
CORS_ORIGIN=https://your-frontend-domain.com

# Cloudinary
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret

# Frontend
NEXT_PUBLIC_API_BASE_URL=https://your-backend-domain.com
```

---

### 7) Data Models (Mongoose)
- **User**
```ts
{
  _id: ObjectId,
  role: 'SEEKER' | 'PROVIDER' | 'ADMIN',
  email: string (unique, lowercase),
  passwordHash: string,
  name: string,
  phone?: string,
  location?: string, // default Freetown
  bio?: string,
  avatarUrl?: string,
  skills?: string[], // providers only
  averageRating: number (default 0),
  reviewCount: number (default 0),
  isSuspended: boolean (default false),
  createdAt: Date,
  updatedAt: Date
}
```
- **Service**
```ts
{
  _id: ObjectId,
  providerId: ObjectId (ref User),
  title: string,
  description: string,
  category: 'Plumbing' | 'Electrical' | 'Graphic Design' | 'Tutoring' | 'Cleaning' | 'Delivery' | 'Construction' | 'Other',
  price: number, // per job or per hour (MVP: simple numeric)
  location: string, // Freetown default
  tags: string[],
  images: string[],
  isActive: boolean (default true),
  ratingAverage: number (default 0), // denormalized for quick filter
  createdAt: Date,
  updatedAt: Date
}
```
- **Booking**
```ts
{
  _id: ObjectId,
  serviceId: ObjectId (ref Service),
  seekerId: ObjectId (ref User),
  providerId: ObjectId (ref User), // denormalized from service
  status: 'Pending' | 'Accepted' | 'Completed' | 'Canceled',
  scheduledAt?: Date,
  notes?: string,
  conversationId?: ObjectId (ref Conversation),
  createdAt: Date,
  updatedAt: Date
}
```
- **Conversation**
```ts
{
  _id: ObjectId,
  bookingId: ObjectId (ref Booking),
  participants: ObjectId[], // [seekerId, providerId]
  lastMessageAt?: Date,
  createdAt: Date,
  updatedAt: Date
}
```
- **Message**
```ts
{
  _id: ObjectId,
  conversationId: ObjectId (ref Conversation),
  senderId: ObjectId (ref User),
  recipientId: ObjectId (ref User),
  body: string,
  readAt?: Date,
  createdAt: Date
}
```
- **Review**
```ts
{
  _id: ObjectId,
  bookingId: ObjectId (ref Booking),
  seekerId: ObjectId (ref User),
  providerId: ObjectId (ref User),
  rating: number (1-5),
  comment?: string,
  createdAt: Date
}
```
- **AdminAuditLog** (optional MVP)
```ts
{
  _id: ObjectId,
  adminId: ObjectId (ref User),
  action: string,
  entityType: 'USER' | 'SERVICE' | 'BOOKING' | 'REVIEW',
  entityId: ObjectId,
  metadata?: object,
  createdAt: Date
}
```

---

### 8) RBAC & Access Control
- **SEEKER**: can search services, create bookings, message post-acceptance, leave reviews post-completion
- **PROVIDER**: can create/manage own services, accept/reject bookings, message post-acceptance
- **ADMIN**: full access
- **Middleware**:
  - `requireAuth` (validate JWT, load user)
  - `requireRole(...roles)`
  - `requireOwnership` for service and booking modifications
- **Rate Limits**: auth routes stricter; messaging moderate; search moderate

---

### 9) API Contracts (Express)
Base URL: `/api`

Auth
- `POST /auth/register`
  - Body: `{ email, password, role, name }`
  - Returns: `{ user, accessToken, refreshToken }`
- `POST /auth/login`
  - Body: `{ email, password }`
  - Returns: `{ user, accessToken, refreshToken }`
- `POST /auth/refresh`
  - Body: `{ refreshToken }` → `{ accessToken }`
- `POST /auth/logout` → 204

Users
- `GET /users/me` (auth)
- `PATCH /users/me` (auth) → update profile fields
- `GET /providers/:id` → public provider profile + rating summary

Services
- `POST /services` (auth: PROVIDER)
  - Body: `{ title, description, category, price, location, tags, images }`
- `GET /services` (public)
  - Query: `q, category, minPrice, maxPrice, minRating, location, sort, page, limit`
- `GET /services/:id` (public)
- `PATCH /services/:id` (auth: PROVIDER owner or ADMIN)
- `DELETE /services/:id` (auth: PROVIDER owner or ADMIN)

Bookings
- `POST /bookings` (auth: SEEKER)
  - Body: `{ serviceId, scheduledAt?, notes? }`
- `GET /bookings/mine` (auth) → bookings for current user (as seeker or provider)
- `GET /bookings/:id` (auth: participant or ADMIN)
- `PATCH /bookings/:id` (auth: PROVIDER owner or ADMIN)
  - Body: `{ status }` (Accepted|Completed|Canceled)

Messaging
- `GET /conversations/:bookingId` (auth: participant)
- `GET /messages/:conversationId` (auth: participant) with pagination
- `POST /messages` (auth: participant)
  - Body: `{ conversationId, recipientId, body }`
  - Socket event: `message:new`

Reviews
- `POST /reviews` (auth: SEEKER)
  - Body: `{ bookingId, providerId, rating, comment? }`
- `GET /reviews/:providerId` (public)

Admin
- `GET /admin/dashboard` (auth: ADMIN) → counts/stats
- `PATCH /admin/users/:id/suspend` (auth: ADMIN) → `{ isSuspended: true }`
- `PATCH /admin/services/:id/deactivate` (auth: ADMIN)

Common Responses
- Use RFC7807-like errors: `{ error: { code, message, details? } }`
- Paginated lists: `{ data: T[], page, limit, total }`

---

### 10) Validation & Security
- Strong password rules; bcrypt cost 10-12
- Validate all inputs (Zod/Joi), sanitize text
- CORS allow-list frontend origin
- Helmet for security headers
- Rate limit auth and messaging
- Never return `passwordHash`; filter sensitive fields
- On review create, verify booking is Completed and seeker is the creator

---

### 11) Search & Filter Behavior
- `q` searches service `title`, `description`, and `tags`
- `category` matches exact value from enum
- `minPrice|maxPrice` numeric filters
- `minRating` uses `ratingAverage`
- `location` string contains (MVP); future: geo queries
- Default sort: `createdAt` desc; optional by `price` or `ratingAverage`

---

### 12) Real-time Messaging (Socket.io)
- Namespace: `/chat`
- Auth via access token on connection
- Join room: `conversation:<conversationId>`
- Events:
  - Client → Server: `message:send` `{ conversationId, body }`
  - Server → Clients: `message:new` `{ message }`
  - Typing indicators optional

---

### 13) Frontend Pages (Next.js suggested)
- Public
  - `/` Landing + search
  - `/services` Search results with filters
  - `/services/[id]` Service detail + provider info
  - `/auth/login`, `/auth/register`
- Authenticated
  - `/dashboard` (role-aware)
    - SEEKER: my bookings, messages
    - PROVIDER: my services, incoming bookings, messages
  - `/services/new` (PROVIDER)
  - `/services/[id]/edit` (PROVIDER owner)
  - `/bookings/[id]` Booking detail + chat
- Admin
  - `/admin` Overview
  - `/admin/users`, `/admin/services`, `/admin/bookings`, `/admin/reviews`

---

### 14) Acceptance Criteria (MVP)
- Seeker: sign up → search → view provider → request booking → provider accepts → chat → mark completed → leave review
- Provider: sign up → create service → receive booking → accept → complete
- Admin: login → view entities → suspend user → deactivate service
- Data persists across sessions; JWT refresh works; basic rate limiting in place

---

### 15) Non-goals (MVP)
- Payments and escrow
- Advanced geo-fencing and maps
- Complex dispute resolution workflows
- Multi-language support

---

### 16) Deployment & DevOps
- **Frontend** (Vercel)
  - Set `NEXT_PUBLIC_API_BASE_URL`
  - Protect env vars; preview deployments
- **Backend** (Render/Railway)
  - Health check endpoint `/api/health`
  - Configure env vars; autosleep disabled if needed
  - Enable logs and alerts
- **Database** (MongoDB Atlas)
  - Network access rules; user with least privilege
  - Create collections indexes (see below)
- **Cloudinary**
  - Upload presets; restrict by unsigned/unsigned as needed
- **Indexes**
  - Services: `{ category: 1, price: 1, ratingAverage: -1, createdAt: -1 }`
  - Text index on `title, description, tags`: `{ title: 'text', description: 'text', tags: 'text' }`
  - Messages: `{ conversationId: 1, createdAt: -1 }`

---

### 17) Seed & Admin
- Create a seed script to insert:
  - 1 admin user (email & temp password), 5 providers (with services), 10 seekers
- On startup, ensure at least one ADMIN exists or log warning
- Store admin creds securely; rotate after deployment

---

### 18) Error Codes (Examples)
- `AUTH_INVALID_CREDENTIALS`
- `AUTH_FORBIDDEN`
- `ENTITY_NOT_FOUND`
- `VALIDATION_ERROR`
- `BOOKING_INVALID_STATUS_TRANSITION`
- `REVIEW_NOT_ALLOWED`

---

### 19) Analytics & Monitoring (Nice-to-have)
- Capture page views, service views, search filters used
- Track funnel: search → service detail → booking request → acceptance → completion
- Error logging with Sentry (frontend + backend)

---

### 20) Performance & Accessibility Targets
- LCP < 2.5s on 3G fast for main pages
- Lighthouse Accessibility ≥ 90
- Image optimization via Next/Image or Cloudinary transforms

---

### 21) Timeline (Suggested)
- Week 1: Auth, profiles, services CRUD, search
- Week 2: Bookings workflow + RBAC + messaging
- Week 3: Reviews + Admin + polishing + deployment
- Week 4: UX refinements, seed data, analytics, test with pilot group

---

### 22) Test Plan (High-level)
- Unit: utils, validators, controllers
- Integration: auth flow, service search, booking transitions, review gating
- E2E: critical journeys with 2 roles + admin
- Load test: simple `GET /services` and messaging spike

---

### 23) Deliverables
- Hosted frontend + backend
- GitHub repo (clean history, README with setup)
- Admin credentials (rotated post-handover)
- `.env.example` with all required variables
- Deployment guide (README)

---

### 24) Notes for Developers
- Prefer TypeScript for safety
- Keep controllers thin; extract business logic into services
- Do not over-engineer; stick to MVP scope
- Log key actions (auth, create/update/delete, booking transitions)
- Add meaningful empty states and toasts for actions