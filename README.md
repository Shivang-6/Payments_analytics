# PayFriction Analytics

> End-to-end payment-friction analytics platform for understanding payment failures, retry behavior, bank response codes, revenue at risk, and payment recovery.

PayFriction Analytics brings payment data and analytical workflows into a unified environment for finance, revenue operations, payment operations, risk analysts, product teams, BI teams, engineering, and customer support.

The repository combines:

- A **NestJS + Prisma + PostgreSQL** backend for payment, transaction, gateway, customer, merchant, and analytics data.
- **Python/Pandas/SQL** analytics and validation workflows.
- **Streamlit** dashboards for KPI monitoring and interactive exploration.
- SQL views and pre-aggregated tables for reusable and performant analytical metrics.
- Visualization scripts and notebooks for deeper analysis.
- Validation tooling that compares SQL and Python calculations to detect metric drift.
- Documentation for data conventions, dashboard design, reporting, exports, and team Git workflow.

---

## Table of Contents

- [Product Overview](#product-overview)
- [Problem](#problem)
- [Objectives](#objectives)
- [Target Users](#target-users)
- [Core Capabilities](#core-capabilities)
- [System Architecture](#system-architecture)
- [Repository Structure](#repository-structure)
- [Technology Stack](#technology-stack)
- [Backend](#backend)
- [Database Model](#database-model)
- [Analytics API](#analytics-api)
- [Transaction API](#transaction-api)
- [Python Analytics Layer](#python-analytics-layer)
- [Data Pipeline](#data-pipeline)
- [Dashboard Layer](#dashboard-layer)
- [SQL Data Layer](#sql-data-layer)
- [KPI Layer](#kpi-layer)
- [Metric Validation and Drift Detection](#metric-validation-and-drift-detection)
- [Visualization Layer](#visualization-layer)
- [Notebooks](#notebooks)
- [Sample Data and Seed Data](#sample-data-and-seed-data)
- [Local Setup](#local-setup)
- [Running the Backend](#running-the-backend)
- [Running the Streamlit Dashboards](#running-the-streamlit-dashboards)
- [Running Analytics Scripts](#running-analytics-scripts)
- [Database Setup](#database-setup)
- [Testing](#testing)
- [Data Model and Business Definitions](#data-model-and-business-definitions)
- [Payment-Friction Classification](#payment-friction-classification)
- [Reporting and Exports](#reporting-and-exports)
- [Performance and Non-Functional Requirements](#performance-and-non-functional-requirements)
- [Security Requirements](#security-requirements)
- [Data Quality and Validation](#data-quality-and-validation)
- [Business Analysis Included in the Repository](#business-analysis-included-in-the-repository)
- [Development Workflow](#development-workflow)
- [Contribution Guidelines](#contribution-guidelines)
- [Known Implementation Notes](#known-implementation-notes)
- [Future Enhancements](#future-enhancements)
- [License](#license)

---

## Product Overview

Payment systems produce data across gateways, retries, bank response codes, transaction histories, and settlement/reporting systems.

PayFriction Analytics is designed to turn that fragmented information into operational metrics and financial insights.

The platform focuses on an important distinction:

**Not every failed payment represents permanent revenue loss.**

Some failures are temporary and may be recovered through an appropriate retry. Others represent permanent failures where additional retries are unlikely to recover the transaction.

The analytical model therefore separates:

- temporary payment friction,
- recoverable revenue,
- permanent payment failure,
- permanently lost revenue,
- revenue at risk,
- retry effectiveness,
- gateway performance,
- bank performance.

This enables teams to investigate payment problems from both an operational and financial perspective.

---

# Problem

Modern payment systems generate data from multiple sources:

- Payment gateway logs
- Retry events
- Bank response codes
- Transaction history
- Settlement reports

When these sources are analyzed separately, teams have difficulty answering questions such as:

- Why did a payment fail?
- Is the failure temporary or permanent?
- Which transactions should be retried?
- How effective are retries?
- How much revenue is recoverable?
- How much revenue has already been lost?
- Which gateway performs better?
- Which banks generate the highest failure rates?
- Are payment failures increasing over time?

Manual analysis also creates the risk of inconsistent metric definitions between SQL queries, Python notebooks, dashboards, and reports.

PayFriction Analytics addresses this by establishing reusable data-layer objects, analytical functions, dashboards, and validation checks.

---

# Objectives

## Business Objectives

- Reduce revenue leakage.
- Increase successful payment recovery.
- Improve retry strategy effectiveness.
- Accelerate financial reporting.
- Standardize payment analytics.

## Product Objectives

- Centralize payment analytics.
- Decode bank response codes.
- Classify payment failures.
- Provide interactive dashboards.
- Generate analytical insights.
- Support reproducible workflows.

---

# Target Users

### Primary Users

- Finance teams
- Revenue Operations
- Payment Operations
- Risk Analysts

### Secondary Users

- Product Managers
- Business Intelligence teams
- Engineering teams
- Customer Support

---

# Core Capabilities

## 1. Payment Analytics

The platform is designed to track:

- Total transactions
- Successful payments
- Failed payments
- Pending transactions
- Retry attempts
- Retry success rate
- Revenue recovered
- Revenue lost
- Revenue at risk

## 2. Retry Analysis

Retry analysis includes:

- Retry count
- Retry timeline
- Retry success percentage
- Average retry delay
- Retry effectiveness
- Retry distribution by gateway

## 3. Bank Response Code Decoder

Bank response codes are stored with:

- Code
- Meaning
- Classification
- Recommended action

Examples from the product definition include:

| Code | Meaning | Classification / Action |
|---|---|---|
| `00` | Approved | Success |
| `05` | Do Not Honor | Manual review |
| `51` | Insufficient Funds | Retry later |
| `54` | Expired Card | Permanent failure / stop retry |
| `91` | Issuer Unavailable | Retry automatically |

## 4. Payment-Friction Classification

### Temporary Friction

Examples:

- Network timeout
- Gateway timeout
- Bank unavailable
- System maintenance
- Processing delay

Typical action:

**Retry automatically**, subject to business rules.

### Permanent Failure

Examples:

- Expired card
- Invalid card
- Closed account
- Fraud block
- Customer cancellation

Typical action:

**Stop retrying and notify the customer.**

## 5. Revenue Intelligence

The platform is designed to expose:

- Recoverable revenue
- Permanently lost revenue
- Revenue leakage percentage
- Average recovery time
- Revenue by bank
- Revenue by gateway
- Revenue by merchant
- Revenue trends

## 6. Bank and Gateway Performance

Performance can be compared using:

- Approval rate
- Failure rate
- Retry success
- Average processing time
- Revenue impact

## 7. Failure Trend Analysis

The repository contains analytical workflows for:

- Daily failures
- Weekly trends
- Monthly trends
- Failure heatmaps
- Hourly failure distributions

## 8. Search and Filtering

The product definition supports filtering by:

- Date range
- Payment gateway
- Bank
- Merchant
- Transaction status
- Failure reason
- Retry status
- Amount range

---

# System Architecture

The repository contains three main analytical/application layers.

```text
                         ┌───────────────────────────┐
                         │       Users / Teams       │
                         │ Finance / Ops / Product   │
                         └─────────────┬─────────────┘
                                       │
                    ┌──────────────────┴──────────────────┐
                    │                                     │
                    ▼                                     ▼
          ┌──────────────────┐                 ┌──────────────────┐
          │ Streamlit        │                 │ NestJS Backend   │
          │ Dashboards       │                 │ REST API         │
          └────────┬─────────┘                 └────────┬─────────┘
                   │                                    │
                   │                                    ▼
                   │                           ┌──────────────────┐
                   │                           │ Prisma ORM       │
                   │                           └────────┬─────────┘
                   │                                    │
                   │                                    ▼
                   │                           ┌──────────────────┐
                   │                           │ PostgreSQL       │
                   │                           └──────────────────┘
                   │
                   ▼
          ┌──────────────────┐
          │ Python Analytics │
          │ Pandas / SQL     │
          │ Validation       │
          └────────┬─────────┘
                   │
                   ▼
          ┌──────────────────┐
          │ SQL Views /      │
          │ Aggregations     │
          └──────────────────┘
```

The repository also contains notebooks, visualization utilities, exported analysis artifacts, and supporting documentation.

---

# Repository Structure

A high-level view of the repository:

```text
.
├── Readme.MD
├── CONTRIBUTING.md
├── WORKFLOW.md
├── requirements.txt
│
├── analysis_narrative.md
├── executive_summary.md
├── technical_analysis.md
├── discrepancy_analysis.md
├── dashboard_design.md
├── data_layer_conventions.md
├── export_documentation.md
├── feedback_and_edits.md
├── audience_versions_A_B_C.md
│
├── dashboard_app.py
├── kpi_dashboard.py
├── streamlit_app.py
├── validation_script.py
├── assignment-33-python.py
├── assignment-35-visualizations.py
│
├── assignments/
│   ├── assignment_2_50_insight_export.md
│   ├── assignment_2_51_streamlit_app_structure.md
│   ├── assignment_2_52_dataset_upload_preview.md
│   ├── assignment_2_53_streamlit_filters_widgets.md
│   ├── assignment_2_54_session_state_persistence.md
│   ├── assignment_2_55_realtime_kpi_dashboard.md
│   ├── assignment_2_56_alert_monitoring_thresholds.md
│   ├── assignment_2_57_insight_email_reports.md
│   ├── assignment_2_58_automated_data_pipeline.md
│   ├── assignment_2_59_github_workflow_validation.md
│   └── assignment_2_60_data_product_documentation.md
│
├── data/
│   ├── raw/
│   └── processed/
│
├── database/
│   ├── aggregations/
│   │   └── agg_daily_metrics.sql
│   └── views/
│       ├── vw_active_customers.sql
│       └── vw_product_performance.sql
│
├── docs/
│   ├── DATA_DICTIONARY.md
│   └── data_dictionary.csv
│
├── interactive_charts/
│   ├── generate_interactive_charts.py
│   └── task5_date_range_answer.md
│
├── kpis/
│   ├── kpi_functions.py
│   ├── kpi_reference.md
│   └── kpi_validation_targets.json
│
├── notebooks/
│   ├── funnel_analysis.ipynb
│   └── time_series_analysis.ipynb
│
├── output/
│   ├── charts
│   ├── reports
│   ├── validation artifacts
│   └── analytical JSON/TXT outputs
│
├── output_exports/
│   └── 2026-08-13_202955_analysis/
│
└── payfriction-backend/
    ├── docker-compose.yml
    ├── package.json
    ├── prisma.config.ts
    ├── prisma/
    │   ├── schema.prisma
    │   ├── seed.ts
    │   └── migrations/
    ├── src/
    │   ├── analytics/
    │   ├── orders/
    │   └── prisma/
    └── test/
```

The repository also includes Prisma CLI/client reference documentation under `payfriction-backend/.agents/skills/`.

---

# Technology Stack

## Backend

- Node.js
- TypeScript
- NestJS
- Prisma ORM
- PostgreSQL
- `pg`
- Jest
- Supertest
- ESLint
- Prettier

## Analytics

- Python
- Pandas
- NumPy
- SQLAlchemy
- Scikit-learn
- SciPy

## Visualization

- Streamlit
- Matplotlib
- Plotly
- Seaborn
- Recharts is specified in the product PRD for the planned application analytics stack.

## Infrastructure

- Docker
- Docker Compose
- PostgreSQL container
- GitHub Actions is specified in the product requirements for CI/CD infrastructure.

## Development

- Git
- GitHub
- Conventional commits
- Pull requests
- Branch-based workflow

---

# Backend

The backend is located in:

```text
payfriction-backend/
```

It is implemented with NestJS and TypeScript.

## Application Modules

The application currently includes:

- `OrdersModule`
- `PrismaModule`
- `ConfigModule`

The analytics controller/service files are also present in the repository.

## Application Entry Point

The server is bootstrapped from:

```text
payfriction-backend/src/main.ts
```

The current implementation:

- creates the NestJS application,
- enables CORS,
- listens on port `3000`,
- binds to `127.0.0.1`.

For local development:

```bash
cd payfriction-backend
npm install
npm run start:dev
```

---

# Database Model

The Prisma schema uses PostgreSQL.

Core entities include:

```text
Merchant
Customer
PaymentGateway
BankResponseCode
Transaction
PaymentAttempt
```

## Entity Relationships

```text
Merchant
   │
   └───< Transaction >─── Customer
              │
              └───< PaymentAttempt
                         │
                         ├── PaymentGateway
                         │
                         └── BankResponseCode
```

## Merchant

Represents the business using the payment system.

Fields include:

- `id`
- `name`
- `industry`
- `createdAt`

A merchant can have multiple transactions.

## Customer

Represents a customer associated with transactions.

Fields include:

- `id`
- `email`
- `customerSegment`
- `createdAt`

Customer segments include examples such as:

- Enterprise
- SMB
- Startup

## PaymentGateway

Represents a payment gateway.

Fields:

- `id`
- `name`

The seed data includes:

- Stripe
- Razorpay

## BankResponseCode

Stores standardized bank/payment response information.

Fields:

- `id`
- `code`
- `meaning`
- `classification`
- `actionRequired`

## Transaction

Represents a payment transaction.

Fields include:

- `id`
- `amount`
- `currency`
- `status`
- `createdAt`
- `updatedAt`
- `merchantId`
- `customerId`

Transaction status values documented in the schema include:

```text
PENDING
SUCCESS
FAILED
```

## PaymentAttempt

Represents an initial payment attempt or retry.

Fields include:

- `id`
- `attemptNumber`
- `isSuccess`
- `processedAt`
- `transactionId`
- `gatewayId`
- `responseCodeId`

`attemptNumber = 1` represents the initial attempt, while values greater than `1` represent retries.

---

# Analytics API

The analytics controller is exposed under:

```text
/analytics
```

## Dashboard Summary

```http
GET /analytics/summary
```

Returns summary information including:

- revenue at risk from failed transactions,
- number of failed transactions.

## Gateway Performance

```http
GET /analytics/gateways
```

Returns gateway-level:

- total attempts,
- successful attempts,
- failed attempts,
- success rate.

## Error Breakdown

```http
GET /analytics/errors
```

Returns response-code information including:

- code,
- meaning,
- classification,
- recommended action,
- occurrence count.

Results are ordered by occurrence count.

## Retry Metrics

```http
GET /analytics/retries
```

Returns:

- total retries,
- successful retries,
- retry recovery rate.

Retries are identified as payment attempts with:

```text
attemptNumber > 1
```

---

# Transaction API

Transaction endpoints are exposed under:

```text
/api/orders
```

## Get Transactions

```http
GET /api/orders
```

Returns transactions with related:

- customer,
- merchant,
- payment attempts,
- gateway,
- bank response code.

Transactions are returned newest first.

## Dashboard Metrics

```http
GET /api/orders/metrics
```

Returns:

- successful transaction revenue,
- total customers,
- total transactions,
- failed payment attempts.

---

# Python Analytics Layer

The Python portion of the repository supports:

- data preparation,
- KPI computation,
- SQL analysis,
- validation,
- visualization,
- Streamlit dashboards,
- exports,
- reporting.

The dependency set is defined in:

```text
requirements.txt
```

Important packages include:

```text
pandas
numpy
sqlalchemy
streamlit
matplotlib
scikit-learn
scipy
seaborn
jupyter
openpyxl
```

---

# Data Pipeline

The repository defines a staged analytical pipeline.

```text
Raw Data
   │
   ▼
Ingestion
   │
   ▼
Validation
   │
   ▼
Cleaning
   │
   ▼
Aggregation
   │
   ▼
Dashboard / Analytics
   │
   ▼
Alerts
   │
   ▼
Reports / Delivery
```

## Stage 1 — Ingestion

Raw CSV data is loaded into a Pandas DataFrame.

The documented workflow uses an ingestion function that:

- accepts a CSV path,
- reads the raw data,
- does not perform transformations,
- raises an error for missing or empty input.

## Stage 2 — Validation

Validation checks include:

- schema,
- data types,
- non-null requirements,
- minimum row counts.

The repository documents validation as a CI/CD quality gate.

## Stage 3 — Cleaning

Documented cleaning operations include:

- removing duplicate rows,
- removing records missing mandatory identifiers,
- handling null numeric values,
- filtering invalid/negative transactions,
- coercing datetime values,
- applying business filters.

## Stage 4 — Aggregation

The analytics layer computes aggregated metrics such as:

- revenue,
- transaction counts,
- customer activity,
- product performance,
- segment-level revenue.

Processed outputs are written to the processed/output layers.

## Stage 5 — Dashboard

Streamlit applications render:

- KPI cards,
- trends,
- segment breakdowns,
- filters,
- transaction tables,
- exports.

## Stage 6 — Alerting

The repository documents threshold-based alerts using Streamlit warning/error states.

## Stage 7 — Delivery

The documented delivery layer can create structured summaries containing:

1. KPIs
2. Findings
3. Actions

The repository also contains documentation for email/report delivery.

---

# Dashboard Layer

The repository contains multiple dashboard implementations for different analytical tasks.

## PayFriction Dashboard

`dashboard_app.py` implements a Streamlit payment-friction dashboard using generated transaction data.

The interface is organized into four levels.

### Level 1 — Executive Status

Five KPI cards are displayed:

- Total Processed Volume
- Payment Success Rate
- Revenue at Risk
- Recovered via Retry
- Average Recovery Time

### Level 2 — Trends

The dashboard includes:

- monthly revenue-at-risk trend,
- weekly successful vs failed transaction volume.

### Level 3 — Segments

Revenue leakage is broken down by:

- Enterprise
- SMB
- Startup

### Level 4 — Transaction Explorer

The detailed explorer provides:

- customer-segment filtering,
- transaction-status filtering,
- filtered record counts,
- transaction table,
- CSV download.

---

# Interactive Sales Dashboard

`streamlit_app.py` demonstrates a separate interactive Streamlit dashboard for analytical exploration.

It provides:

- minimum order amount filter,
- product filter,
- date range filter,
- total revenue KPI,
- total orders KPI,
- average order value,
- unique customers.

It also includes Plotly charts with:

- hover interaction,
- zoom,
- pan,
- reset,
- date range selection.

---

# KPI Dashboard

`kpi_dashboard.py` provides KPI computation and display logic.

The documented KPI set includes:

1. Revenue
2. Active Users
3. Average Order Value
4. Churn Rate
5. Customer Satisfaction

Each KPI is calculated for:

- current period,
- previous period.

Percentage change is then calculated.

## Directional Logic

Metrics are classified according to whether an increase or decrease is favorable.

For example:

- Revenue increasing → positive
- Satisfaction increasing → positive
- Churn decreasing → positive

The KPI dashboard uses a threshold of approximately `2%` to distinguish significant movement from stable values.

---

# SQL Data Layer

The repository defines conventions for reusable SQL objects.

## Views

Views use the prefix:

```text
vw_
```

Examples:

```text
vw_active_customers
vw_product_performance
```

Views are intended to encapsulate reusable business logic and provide a consistent source of metrics.

### `vw_active_customers`

The documented view provides information related to:

- customer activity,
- revenue,
- recent ordering behavior,
- segment analysis.

### `vw_product_performance`

The documented view aggregates:

- product,
- category,
- units sold,
- lifetime revenue.

---

# Pre-Aggregated Tables

Pre-aggregated analytical tables use:

```text
agg_
```

The repository includes:

```text
agg_daily_metrics
```

This table is designed to store pre-computed daily metrics.

Standard aggregate columns include:

- `aggregation_date`
- `metric_name`
- `metric_value`
- `row_count`
- `updated_at`

## Why Pre-Aggregation?

Pre-aggregation reduces the need for dashboards to repeatedly scan large transactional tables.

Benefits documented in the repository include:

- reduced dashboard latency,
- consistent business logic,
- easier maintenance,
- clearer data-layer ownership.

---

# KPI Layer

The `kpis/` directory contains:

```text
kpi_functions.py
kpi_reference.md
kpi_validation_targets.json
```

This layer centralizes KPI definitions and validation expectations.

The repository emphasizes that KPI logic should be consistent across:

- SQL,
- Python,
- dashboards,
- reports.

This is intended to prevent metric drift.

---

# Metric Validation and Drift Detection

One of the important engineering aspects of the repository is cross-validation between SQL and Python calculations.

The main validation script is:

```text
validation_script.py
```

It compares metrics calculated using:

- SQL,
- Pandas/Python.

## Metrics Tested

The validation workflow includes:

- Active Users over a 30-day window
- Average Order Value
- Monthly Customer Churn

## Example of the Drift Problem

The repository demonstrates a churn calculation bug caused by comparing only the numerical month:

```sql
strftime('%m', order_date)
```

This discards the year.

For example, July 2025 and July 2026 both produce:

```text
07
```

That can incorrectly classify historical customers as belonging to the current comparison period.

## Correct Approach

The fixed SQL uses explicit calendar boundaries:

```sql
WHERE order_date >= date('now', 'start of month', '-1 month')
  AND order_date < date('now', 'start of month')
```

This preserves the year/month relationship.

## Validation Principle

A validation system can detect that two calculations differ, but it cannot automatically determine which implementation is correct.

Therefore:

```text
Metric mismatch
      │
      ▼
Automated validation
      │
      ▼
FAIL / discrepancy
      │
      ▼
Manual investigation
      │
      ▼
Root-cause analysis
      │
      ▼
Corrected business logic
      │
      ▼
Re-validation
```

The repository records a corrected result where SQL and Python both return the same churn count after the calendar-boundary fix.

---

# Visualization Layer

The repository includes:

```text
assignment-35-visualizations.py
interactive_charts/
```

The visualization assignment generates five chart types.

## 1. Revenue by Product

Horizontal bar chart showing revenue by product line.

## 2. Revenue Trend

Line chart showing monthly revenue trends for the top products.

## 3. Order Value Distribution

Histogram showing the distribution of order values.

The analysis also displays:

- mean,
- median.

## 4. Quarterly Revenue Composition

Stacked bar chart showing the contribution of product lines to quarterly revenue.

## 5. Marketing Spend vs Revenue

Scatter plot comparing monthly marketing spend with revenue.

A trend line is included for relationship analysis.

Generated chart files are documented as PNG outputs in the `output/` directory.

---

# Notebooks

The repository contains Jupyter notebooks for analytical exploration.

```text
notebooks/
├── funnel_analysis.ipynb
└── time_series_analysis.ipynb
```

These notebooks support exploratory and time-series analysis outside the production dashboard layer.

---

# Sample Data and Seed Data

The repository uses simulated/sample data in several analytical applications.

The PayFriction dashboard generates synthetic transactions including:

- date,
- customer segment,
- amount,
- transaction status,
- transaction ID.

The backend also includes a Prisma seed script:

```text
payfriction-backend/prisma/seed.ts
```

The seed creates example:

- payment gateways,
- bank response codes,
- merchant,
- customer,
- successful transaction,
- failed transaction due to insufficient funds,
- failed transaction due to expired card.

---

# Local Setup

## Prerequisites

Recommended tools:

- Python 3.x
- Node.js
- npm
- Docker
- Docker Compose
- PostgreSQL, if running without the provided container

---

# Python Setup

From the repository root:

```bash
python -m venv .venv
```

### Windows

```bash
.venv\Scripts\activate
```

### macOS / Linux

```bash
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

---

# Backend Setup

```bash
cd payfriction-backend
npm install
```

The backend package provides the following important commands:

```bash
npm run build
npm run start
npm run start:dev
npm run start:prod
npm run lint
npm test
npm run test:watch
npm run test:cov
npm run test:e2e
```

---

# Database Setup

The backend provides a Docker Compose PostgreSQL service.

From:

```text
payfriction-backend/
```

run:

```bash
docker compose up -d
```

The supplied Compose configuration exposes PostgreSQL on:

```text
localhost:5433
```

The container uses:

```text
Database: payfriction
User: admin
Password: adminpassword
```

> These credentials are development/demo configuration from the repository. Do not reuse them for production.

---

# Environment Variables

The Prisma seed and backend use:

```text
DATABASE_URL
```

Set it according to your PostgreSQL environment.

For example, with the supplied Docker Compose configuration, the application connection should target the PostgreSQL container/port appropriate to where the application is running.

Do not commit production secrets to source control.

---

# Prisma Workflow

The backend uses Prisma for database access and schema management.

Useful commands include:

```bash
npx prisma generate
npx prisma validate
npx prisma migrate dev
npx prisma migrate deploy
npx prisma studio
npx prisma db seed
```

Typical local workflow:

```bash
cd payfriction-backend

npm install

npx prisma generate
npx prisma migrate dev
npx prisma db seed

npm run start:dev
```

The exact migration command should be selected according to whether the database is a development or deployment environment.

---

# Running the Backend

Start the development server:

```bash
cd payfriction-backend
npm run start:dev
```

The current application listens on:

```text
http://127.0.0.1:3000
```

Example endpoints:

```text
GET /api/orders
GET /api/orders/metrics

GET /analytics/summary
GET /analytics/gateways
GET /analytics/errors
GET /analytics/retries
```

---

# Running the Streamlit Dashboards

## PayFriction Dashboard

```bash
streamlit run dashboard_app.py
```

## Interactive Sales Dashboard

```bash
streamlit run streamlit_app.py
```

## KPI Dashboard

```bash
streamlit run kpi_dashboard.py
```

The dashboards in the repository use generated or analytical data depending on the individual application.

---

# Running Analytics Scripts

## Data Workflow

The documented production-style workflow is:

```bash
python scripts/data_workflow.py
```

The workflow performs:

```text
Ingest
  ↓
Process
  ↓
Output
```

The documented processing includes:

- duplicate removal,
- null handling,
- amount filtering,
- retry-count filtering,
- friction categorization.

The documented output is:

```text
output/processed.csv
```

Execution logs are documented under:

```text
logs/workflow.log
```

## Validation

Run:

```bash
python validation_script.py
```

The script creates:

```text
validation_report.csv
output/validation_report.csv
```

---

# Database Analytics Assignment

`assignment-33-python.py` demonstrates an SQL/data-layer workflow that:

1. Initializes or seeds an SQLite analytical database when required.
2. Creates SQL views.
3. Creates a pre-aggregated table.
4. Populates daily metrics.
5. Queries the views.
6. Queries the aggregate table.
7. Demonstrates segmentation queries.
8. Benchmarks an aggregate query.

The example database file is:

```text
optimization.db
```

---

# Data Quality and Validation

The repository treats data quality as a first-class concern.

Validation concepts include:

- schema validation,
- data type validation,
- non-null validation,
- minimum row-count validation,
- duplicate detection,
- missing-value handling,
- metric reconciliation,
- SQL/Python cross-checking.

A particularly important principle is:

> Passing a numerical tolerance check does not prove that the business logic is correct.

A discrepancy should therefore be investigated rather than automatically overwritten.

---

# Data Dictionary

Documentation is available under:

```text
docs/
├── DATA_DICTIONARY.md
└── data_dictionary.csv
```

The data dictionary provides a central reference for analytical fields and their intended meaning.

---

# Business Definitions

The repository's analytical model distinguishes several important concepts.

## Revenue at Risk

Revenue associated with failed transactions that may potentially be recovered.

## Revenue Recovered

Revenue successfully recovered after payment retry/recovery activity.

## Permanent Revenue Loss

Revenue associated with failures classified as permanent and not expected to be recovered through another retry.

## Retry Success Rate

The proportion of retry attempts that result in successful payment.

## Payment Friction

A temporary payment-processing problem that may be resolved through another attempt or operational action.

## Permanent Failure

A failure for which additional retries are generally not appropriate, such as an expired card or customer cancellation.

---

# Reporting and Exports

The product requirements describe report generation for:

- daily reports,
- weekly reports,
- monthly reports,
- revenue leakage reports,
- retry effectiveness reports,
- bank performance reports.

Supported export formats specified by the product definition include:

- CSV
- Excel
- PDF

The repository also contains exported analytical artifacts under:

```text
output_exports/
```

---

# Performance and Non-Functional Requirements

The product requirements define the following targets.

## Performance

- Dashboard load: under 2 seconds
- Analytics queries: within 5 seconds
- Dataset scale: over 10 million transaction records

## Availability

Target:

```text
99.9% uptime
```

## Scalability

The product definition calls for:

- horizontal scaling,
- containerized deployment,
- cloud-native architecture.

These are product requirements/targets; they should not be interpreted as proof that every target is already achieved by the current implementation.

---

# Security Requirements

The product requirements specify:

- Role-Based Access Control (RBAC)
- Audit logging
- Encrypted data storage
- Secure authentication

Production deployments should also keep credentials and database connection strings outside source control.

---

# Business Analysis Included in the Repository

The repository contains an executive/customer-churn analysis alongside the payment-friction product work.

The analysis examines:

- customer churn,
- support response time,
- customer segments,
- recurring revenue impact,
- operational recommendations.

The included narrative analyzes 50,000 customer records across a 24-month period and reports a relationship between support response time and churn.

The repository also contains:

- executive summary,
- audience-specific versions,
- supporting analysis,
- discrepancy investigation,
- reporting/export documentation.

These documents represent analytical/reporting artifacts contained in the repository and should be treated separately from the core payment analytics application's runtime behavior.

---

# Audience-Specific Communication

`audience_versions_A_B_C.md` documents how the same analytical findings can be framed differently for:

- Board/Executive stakeholders
- Operations
- Support teams
- Engineering leadership

The principle is to keep the underlying findings consistent while changing:

- level of technical detail,
- business framing,
- operational focus,
- requested action.

---

# Development Workflow

The repository uses a GitHub-oriented workflow.

## Branching

Keep the main branch releasable.

Create a short-lived branch for each task:

```text
feature/<issue>-<description>
fix/<issue>-<description>
docs/<issue>-<description>
chore/<issue>-<description>
```

Example:

```bash
git checkout main
git pull origin main
git checkout -b feature/123-churn-model
```

## Issues

Every change should begin with a GitHub issue containing:

- clear title,
- problem/request description,
- acceptance criteria,
- appropriate labels,
- assignee.

## Pull Requests

A completed branch should:

1. Be pushed to GitHub.
2. Open a pull request.
3. Link the relevant issue.
4. Request at least one review.
5. Address feedback.
6. Merge after approval.

---

# Commit Convention

The repository recommends conventional commits.

Format:

```text
type: short summary
```

Examples:

```text
feat: add churn model training pipeline
fix: correct null handling in data profiler
docs: update onboarding steps for analysts
refactor: extract shared preprocessing logic
test: add validation for payment metrics
chore: update dependencies
```

Common types:

| Type | Purpose |
|---|---|
| `feat` | New functionality |
| `fix` | Bug fix |
| `docs` | Documentation |
| `refactor` | Code restructuring |
| `test` | Tests/validation |
| `chore` | Maintenance |

---

# Testing

## Backend Unit Tests

Run:

```bash
cd payfriction-backend
npm test
```

Coverage:

```bash
npm run test:cov
```

Watch mode:

```bash
npm run test:watch
```

## End-to-End Tests

Run:

```bash
npm run test:e2e
```

The repository includes tests for:

- application controller,
- analytics controller,
- analytics service,
- orders controller,
- orders service,
- Prisma service,
- application-level E2E behavior.

## Analytical Validation

Run:

```bash
python validation_script.py
```

This validates that SQL and Python calculations remain aligned for the tested metrics.

---

# Design Principles

## 1. Single Source of Truth

Business metrics should be defined once and reused across dashboards and reports.

## 2. Reproducibility

The same analytical workflow should be executable consistently across environments.

## 3. Progressive Disclosure

Executives see high-level KPIs first, while detailed transaction data remains available for deeper investigation.

## 4. Metric Consistency

SQL and Python implementations should be reconciled through validation.

## 5. Separation of Concerns

The repository separates:

- ingestion,
- cleaning,
- transformation,
- aggregation,
- visualization,
- validation,
- reporting.

## 6. Root-Cause Analysis

A failed validation should lead to investigation of the underlying logic rather than simply forcing two outputs to match.

---

# Known Implementation Notes

This repository contains both product requirements and implemented analytical/application code. They should be distinguished.

### Product-level requirements include

- Next.js frontend
- React
- TypeScript
- Tailwind CSS
- Recharts
- TanStack Query
- NestJS
- PostgreSQL
- Prisma
- Redis
- BullMQ
- Docker
- AWS
- NGINX
- GitHub Actions

### Current repository implementation visibly includes

- NestJS backend
- Prisma/PostgreSQL data model
- Streamlit dashboards
- Python/Pandas analytics
- SQL views and aggregation tables
- validation scripts
- Jupyter notebooks
- visualization scripts
- Docker Compose PostgreSQL setup
- Jest tests

The repository export does not show a complete Next.js frontend implementation. The Next.js stack should therefore be treated as part of the documented product architecture rather than assumed to be fully implemented in the supplied source export.

Similarly, the non-functional requirements such as 99.9% uptime and sub-2-second dashboard loading are targets defined by the product requirements, not measured production guarantees.

---

# Future Enhancements

The product requirements identify several future directions:

- AI-powered payment failure prediction
- Intelligent retry recommendation engine
- Payment gateway health monitoring
- Predictive revenue forecasting
- Customer payment behavior analysis
- Real-time anomaly detection
- Multi-currency analytics
- Executive AI-generated insights and summaries
- ERP integrations
- Accounting integrations
- BI platform integrations

---

# Suggested Production Architecture

A production deployment can evolve toward:

```text
                    ┌─────────────────────┐
                    │ Web / BI / Streamlit│
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ API / NestJS        │
                    │ Authentication      │
                    │ RBAC                │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
        PostgreSQL           Redis          Background
        Transactions         Cache           Jobs/BullMQ
              │                │                │
              └────────────────┼────────────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Analytics Layer     │
                    │ SQL / Python        │
                    │ Pandas              │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Reports / Alerts    │
                    │ CSV / Excel / PDF   │
                    └─────────────────────┘
```

This diagram represents an architectural direction based on the repository's product requirements and technology choices; it is not a claim that every component is currently deployed.

---

# Quick Start

For a local development setup:

```bash
# 1. Create Python environment
python -m venv .venv

# 2. Activate it
# Windows:
.venv\Scripts\activate

# macOS/Linux:
source .venv/bin/activate

# 3. Install Python dependencies
pip install -r requirements.txt

# 4. Start PostgreSQL
cd payfriction-backend
docker compose up -d

# 5. Install backend dependencies
npm install

# 6. Generate Prisma client
npx prisma generate

# 7. Apply development migrations
npx prisma migrate dev

# 8. Seed sample data
npx prisma db seed

# 9. Start backend
npm run start:dev
```

In another terminal, from the repository root:

```bash
streamlit run dashboard_app.py
```

Or:

```bash
streamlit run kpi_dashboard.py
```

Or:

```bash
streamlit run streamlit_app.py
```

---

# Project Summary

PayFriction Analytics is a payment analytics and data-product repository focused on making payment failures measurable, explainable, and actionable.

Its central analytical workflow is:

```text
Payment Data
     ↓
Validation
     ↓
Cleaning
     ↓
Classification
     ↓
Aggregation
     ↓
KPI Calculation
     ↓
Dashboard
     ↓
Investigation
     ↓
Reporting
```

The backend provides structured payment entities and REST endpoints. The Python layer provides analytical processing and validation. SQL views and aggregate tables centralize business logic. Streamlit dashboards expose KPIs and transaction-level analysis. Validation tooling helps detect differences between SQL and Python implementations.

The repository therefore combines application development, database engineering, analytics engineering, data visualization, validation, and business reporting into a single payment-friction analytics project.

---

# License

The product requirements and repository documentation define the intended product and technical architecture. Check the repository's authoritative license file before redistributing or deploying the project.

