# BACKEND MODULE
### NestJS REST API Development
**Time Allowed:** 3 Hours | **Total Marks:** 100

---

## Scenario

**SmartLib** is a startup building a digital library management platform for universities. Their current system is a mess of spreadsheets and manual processes — students can't check book availability online, librarians have no way to track loans, and the admin team has zero reporting visibility.

You have been contracted as a backend engineer to build the core API that will power the new SmartLib platform. The frontend team is already waiting on your endpoints. The CTO has reviewed the architecture and decided on **NestJS** with **Prisma ORM** and **MySQL** as the stack.

You will build the system from scratch. Every endpoint must be production-ready — validated inputs, proper error responses, role-based access, and a clean database schema.

---

## Overview of the System

SmartLib has three types of users:

| Role | Description |
|------|-------------|
| `ADMIN` | Manages books, users, and views reports |
| `LIBRARIAN` | Processes loans and returns |
| `MEMBER` | Browses books and views their own loan history |

There are three core resources:

- **Users** — accounts with roles
- **Books** — the library catalog
- **Loans** — records of who borrowed which book and when

---

## Technical Requirements

### Stack
- **Framework:** NestJS (v10+)
- **ORM:** Prisma
- **Database:** MySQL
- **Auth:** JWT (Bearer token)
- **Validation:** `class-validator` + `class-transformer`

### Project Setup
```
src/
├── main.ts
├── app.module.ts
├── prisma/
│   ├── prisma.module.ts
│   └── prisma.service.ts
├── auth/
│   ├── auth.module.ts
│   ├── auth.service.ts
│   ├── auth.controller.ts
│   ├── dto/
│   ├── guards/
│   └── strategies/
├── users/
│   ├── users.module.ts
│   ├── users.service.ts
│   ├── users.controller.ts
│   └── dto/
├── books/
│   ├── books.module.ts
│   ├── books.service.ts
│   ├── books.controller.ts
│   └── dto/
└── loans/
    ├── loans.module.ts
    ├── loans.service.ts
    ├── loans.controller.ts
    └── dto/
```

### Environment Variables
```env
DATABASE_URL="mysql://root:password@localhost:3306/smartlib"
JWT_SECRET="your-secret-key"
JWT_EXPIRES_IN="7d"
```

---

## Database Schema

Design the Prisma schema to satisfy all the requirements below.

### Required Models

#### `User`
| Field | Type | Notes |
|-------|------|-------|
| `id` | Int (PK) | Auto-increment |
| `name` | String | |
| `email` | String | Unique |
| `password` | String | Hashed with bcrypt |
| `role` | Enum | `ADMIN`, `LIBRARIAN`, `MEMBER` |
| `createdAt` | DateTime | Default now |
| `updatedAt` | DateTime | Auto-update |

#### `Book`
| Field | Type | Notes |
|-------|------|-------|
| `id` | Int (PK) | Auto-increment |
| `title` | String | |
| `author` | String | |
| `isbn` | String | Unique |
| `category` | String | |
| `stock` | Int | Must be ≥ 0 |
| `createdAt` | DateTime | Default now |
| `updatedAt` | DateTime | Auto-update |

#### `Loan`
| Field | Type | Notes |
|-------|------|-------|
| `id` | Int (PK) | Auto-increment |
| `userId` | Int (FK) | References `User` |
| `bookId` | Int (FK) | References `Book` |
| `borrowedAt` | DateTime | Default now |
| `dueDate` | DateTime | Set at creation |
| `returnedAt` | DateTime? | Null until returned |
| `status` | Enum | `ACTIVE`, `RETURNED`, `OVERDUE` |

---

## Tasks & Marking Criteria

---

### Task 1 — Authentication Module `(20 marks)`

Implement a JWT-based authentication system.

#### Endpoints

**`POST /auth/register`**

Register a new user. Default role is `MEMBER`.

Request body:
```json
{
  "name": "Budi Santoso",
  "email": "budi@ui.ac.id",
  "password": "secret123"
}
```

Success response `201`:
```json
{
  "token": "<jwt>",
  "user": {
    "id": 1,
    "name": "Budi Santoso",
    "email": "budi@ui.ac.id",
    "role": "MEMBER"
  }
}
```

Error cases:
- `409 Conflict` — email already registered
- `400 Bad Request` — missing or invalid fields

---

**`POST /auth/login`**

Authenticate and receive a token.

Request body:
```json
{
  "email": "budi@ui.ac.id",
  "password": "secret123"
}
```

Success response `200`:
```json
{
  "token": "<jwt>",
  "user": { ... }
}
```

Error cases:
- `401 Unauthorized` — wrong email or password (use same message for both, do not reveal which)

---

#### Marking Criteria — Task 1

| Criteria | Marks |
|----------|-------|
| Register endpoint works, returns token | 4 |
| Password is hashed in the database | 3 |
| Duplicate email returns 409 | 2 |
| Login with correct credentials returns token | 4 |
| Login with wrong credentials returns 401 | 2 |
| JWT is valid and signed with secret | 3 |
| Validation rejects missing/invalid fields | 2 |

---

### Task 2 — Users Module `(15 marks)`

Manage user accounts. All endpoints in this module require authentication.

#### Endpoints

**`GET /users/me`** — `Any authenticated user`

Returns the currently logged-in user's profile. Read from the JWT payload.

---

**`GET /users`** — `ADMIN only`

Returns a list of all users. Never return `password` in any response.

---

**`PATCH /users/:id`** — `ADMIN only`

Update a user's name, email, or role. Validate the role against the enum.

---

**`DELETE /users/:id`** — `ADMIN only`

Delete a user. Return `404` if user not found.

---

#### Marking Criteria — Task 2

| Criteria | Marks |
|----------|-------|
| `GET /users/me` returns authenticated user | 3 |
| `GET /users` returns all users (no passwords) | 3 |
| `PATCH /users/:id` updates fields correctly | 4 |
| `DELETE /users/:id` deletes and handles 404 | 3 |
| Non-ADMIN access returns 403 | 2 |

---

### Task 3 — Books Module `(25 marks)`

Manage the library catalog.

#### Endpoints

**`GET /books`** — `Public` (no auth required)

Returns all books. Support optional query parameter `?category=fiction` to filter by category.

---

**`GET /books/:id`** — `Public`

Returns a single book. Return `404` if not found.

---

**`POST /books`** — `ADMIN only`

Add a new book to the catalog.

Request body:
```json
{
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "9780132350884",
  "category": "Programming",
  "stock": 5
}
```

Error cases:
- `409 Conflict` — ISBN already exists
- `400 Bad Request` — `stock` is negative

---

**`PATCH /books/:id`** — `ADMIN only`

Update book details or stock. Partial update (all fields optional).

---

**`DELETE /books/:id`** — `ADMIN only`

Delete a book. Return `400 Bad Request` if the book has any `ACTIVE` loans (cannot delete a borrowed book).

---

#### Marking Criteria — Task 3

| Criteria | Marks |
|----------|-------|
| `GET /books` returns all books | 3 |
| `GET /books?category=x` filters correctly | 3 |
| `GET /books/:id` returns single book or 404 | 3 |
| `POST /books` creates a book | 4 |
| Duplicate ISBN returns 409 | 2 |
| Negative stock is rejected | 2 |
| `PATCH /books/:id` updates partial fields | 4 |
| `DELETE /books/:id` with active loan guard | 4 |

---

### Task 4 — Loans Module `(30 marks)`

This is the core business logic of SmartLib.

#### Business Rules

> Read these carefully. They are enforced by your API.

1. A `MEMBER` can only borrow a book if `stock > 0`.
2. When a loan is created, the book's `stock` decreases by 1.
3. `dueDate` is always **14 days** from `borrowedAt`.
4. A `MEMBER` cannot borrow the **same book** if they already have an `ACTIVE` loan for it.
5. When a loan is returned, `stock` increases by 1 and `returnedAt` is set to now.
6. A loan is `OVERDUE` if `status` is still `ACTIVE` and today's date is past `dueDate`.

---

#### Endpoints

**`POST /loans`** — `MEMBER only`

Borrow a book. The `userId` is taken from the JWT (do not accept it in the body).

Request body:
```json
{
  "bookId": 3
}
```

Success response `201`:
```json
{
  "id": 12,
  "bookId": 3,
  "userId": 1,
  "borrowedAt": "2025-06-16T08:00:00.000Z",
  "dueDate": "2025-06-30T08:00:00.000Z",
  "status": "ACTIVE"
}
```

Error cases:
- `400` — book out of stock
- `409` — member already has an active loan for this book

---

**`GET /loans/my`** — `MEMBER only`

Returns all loans belonging to the logged-in member. Include a computed field `isOverdue: boolean`.

---

**`GET /loans`** — `ADMIN` or `LIBRARIAN`

Returns all loans across all members. Support query params:
- `?status=ACTIVE` — filter by status
- `?userId=5` — filter by member

---

**`PATCH /loans/:id/return`** — `LIBRARIAN` or `ADMIN`

Mark a loan as returned. Executes a Prisma transaction:
1. Set `returnedAt` = now
2. Set `status` = `RETURNED`
3. Increase `book.stock` by 1

Return `400` if the loan is already returned.

---

**`GET /loans/overdue`** — `ADMIN` or `LIBRARIAN`

Returns all loans where `status` is `ACTIVE` and `dueDate` is in the past. Include the `user` and `book` in the response.

---

#### Marking Criteria — Task 4

| Criteria | Marks |
|----------|-------|
| `POST /loans` creates loan and decrements stock | 5 |
| Out-of-stock returns 400 | 3 |
| Duplicate active loan returns 409 | 3 |
| `dueDate` is exactly 14 days from now | 2 |
| `GET /loans/my` returns member's own loans | 3 |
| `isOverdue` computed field is correct | 2 |
| `GET /loans` with filters works | 3 |
| `PATCH /loans/:id/return` uses transaction | 5 |
| Already-returned loan returns 400 | 2 |
| `GET /loans/overdue` returns correct results | 2 |

---

### Task 5 — Code Quality `(10 marks)`

Your code will be reviewed for structure and best practices.

| Criteria | Marks |
|----------|-------|
| `password` is never returned in any response | 2 |
| DTOs use `class-validator` decorators throughout | 2 |
| `ValidationPipe` with `whitelist: true` is active globally | 1 |
| Prisma errors are handled (no unhandled exceptions) | 2 |
| `PrismaModule` is `@Global()` and not re-imported | 1 |
| Modules are decoupled (no cross-importing services) | 2 |

---

## API Summary

| Method | Endpoint | Auth | Role |
|--------|----------|------|------|
| POST | `/auth/register` | ✗ | — |
| POST | `/auth/login` | ✗ | — |
| GET | `/users/me` | ✓ | Any |
| GET | `/users` | ✓ | ADMIN |
| PATCH | `/users/:id` | ✓ | ADMIN |
| DELETE | `/users/:id` | ✓ | ADMIN |
| GET | `/books` | ✗ | — |
| GET | `/books/:id` | ✗ | — |
| POST | `/books` | ✓ | ADMIN |
| PATCH | `/books/:id` | ✓ | ADMIN |
| DELETE | `/books/:id` | ✓ | ADMIN |
| POST | `/loans` | ✓ | MEMBER |
| GET | `/loans/my` | ✓ | MEMBER |
| GET | `/loans` | ✓ | ADMIN, LIBRARIAN |
| PATCH | `/loans/:id/return` | ✓ | ADMIN, LIBRARIAN |
| GET | `/loans/overdue` | ✓ | ADMIN, LIBRARIAN |

---

## Judging Notes

- The judges will run a Postman collection against your server on `http://localhost:3000`
- Your database must be migrated and seeded before submission (`npx prisma migrate dev`)
- The `.env` file must be present and correct
- **Do not use any database seeding for test data** — the Postman collection creates its own data from scratch
- Partial credit is awarded per criteria row — a working endpoint with a missing guard still earns partial marks

---

## Submission Checklist

- [ ] `prisma/schema.prisma` is complete and migrated
- [ ] `.env` file is configured
- [ ] Server starts with `npm run start:dev`
- [ ] All 16 endpoints respond correctly
- [ ] Passwords are never exposed in responses
- [ ] Business rules in Task 4 are enforced

---

*SmartLib Module C — Backend Web Technologies*
*Competition Module v1.0*