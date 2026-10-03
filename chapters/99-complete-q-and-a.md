# 99. Complete questions and answers

[Back to notes index](../README.md)

| Previous | Notes index | End |
| --- | --- | --- |
| [Previous: All code samples](./98-all-code-samples.md) | [Notes index](../README.md) | End of notes |

These questions review the core CodeIgniter 4 topics in this repository. Each answer is kept practical and points back to the relevant chapter when more detail is useful.

## Setup and application structure

### 1. What does CodeIgniter do?

CodeIgniter is a PHP framework for building web applications and APIs. It provides request routing, controllers, views, database tools, validation, sessions, filters, and other common building blocks.

### 2. What does Composer do in a CodeIgniter project?

Composer installs PHP packages and records dependency versions. For an application, commit `composer.lock` so local, test, and production environments install the same resolved versions.

### 3. What is Spark?

Spark is the project's command-line tool. Use it to inspect routes, run migrations, generate files, start a local server, and run custom commands. Run it from the project root with `php spark`.

### 4. Why is the `public/` directory the web root?

It contains the front controller and public assets. Pointing the web server there prevents direct web access to application code, environment files, dependencies, and writable storage.

### 5. What belongs in `writable/`?

Runtime files such as logs, cache data, sessions, and uploads. The web server needs permission to write there, but the directory should not be exposed as a public file location.

### 6. Why keep configuration in environment variables?

Different environments need different database credentials, base URLs, and service secrets. Environment settings keep those values out of source control and let each deployment configure its own connection details.

### 7. What is a namespace?

A namespace groups PHP classes and prevents class-name collisions. CodeIgniter's `App` namespace normally maps to the project's `app/` directory through PSR-4 autoloading.

## Routing and request handling

### 8. What is a route?

A route maps a URL and HTTP method to a controller action. Explicit routes make the application's public endpoints easier to understand and review.

### 9. Why define HTTP verbs explicitly?

The HTTP verb communicates the request's intent and helps restrict what an endpoint can do. A read should use GET, while changes should use POST, PUT, PATCH, or DELETE as appropriate.

### 10. What is a route group?

A route group applies shared settings such as a URL prefix, namespace, or filter to several routes. It reduces repeated configuration while keeping related endpoints together.

### 11. What is the role of a controller?

A controller receives a request, coordinates application work, and returns a response. It should avoid holding unrelated business rules or large database workflows directly.

### 12. When should a controller return a redirect?

Use a redirect after a browser form submission or when the visitor should continue at another URL. For an API, return a structured response and status code instead.

### 13. What is a filter?

A filter runs before or after a controller. Filters are useful for cross-cutting checks such as authentication, CSRF protection, or response headers.

### 14. How is a filter different from validation?

A filter checks or changes request handling across routes. Validation checks whether specific request data follows the rules for an operation.

## Views and user input

### 15. Why escape data in a view?

Escaping prevents text from being interpreted as HTML or script in the output context. Use `esc()` when rendering user-controlled values, including values read from a database.

### 16. Does validation replace output escaping?

No. Validation decides whether data is acceptable for an operation. Escaping protects the context where data is displayed. Many apps need both.

### 17. Why use CSRF protection?

CSRF protection helps prevent another site from causing a visitor's browser to submit an unintended state-changing request. It does not replace authentication, authorization, or validation.

### 18. Which methods does the CSRF filter protect?

CodeIgniter's CSRF filter protects POST, PUT, PATCH, and DELETE requests. Configure it for browser forms that change state and handle external API callbacks with their own verified authentication method.

### 19. Why use `old()` in a form?

It restores a submitted value after validation fails, so the visitor does not have to re-enter every field. Escape the restored value when rendering it.

## Database and models

### 20. What is Query Builder?

Query Builder is CodeIgniter's interface for composing database queries. It handles value escaping for many operations and supports readable query construction.

### 21. Does Query Builder make every query automatically safe?

No. Use its methods and bind values instead of concatenating untrusted input into SQL. Dynamic table names, column names, and sort directions still need explicit allowlists.

### 22. What does a model provide?

A model centralizes common operations for a table or data source, including allowed fields, validation, timestamps, and CRUD methods. It helps controllers avoid repeating data access details.

### 23. What does `allowedFields` protect?

It limits which submitted keys the model can insert or update. It helps prevent callers from changing fields that should be controlled by the server, but it does not replace authorization or validation.

### 24. What is a database transaction?

A transaction groups related database operations so they can succeed or roll back together. Use one when partial completion would leave data inconsistent.

### 25. What is a migration?

A migration is a versioned schema change stored in code. It gives the team a repeatable way to create, update, and roll back database structure.

### 26. What is a seeder?

A seeder inserts known data, such as local examples or test fixtures. Keep production account secrets and private data out of general seed files.

### 27. Why use pagination?

Pagination limits the number of rows returned and the memory and time used to build a response. Enforce a maximum page size on the server, even if the client sends a larger value.

## Validation, sessions, and files

### 28. Why validate on the server if the browser already checks a form?

Browser checks can be bypassed or changed. The server must validate every request before it performs a write or other sensitive action.

### 29. What are strict validation rules useful for?

Strict rules preserve and check native types such as integers, booleans, arrays, and null values. They are especially useful when validating decoded JSON.

### 30. What is flash data?

Flash data is session data intended for one following request, commonly used for a short success or error message after a redirect.

### 31. What is the difference between authentication and authorization?

Authentication identifies who is making a request. Authorization decides whether that identity may perform a specific action on a specific resource.

### 32. How should an upload be checked?

Validate required upload status, file size, and allowed content type on the server. For images, use image-specific checks. Treat the browser's `accept` attribute as a convenience only.

### 33. Why generate a filename for an uploaded file?

Client-provided names are untrusted and may collide or contain misleading extensions. A generated name avoids trusting the submitted name as a storage path.

### 34. Where should private uploads be stored?

Keep them outside the public document root, such as under `writable/`. Serve them through a route that verifies the current user's permission.

## APIs, services, and commands

### 35. What makes a resource API predictable?

Stable resource URLs, standard HTTP methods, useful status codes, consistent JSON shapes, and documented validation errors help clients understand what each endpoint does.

### 36. What status should a create endpoint return?

A successful create commonly returns 201 Created, optionally with the new resource in the response body. Use a validation or client-error status when the submitted data is invalid.

### 37. Is CORS an authentication mechanism?

No. CORS controls which browser origins may read responses. It does not prove the identity or permissions of a caller.

### 38. What does `ResponseTrait` provide?

It provides helpers for returning formatted API success and failure responses, including common HTTP status handling. Controllers still need to validate, authorize, and choose what data to expose.

### 39. What is a CodeIgniter service?

A service is a named factory for retrieving or creating an object. It can centralize construction and optionally return a shared instance.

### 40. When are events useful?

Events let application code announce that something has happened so separate listeners can handle optional side effects. Do not rely on a listener for a required change that must be atomic with a database write.

### 41. When should I create a custom Spark command?

Use a custom command for repeatable operator or scheduled work that should run from the command line, such as cleanup, imports, or a scheduled digest.

## Errors, caching, tests, and deployment

### 42. What should production error pages show?

A production visitor should see a safe, useful message without stack traces, SQL, file paths, or secrets. Record diagnostic details in protected logs.

### 43. What should not be written to application logs?

Do not log passwords, access tokens, session IDs, full payment details, or private request bodies. Logs need enough context to diagnose behavior without exposing sensitive values.

### 44. What makes a good cache key?

A key includes every value that changes the cached result, such as locale, tenant, or user scope. If the response contains private or personalized data, do not share it through a public cache key.

### 45. When is full-page caching appropriate?

It is appropriate for public pages whose output is the same for every visitor during the cache lifetime. Avoid it for account pages, private dashboards, and pages containing user-specific tokens or data.

### 46. What is a feature test?

A feature test sends a request through the application's routing and request lifecycle. It can check route behavior, status, redirects, rendered content, and JSON responses.

### 47. Why use a separate test database?

Database tests can reset schema and data. A dedicated database prevents those test operations from damaging development or production records.

### 48. What does `DatabaseTestTrait` do?

It provides CodeIgniter-specific database setup for tests, including migration, refresh, and seed options. The test must use a configured test database group and should call parent setup and teardown methods when overriding them.

### 49. Why must the web root point to `public/` in production?

The public directory is designed to be the only directly served part of the application. It keeps configuration, source code, dependencies, and writable data outside normal web access.

### 50. What should happen before a production migration?

Back up important data, review the schema change, test it on a production-like copy, and check compatibility with the currently deployed code. Never run `migrate:refresh` on production.

### 51. What is a safe deployment rollback plan?

Keep the previous application release available, know how to switch traffic back, and plan database changes so the older app can still run during rollback. A code rollback alone cannot restore data removed by a migration.

### 52. What should I check after deployment?

Check the production environment, HTTPS, important routes, authentication, database reads and writes, logs, uploads, and error pages. Confirm that no development tools or detailed errors are exposed.

## Chapter links

- [Installation and the first route](./01-installation-and-first-route.md)
- [Project structure and configuration](./02-project-structure-and-configuration.md)
- [Routing and route groups](./03-routing-and-route-groups.md)
- [Controllers, requests, and responses](./04-controllers-requests-and-responses.md)
- [Views, layouts, and helpers](./05-views-layouts-and-helpers.md)
- [Databases, Query Builder, and transactions](./06-databases-query-builder-and-transactions.md)
- [Models and CRUD](./07-models-and-crud.md)
- [Migrations and seeders](./08-migrations-and-seeders.md)
- [Validation and form errors](./09-validation-and-form-errors.md)
- [Sessions, cookies, and flash data](./10-sessions-cookies-and-flash-data.md)
- [Filters, security, and file uploads](./11-filters-security-and-file-uploads.md)
- [Building REST APIs](./12-building-rest-apis.md)
- [Services, events, and Spark commands](./13-services-events-and-spark-commands.md)
- [Errors, logging, and caching](./14-errors-logging-and-caching.md)
- [Testing controllers and models](./15-testing-controllers-and-models.md)
- [Deployment and production checks](./16-deployment-and-production-checklist.md)

## Official references

- [CodeIgniter 4 User Guide](https://codeigniter.com/user_guide/)
- [Server requirements](https://codeigniter.com/user_guide/intro/requirements.html)
- [Security guidelines](https://codeigniter.com/user_guide/concepts/security.html)

| Previous | Notes index | End |
| --- | --- | --- |
| [Previous: All code samples](./98-all-code-samples.md) | [Notes index](../README.md) | End of notes |
