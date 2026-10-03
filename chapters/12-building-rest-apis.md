# 12. Building REST APIs

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Filters, security, and file uploads](./11-filters-security-and-file-uploads.md) | [Notes index](../README.md) | [Next: Services, events, and Spark commands](./13-services-events-and-spark-commands.md) |

## In this chapter

Build predictable JSON endpoints around resources, use HTTP methods and status codes consistently, validate request bodies, and keep API output under control.

## Model resources with URLs and HTTP methods

A resource is a thing the API manages, such as an article. Use a stable collection URL and identify one record by its ID:

| Request | Meaning |
| --- | --- |
| `GET /api/articles` | List articles |
| `GET /api/articles/42` | Read article 42 |
| `POST /api/articles` | Create an article |
| `PUT /api/articles/42` | Replace or update article 42 |
| `DELETE /api/articles/42` | Delete article 42 |

Keep the URL about the resource. The HTTP method communicates the action. Use query parameters for filters and pagination, for example `GET /api/articles?page=2&limit=20`.

## Define routes explicitly

CodeIgniter can create conventional resource routes with `resource()`. Explicit routes are easy to audit and make the allowed HTTP methods clear:

```php
$routes->group('api', ['namespace' => 'App\\Controllers\\Api'], static function ($routes) {
    $routes->get('articles', 'Articles::index');
    $routes->get('articles/(:num)', 'Articles::show/$1');
    $routes->post('articles', 'Articles::create');
    $routes->put('articles/(:num)', 'Articles::update/$1');
    $routes->delete('articles/(:num)', 'Articles::delete/$1');
});
```

Use the same database model and validation rules as a browser form, but return JSON with a meaningful HTTP status. API routes still need authentication, authorization, validation, and rate limits where appropriate.

## Return JSON with ResponseTrait

The `ResponseTrait` provides helpers for successful and failed API responses. This example lists articles and reads one record. It assumes an `ArticleModel` with an `allowedFields` list:

```php
<?php

namespace App\\Controllers\\Api;

use App\\Controllers\\BaseController;
use App\\Models\\ArticleModel;
use CodeIgniter\\API\\ResponseTrait;

class Articles extends BaseController
{
    use ResponseTrait;

    protected $format = 'json';

    public function index()
    {
        $articles = model(ArticleModel::class)
            ->orderBy('created_at', 'DESC')
            ->findAll(20);

        return $this->respond([
            'data' => $articles,
        ]);
    }

    public function show(int $id)
    {
        $article = model(ArticleModel::class)->find($id);

        if ($article === null) {
            return $this->failNotFound('Article not found.');
        }

        return $this->respond([
            'data' => $article,
        ]);
    }
}
```

A response wrapper gives clients a stable place for data and future metadata. Avoid returning database columns that the client does not need. If the model returns sensitive fields, map the result to an explicit response shape instead.

## Validate JSON before writing it

JSON requests do not use form post data. Read the decoded body, validate it, then use only the validated fields:

```php
public function create()
{
    $data = $this->request->getJSON(true) ?? [];

    $rules = [
        'title' => 'required|min_length[3]|max_length[160]',
        'body'  => 'required',
    ];

    if (! $this->validateData($data, $rules)) {
        return $this->failValidationErrors($this->validator->getErrors());
    }

    $values = $this->validator->getValidated();
    $id = model(ArticleModel::class)->insert($values);

    if ($id === false) {
        return $this->failServerError('The article could not be saved.');
    }

    $article = model(ArticleModel::class)->find($id);

    return $this->respondCreated([
        'data' => $article,
    ]);
}
```

Use strict validation rules for values decoded from JSON, especially when the request may contain numbers, booleans, arrays, or null. The model's `allowedFields` remains an important second boundary. Do not pass an unfiltered request body directly to `insert()` or `update()`.

For an update, first load the record and return 404 if it does not exist. Validate only the fields that may change, then update the known ID. Decide whether the endpoint uses PUT for replacement or PATCH for partial changes, and apply that rule consistently.

## Choose useful status codes

| Status | Use |
| --- | --- |
| 200 OK | Request succeeded and a response body is present |
| 201 Created | A new resource was created |
| 204 No Content | Request succeeded and there is no response body |
| 400 Bad Request | Request shape or syntax is invalid |
| 401 Unauthorized | Authentication is missing or invalid |
| 403 Forbidden | Identity is known but access is not allowed |
| 404 Not Found | Resource does not exist or should not be revealed |
| 422 Unprocessable Content | Request data failed validation |
| 429 Too Many Requests | Client exceeded a rate limit |
| 500 Internal Server Error | Unexpected server failure |

Do not return status 200 for every result. Clients use status codes to decide whether to retry, show validation feedback, or request authentication. Avoid exposing stack traces and SQL errors in production responses.

## Paginate and constrain list endpoints

Do not return an unbounded table. Read pagination values from the query string, validate them, and cap the page size:

```php
$page = max(1, (int) $this->request->getGet('page'));
$limit = (int) $this->request->getGet('limit');

if ($limit < 1) {
    $limit = 20;
}

$limit = min($limit, 100);
$offset = ($page - 1) * $limit;

$rows = model(ArticleModel::class)
    ->orderBy('created_at', 'DESC')
    ->findAll($limit, $offset);
```

The response can include `data`, `page`, `limit`, and a total count if the extra query is appropriate. Apply the same size limits to search terms, arrays, nested objects, and uploaded data. A caller-controlled limit can consume too much memory and database time.

## Test requests with curl

Run the application and try a request with a JSON body:

```sh
curl -i http://localhost:8080/api/articles
```

Create a record:

```sh
curl -i -X POST http://localhost:8080/api/articles \
  -H "Content-Type: application/json" \
  -d '{"title":"First article","body":"A short example."}'
```

Check the status, response headers, JSON structure, and validation error shape. Also try missing fields, malformed JSON, an unknown ID, an unauthorized user, and an oversized page limit.

## Common mistakes

- Using GET to change or delete data.
- Returning every database column and every row.
- Trusting decoded JSON without validation.
- Returning success status for a failed write.
- Mixing form validation messages with a different API error shape.
- Forgetting that CORS is not authentication.
- Disabling CSRF or authentication broadly to make an API request work.
- Allowing unlimited search sizes, page sizes, or request bodies.

## Official references

- [RESTful resource handling](https://codeigniter.com/user_guide/incoming/restful.html)
- [API response helpers](https://codeigniter.com/user_guide/outgoing/api_responses.html)
- [Building a RESTful controller](https://codeigniter.com/user_guide/guides/api/controller.html)
- [Validation](https://codeigniter.com/user_guide/libraries/validation.html)
- [Security guidelines](https://codeigniter.com/user_guide/concepts/security.html)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Filters, security, and file uploads](./11-filters-security-and-file-uploads.md) | [Notes index](../README.md) | [Next: Services, events, and Spark commands](./13-services-events-and-spark-commands.md) |
