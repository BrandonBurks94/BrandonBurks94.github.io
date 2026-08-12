# Enhancement One Narrative: Software Design and Engineering

## Artifact Overview

The artifact selected for Enhancement One is Travlr Getaways, a MEAN-stack travel application originally created in CS 465 Full Stack Development in 2026. The application includes a public customer-facing travel site, an Express and Node.js API, MongoDB persistence through Mongoose, and an Angular administrative single-page application for managing trip records. The original artifact supported user registration, login, JSON Web Token authentication, and create, read, update, and delete operations for trips.

This artifact was selected because it is a complete full-stack application with enough architecture, authentication, routing, persistence, and client-server interaction to demonstrate meaningful software design and engineering growth. It also fits the direction of my ePortfolio because it shows practical backend and full-stack development work rather than an isolated classroom exercise.

## Rationale for Inclusion

Travlr Getaways belongs in my ePortfolio because it demonstrates my ability to work across a full application stack. The original version already showed familiarity with Express routes, Angular components, MongoDB models, REST endpoints, and JWT-based login. However, the original implementation also showed common early-stage project limitations: controllers were tightly coupled to Mongoose, JWT verification was defined inline in the route file, validation logic was repeated near endpoint handlers, server errors could expose internal messages to API callers, required configuration was not validated at startup, and there was no automated backend test script.

The enhanced version improves the professional value of the artifact by turning those findings into specific design changes. The backend now separates routing, authentication middleware, validation, service logic, persistence access, and error handling. This makes the application easier to test, easier to maintain, and safer to extend in later enhancements.

## Enhancement and Evidence

The software design and engineering enhancement focused on backend architecture, secure configuration, safe API responses, authentication structure, password handling, and automated testing.

The enhanced artifact adds a startup configuration module at `app_api/config/environment.js`. Instead of silently falling back to a local MongoDB URI or reading `JWT_SECRET` only when a token is created or verified, the application now validates required environment variables as soon as it starts. If `MONGODB_URI` or `JWT_SECRET` is missing, the application fails fast with a clear configuration error. This supports safer deployment because the application does not continue running with an undefined JWT secret or an unintended database target.

JWT authorization was moved from an inline function in `app_api/routes/index.js` into reusable middleware at `app_api/middleware/authentication.js`. This keeps the route table focused on endpoint structure and makes authentication behavior reusable and testable. Trip CRUD logic was also refactored so controllers no longer call Mongoose directly. The new flow is controller to service to repository, with reusable validation in `app_api/validators/trip-validator.js`. This creates clearer boundaries between HTTP handling, business rules, input validation, and database access.

Centralized error handling was added through `app_api/middleware/error-handler.js`, supported by a small `HttpError` class. Before the enhancement, several controller branches returned `err.message` values to clients. After the enhancement, expected client errors return consistent public messages and validation details, while unexpected server errors return a sanitized message: "An unexpected server error occurred." Diagnostic detail remains server-side through logging instead of being exposed in the API response.

Password hashing was changed in `app_api/models/user.js`. The original artifact used synchronous PBKDF2 with 1,000 iterations and compared hashes directly as strings. The enhanced version uses Node's memory-hard `scrypt` function and timing-safe comparison with `crypto.timingSafeEqual`. This is a stronger authentication design choice and directly addresses the security weakness identified in the code review.

The original artifact had no backend test script. The enhanced artifact adds a Node test suite using the built-in `node:test` runner. The tests cover configuration validation, JWT middleware behavior, error-response sanitization, trip payload validation, trip service behavior, not-found handling, duplicate-code handling, and field allowlisting. Local verification on July 19, 2026 passed 13 of 13 tests:

```text
npm.cmd test
tests 13
pass 13
fail 0
```

The enhancement also adds a GitHub Actions quality gate at `.github/workflows/quality-gate.yml` so the same backend test suite can run automatically when the enhanced artifact is pushed or reviewed.

## Engineering Decisions and Trade-Offs

The main design decision was to refactor the backend into explicit layers without rewriting the entire application or changing the database schema yet. A larger rewrite could have introduced a new framework, TypeScript on the backend, or a different authentication approach, but that would have expanded the scope beyond the software design milestone and overlapped too much with later database and algorithm enhancements. The selected approach keeps the Express and CommonJS structure intact while adding professional boundaries around the code that most needed them.

I also chose focused automated tests instead of immediately building a full end-to-end test environment with a live MongoDB instance. The unit tests prove the most important design and security behavior without requiring a database service. This made the tests easier to run consistently and more appropriate for the milestone timeline. A future improvement would be to add integration tests that start the API against a dedicated test database and verify complete request and response behavior.

For error handling, I chose sanitized responses for unexpected errors rather than returning raw exception details. This trade-off improves security and privacy, but it also means developers need to rely on server-side logs for diagnosis. That is the correct boundary for a production-style API because external users should not receive internal database or stack information.

## Process, Challenges, and Feedback

The enhancement process began with the Module One code review findings and the baseline report. Those documents identified specific weaknesses rather than general style concerns, which made the implementation more focused. The most useful feedback from Module One was the reminder to make each category enhancement distinguished. For this milestone, that meant the software design enhancement needed to stand on its own through architecture, security, and testing evidence instead of becoming a general cleanup pass.

One challenge was keeping the enhancement substantial without making it too broad. The artifact also needs later algorithm and database enhancements, so I intentionally avoided changing trip price and duration data types or adding recommendation logic in this milestone. Those changes belong to later categories. Instead, I concentrated on the backend structure that those future enhancements will rely on.

Another challenge was improving authentication safely while staying compatible with the existing Express and Passport structure. Moving JWT verification into middleware and validating configuration at startup improved design without replacing the whole login flow. Updating password hashing to `scrypt` and timing-safe comparison strengthened the security posture while keeping the user model understandable.

The most important learning from this enhancement was that software design quality is not only about adding more files or abstractions. The new layers are valuable because each one removes a specific problem: repeated validation, repeated error handling, controller-to-database coupling, inline authentication, unsafe configuration, and untested security behavior.

## Course Outcome Alignment

This enhancement directly supports Outcome 4 and Outcome 5. It does not attempt to demonstrate every course outcome on its own. Outcomes 1, 2, and 3 are addressed more directly through the code review, portfolio communication, and the later algorithms and database enhancements.

Outcome 4 is demonstrated through the use of well-founded software engineering techniques that improve maintainability and reliability. The enhanced artifact separates routing, authentication middleware, validation, service logic, repository access, and centralized error handling. It also adds automated backend tests and a GitHub Actions quality gate. These changes show the ability to apply professional software design practices and tools to implement a computing solution that delivers value beyond cosmetic cleanup.

Outcome 5 is demonstrated through security-focused design changes. The application now validates required secrets at startup, avoids leaking internal error details, uses reusable JWT middleware, strengthens password hashing with `scrypt`, performs timing-safe password verification, and tests security-relevant behavior. These changes show a security mindset because they anticipate configuration mistakes, authentication weaknesses, and information-disclosure risks before they become production defects.
