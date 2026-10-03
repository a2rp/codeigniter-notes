# 3. Routing and route groups

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Project structure and configuration](./02-project-structure-and-configuration.md) | [Notes index](../README.md) | [Next: Controllers, requests, and responses](./04-controllers-requests-and-responses.md) |

## Map URLs to actions explicitly

Routes are defined in `app/Config/Routes.php`. Each route connects an HTTP method and URI pattern to a controller action or closure. CodeIgniter 4 has automatic routing disabled by default, so an undeclared URL does not silently expose a controller method.

```php
$routes->get('/', 'Home::index');
$routes->get('articles', 'Articles::index');
$routes->get('articles/(:num)', 'Articles::show/$1');
$routes->post('articles', 'Articles::create');
$routes->put('articles/(:num)', 'Articles::update/$1');
$routes->delete('articles/(:num)', 'Articles::delete/$1');
```

The `(:num)` placeholder matches a numeric segment. The `$1` in the destination passes that captured value to the action. Other built-in placeholders include `(:segment)` for a single non-slash segment and `(:any)` for a broader match. Prefer the narrowest pattern that fits the route.

## Match the HTTP method to the action

Use GET to read a resource. Use POST to create or submit data. Use PUT or PATCH to update a resource, and DELETE to remove one. A route using only GET should not change saved data.

```php
$routes->get('articles', 'Articles::index');
$routes->get('articles/(:num)', 'Articles::show/$1');
$routes->post('articles', 'Articles::create');
$routes->patch('articles/(:num)', 'Articles::update/$1');
$routes->delete('articles/(:num)', 'Articles::delete/$1');
```

This makes the route table easier to review and prevents a link preview or browser prefetch from accidentally performing a write.

## Group related routes

A group can add a shared URI prefix to several routes:

```php
$routes->group('admin', static function ($routes) {
    $routes->get('reports', 'Reports::index');
    $routes->get('reports/(:num)', 'Reports::show/$1');
    $routes->post('reports', 'Reports::create');
});
```

These routes start with `/admin`. Groups can also share options such as namespace or filters. Use a group when the routes genuinely share configuration, and keep the declared actions explicit.

## Name routes that are used elsewhere

A named route lets views and controllers generate a URL without repeating the path:

```php
$routes->get('articles/(:num)', 'Articles::show/$1', [
    'as' => 'article_show',
]);
```

Generate its URL from application code:

```php
$url = url_to('article_show', $articleId);
```

If the path changes later, the named route keeps callers focused on the route's purpose. Prefer the route name over hand-building the same URL in several views.

## Use resource routes deliberately

CodeIgniter can generate a standard set of routes for a resource:

```php
$routes->resource('photos');
```

Resource routes are useful when the controller follows the expected resource action names. Inspect the routes the framework generates and disable actions that an application does not use. For a small app or an API with special behavior, explicit route definitions can be easier to understand.

## Inspect the final route table

Run Spark from the project root:

```sh
php spark routes
```

Check the HTTP method, URI, name, and handler. A 404 often means the path or method does not match a registered route. A method error can mean the path exists, but it does not accept that HTTP verb.

## Route design guidelines

- Keep public paths predictable and use nouns for resources.
- Restrict each route to the HTTP methods it needs.
- Validate route parameters before using them in a query.
- Keep sensitive operations behind authorization checks and filters.
- Do not expose internal controller methods through automatic routing.
- Group routes only when shared configuration makes the route file clearer.

## Practice

1. Add an article list and numeric detail route.
2. Add a POST route for creating an article.
3. Give the detail route a name and generate its URL with `url_to()`.
4. Run `php spark routes` and verify the final route table.

## References

- [URI routing](https://codeigniter.com/user_guide/incoming/routing.html)
- [Improved auto routing](https://codeigniter.com/user_guide/incoming/auto_routing_improved.html)
- [Route filters](https://codeigniter.com/user_guide/incoming/filters.html)
