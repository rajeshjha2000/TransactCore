# TransactCore

A backend ledger service built with Node.js, Express, and MongoDB. It provides APIs for user authentication, account management, and financial transactions with double-entry bookkeeping.

## Features

- **User Authentication** — Register, login, and logout with JWT-based auth and token blacklisting
- **Account Management** — Create accounts, view balances, and manage account status (Active / Frozen / Closed)
- **Transactions** — Transfer funds between accounts with balance validation and idempotency keys
- **Double-Entry Ledger** — Every transaction creates immutable DEBIT and CREDIT ledger entries
- **Email Notifications** — Sends registration and transaction confirmation emails via Nodemailer (Gmail OAuth2)
- **System User Support** — Protected routes for initial fund loading by system-level users

## Tech Stack

- **Runtime**: Node.js
- **Framework**: Express.js
- **Database**: MongoDB (Mongoose ODM)
- **Auth**: JWT + bcrypt
- **Email**: Nodemailer (Gmail OAuth2)

## Project Structure

```
TransactCore/
├── server.js                     # Entry point
├── src/
│   ├── app.js                    # Express app setup & route mounting
│   ├── config/
│   │   └── db.js                 # MongoDB connection
│   ├── controllers/
│   │   ├── auth.controller.js    # Register, login, logout
│   │   ├── account.controller.js # Create account, get balance
│   │   └── transaction.controller.js # Fund transfers, initial funds
│   ├── middleware/
│   │   └── auth.middleware.js    # JWT verification & system user check
│   ├── models/
│   │   ├── user.model.js         # User schema with password hashing
│   │   ├── account.model.js      # Account schema with balance aggregation
│   │   ├── transaction.model.js  # Transaction schema with idempotency
│   │   ├── ledger.model.js       # Immutable ledger entries
│   │   └── blackList.model.js    # Token blacklist with TTL
│   ├── routes/
│   │   ├── auth.routes.js
│   │   ├── account.routes.js
│   │   └── transaction.routes.js
│   └── services/
│       └── email.service.js      # Email notifications
```

## Getting Started

### Prerequisites

- Node.js (v18+)
- MongoDB Atlas or local MongoDB instance
- Gmail account with OAuth2 credentials (for email notifications)

### Installation

```bash
git clone https://github.com/rajeshjha2000/TransactCore.git
cd TransactCore
npm install
```

### Environment Variables

Create a `.env` file in the root directory:

```env
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret

# Email (Gmail OAuth2)
EMAIL_USER=your_email@gmail.com
CLIENT_ID=your_google_client_id
CLIENT_SECRET=your_google_client_secret
REFRESH_TOKEN=your_google_refresh_token
```

### Run the Server

```bash
# Development
npm run dev

# Production
npm start
```

Server runs on `http://localhost:3000`

## API Endpoints

### Auth
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Register a new user |
| POST | `/api/auth/login` | Login and get JWT token |
| POST | `/api/auth/logout` | Logout and blacklist token |

### Accounts
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/accounts/` | Create a new account |
| GET | `/api/accounts/` | Get all accounts of logged-in user |
| GET | `/api/accounts/balance/:accountId` | Get account balance |

### Transactions
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/transactions/` | Create a new transaction |
| POST | `/api/transactions/system/initial-funds` | Load initial funds (system user only) |

> All account and transaction routes are protected and require a valid JWT token.

## How Transactions Work

1. Validate request params and idempotency key
2. Check both accounts are ACTIVE
3. Verify sender has sufficient balance (derived from ledger)
4. Create transaction record with PENDING status
5. Create DEBIT entry for sender in ledger
6. Create CREDIT entry for receiver in ledger
7. Mark transaction as COMPLETED
8. All steps 4-7 run inside a MongoDB session for atomicity
9. Send email notification on success
