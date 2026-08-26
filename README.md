# RecoverAI - Revenue Recovery Intelligence

> Recover the right payment. At the right time. With the right strategy.

RecoverAI is an adaptive revenue recovery platform designed to help merchants make better decisions when payments fail. Instead of blindly retrying every failed transaction, it analyzes failure context, estimates recovery probability and expected recovery value, compares recovery strategies, applies merchant policies, and recommends the next action.

```text
Failed Payment -> Understand -> Evaluate -> Decide -> Recover -> Measure
```

This is a portfolio and demonstration application. Transactions and recovery outcomes are synthetic, payment execution is simulated, and no real money is transferred.

## Why RecoverAI?

A temporary timeout, insufficient funds, invalid payment details, repeated failure, and potentially permanent failure should not receive the same retry strategy. RecoverAI makes the reasoning behind the next action visible to an operator.

- Why did the payment fail?
- Is it recoverable?
- How valuable is the opportunity?
- What action should be taken and when?
- Should the payment be retried at all?
- What would happen under another recovery strategy?

## Core Decision Flow

```text
Payment Failed
      |
      v
Understand Failure
      |
      v
Determine Recoverability
      |
      v
Estimate Recovery Probability
      |
      v
Calculate Expected Recovery Value
      |
      v
Compare Recovery Strategies
      |
      v
Apply Merchant Policies
      |
      v
Select Best Action
      |
      v
Simulate Recovery Safely
      |
      v
Track Outcome
      |
      v
Measure Recovery Performance
```

## What Makes RecoverAI Different?

RecoverAI is a recovery **decision engine**, not simply a failed-payment dashboard or automatic retry system. It combines failure intelligence, deterministic recovery probability, expected recovery value, strategy selection, timing recommendations, payment-method considerations, merchant policies, explainable decisions, counterfactual analysis, recovery simulation, audit events, and revenue leakage signals.

The scoring engine is transparent and deterministic. It is not a trained machine-learning model and does not claim production prediction accuracy.

## Key Features

### Recovery Decision Engine

`src/lib/aiEngine.js` classifies failure reasons, estimates recovery probability and confidence, recommends timing, calculates opportunity scores, ranks strategies, and creates explanations from transaction, customer, and settings data.

### Expected Recovery Value

```text
Expected Recovery Value = Transaction Amount x Recovery Probability - Estimated Recovery Cost
```

This is an application decision metric for prioritization and comparison, not a claim of real-world financial prediction accuracy.

### Recovery Strategy Selection

The engine can recommend retry now, retry later, changing the payment method, sending a payment link, sending a reminder, requesting customer action, escalating, or not retrying. `src/lib/recovery.js` implements retry, scheduling, in-app reminder, alternative-method, escalation, closure, approval, and manual-override paths.

Payment-link and do-not-retry labels are represented by the decision engine; they do not imply an external payment-link provider or separate payment integration.

### Do-Not-Retry Intelligence

Low recovery opportunity, permanent failure classifications, retry limits, and policy conditions can lead to a recommendation to avoid another attempt. This is explainable rule-based scoring, not a trained ML model.

### Explainable and Counterfactual Decisions

Recovery cases expose failure reason and category, transaction amount, attempts, probability, confidence, expected value, timing, policy state, recommendation, and alternative strategy comparisons. The simulator compares expected probability, value, and risk across strategies without inventing production results.

### Recovery Command Center

The Dashboard dynamically calculates revenue at risk, recovered revenue, revenue lost, recoverable revenue, expected recovery, recovery rate, recovery health, recovery opportunities, failure-reason breakdowns, payment-method performance, trends, and generated insights from stored synthetic records.

### Recovery Queue and Operations

The Recovery Queue and case detail views support operational review of failed transactions, recovery cases, policy decisions, manual overrides, alternative payment methods, simulated retries, scheduled retries, escalation, closure, notifications, and audit history.

### Failure Intelligence and Revenue Leakage

The analytics layer provides failure fingerprints, failure-reason breakdowns, payment-method performance, recovery funnel data, trends, recovery health, and revenue leakage signals.

### Strategy and Synthetic Payment Simulators

`src/pages/Simulator.jsx` provides synthetic payment scenarios and strategy comparisons using the same decision libraries. Recovery actions produce deterministic simulated outcomes; no payment gateway is contacted.

### Merchant Policies and Rules

Recovery settings include retry limits, retry timing, minimum probability, automatic-retry preference, escalation threshold, notification preferences, allowed payment methods, strategy preference, and operating costs. `RecoveryRules.jsx` supports creating, editing, enabling, disabling, and deleting rules. Rules are stored locally and are not automatically evaluated by a background worker.

### Notifications and Audit History

Recovery actions create browser-local in-app notifications and audit records. Events include risk detection, policy decisions, action execution or failure, case resolution, escalation, scheduling, settings changes, rule changes, and notifications.

### Safety and Reliability Controls

Implemented controls include retry limits, high-value approval gates, terminal-case protection, local idempotency checks, manual overrides, signed sessions, protected routes, rate limiting, hashed expiring email tokens, and audit events. These controls support demonstration use and are not a production payment authorization boundary.

## Dynamic Data and Storage

Authentication is handled by the Express API and SQLite. Operational records are handled by `src/api/localDataClient.js` and stored in browser `localStorage`, scoped by workspace or demo scope.

```text
SQLite via backend/db.js
  users, workspaces, workspace_members, email_tokens

Browser localStorage via src/api/localDataClient.js
  Customer, Transaction, RecoveryCase, RecoveryAttempt,
  RecoveryRule, Settings, Notification, AIInsight, AuditLog

Synthetic seed data
  src/lib/seeData.js -> src/hooks/useSeedData.js
```

The demo workspace uses the `demo-workspace` storage scope. Client-side scope isolation is useful for this prototype but is not server-enforced multi-tenant authorization.

## Authentication and Email

The backend implements email/password registration, login, logout, email verification, verification resend, password reset, protected session lookup, workspace creation, and workspace membership.

- Passwords are hashed with `bcryptjs`.
- Verification and reset tokens are SHA-256 hashed, expire after one hour, and are single-use.
- Sessions use seven-day JWTs in HTTP-only `SameSite=Lax` cookies.
- Production cookies enable the `secure` flag.
- Authentication requests are rate-limited to 30 requests per 15 minutes.
- Nodemailer can send verification and reset messages when SMTP is configured.
- Without SMTP configuration, development messages are written to `data/email-outbox`.
- Recovery reminders create in-app notifications; they are not connected to external email or SMS providers.

Google login, real payment-link delivery, and production email delivery are not implemented.

## Architecture

```text
React pages and shared UI
          |
          v
src/api/localDataClient.js
          |
          +--> src/lib/aiEngine.js  deterministic scoring
          +--> src/lib/recovery.js  orchestration and actions
          +--> src/lib/analytics.js metrics and insights
          +--> browser localStorage  operational data
          |
          +--> /api/auth/* via Vite proxy
                    |
                    v
              backend/server.js
                    |
                    v
              SQLite via backend/db.js
```

## Technology Stack

| Area | Technology |
| --- | --- |
| Frontend | React 18, React Router 6 |
| Build tool | Vite |
| Backend | Node.js, Express 5 |
| Database | SQLite via `better-sqlite3` |
| Styling | Tailwind CSS, PostCSS |
| UI components | Radix UI, Lucide React |
| Charts | Recharts |
| Authentication | JWT, `bcryptjs`, `cookie-parser`, CORS, `express-rate-limit` |
| Email | Nodemailer |
| Testing | Node.js test runner |
| Code quality | ESLint, TypeScript type checking |

## Project Structure

```text
backend/
  server.js              Express API and health endpoint
  authRoutes.js          Registration, login, verification, and reset routes
  db.js                  SQLite connection and schema
  emailService.js        SMTP and development email handling

src/
  App.jsx                Application routes and providers
  api/localDataClient.js Browser-local entities and auth client
  components/            Layout, auth, operational, and UI components
  hooks/                 Responsive hooks and synthetic data seeding
  lib/
    aiEngine.js          Deterministic recovery scoring and explanations
    analytics.js         Recovery metrics and insights
    audit.js              Audit helpers
    recovery.js           Recovery orchestration and policy-aware actions
    seeData.js            Synthetic seed records
  pages/                 Dashboard, operations, analytics, simulator, rules, and auth views

tests/engine.test.js     Decision-engine tests
index.html               Vite entry document
vite.config.js           Source alias and API proxy
package.json             Scripts and dependencies
.env.example             Local server and SMTP configuration template
```

Application routes include `/`, `/transactions`, `/recovery`, `/recovery/:id`, `/customers`, `/customers/:id`, `/analytics`, `/ai-insights`, `/simulator`, `/rules`, `/notifications`, `/audit-logs`, `/settings`, `/onboarding`, `/login`, `/register`, `/forgot-password`, `/reset-password`, and `/verify-email`.

## Local Development

Use a current Node.js release and npm.

```bash
npm install
copy .env.example .env
npm run dev:full
```

Open the frontend at `http://localhost:5173`. The Express API runs at `http://localhost:8787`; its health endpoint is `http://localhost:8787/api/health`.

To run the services separately:

```bash
npm run server
npm run dev
```

Available checks and build commands:

```bash
npm test
npm run lint
npm run typecheck
npm run build
npm run preview
```

## Environment Variables

The variables are defined in `.env.example`:

| Variable | Purpose | Required |
| --- | --- | --- |
| `API_PORT` | Express API port; defaults to `8787` | Optional |
| `SESSION_SECRET` | JWT signing secret | Required in production; use a long random value |
| `PUBLIC_APP_URL` | Verification and reset link base URL | Optional |
| `FRONTEND_ORIGIN` | Allowed frontend origin for CORS | Optional |
| `EMAIL_PROVIDER` | Email provider selection | Optional |
| `EMAIL_FROM` | Sender address | Optional |
| `SMTP_HOST` | SMTP host | Optional |
| `SMTP_PORT` | SMTP port | Optional |
| `SMTP_SECURE` | SMTP TLS setting | Optional |
| `SMTP_USER` | SMTP username | Optional |
| `SMTP_PASSWORD` | SMTP password | Optional |
| `VITE_APP_NAME` | Frontend template variable | Optional; not currently read by runtime code |
| `VITE_SYNTHETIC_DATA_ONLY` | Frontend template variable | Optional; not currently read by runtime code |

Without SMTP values, development emails are previewed in `data/email-outbox`. Never commit `.env` files or secrets.

## Demo Walkthrough

1. Open `http://localhost:5173`.
2. Select **Explore Demo** on the login page.
3. Review the Dashboard recovery metrics and health.
4. Open Transactions and select a failed synthetic payment.
5. Review failure classification, probability, confidence, expected value, timing, and recommendation.
6. Open the recovery case to compare strategies and review policy state.
7. Execute an available simulated recovery action or schedule a retry.
8. Review the updated transaction, case, notification, and audit records.
9. Open Analytics, AI Insights, and Simulator to inspect dynamic performance views and counterfactual strategy comparisons.

Demo records remain in browser-local storage. The demo reset control clears records in the demo workspace scope.

## Example Recovery Case

The following is a synthetic example, not a production result:

```text
Transaction:    INR 18,500
Payment method: Credit Card
Status:         Failed
Failure:        Network Timeout
Attempts:       1

Failure Classification
        -> Recovery Analysis
        -> Strategy Comparison
        -> Policy Evaluation
        -> Recommended Action
        -> Simulated Recovery
        -> Updated Metrics
        -> Audit Event
```

The actual recommendation depends on the transaction, customer signals, settings, and deterministic scoring logic at runtime.

## Current Implementation vs Future Roadmap

### Currently Implemented

- React operations console with protected application routes
- Express API with SQLite-backed authentication
- Email/password registration, verification, login, logout, and password reset
- Browser-local operational entities and workspace-scoped demo data
- Synthetic customers, transactions, recovery cases, attempts, rules, notifications, insights, and audit records
- Deterministic and explainable failure classification, probability, confidence, timing, expected value, and strategy ranking
- Recovery queue, case detail, policy gates, manual overrides, scheduling, simulated actions, retry limits, and idempotency protection
- Dynamic analytics, failure intelligence, revenue leakage signals, counterfactual comparisons, and strategy simulation

### Future Improvements

- Real payment gateway integrations and webhooks
- Server-side operational repositories and durable multi-tenant authorization
- Background workers for rule evaluation and scheduled recovery
- Provider-backed recovery email, SMS, and payment-link delivery
- Trained recovery models, calibration, feature engineering, anomaly detection, segmentation, and feedback-based learning
- Production observability, compliance controls, key management, and deployment infrastructure

## Limitations and Safety

RecoverAI is a portfolio and demonstration system. Payment data and outcomes are synthetic, payment execution is simulated, and no real money is transferred. Do not enter real card credentials, payment secrets, or sensitive customer data.

The current scoring is deterministic and explainable. Operational records are browser-local and mutable, and client-side workspace scoping is not a production authorization boundary. Settings such as two-factor authentication and IP allowlisting are represented in the interface but are not enforced as complete security controls. Rules are stored but are not run by a background worker. The project requires substantial security, compliance, infrastructure, and payment-provider work before any real financial operation.

## Project Objective

The objective is to demonstrate how payment recovery can move beyond simple retry automation toward an intelligent recovery decision workflow.

Instead of asking:

> Should we retry this payment?

RecoverAI asks:

> What is the best recovery action for this payment, when should it happen, what value could it recover, and should we avoid the retry altogether?

## Product Vision

```text
Failure -> Intelligence -> Decision -> Action -> Outcome -> Optimization
```

The goal is to improve the quality of recovery decisions rather than simply increase retry volume.

## Differentiation

RecoverAI combines:

```text
Failure Intelligence
        + Recovery Probability
        + Expected Recovery Value
        + Strategy Selection
        + Timing
        + Payment-Method Considerations
        + Policy Decisioning
        + Explainability
        + Counterfactual Analysis
        + Revenue Leakage Signals
        + Strategy Simulation
        + Safe Simulated Execution
        + Auditability
```

RecoverAI transforms failed payments from passive error records into actionable recovery opportunities.

Don't just retry failed payments. Understand the failure, evaluate the opportunity, choose the right action, and recover intelligently.

Recover the right payment.\
At the right time.\
With the right strategy.