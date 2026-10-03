# 15. Testing controllers and models

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Errors, logging, and caching](./14-errors-logging-and-caching.md) | [Notes index](../README.md) | [Next: Deployment and production checks](./16-deployment-and-production-checklist.md) |

## In this chapter

Test the behavior a user can observe, keep database tests isolated, and make test results repeatable.

## Run the test suite

CodeIgniter uses PHPUnit. Install it as a development dependency if the project does not already include it:

```sh
composer require --dev phpunit/phpunit
vendor/bin/phpunit
```

On Windows, the executable is usually `vendor\\bin\\phpunit`. Check the project's `composer.json` for a test script or configuration. Run tests from the project root so PHPUnit finds the correct bootstrap and test suites.

A useful test checks behavior and a meaningful edge case. Avoid tests that only restate the code line by line. When a test fails, read the assertion and the application log before changing the expected result.

## Start with a unit test

A unit test checks one small piece of behavior without making a real HTTP request or depending on a production database:

```php
<?php

namespace Tests\\Unit;

use CodeIgniter\\Test\\CIUnitTestCase;

final class ArticleTitleTest extends CIUnitTestCase
{
    public function testTitleIsTrimmedBeforeItIsShown(): void
    {
        $title = trim('  Notes about routing  ');

        $this->assertSame('Notes about routing', $title);
    }
}
```

As the application grows, move non-trivial rules into plain classes or services so they can be tested directly. A controller test should not need to prove every internal line; it should check its observable response and important side effects.

## Test a complete HTTP request

Feature tests run a request through routing and the application lifecycle. This is useful for checking route wiring, response status, view output, redirects, and JSON:

```php
<?php

namespace Tests\\Feature;

use CodeIgniter\\Test\\CIUnitTestCase;
use CodeIgniter\\Test\\FeatureTestTrait;

final class HomePageTest extends CIUnitTestCase
{
    use FeatureTestTrait;

    public function testHomePageLoads(): void
    {
        $result = $this->get('/');

        $result->assertOK();
        $result->assertSee('Welcome');
    }
}
```

For an API route, set the request body format and inspect the JSON response:

```php
$result = $this->withBodyFormat('json')
    ->post('/api/articles', [
        'title' => 'A test article',
        'body'  => 'A short body.',
    ]);

$result->assertStatus(201);
$result->assertJSONFragment([
    'title' => 'A test article',
]);
```

Feature tests can also set headers, session values, or temporary routes. Prefer the real route configuration when route behavior is part of what the test needs to prove.

## Keep database tests isolated

Never point tests at a database containing real user data. Configure a separate `tests` database group in `app/Config/Database.php`, and keep its credentials in an untracked environment file.

Use `DatabaseTestTrait` to run migrations and reset data around tests:

```php
<?php

namespace Tests\\Feature;

use CodeIgniter\\Test\\CIUnitTestCase;
use CodeIgniter\\Test\\DatabaseTestTrait;
use CodeIgniter\\Test\\FeatureTestTrait;

final class ArticleApiTest extends CIUnitTestCase
{
    use DatabaseTestTrait;
    use FeatureTestTrait;

    protected $migrate = true;
    protected $migrateOnce = false;
    protected $refresh = true;

    public function testUnknownArticleReturnsNotFound(): void
    {
        $result = $this->get('/api/articles/999999');

        $result->assertStatus(404);
    }
}
```

The database trait can use migrations and seeders to create a known state. Each test should create the rows it needs and avoid depending on test execution order. If a test class defines `setUp()` or `tearDown()`, call the parent method so the traits can prepare and clean the test database.

## Test invalid and permitted behavior

For important endpoints, cover more than the successful path:

- A missing record returns 404.
- Invalid input returns a validation response and does not write a row.
- A guest cannot reach a protected route.
- A signed-in user cannot access another user's private record.
- A valid request stores the expected data.
- A failed database operation returns a safe response.
- An API response has the expected status and JSON fields.

Check authorization separately from authentication. A test that proves a logged-in user can access a route does not prove that the user is allowed to read every record.

## Make test setup repeatable

- Use dedicated test database credentials and verify the database name before a destructive refresh.
- Use migrations to create the schema and seed only the records required by a test.
- Avoid relying on network services, email delivery, or current wall-clock time.
- Replace external services with controlled test doubles where appropriate.
- Keep event listeners from sending real notifications during tests.
- Use deterministic values for dates and IDs when the behavior depends on them.
- Run the suite after changing routes, filters, models, or validation rules.

## Common mistakes

- Running a database refresh against development or production data.
- Writing tests that pass only because another test ran first.
- Testing controller internals when the contract is the response.
- Checking only a 200 status while ignoring the body.
- Treating a test database as optional for a test that writes data.
- Letting tests send real emails or call paid external services.
- Disabling an assertion because it reveals a real authorization gap.

## Official references

- [Testing overview](https://codeigniter.com/user_guide/testing/overview.html)
- [HTTP feature testing](https://codeigniter.com/user_guide/testing/feature.html)
- [Testing controllers](https://codeigniter.com/user_guide/testing/controllers.html)
- [Testing the database](https://codeigniter.com/user_guide/testing/database.html)
- [Testing responses](https://codeigniter.com/user_guide/testing/response.html)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Errors, logging, and caching](./14-errors-logging-and-caching.md) | [Notes index](../README.md) | [Next: Deployment and production checks](./16-deployment-and-production-checklist.md) |
