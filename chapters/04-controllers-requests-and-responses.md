# 4. Controllers, requests, and responses

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Routing and route groups](./03-routing-and-route-groups.md) | [Notes index](../README.md) | [Next: Views, layouts, and helpers](./05-views-layouts-and-helpers.md) |

## Give a controller one request-handling job

A controller receives a routed request, coordinates the work needed for that request, and returns a response. Keep it focused on HTTP concerns. Put reusable data access in a model and repeated business rules in a suitable service.

Controllers live in `app/Controllers/`, use the `App\Controllers` namespace, and extend the application's `BaseController`.

```php
<?php

namespace App\Controllers;

use App\Models\ArticleModel;
use CodeIgniter\Http\ResponseInterface;

class Articles extends BaseController
{
    public function index(): string
    {
        $articles = model(ArticleModel::class)->findAll();

        return view('articles/index', [
            'articles' => $articles,
        ]);
    }
}
```

The route can point to `Articles::index`. The controller asks the model for data and passes that data to a view.

## Read the correct request source

The request object is available as `$this->request` in a controller. Use the method that matches the input source:

```php
$search = $this->request->getGet('q');
$title  = $this->request->getPost('title');
$method = $this->request->getMethod();
```

Do not use a combined input lookup for security-sensitive fields when you mean to accept only POST data. Be explicit about whether a value comes from a query string, form submission, cookie, or JSON body.

For a JSON request, decode the body as an associative array:

```php
$payload = $this->request->getJSON(true);
```

Validate untrusted values before using them. The validation chapter covers rules and retrieving only the validated fields.

## Return an HTML view

A controller can return a rendered view string:

```php
public function show(int $id): string
{
    $article = model(ArticleModel::class)->find($id);

    if ($article === null) {
        throw \CodeIgniter\Exceptions\PageNotFoundException::forPageNotFound();
    }

    return view('articles/show', [
        'article' => $article,
    ]);
}
```

Use the actual model and view paths in your app. Do not assume that a request parameter identifies an existing record. Handle a missing record as a 404 response or a deliberate alternative.

## Return JSON with a status code

For an API endpoint, use the response object rather than manually setting PHP headers:

```php
public function create(): ResponseInterface
{
    $payload = $this->request->getJSON(true);

    if (! is_array($payload)) {
        return $this->response
            ->setStatusCode(400)
            ->setJSON(['error' => 'A JSON object is required.']);
    }

    return $this->response
        ->setStatusCode(201)
        ->setJSON(['message' => 'Request accepted.']);
}
```

The example checks the broad shape of the request only. A real endpoint must validate every required field and save data before returning a successful creation response.

## Redirect after a form submission

After processing a browser form, redirect to a safe GET route to avoid resubmitting the same POST when the user refreshes:

```php
return redirect()
    ->to('/articles')
    ->with('message', 'Article created.');
```

The message is flash data, which lasts for the next request. Display it in the destination view and do not treat it as durable storage.

## Keep response types clear

A controller can return a view string, a response object, or a redirect response. Choose the one that matches the request:

- HTML page request: return `view(...)`.
- JSON API request: return a JSON response with an appropriate status.
- Successful browser form submission: return a redirect.
- Missing or invalid request: return a suitable error response.

Avoid printing output directly or calling PHP header functions in the controller. Returning the framework response keeps status codes, headers, and body handling together.

## Practice

1. Add a controller action that reads a search term with `getGet()`.
2. Pass results to a view using an associative array.
3. Return a 201 JSON response for a valid create request.
4. Redirect after a form submission and display one flash message.

## References

- [Controllers](https://codeigniter.com/user_guide/incoming/controllers.html)
- [Incoming requests](https://codeigniter.com/user_guide/incoming/incomingrequest.html)
- [HTTP responses](https://codeigniter.com/user_guide/outgoing/response.html)
- [CodeIgniter first app: create items](https://codeigniter.com/user_guide/guides/first-app/create_news_items.html)
