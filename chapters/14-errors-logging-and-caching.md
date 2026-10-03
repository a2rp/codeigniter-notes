# 14. Errors, logging, and caching

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Services, events, and Spark commands](./13-services-events-and-spark-commands.md) | [Notes index](../README.md) | [Next: Testing controllers and models](./15-testing-controllers-and-models.md) |

## In this chapter

Show safe errors to users, record useful diagnostics for developers, and cache only data that can be safely reused.

## Handle errors by environment

CodeIgniter has environment-specific error settings. Development should show enough detail to diagnose a problem. Production should show a generic page and keep diagnostic details in protected logs. Set the environment deliberately in the server configuration; do not leave a production app in development mode.

Return the right HTTP status for expected conditions. For example, return 404 when a record is missing and 403 when the current user is not allowed to access it. Do not use exceptions as normal branching for expected user input. For unexpected failures, let the framework's error handler record the failure and return a safe error response.

A custom not-found page can explain what the visitor can do next without exposing file paths, SQL, stack traces, or configuration values.

## Write useful logs

Use `log_message()` with a level that matches the event:

```php
log_message('info', 'Article {id} published by user {userId}', [
    'id' => $articleId,
    'userId' => $currentUserId,
]);

try {
    $publisher->publish($articleId);
} catch (\\Throwable $exception) {
    log_message('error', 'Publishing article {id} failed: {message}', [
        'id' => $articleId,
        'message' => $exception->getMessage(),
    ]);

    throw $exception;
}
```

Use placeholders and context values to make logs searchable. The default file handler writes logs under `writable/logs`; inspect `app/Config/Logger.php` for thresholds and handlers. Make sure the web server can write the log directory and that log files cannot be served publicly.

Logs help investigate a failure, but they can become a source of data leaks. Do not write passwords, access tokens, full payment details, session IDs, or private request bodies. Add a request or correlation ID when one is available so a request can be traced across log lines. Rotate and retain logs according to the app's needs.

## Cache values with an expiry

Use the cache service for data that is expensive to compute and safe to reuse:

```php
$key = 'homepage:published-articles';

$articles = cache($key);

if ($articles === null) {
    $articles = model(ArticleModel::class)
        ->where('status', 'published')
        ->orderBy('created_at', 'DESC')
        ->findAll(10);

    cache()->save($key, $articles, 300);
}
```

The third argument is the lifetime in seconds. Cache keys should describe the data and every input that changes the result. If output depends on a user, tenant, locale, permission, or query parameter, include that value in the key or do not share the cached value.

When a write changes the source data, delete related keys:

```php
cache()->delete('homepage:published-articles');
```

A cache is an optimization, not the source of truth. Decide what happens when the cache expires, is cleared, or the configured handler is unavailable. Start with a short expiry, measure the result, and adjust based on observed load and freshness needs.

## Cache full pages only when output is public

A controller can cache a rendered page for a limited time:

```php
public function index()
{
    $this->cachePage(60);

    return view('home', [
        'articles' => $this->articles->publishedForHome(),
    ]);
}
```

Page caching is keyed by URI and, in current CodeIgniter versions, HTTP method. Query-string behavior is configured in `app/Config/Cache.php`. Be deliberate about whether query parameters affect the output.

Full-page caching is suitable for public pages with the same output for all visitors. Do not use it for account pages, private dashboards, personalized forms, or responses that contain CSRF tokens or user-specific information. In current versions, configure cacheable status codes deliberately. For a production site that only wants successful pages, `$cacheStatusCodes = [200]` avoids keeping temporary error pages in cache.

After changing content, clear the affected cache or allow its expiry to pass. Removing a cache call does not necessarily erase an already stored response immediately.

## Debug locally, protect production

Use Spark to inspect available commands and routes:

```sh
php spark
php spark routes
```

Inspect the latest file in `writable/logs`, check the environment value, and reproduce the request with the smallest input that still fails. For database problems, enable focused local diagnostics rather than logging every query in production.

Before release, confirm:

- The production environment is selected.
- Detailed error display is disabled for visitors.
- The log threshold captures operational failures.
- The log directory is writable and private.
- Secrets and private request data are not written to logs.
- Cache keys separate data with different access or locale rules.
- Private pages and error responses are not cached as public content.

## Common mistakes

- Displaying a stack trace or database error to a visitor.
- Logging a whole request that contains credentials or personal data.
- Using the same cache key for different users or languages.
- Caching a page before checking permissions.
- Treating cached data as permanent truth.
- Caching a 404 page and wondering why a newly created record stays missing.
- Increasing cache duration without deciding how changed data invalidates it.

## Official references

- [Error handling](https://codeigniter.com/user_guide/general/errors.html)
- [Logging information](https://codeigniter.com/user_guide/general/logging.html)
- [Caching driver](https://codeigniter.com/user_guide/libraries/caching.html)
- [Web page caching](https://codeigniter.com/user_guide/general/caching.html)
- [Debugging](https://codeigniter.com/user_guide/testing/debugging.html)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Services, events, and Spark commands](./13-services-events-and-spark-commands.md) | [Notes index](../README.md) | [Next: Testing controllers and models](./15-testing-controllers-and-models.md) |
