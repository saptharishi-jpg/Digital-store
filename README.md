# Digital Product Store

A full-stack digital product store built with **FastAPI, SQLAlchemy,
SQLite, React, Vite, JWT authentication, and Stripe Checkout/Webhooks**.

## 1. Project Setup

### Technologies

  Technology        Purpose
  ----------------- -----------------------------
  Python            Backend programming
  FastAPI           REST API
  Uvicorn           Backend server
  SQLAlchemy        Database ORM
  SQLite            Default database
  Pydantic          Request validation
  JWT               User authentication
  React             Frontend UI
  Vite              Frontend development server
  Axios             Frontend API requests
  React Router      Frontend routing
  React Toastify    Notifications
  Stripe Checkout   Online test payments
  Stripe CLI        Local webhook forwarding
  Pytest            Backend testing

### Prerequisites

-   Python 3.10+
-   Node.js and npm
-   Stripe account in Sandbox/Test mode
-   Stripe CLI for local webhook testing

Install backend dependencies:

``` powershell
pip install -r requirements.txt
```

The current `main.py` imports the Stripe Python package. If `stripe` is
not already installed:

``` powershell
pip install stripe
```

## 2. Database Configuration

The project uses SQLite by default:

``` env
DATABASE_URL=sqlite:///./store.db
```

The application creates the database tables automatically when it
starts.

Main tables:

-   `users`
-   `products`
-   `carts`
-   `cart_items`
-   `orders`
-   `order_items`
-   `payments`

## 3. Environment Variables

Create a `.env` file in the backend/project directory.

**Never commit the real `.env` file or publish Stripe secrets.**

``` env
DATABASE_URL=sqlite:///./store.db
JWT_SECRET=your-long-random-secret
STRIPE_SECRET_KEY=sk_test_REPLACE_WITH_YOUR_TEST_SECRET_KEY
STRIPE_WEBHOOK_SECRET=whsec_REPLACE_WITH_YOUR_WEBHOOK_SECRET
FRONTEND_URL=http://localhost:5173
PAYMENT_SUCCESS_URL=http://localhost:5173/orders?success=1
PAYMENT_CANCEL_URL=http://localhost:5173/cart?cancelled=1
```

  Variable                  Purpose
  ------------------------- ---------------------------------------
  `DATABASE_URL`            Database connection
  `JWT_SECRET`              JWT signing secret
  `STRIPE_SECRET_KEY`       Stripe test secret key
  `STRIPE_WEBHOOK_SECRET`   Stripe webhook signature verification
  `FRONTEND_URL`            Frontend origin allowed by CORS
  `PAYMENT_SUCCESS_URL`     Stripe success redirect
  `PAYMENT_CANCEL_URL`      Stripe cancel redirect

Because `main.py` reads variables with `os.getenv()`, start FastAPI
with:

``` powershell
uvicorn main:app --reload --env-file .env
```

## 4. Stripe Configuration

The application uses Stripe Checkout in Sandbox/Test mode.

### Secret key

Use a Stripe test key beginning with `sk_test_`:

``` env
STRIPE_SECRET_KEY=sk_test_...
```

### Webhook secret

The backend expects a webhook signing secret beginning with `whsec_`:

``` env
STRIPE_WEBHOOK_SECRET=whsec_...
```

The FastAPI webhook endpoint is:

``` text
POST http://localhost:8000/payments/webhook
```

### Stripe CLI

For local webhook testing:

``` powershell
stripe.cmd listen --events checkout.session.completed,checkout.session.async_payment_succeeded,checkout.session.async_payment_failed,checkout.session.expired --forward-to localhost:8000/payments/webhook
```

When the listener starts successfully, Stripe CLI provides a `whsec_...`
signing secret. Put that secret in `STRIPE_WEBHOOK_SECRET`.

Keep the Stripe CLI listener running while testing checkout.

The backend handles:

-   `checkout.session.completed` → `PAID`
-   `checkout.session.async_payment_succeeded` → `PAID`
-   `checkout.session.async_payment_failed` → `FAILED`
-   `checkout.session.expired` → `CANCELLED`

For Stripe test checkout, use:

``` text
4242 4242 4242 4242
```

with a future expiry and test CVC.

Stripe cannot directly reach a local `localhost` endpoint from its
servers, so Stripe CLI forwarding is required for local webhook testing.
If Stripe CLI reports an account-role/permission error, the Stripe
account administrator must grant the required permissions.

## 5. Backend Execution Steps

Open the VS Code terminal in the project directory.

Install dependencies:

``` powershell
pip install -r requirements.txt
```

If required:

``` powershell
pip install stripe
```

Create/configure `.env`, then run:

``` powershell
uvicorn main:app --reload --env-file .env
```

Backend:

``` text
http://localhost:8000
```

Health check:

``` text
http://localhost:8000/health
```

Swagger:

``` text
http://localhost:8000/docs
```

ReDoc:

``` text
http://localhost:8000/redoc
```

## 6. Frontend Execution Steps

Open another VS Code terminal.

If the frontend is in a `frontend` directory:

``` powershell
cd frontend
```

Install dependencies:

``` powershell
npm install
```

Start Vite:

``` powershell
npm run dev
```

Frontend:

``` text
http://localhost:5173
```

The frontend defaults to the backend URL:

``` text
http://localhost:8000
```

If required, set:

``` env
VITE_API_URL=http://localhost:8000
```

## 7. Main Features

### Authentication

``` text
POST /auth/register
POST /auth/login
GET  /users/me
```

JWT authentication protects profile, cart, and order APIs.

### Products

``` text
GET    /products
GET    /products/{id}
POST   /products
PUT    /products/{id}
DELETE /products/{id}
```

Product writes require admin access.

### Cart

``` text
GET    /cart
POST   /cart/items
PUT    /cart/items/{product_id}
DELETE /cart/items/{product_id}
DELETE /cart
```

### Payments

``` text
POST /payments/create-checkout-session
POST /payments/webhook
```

### Orders

``` text
GET /orders
GET /orders/{id}
GET /admin/orders
```

### Admin statistics and reports

``` text
GET /admin/stats
GET /reports/most-purchased
GET /reports/never-purchased
```

## 8. Admin Access

Registration creates regular users.

After registering a user, an existing user can be promoted to admin
with:

``` powershell
py -c "from main import SessionLocal,User; s=SessionLocal(); u=s.query(User).filter_by(email='YOUR_EMAIL@example.com').one(); u.is_admin=True; s.commit(); s.close()"
```

Replace `YOUR_EMAIL@example.com` with the registered email.

Log in again after changing the admin status so the frontend receives a
fresh JWT.

Admin functionality includes product CRUD, viewing all orders,
statistics, and reports.

## 9. Testing Commands

Run tests from the project/backend directory:

``` powershell
pytest -q
```

For detailed output:

``` powershell
pytest -v
```

The current test suite covers:

-   Registration and duplicate registration
-   Login and invalid login
-   Unauthorized profile access
-   Admin protection for product creation
-   Product listing
-   Product pagination
-   Product search
-   Cart operations
-   Empty checkout validation
-   Order ownership protection

The tests use an isolated SQLite database. Stripe network checkout is
not called by the automated tests.

## 10. Useful URLs

``` text
Frontend:       http://localhost:5173
Backend:        http://localhost:8000
Swagger:        http://localhost:8000/docs
ReDoc:          http://localhost:8000/redoc
Health:         http://localhost:8000/health
```

## 11. End-to-End Payment Flow

``` text
React Cart
    ↓
POST /payments/create-checkout-session
    ↓
FastAPI
    ↓
Stripe Checkout
    ↓
Test Payment
    ↓
Stripe Event
    ↓
Stripe CLI
    ↓
POST /payments/webhook
    ↓
FastAPI verifies signature
    ↓
Order/Payment status updated
    ↓
PAID / FAILED / CANCELLED
```

A successful `PAID` webhook also clears the user's cart.

## 12. Security Notes

-   Never commit `.env`.
-   Never publish `STRIPE_SECRET_KEY`.
-   Never publish `STRIPE_WEBHOOK_SECRET`.
-   Use Stripe Sandbox/Test keys for development.
-   Do not use real card information in test checkout.
-   JWT authentication protects user-specific APIs.
-   Admin routes require an admin user.

## 13. Typical Startup Order

### Terminal 1 --- FastAPI

``` powershell
uvicorn main:app --reload --env-file .env
```

### Terminal 2 --- Stripe CLI

``` powershell
stripe.cmd listen --events checkout.session.completed,checkout.session.async_payment_succeeded,checkout.session.async_payment_failed,checkout.session.expired --forward-to localhost:8000/payments/webhook
```

### Terminal 3 --- React/Vite

``` powershell
npm run dev
```

Then open:

``` text
http://localhost:5173
```

## 14. Troubleshooting

### 401 Unauthorized

Log in again and make sure the frontend has a valid JWT token.

### Stripe checkout failed

Check that:

1.  `STRIPE_SECRET_KEY` is configured.
2.  It is a Stripe test/sandbox key.
3.  FastAPI was started with `--env-file .env`.
4.  The key has not been revoked.

### Webhook not updating the order

Check that:

1.  Stripe CLI is running.
2.  The forwarding target is `localhost:8000/payments/webhook`.
3.  `STRIPE_WEBHOOK_SECRET` matches the signing secret for the active
    local forwarding session.
4.  FastAPI is running.
5.  Stripe CLI has the required account permissions.

## 15. Project Checklist

-   FastAPI backend
-   SQLite database
-   SQLAlchemy models and relationships
-   User registration/login
-   JWT authentication
-   Product listing/search/pagination
-   Admin product CRUD
-   User-specific cart
-   Stripe Checkout
-   Stripe webhook endpoint
-   Order management
-   Payment status handling
-   Admin statistics
-   Purchase reports
-   React + Vite frontend
-   Axios API integration
-   React routing
-   Toast notifications
-   Pytest tests
-   Swagger documentation
