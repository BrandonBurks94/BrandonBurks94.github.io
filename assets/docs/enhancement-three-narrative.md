# Enhancement Three Narrative: Databases

## Artifact Overview

The artifact selected for Enhancement Three is Travlr Getaways, a MEAN-stack travel application originally created in CS 465 Full Stack Development in 2026. The application includes a public travel site, an Express and Node.js API, MongoDB persistence through Mongoose, and an Angular administrative single-page application for managing trip records. The original artifact supported viewing trips, user registration, login, JSON Web Token authentication, and CRUD operations for trip data.

For this milestone, I created a database-focused enhanced copy at `artifacts/travlr-getaways-enhancement-three`. The original artifact remains preserved separately, and the Enhancement Three copy builds on the earlier software design and algorithms work. This makes it possible to show how the same full-stack application improved across architecture, algorithms, and database design.

## Rationale for Inclusion

Travlr Getaways belongs in my ePortfolio because it is a complete full-stack application rather than a small isolated exercise. It demonstrates API design, authentication, routing, frontend interaction, backend service structure, and MongoDB persistence. That makes it a strong artifact for showing the type of backend and full-stack development skills I want to use professionally.

The database portion of the original artifact had clear room for improvement. Trip data was stored in MongoDB, but important values such as price and duration were stored only as display strings like `$799` and `4 nights`. Those strings worked for the user interface, but they were weak for database querying, validation, indexing, and future reporting. For example, a database cannot reliably perform budget or duration queries if those values are stored only as formatted text. The original schema also had limited indexes and did not include normalized searchable text or timestamps for record auditing.

Enhancement Three improves the artifact by strengthening the MongoDB/Mongoose data model. The schema now derives and stores normalized database fields: `priceAmount`, `durationNights`, and `searchText`. These fields preserve the original display values while also giving the database structured values that can be validated, indexed, and queried. The enhancement also adds timestamps, uppercase normalized trip codes, schema validation, a compound index for budget and duration queries, and a text index for searchable trip content.

## Enhancement and Evidence

The main database changes are in `app_api/models/travlr.js`, `app_api/repositories/trip-repository.js`, `app_api/validators/trip-validator.js`, and `app_api/utils/trip-normalization.js`.

The new `trip-normalization` utility centralizes parsing and derived-field logic. It converts display prices such as `$799` into numeric values such as `799`, parses durations such as `4 nights` into numeric night counts, normalizes trip codes to uppercase, and builds searchable text from the trip code, name, description, and details. Centralizing this logic helps keep the API validator and Mongoose schema consistent.

The Mongoose schema now includes `priceAmount`, `durationNights`, and `searchText`. These fields are required, derived from the original display fields, and validated before persistence. This means the database now rejects trip records that cannot populate meaningful numeric query fields. The schema also enables timestamps, so records can track creation and update times.

The indexing strategy was also improved. The schema defines a unique index on `code`, a compound index on `priceAmount` and `durationNights`, and a text index on `searchText`. These indexes align with the application domain. Trip codes should be unique, budget and duration are natural filters for travel search, and text search supports trip discovery by destination or trip description.

The repository layer now includes `findForRecommendationCriteria`. This method builds MongoDB query criteria from normalized recommendation inputs, allowing the database to prefilter records by budget, duration, and text search before the application-level recommendation service ranks the candidates. This connects the database enhancement to the earlier algorithms enhancement without making the database work redundant. The algorithms enhancement still handles scoring and tie-breaking, while the database enhancement improves how candidate records are stored and selected.

Automated tests were expanded to verify the database work. The test suite now checks that the schema derives normalized fields, rejects invalid values that cannot populate numeric database fields, and defines indexes for uniqueness, filtering, and text search. It also verifies that API payload validation builds the same normalized fields before persistence. Local verification on August 2, 2026 passed 23 of 23 backend tests:

```text
npm.cmd test
tests 23
pass 23
fail 0
```

## Course Outcome Alignment

This enhancement directly supports Course Outcome 4: demonstrating the ability to use well-founded techniques, skills, and tools in computing practices to implement solutions that deliver value and accomplish industry-specific goals. The database enhancement uses Mongoose schema design, validation, normalized stored fields, indexes, timestamps, and repository query logic to make the travel application more reliable and useful.

This enhancement also supports Course Outcome 5 because validation and predictable persistence are part of a security mindset. The application now rejects malformed price and duration values instead of allowing weak or unusable data into the database. While this is not an encryption or authentication change, it still reduces data integrity risk and supports safer backend behavior.

My outcome-coverage plan remains consistent with the plan from Module One. Enhancement One provided strong evidence for software design and security. Enhancement Two provided direct evidence for algorithms and data structures. Enhancement Three now provides direct evidence for database design and persistence quality. Together, the three enhancements show a broader improvement path across the full-stack artifact.

## Process, Challenges, and Learning

The main learning from this enhancement was that database design is not only about choosing MongoDB or writing basic CRUD operations. A database-backed application needs data that is structured for the questions the system must answer. In the original artifact, storing `$799` and `4 nights` as strings was enough for display, but it was not enough for reliable filtering, indexing, or future analysis. Turning those values into normalized fields made the database more useful without breaking the existing user interface.

One challenge was keeping the database enhancement distinct from the algorithms enhancement. The recommendation feature already used price and duration during ranking, so it would have been easy to repeat the same work. I avoided that by focusing this milestone on persistence and query readiness. The database now stores and indexes fields that support recommendation and search, while the recommendation service remains responsible for scoring and ranking.

Another challenge was preserving compatibility with the existing application. I did not want to remove the original display fields because the public and administrative UI already use them. Instead, I kept `price` and `duration` for display and added `priceAmount` and `durationNights` for database behavior. This was a practical trade-off because it improved the database model without requiring a full frontend rewrite.

The testing process also reinforced the importance of understanding framework behavior. I initially expected one validation path to derive fields in every test case, but the synchronous validation path behaved differently. I adjusted the schema so derived values are available through defaults as well as validation logic. That made the model more reliable and gave me stronger evidence that the database enhancement works as intended.

Overall, this enhancement helped me think more carefully about persistence quality. Good database design supports the rest of the application by improving validation, query performance, search capability, and maintainability. That is the kind of backend work I want to demonstrate in my ePortfolio.
