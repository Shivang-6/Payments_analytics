# SalesSaarthi AI

> **AI-powered WhatsApp sales assistant for small businesses**

SalesSaarthi is a full-stack conversational commerce platform where customers interact with a business through WhatsApp and an AI-assisted workflow handles product discovery, pricing questions, lead capture, order collection, payment confirmation, and order management.

The system also provides a business dashboard for product/inventory management, analytics, user statistics, conversation history, and operational visibility.

The repository includes a Node.js/Express backend, Firebase Firestore persistence, Twilio WhatsApp integration, Groq-powered AI responses, a React/TypeScript dashboard, payment/order flows, stateful conversations, analytics, and deployment configuration.

---

## Table of Contents

- [Live Deployment](#live-deployment)
- [What the Project Does](#what-the-project-does)
- [Core Features](#core-features)
- [Architecture](#architecture)
- [Conversation Flow](#conversation-flow)
- [AI and Intent Handling](#ai-and-intent-handling)
- [WhatsApp Integration](#whatsapp-integration)
- [Product Management](#product-management)
- [Lead Tracking](#lead-tracking)
- [Order and Payment Flow](#order-and-payment-flow)
- [Analytics](#analytics)
- [Data Model](#data-model)
- [Backend API](#backend-api)
- [Project Structure](#project-structure)
- [Technology Stack](#technology-stack)
- [Prerequisites](#prerequisites)
- [Environment Variables](#environment-variables)
- [Local Development](#local-development)
- [Testing and Performance](#testing-and-performance)
- [Deployment](#deployment)
- [Troubleshooting](#troubleshooting)
- [Security Notes](#security-notes)
- [Repository Notes](#repository-notes)
- [Contributing](#contributing)
- [License](#license)

---

## Live Deployment

- **Frontend Dashboard:** https://salessaarthi.vercel.app/
- **Backend API:** https://salessaarthi-8iue.onrender.com

The repository README identifies these as the deployed frontend and backend endpoints.

---

# What the Project Does

SalesSaarthi connects a business's WhatsApp inbox to an automated sales workflow.

```text
Customer
   |
   | WhatsApp message
   v
Twilio WhatsApp API
   |
   v
Express Webhook
   |
   +--------------------------+
   |                          |
   v                          v
State / Intent Logic       AI Fallback
   |                          |
   +------------+-------------+
                |
                v
       Conversation Engine
                |
        +-------+-------+
        |               |
        v               v
   Product Data      Order Flow
        |               |
        |               v
        |          Payment Flow
        |               |
        +-------+-------+
                |
                v
          Firestore
                |
                v
        Business Dashboard
```

The assistant is designed to handle common commerce interactions without requiring a business owner to manually answer every WhatsApp message.

---

# Core Features

## WhatsApp Integration

Incoming customer messages are received through a Twilio WhatsApp webhook.

The backend:

1. receives the WhatsApp request
2. extracts the sender and message
3. normalizes the message
4. loads the user's current conversation state
5. detects the intent
6. generates the appropriate response
7. sends the response through Twilio
8. stores the conversation in Firestore
9. updates the user's sales stage

---

## AI-Powered Responses

The application uses the Groq API with a Llama-based model for AI-generated responses.

AI is primarily used as a fallback for messages that cannot be handled by the deterministic intent/state layer.

This gives the system a hybrid architecture:

```text
Known intent
    |
    v
Deterministic conversation logic
    |
    v
Fast predictable response

Unknown/general query
    |
    v
AI service
    |
    v
Contextual response
```

This reduces unnecessary AI calls while retaining natural-language handling for queries outside the predefined flows.

---

## Product Management

The dashboard supports:

- add product
- edit product
- delete product
- view product catalog
- track stock
- display stock status

Product records are stored in Firestore.

The backend validates required fields such as product name and price before creating or updating a product.

---

## Lead Capture

Customers are tracked through sales stages:

```text
new
  |
  v
interested
  |
  v
converted
```

The webhook updates the user's stage based on their conversation flow.

For example:

- normal conversation → `new`
- order/payment flow → `interested`
- successful payment/order confirmation → `converted`

The analytics layer uses these stages to calculate leads and conversion rate.

---

## Analytics Dashboard

The backend exposes analytics for:

- total users
- total messages
- total products
- leads generated
- converted users
- conversion rate
- users by stage
- messages over the previous seven days
- recent chats

The dashboard can visualize this data using charts.

---

# Architecture

## High-Level Architecture

```text
                     ┌─────────────────────┐
                     │      Customer       │
                     │      WhatsApp       │
                     └──────────┬──────────┘
                                │
                                ▼
                     ┌─────────────────────┐
                     │        Twilio       │
                     │   WhatsApp API      │
                     └──────────┬──────────┘
                                │ Webhook
                                ▼
┌─────────────────────────────────────────────────────────┐
│                    Express Backend                      │
│                                                         │
│  Webhook Controller                                     │
│        │                                                │
│        ├── State Service                                │
│        ├── Intent Service                               │
│        ├── Conversation Service                         │
│        ├── Product Service                              │
│        ├── AI Service                                   │
│        ├── Payment Service                              │
│        ├── Order Service                                │
│        └── Twilio Service                               │
│                                                         │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
                 ┌─────────────────────┐
                 │  Firebase Firestore  │
                 │                     │
                 │ users               │
                 │ chats               │
                 │ products            │
                 │ orders              │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │ React Dashboard     │
                 │                     │
                 │ Analytics           │
                 │ Products            │
                 │ Chats               │
                 │ Orders              │
                 └─────────────────────┘
```

---

# Conversation Flow

The current implementation contains both a state-driven conversation system and a newer intent/conversation service path.

The main production webhook uses:

```text
Incoming message
      |
      v
Normalize message
      |
      v
Load user state
      |
      +--------------------------+
      |                          |
      v                          v
Fast commands              Payment state
menu/start/home            verification
cancel/stop/exit                |
      |                          |
      +------------+-------------+
                   |
                   v
             Intent detection
                   |
            +------+------+
            |             |
       Known intent    UNKNOWN
            |             |
            v             v
      Conversation     AI Service
         Service          |
            |             |
            +------+------+
                   |
                   v
             Send via Twilio
                   |
                   v
          Save chat + user data
```

The webhook intentionally sends quick acknowledgements for heavier operations such as payment/order requests or long messages.

---

# Conversation States

The legacy/state-oriented implementation defines flows around concepts such as:

```text
HOME
PRODUCTS
PRODUCT_DETAIL
COLLECTING_DETAILS
PAYMENT
PAYMENT_VERIFYING
MENU
ORDER
```

The exact active state representation is handled by the state service.

A typical purchase journey is:

```text
HOME
  |
  | "products"
  v
PRODUCTS
  |
  | "1"
  v
PRODUCT_DETAIL
  |
  | "buy"
  v
COLLECTING_DETAILS
  |
  | name + address
  v
PAYMENT
  |
  | PAID
  v
PAYMENT_VERIFYING
  |
  v
ORDER CONFIRMED
  |
  v
HOME / MENU
```

---

# AI and Intent Handling

The system uses a rule-first strategy.

Common intents include:

### Greeting

Examples:

```text
hi
hello
hey
good morning
good afternoon
good evening
namaste
```

### Product intent

Examples:

```text
product
products
catalog
show products
items
what do you have
menu
list
```

### Price intent

Examples:

```text
price
cost
rates
how much
pricing
amount
rupees
rs
money
```

### Buy intent

Examples:

```text
buy
order
purchase
want to buy
get
shop
book
```

### Discount intent

Examples:

```text
discount
```

If the deterministic layer cannot identify the message, the application falls back to the AI service.

---

# WhatsApp Integration

## Webhook

The backend exposes:

```http
POST /webhook
```

Twilio sends fields including:

```text
Body
From
```

The application removes the `whatsapp:` prefix from the sender identifier and normalizes the message.

---

## Webhook Health

```http
GET /webhook
```

The webhook controller exposes a readiness response indicating that the WhatsApp webhook is active.

---

## Local Webhook Testing

For local development, the repository recommends exposing the local server through ngrok:

```bash
ngrok http 5000
```

Then configure the resulting public URL in Twilio.

Example:

```text
https://your-ngrok-domain.ngrok.io/webhook
```

---

# Product Management

## Product Fields

The controller currently handles:

```text
name
price
description
stock
createdAt
updatedAt
```

A product is rejected if the required `name` or `price` is missing.

Price is converted to a floating-point number and stock to an integer.

---

# Lead Tracking

User documents contain information including:

```text
phone
stage
lastMessage
updatedAt
```

The stage logic is connected to the conversation flow.

Analytics interprets the stages as:

```text
new
interested
converted
```

The conversion rate is calculated as:

```text
converted users / total users × 100
```

and rounded to two decimal places.

---

# Order and Payment Flow

SalesSaarthi includes a WhatsApp purchase workflow.

## 1. Product Selection

The customer selects a product from the catalog.

The selected product is stored in the user's conversation state.

---

## 2. Customer Details

The assistant requests:

```text
Full name
Delivery address
```

The current implementation performs basic message-pattern validation and stores the collected information in state.

---

## 3. Payment

The system generates a UPI payment message using the product amount.

The implementation contains a payment flow around:

```text
PAID
CANCEL
```

and payment verification.

---

## 4. Payment Confirmation

After a valid payment confirmation:

- order data is created
- the order is stored
- the user state is reset
- a confirmation is sent
- the user can be marked as converted

The newer webhook implementation also generates transaction/order identifiers and sends a final confirmation message.

---

## 5. Order Data

The order flow works with information including:

```text
orderId
product
name
address
phone
amount
```

---

# Analytics

## Analytics Endpoint

```http
GET /api/analytics
```

The analytics controller queries Firestore collections for:

- users
- chats
- products

It calculates:

```text
totalUsers
totalMessages
totalProducts
leadsGenerated
convertedUsers
conversionRate
usersByStage
messagesOverTime
recentChats
```

---

## User Statistics

```http
GET /api/analytics/users
```

Returns user records with timestamps converted to JavaScript dates.

---

## Messages Over Time

The analytics controller calculates message activity over the previous seven days.

The result is grouped by date.

---

## Recent Chats

The backend retrieves the ten newest chat records ordered by timestamp.

---

# Data Model

The implementation uses Firebase Firestore.

The main collections visible in the backend are:

```text
users
chats
products
orders
```

## Users

Representative fields:

```json
{
  "phone": "customer-phone",
  "stage": "new",
  "lastMessage": "timestamp",
  "updatedAt": "timestamp"
}
```

---

## Chats

Representative fields:

```json
{
  "userId": "customer-phone",
  "message": "customer message",
  "reply": "assistant response",
  "timestamp": "timestamp"
}
```

---

## Products

Representative fields:

```json
{
  "name": "Product name",
  "price": 999,
  "description": "Product description",
  "stock": 10,
  "createdAt": "timestamp",
  "updatedAt": "timestamp"
}
```

---

## Orders

The order service is responsible for creating and updating orders. The webhook passes information including customer, product, address, and amount into the order/payment workflow.

---

# Backend API

The Express application mounts these route groups:

```text
/webhook
/api/products
/api/analytics
/api/orders
/api/payment
/api/chats
```

---

## Health

### `GET /health`

Example response:

```json
{
  "status": "OK",
  "message": "AI WhatsApp Sales Assistant is running!"
}
```

---

## Root

### `GET /`

The root endpoint exposes the API version and route groups.

The current implementation identifies version:

```text
2.0.0
```

and exposes:

```text
webhook
products
orders
payment
analytics
health
```

---

## Products

### `GET /api/products`

Returns all products ordered by creation time.

### `POST /api/products`

Creates a product.

Example:

```json
{
  "name": "Example Product",
  "price": 999,
  "description": "Product description",
  "stock": 25
}
```

### `PUT /api/products/:id`

Updates a product.

### `DELETE /api/products/:id`

Deletes a product.

---

## Analytics

### `GET /api/analytics`

Returns dashboard analytics.

### `GET /api/analytics/users`

Returns user statistics.

---

## Orders

```text
/api/orders
```

Order routes are registered by the backend and handled by the order controller/service layer.

---

## Payments

```text
/api/payment
```

Payment routes are registered by the backend.

The WhatsApp conversation also uses the payment service directly for payment confirmation and order completion.

---

## Chats

```text
/api/chats
```

Chat routes are registered by `server.js` for dashboard/chat functionality.

---

# Project Structure

```text
SalesSaarthi AI/
│
├── package.json
├── railway.toml
├── start.sh
│
├── backend/
│   ├── package.json
│   ├── index.js
│   ├── server.js
│   ├── firebase.js
│   ├── debug-firebase.js
│   ├── performance-test.js
│   │
│   ├── controllers/
│   │   ├── analyticsController.js
│   │   ├── productController.js
│   │   ├── webhookController.js
│   │   └── webhookController_old.js
│   │
│   ├── routes/
│   │   ├── webhook
│   │   ├── products
│   │   ├── analytics
│   │   ├── orders
│   │   ├── payment
│   │   └── chats
│   │
│   ├── services/
│   │   ├── AI service
│   │   ├── conversation service
│   │   ├── intent service
│   │   ├── state service
│   │   ├── product service
│   │   ├── order service
│   │   ├── payment service
│   │   └── Twilio service
│   │
│   ├── utils/
│   │   └── cleanup utilities
│   │
│   └── seedData.js
│
└── frontend/
    └── src/
        ├── pages/
        ├── services/
        ├── App.tsx
        └── ...
```

The repository export confirms separate backend and frontend applications and a Node-based root package configuration.

---

# Technology Stack

## Backend

| Technology | Purpose |
|---|---|
| Node.js | Runtime |
| Express.js | REST API/server |
| Firebase Admin | Firestore access |
| Firebase Firestore | Database |
| Twilio | WhatsApp integration |
| Groq SDK | AI generation |
| Axios | HTTP requests |
| CORS | Cross-origin access |
| dotenv | Environment configuration |
| Nodemon | Development reload |

The backend package currently declares:

```text
axios ^1.7.7
cors ^2.8.5
dotenv ^16.4.5
express ^4.19.2
firebase-admin ^12.5.0
groq-sdk ^1.3.0
twilio ^5.3.0
nodemon ^3.1.0
```

---

## Frontend

The project README identifies:

- React.js
- TypeScript
- Tailwind CSS
- Chart.js
- Axios

The frontend provides the business dashboard.

---

## Infrastructure

The repository contains deployment configuration for Railway and the deployed frontend/backend are associated with Vercel and Render respectively.

---

# Prerequisites

The project README specifies:

- Node.js
- npm or yarn
- Firebase project
- Twilio account
- Groq API key

The root `package.json` specifies:

```text
Node >= 16.0.0
npm >= 8.0.0
```

For a clean local setup, use Node 16+.

---

# Environment Variables

Create:

```text
backend/.env
```

The documented backend configuration includes:

```env
# Server
PORT=5000

# Twilio
TWILIO_ACCOUNT_SID=your_twilio_account_sid
TWILIO_AUTH_TOKEN=your_twilio_auth_token
TWILIO_PHONE_NUMBER=your_twilio_phone_number

# Groq
GROQ_API_KEY=your_groq_api_key

# Firebase
FIREBASE_PROJECT_ID=your_firebase_project_id
FIREBASE_PRIVATE_KEY_ID=your_firebase_private_key_id
FIREBASE_PRIVATE_KEY=your_firebase_private_key
FIREBASE_CLIENT_EMAIL=your_firebase_client_email
FIREBASE_CLIENT_ID=your_firebase_client_id
FIREBASE_AUTH_URI=your_firebase_auth_uri
FIREBASE_TOKEN_URI=your_firebase_token_uri
```

The Firebase initialization code also supports additional service-account fields:

```env
FIREBASE_TYPE=
FIREBASE_AUTH_PROVIDER_X509_CERT_URL=
FIREBASE_CLIENT_X509_CERT_URL=
FIREBASE_UNIVERSE_DOMAIN=
```

These should be populated according to the Firebase service-account configuration being used.

---

# Local Development

## 1. Clone

```bash
git clone <repository-url>
cd "SalesSaarthi AI"
```

---

## 2. Install Backend

```bash
cd backend
npm install
```

---

## 3. Configure Firebase

Create a Firebase project and enable Firestore.

Create a service account and provide the required credentials through environment variables.

The Firebase module converts escaped newline sequences in the private key:

```javascript
process.env.FIREBASE_PRIVATE_KEY?.replace(/\n/g, '
')
```

Do not commit Firebase credentials.

---

## 4. Configure Twilio

Create/configure a Twilio WhatsApp environment.

Set:

```text
TWILIO_ACCOUNT_SID
TWILIO_AUTH_TOKEN
TWILIO_PHONE_NUMBER
```

Then configure the WhatsApp webhook:

```text
POST https://your-domain.com/webhook
```

For local testing:

```bash
ngrok http 5000
```

---

## 5. Configure Groq

Create a Groq API key and set:

```env
GROQ_API_KEY=...
```

---

## 6. Seed Data

The repository documents an optional seed command:

```bash
node seedData.js
```

---

## 7. Start Backend

Development:

```bash
npm run dev
```

Production-style start:

```bash
npm start
```

The backend uses:

```javascript
const PORT = process.env.PORT || 3000;
```

The README's local setup uses port `5000`, so the effective port should be kept consistent between `.env`, webhook configuration, and local tooling.

---

## 8. Start Frontend

From the frontend directory:

```bash
cd frontend
npm install
npm start
```

The documented development dashboard is:

```text
http://localhost:3000
```

---

# Root Scripts

The root `package.json` contains:

```json
{
  "scripts": {
    "start": "chmod +x start.sh && ./start.sh",
    "build": "cd backend && npm install",
    "dev": "cd backend && npm run dev"
  }
}
```

The root project therefore also supports:

```bash
npm run dev
npm run build
npm start
```

---

# Startup Script

`start.sh` performs:

```bash
cd backend
npm install
npm start
```

This is also used by the Railway deployment configuration.

---

# Testing and Performance

## Backend Tests

The project README documents:

```bash
cd backend
npm test
```

The actual backend package currently exposes `start` and `dev` scripts; therefore, a separate `test` script should be added if automated npm test execution is required.

---

## Frontend Tests

The project README documents:

```bash
cd frontend
npm test
```

The exact frontend test command depends on the frontend package configuration.

---

# Performance Test

The repository includes:

```text
backend/performance-test.js
```

It exercises the intent and conversation services using messages such as:

```text
menu
browse
1
buy
speaker
cancel
help
track
laptop
2
```

It measures:

```text
intent detection time
response generation time
total time
```

It also performs a repeated intent-detection cache test.

The script defines target goals:

```text
Average total response time < 100 ms
Intent detection < 50 ms
Response generation < 50 ms
```

These are implementation targets defined by the project, not independently verified performance guarantees.

Run it with:

```bash
node performance-test.js
```

---

# Manual Testing

A basic end-to-end test is:

1. start backend
2. start frontend
3. add sample products
4. configure Twilio WhatsApp
5. send WhatsApp messages
6. verify bot responses
7. complete an order flow
8. inspect Firestore
9. verify analytics
10. inspect chat history

Example customer inputs:

```text
hi
products
1
buy
John Doe, 123 Main Street, City
PAID
```

Other supported commands include:

```text
menu
start
home
cancel
stop
exit
```

---

# Smart Response Logic

The system has explicit handling for several categories.

## Price inquiries

The system recognizes terms such as:

```text
price
cost
rates
how much
pricing
amount
rupees
rs
money
```

---

## Product requests

It recognizes:

```text
product
products
catalog
show products
items
what do you have
menu
list
```

---

## Purchase requests

It recognizes:

```text
buy
order
purchase
want to buy
get
shop
book
```

---

## General queries

If deterministic intent detection returns:

```text
UNKNOWN
```

the webhook loads available products and calls the AI service.

---

# Error Handling

The Express application has centralized error middleware that logs the error and returns:

```json
{
  "error": "Internal Server Error"
}
```

The webhook also has its own error handling.

On a webhook failure it attempts to send an error message to the customer and returns an empty TwiML response.

This prevents the webhook handler from simply crashing the request path.

---

# Cleanup

The application starts a cleanup scheduler when the server starts:

```javascript
scheduleCleanup();
```

The cleanup implementation is located under:

```text
backend/utils/cleanup
```

The repository therefore includes background cleanup behavior in addition to request-driven processing.

---

# Deployment

## Backend

The original project documentation supports deployment to platforms such as:

- Render
- Heroku
- Railway

The repository contains a `railway.toml`.

Railway configuration uses:

```toml
[build]
builder = "nixpacks"

[deploy]
startCommand = "chmod +x start.sh && ./start.sh"
healthcheckPath = "/health"
healthcheckTimeout = 100
restartPolicyType = "on_failure"
restartPolicyMaxRetries = 10
```

---

## Frontend

The documented deployment targets include:

- Vercel
- Netlify

The deployed dashboard is currently listed as:

```text
https://salessaarthi.vercel.app/
```

---

# Production Environment

For production:

1. deploy the backend
2. configure all backend environment variables
3. obtain the backend public URL
4. configure Twilio's webhook to use the HTTPS backend URL
5. configure the frontend API base URL
6. deploy the frontend
7. verify CORS
8. verify Firestore access
9. test the full WhatsApp flow
10. verify payment/order behavior

---

# Troubleshooting

## Firebase Connection Error

Check:

```text
FIREBASE_PROJECT_ID
FIREBASE_PRIVATE_KEY_ID
FIREBASE_PRIVATE_KEY
FIREBASE_CLIENT_EMAIL
FIREBASE_CLIENT_ID
FIREBASE_AUTH_URI
FIREBASE_TOKEN_URI
```

Also verify:

- service account validity
- Firestore availability
- project ID
- private-key newline formatting

The repository includes:

```text
backend/debug-firebase.js
```

for inspecting loaded Firebase environment values.

Do not expose private keys in logs or commit them to source control.

---

## Twilio Webhook Not Working

Verify:

- backend is publicly reachable
- webhook URL is correct
- URL points to `/webhook`
- Twilio WhatsApp configuration is correct
- ngrok is running for local testing
- Twilio credentials are correct

---

## Groq API Error

Check:

```text
GROQ_API_KEY
```

Also verify:

- API key validity
- provider availability
- rate limits
- account/billing status

---

## CORS Issues

The Express backend currently uses:

```javascript
app.use(cors());
```

If the deployed frontend cannot reach the backend, verify:

- frontend API URL
- backend availability
- browser network errors
- production CORS policy

For production, CORS should normally be restricted to known frontend origins.

---

# Security Notes

The current repository is a full-stack application/prototype and should be hardened before handling sensitive production traffic.

## Secrets

Never commit:

```text
.env
Firebase private keys
Twilio auth tokens
Groq API keys
```

---

## Firebase Credentials

The backend loads Firebase service-account information from environment variables.

Private keys must remain server-side.

---

## Webhook Verification

The current implementation handles the webhook request but the repository does not document a full Twilio signature-validation layer.

A production deployment should validate Twilio webhook signatures before processing messages.

---

## CORS

The backend currently allows broad CORS through:

```javascript
cors()
```

A production configuration should restrict allowed origins.

---

## Input Validation

Product fields have basic validation.

Customer name/address and payment messages use application-specific validation logic.

For production, stronger schema validation and sanitization should be introduced for every externally supplied field.

---

# Repository Notes

There are currently two closely related backend entry files:

```text
backend/index.js
backend/server.js
```

Both configure Express and route groups, while `server.js` additionally mounts:

```text
/api/chats
```

The repository also contains:

```text
webhookController_old.js
```

which preserves an older state-based webhook implementation.

The active production webhook controller is:

```text
webhookController.js
```

This distinction matters when modifying or deploying the application.

---

# Current System Design

The most important architectural characteristic is the **hybrid deterministic + AI approach**.

```text
                  Incoming Message
                         |
                         v
                 Normalize Input
                         |
                         v
                 Load User State
                         |
                         v
                Intent Detection
                         |
             +-----------+-----------+
             |                       |
       Recognized                Unknown
             |                       |
             v                       v
     State/Conversation          Groq AI
          Service                  Service
             |                       |
             +-----------+-----------+
                         |
                         v
                  Final Response
                         |
                         v
                      Twilio
                         |
                         v
                     Customer
```

This is preferable to sending every message directly to an LLM because common commerce operations remain deterministic and state-aware.

---

# End-to-End Example

Suppose a customer sends:

```text
Hi
```

The system identifies the message as a greeting and returns the menu/home experience.

The customer then sends:

```text
products
```

The system recognizes product intent and retrieves the product catalog.

The customer sends:

```text
1
```

The selected product is placed into conversation state.

The customer sends:

```text
buy
```

The system moves into the purchase flow and requests customer details.

The customer provides:

```text
John Doe, 123 Main Street, Delhi
```

The system stores the relevant information and creates the payment message.

The customer sends:

```text
PAID
```

The payment flow validates the confirmation, creates/stores order information, sends a confirmation, resets the conversation state, and updates the customer toward the converted stage.

The business dashboard can then reflect:

```text
New user
    ↓
Interested lead
    ↓
Converted customer
```

---

# Design Principles

## 1. Fast path first

Frequently used commands are handled without unnecessary external AI calls.

---

## 2. Stateful conversations

The assistant remembers the user's current step and selected product during the interaction.

---

## 3. AI as fallback

The LLM extends the assistant's natural-language capability instead of being responsible for every deterministic business operation.

---

## 4. Persistent business data

Products, users, chats, and orders are stored in Firestore.

---

## 5. Non-blocking operations

The webhook implementation deliberately avoids making every database/background operation block the customer response.

For example, chat persistence can be triggered after the response is sent.

---

# Potential Improvements

These are engineering extensions rather than claims about the current implementation.

## Security

- Twilio signature verification
- strict CORS
- rate limiting
- authentication for dashboard APIs
- authorization/RBAC
- secret rotation

## Reliability

- persistent conversation state
- transactional order creation
- idempotent payment handling
- webhook retry handling
- background job queue
- structured logging

## AI

- stronger product-aware prompts
- tool/function calling
- structured AI output
- conversation memory
- hallucination controls
- response evaluation

## Commerce

- real payment gateway integration
- payment webhook verification
- inventory reservation
- stock decrement after confirmed payment
- refunds/cancellations
- order tracking

## Analytics

- funnel analytics
- product-level conversion
- revenue analytics
- customer retention
- response latency monitoring
- AI usage/cost metrics

---

# Contributing

The documented contribution process is:

1. Fork the repository.
2. Create a feature branch.
3. Make the changes.
4. Test thoroughly.
5. Submit a pull request.

Example:

```bash
git checkout -b feature/my-feature
git add .
git commit -m "Add my feature"
git push origin feature/my-feature
```

---

# License

The project is licensed under the:

```text
MIT License
```

---

# Project Summary

SalesSaarthi is an end-to-end AI-assisted conversational commerce platform.

Its architecture combines:

```text
WhatsApp
   +
Twilio
   +
Express.js
   +
Firebase Firestore
   +
Groq AI
   +
Stateful conversation logic
   +
Product management
   +
Order/payment workflows
   +
Analytics dashboard
```

The central workflow is:

```text
Customer Message
      ↓
WhatsApp / Twilio
      ↓
Express Webhook
      ↓
State + Intent Detection
      ↓
Deterministic Conversation Logic
      ↓
AI Fallback for Unknown Queries
      ↓
Product / Order / Payment Operations
      ↓
Firestore
      ↓
Analytics + Dashboard
```

This gives small businesses a single system for handling customer conversations, product discovery, sales leads, orders, and operational analytics through WhatsApp.
