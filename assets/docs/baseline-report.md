# Travlr Getaways Baseline Report

## Verification Date

July 3, 2026

## Environment

- Node.js 22.14.0
- npm 11.6.2
- MongoDB Windows service running locally
- Baseline database: `travlr_capstone_baseline`
- Angular 20.3 production build

## Reproduction

1. Install root dependencies with `npm install`.
2. Set `MONGODB_URI` and `JWT_SECRET`.
3. Run `npm run seed`.
4. Run `npm start`.
5. Install `app_admin` dependencies with `npm install`.
6. Run the Angular production build with `npm run build`.

## Successful Checks

| Check | Result |
|---|---|
| MongoDB seed | 3 trips loaded successfully |
| Public home page | HTTP 200 |
| Server-rendered travel page | HTTP 200 |
| `GET /api/trips` | HTTP 200; 3 trips returned |
| Unauthenticated `POST /api/trips` | HTTP 401 |
| User registration | JWT returned |
| Angular production build | Successful; 313.06 kB initial bundle |
| Measured local `GET /api/trips` | 14.66 ms for the verification run |

## Baseline Architecture

- Express serves static public pages and a Handlebars travel route.
- Angular provides the administrative single-page application.
- Express exposes REST endpoints for authentication and trip CRUD operations.
- Mongoose connects the application to MongoDB.
- JWT bearer tokens protect trip create, update, and delete operations.

## Limitations and Risks

- No root or Angular automated test script is defined.
- Dependency installation reported one moderate-severity vulnerability.
- Trip `price` and `duration` values are stored as presentation strings, limiting numeric filtering, sorting, validation, and aggregation.
- The trip list endpoint retrieves every record and has no pagination, filtering, sorting, or result limit.
- The application lacks a recommendation or ranking algorithm.
- Password derivation uses synchronous PBKDF2 with only 1,000 iterations.
- JWT configuration is not validated at startup.
- Authentication endpoints have no visible rate limiting.
- Validation is repeated in controller code rather than enforced through a reusable boundary.
- Database errors can be returned to clients through `err.message`.
- Environment-specific API URLs are constructed directly in the Angular service.
- No continuous integration workflow or automated quality gate is present.

## Planned Measurement Baselines

- Automated tests: 0 defined test scripts.
- API result size: unbounded.
- Search/filter capability: none.
- Recommendation capability: none.
- Structured numeric price/duration data: none.
- Security checks: manual endpoint check only.
- Database query evidence: capture MongoDB `explain()` results before and after indexing using an expanded test dataset.

