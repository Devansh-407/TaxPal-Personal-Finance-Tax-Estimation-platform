# TaxPal – Personal Finance & Tax Estimator for Freelancers

TaxPal is a full-stack MERN web application that lets freelancers and gig workers track income and expenses, set monthly budgets, estimate slab-based tax for five countries, follow advance-tax deadlines, and generate financial reports for tax filing.

## Problem Statement

Freelancers and gig workers earn irregular income from several clients and platforms, yet most track it manually in spreadsheets. This is error-prone and gives little insight into spending. Tax estimation is harder still: slab rates, deductions, cess and quarterly advance-tax deadlines differ by country, so users may overpay, underpay or miss deadlines. TaxPal combines tracking, budgeting, tax estimation and reporting in one platform.

## Features

**Authentication**
- Registration with name, email, country and income bracket
- Email/password and Google sign-in via Firebase
- Firebase ID-token verification on all protected APIs

**Transactions & Categories**
- Add and delete income and expense transactions (amount, category, date, description, notes)
- Filter by type, search by description or category
- Default categories plus custom categories with colors

**Budgets & Dashboard**
- Monthly budget per category with automatic spent-vs-budget calculation
- Income vs expense monthly bar chart
- Expense breakdown pie chart with percentages

**Tax Estimation**
- Countries: India, United States, United Kingdom, Canada, Australia
- Slab-based calculation for FY 2025–26 with quarter selection (Q1–Q4 and Annual)
- Country-specific deduction inputs
- One saved estimate per user, country, year and quarter
- Tax history with payment status (pending / paid)
- Compliance calendar with deadline status and a reminder when a deadline is within 15 days

**Reports**
- Income Statement, Tax Summary and Expense Report
- Period and format selection (PDF / CSV / TXT)
- Preview, print, download and delete, with report history

## Tech Stack

| Layer | Technologies |
|---|---|
| Frontend | React.js, React Router, JavaScript, CSS, Axios |
| Visualization | Recharts |
| Backend | Node.js, Express.js, REST APIs, CORS, dotenv |
| Authentication | Firebase Authentication, Firebase Admin SDK |
| Database | MongoDB with Mongoose |
| Tax Engine | Country-wise calculator functions |
| Deployment | Render / Vercel / Azure / Oracle Cloud |

## System Architecture

The user logs in through Firebase. The React frontend sends the ID token with each API request. Express middleware verifies it and routes the request to the Transaction & Budget, Tax Estimation or Report module. MongoDB data is validated and returned as charts, estimates or reports.

```
USER (Web Browser)
        │
FRONTEND  – React.js, React Router, Axios, Recharts
        │
BACKEND   – Node.js, Express.js, REST APIs, Firebase Token Middleware
        │
BUSINESS LOGIC – Transaction + Budget + Category + Tax Engine + Report Controllers
        │
   ┌────┴─────────────┬──────────────┬─────────────────┐
Firebase Auth   MongoDB/Mongoose   Tax Engine     Deployment & VCS
```

## Data Design

| Entity | Attributes |
|---|---|
| User | firebaseId, name, email, country, income_bracket, createdAt |
| Transaction | _id, firebaseId, type, category, amount, description, date, notes |
| Category | _id, name, type (income/expense), color, firebaseId (`GLOBAL` for defaults) |
| Budget | _id, firebaseId, category, type, amount, month, description |
| Tax Estimate | _id, firebaseId, country, year, quarter, income, deductions, estimatedTax, status |
| Report | _id, firebaseId, reportName, reportType, period, format, status, dataSnapshot, fileUrl |

## Tax Estimation Logic

```
Taxable Income       = max(0, Gross Income − Standard Deduction)
Progressive Slab Tax = Σ rᵢ × max(0, min(T, Uᵢ) − Lᵢ)
Effective Tax Rate   = (Estimated Tax / Income) × 100
```

`T` = taxable income; `Lᵢ`, `Uᵢ` = lower/upper limit of slab *i*; `rᵢ` = slab rate.

- Standard deduction / allowance: India ₹75,000 · United States $15,000 · United Kingdom £12,570
- **India:** Total Tax = Tax × (1 + 0.04) (4% Health & Education Cess)
- **Australia:** Total Tax = Tax + 0.02 × Income (2% Medicare Levy)

Slab values reflect FY 2025–26 and must be updated manually when tax laws change.
