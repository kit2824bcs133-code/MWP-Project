# Examly Online Examination Portal

Examly is a React and Express application for timed online assessments. The current implementation includes student sign-in, an exam dashboard, section-timed attempts, autosaved responses, server-side objective scoring, result review, and a basic teacher attempt monitor.

## Requirements

- Node.js 20 or later
- npm

## Setup

```sh
npm install
copy .env.example .env
npm run dev
```

The Vite app is available at `http://localhost:5173`; the API listens on `http://127.0.0.1:3001`. SQLite creates `data/examly.sqlite` automatically. For local evaluation, the seed users are `student@examly.test` and `admin@examly.test`, both using the password in `DEMO_PASSWORD` (default `Examly!2026`). Change that value before the database is first created if you want a different local password. Seed users are for development only.

Set a long, random `JWT_ACCESS_SECRET` in `.env` before deployment. Production mode refuses the built-in development secret and serves the compiled client from `dist`.

## Implemented

- JWT access tokens kept in memory, rotating refresh tokens stored as hashes in SQLite, and role checks for teacher endpoints.
- A seeded question bank with subject, tags, difficulty, points, negative marks, and explanations.
- Single choice, multiple choice, true/false, fill-in-the-blank, short answer, code, and image-choice questions.
- Per-attempt question and option shuffling; every assigned question is included.
- Section timers enforced by API timestamps, question palette states, response autosave every three seconds, and a browser-local retry queue for temporary offline periods.
- Automatic submission when the active section timer expires through server-side deadline checks.
- Consent-gated fullscreen requests and browser detection for tab changes, focus loss, fullscreen exits, clipboard/context-menu actions, and print shortcuts; three server-recorded violations submit the attempt.
- Server-side objective scoring, partial credit for multiple choice, negative marks, and post-submit answer/explanation review.
- Monaco code editor. Code and subjective answers can be reviewed and manually graded by teachers; code is not executed.
- Student attempt history, a teacher view of recent attempts and violations, pending-response grading, pass-rate/average/facility analytics, and top finalized scores.
- Route-level code splitting for the exam, results, and chart-heavy admin view.

## Commands

```sh
npm run dev       # Vite and Express in watch mode
npm run build     # Production client build
npm run start     # Serve the built client and API
npm run lint      # ESLint
npm test          # Server scoring and API tests
npm run test:e2e  # Playwright student flow
```

Install the Playwright browser once before the E2E test:

```sh
npx playwright install chromium
```

## API

All API responses use JSON unless noted. Authenticated endpoints require `Authorization: Bearer <accessToken>`. Refresh tokens are supplied by the HTTP-only `exam_refresh` cookie.

| Method | Path | Description |
| --- | --- | --- |
| `GET` | `/api/health` | Health check |
| `POST` | `/api/auth/login` | Validate email/password; returns access token and user |
| `POST` | `/api/auth/refresh` | Rotate the refresh cookie and issue a new access token; returns `204` when signed out |
| `POST` | `/api/auth/logout` | Revoke refresh token and clear cookie |
| `GET` | `/api/exams` | List published exams and durations |
| `POST` | `/api/attempts` | Start a randomized attempt; body: `{ "examId": "exam-foundations" }` |
| `GET` | `/api/attempts/:attemptId` | Resume an active attempt or report its submitted result path |
| `POST` | `/api/attempts/:attemptId/consent` | Record monitoring consent for an existing attempt |
| `PATCH` | `/api/attempts/:attemptId/questions/:questionId` | Save an answer, visit/review flags, and elapsed time |
| `POST` | `/api/attempts/:attemptId/submit` | Submit and calculate the result on the server |
| `GET` | `/api/attempts/:attemptId/result` | Retrieve a submitted result and answer review |
| `POST` | `/api/attempts/:attemptId/violations` | Record an allowed violation event; the third event submits the attempt |
| `GET` | `/api/history` | List the signed-in student's attempts |
| `GET` | `/api/admin/attempts` | List recent attempts (admin only) |
| `GET` | `/api/admin/violations` | List recorded violation events (admin only) |
| `GET` | `/api/admin/grading` | List pending manual responses (admin only) |
| `PATCH` | `/api/admin/attempts/:attemptId/questions/:questionId/grade` | Save a manual grade and feedback (admin only) |
| `GET` | `/api/admin/analytics` | Pass rate, average score, item facility, and top scores (admin only) |

Example login request:

```json
{
	"email": "student@examly.test",
	"password": "Examly!2026"
}
```

Correct answers and explanations are excluded from active attempt responses and only included in results after server-side submission.

## Data and deployment notes

The SQLite schema is initialized on server startup and seed data is inserted into an empty database. Back up the database before removing `data/examly.sqlite`; deleting it resets demo users and attempts. Use a persistent volume for `DB_PATH` in a deployment. This starter does not yet include versioned migrations, production account enrollment, rate limiting, or multi-instance database coordination.

## Known limitations / planned work

- No webcam capture, face detection, or microphone monitoring. Browser checks are deterrents, not tamper-proof anti-cheat; users can bypass client-side controls with developer tools or unsupported browsers.
- No exam authoring/publishing API, schedule/group assignment, CSV/Excel import, or question sampling; the seeded exam is read-only and all its assigned questions are shuffled.
- No code execution sandbox, live WebSocket monitor, advanced psychometrics/percentiles, CSV/Excel authoring/import, PDF exports, certificates, or email/in-app notifications. Analytics are basic summaries of this single seeded exam, not cohort or date-filtered reporting.
- No 2FA, password reset, user enrollment, account lockout, or auth rate limiting. Configure HTTPS and a strong secret before deployment; demo credentials are public by design.
- No i18n layer or full accessibility audit. Theme toggle and responsive layouts are present, but WCAG conformance has not been certified.
- Offline mode retains pending responses in browser local storage and retries when connectivity returns; a device/browser loss before sync can still lose the newest queued changes. The server remains authoritative for scoring and section deadlines.
- Results currently show raw score and section accuracy. Percentile, certificate/PDF, and export workflows are not implemented.
# React + Vite

This template provides a minimal setup to get React working in Vite with HMR and some ESLint rules.

Currently, two official plugins are available:

- [@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react) uses [Oxc](https://oxc.rs)
- [@vitejs/plugin-react-swc](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react-swc) uses [SWC](https://swc.rs/)

## React Compiler

The React Compiler is not enabled on this template because of its impact on dev & build performances. To add it, see [this documentation](https://react.dev/learn/react-compiler/installation).

## Expanding the ESLint configuration

If you are developing a production application, we recommend using TypeScript with type-aware lint rules enabled. Check out the [TS template](https://github.com/vitejs/vite/tree/main/packages/create-vite/template-react-ts) for information on how to integrate TypeScript and [`typescript-eslint`](https://typescript-eslint.io) in your project.
