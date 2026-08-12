# Enhancement Two Narrative: Algorithms and Data Structures

## Artifact Overview

The artifact selected for Enhancement Two is Travlr Getaways, a MEAN-stack travel application originally created in CS 465 Full Stack Development in 2026. The application includes a customer-facing travel site, an Express and Node.js API, MongoDB persistence through Mongoose, and an Angular administrative single-page application for managing trip records. The original artifact supported viewing trips, user registration, login, JSON Web Token authentication, and create, read, update, and delete operations for trip data.

For this milestone, I continued using the enhanced copy of the artifact that was created for the software design and engineering milestone. The original artifact remains preserved separately so the before-and-after state can be reviewed. Enhancement Two adds an algorithms and data structures improvement to trip discovery through a backend recommendation endpoint.

## Rationale for Inclusion

I selected Travlr Getaways for my ePortfolio because it is a complete full-stack application and it connects directly to the type of backend and full-stack software engineering work I want to pursue. It is stronger than a small isolated class exercise because it includes routing, API design, authentication, persistence, frontend interaction, and testable backend behavior. That makes it a useful artifact for showing how I can improve an existing application in a disciplined way.

The original trip functionality was mostly CRUD-based. A user or administrator could list trips or look up a trip by code, but the application did not provide meaningful trip discovery. That limitation made the artifact a good candidate for an algorithms and data structures enhancement. A travel application naturally benefits from search, filtering, sorting, and recommendation behavior because users often choose trips by destination, budget, duration, and trip characteristics.

The enhanced artifact now includes a recommendation endpoint at `GET /api/trips/recommendations`. The endpoint accepts query criteria such as text search, maximum price, maximum number of nights, and result limit. It returns ranked trips with a numeric recommendation score and human-readable recommendation reasons. This improvement showcases algorithmic thinking because it goes beyond simply returning stored records. The backend now normalizes user criteria, builds searchable token data, selects candidate trips, filters constraints, scores matches, and sorts the final results with deterministic tie-breakers.

## Enhancement and Evidence

The main implementation is in `app_api/services/trip-recommendation-service.js`. The enhancement uses several algorithmic steps and data structures.

First, the service normalizes input criteria. Price values such as `$900` and `900` are converted to numeric values. Duration values such as `4 nights` are parsed into numbers. Text criteria are tokenized into lowercase search terms. Invalid values are rejected before ranking begins, which prevents misleading results and keeps the API behavior predictable.

Second, the service builds an inverted index using a JavaScript `Map` whose keys are searchable tokens and whose values are `Set` collections of trip positions. This structure allows the algorithm to identify candidate trips by token match instead of repeatedly scanning every searchable field for every ranking step. For the current artifact size, either approach would be fast, but using an inverted index is a better demonstration of scalable search design and makes the trade-off explicit.

Third, the algorithm filters candidate trips against hard constraints. If a maximum price or maximum duration is supplied, trips outside those limits are removed from the result set. This separates eligibility from ranking, which makes the behavior easier to reason about and test.

Fourth, the service applies a weighted scoring model. Text matches, budget fit, and duration fit contribute to the recommendation score. The score is intentionally simple and explainable. A trip receives points for matching search terms, for staying within the budget, and for fitting the duration limit. The returned response includes recommendation reasons so the ranking is not a black box.

Finally, the service sorts the results by recommendation score. If two trips have the same score, the lower price is preferred. If the score and price are still tied, the original collection order is used as a stable tie-breaker. This gives deterministic output, which is important for testing and for user trust.

The API route was added in `app_api/routes/index.js` before the existing `/trips/:tripCode` route. This order matters because Express route matching is sequential; placing `/trips/recommendations` first ensures that the word `recommendations` is treated as the recommendation endpoint rather than as a trip code.

Automated tests were added in `test/trip-recommendation-service.test.js`, and the existing trip service tests were expanded to verify service delegation. The test suite now covers price parsing, duration parsing, inverted index construction, text matching, budget and duration filtering, weighted ranking, tie-breaking, validation errors, and service boundary behavior. Local verification on July 24, 2026 passed 19 of 19 backend tests:

```text
npm.cmd test
tests 19
pass 19
fail 0
```

## Course Outcome Alignment

This enhancement directly supports Course Outcome 3: designing and evaluating computing solutions that solve a given problem using algorithmic principles and computer science practices and standards while managing design trade-offs. The problem was that the artifact did not support meaningful trip discovery. The solution uses tokenization, an inverted index, sets, constraint filtering, weighted scoring, and stable sorting to produce ranked recommendations.

This enhancement also supports Course Outcome 4 as secondary evidence because the recommendation endpoint adds practical value to the travel application. It is not only an academic algorithm placed beside the project; it is integrated into the existing Express API and tested through the backend test suite. The work builds on the software design boundaries from Enhancement One, which made it easier to add the algorithm as a separate service rather than placing ranking code directly in the controller.

My outcome-coverage plan remains mostly the same. Enhancement One provided the strongest evidence for software design and security. Enhancement Two now provides direct evidence for algorithms and data structures. The remaining database enhancement will provide stronger evidence for database design, query behavior, validation, indexes, and persistence quality.

## Process, Challenges, and Learning

The most important learning from this enhancement was that algorithms in an application should be tied to a real user problem. I could have added a standalone sorting algorithm or a simple frontend filter, but that would not have improved the artifact in a meaningful way. The recommendation endpoint is more valuable because it fits the travel domain and creates a feature a user would expect from a trip application.

One challenge was choosing an algorithm that was substantial enough for the milestone without making the artifact unnecessarily complex. A machine-learning recommendation model would have been too large for the available data and would have been difficult to justify in this project. A simple database query filter would have been too basic. The selected approach is a middle ground: it uses recognizable computer science concepts while keeping the implementation explainable, testable, and appropriate for the current size of the application.

Another challenge was deciding how to handle imperfect data. The existing trip records store price and duration as display strings, such as `$799` and `4 nights`. Instead of changing the database schema during this milestone, I wrote parsing logic that converts those strings into numeric values for scoring. This was a useful trade-off because it kept the algorithms enhancement focused while leaving deeper schema improvements for the database milestone.

I also learned the value of deterministic tie-breaking. Recommendation logic can become hard to test if equal scores produce unpredictable order. By sorting first by score, then by price, and then by original position, the algorithm produces stable results. That makes the behavior easier to explain in the narrative and easier to verify with automated tests.

Overall, this enhancement helped me move from basic CRUD thinking toward solution design. I had to define a problem, choose data structures, make trade-offs, implement the algorithm in a maintainable service, and prove the behavior through tests. That process is closer to professional software engineering than simply adding another endpoint.
