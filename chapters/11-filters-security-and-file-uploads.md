# 11. Filters, security, and file uploads

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Sessions, cookies, and flash data](./10-sessions-cookies-and-flash-data.md) | [Notes index](../README.md) | [Next: Building REST APIs](./12-building-rest-apis.md) |

## In this chapter

Use filters around request handling, protect state-changing forms with CSRF checks, escape untrusted output, and validate files before storing them.

## Filters run around a request

A filter runs before a controller, after it, or both. Use one for cross-cutting checks such as authentication, role access, request throttling, or security headers. Keep business rules in the controller or a service.

CodeIgniter includes filters such as `csrf`, `cors`, `honeypot`, `secureheaders`, and `forcehttps`. The aliases are configured in `app/Config/Filters.php`. Inspect the starter project's file before changing it because it already contains framework aliases and environment settings.

A route can require a filter:

```php
$routes->get('account', 'AccountController::index', [
    'filter' => 'session-auth',
]);
```

The filter alias must exist in `Config\\Filters::$aliases`. A custom authentication filter can redirect guests before the controller is reached:

```php
<?php

namespace App\\Filters;

use CodeIgniter\\Filters\\FilterInterface;
use CodeIgniter\\HTTP\\RequestInterface;
use CodeIgniter\\HTTP\\ResponseInterface;

class SessionAuth implements FilterInterface
{
    public function before(RequestInterface $request, $arguments = null)
    {
        if (session()->get('userId') === null) {
            return redirect()->to('/login');
        }
    }

    public function after(
        RequestInterface $request,
        ResponseInterface $response,
        $arguments = null
    ) {
    }
}
```

Register the class in the aliases array:

```php
public $aliases = [
    // Keep the aliases supplied by the starter project.
    'session-auth' => \\App\\Filters\\SessionAuth::class,
];
```

The array above shows the new entry only. Merge it into the existing aliases rather than replacing framework aliases. Route-level filters make the protected routes visible and testable. Use groups when a set of routes shares the same requirement.

## CSRF protection for state changes

Cross-Site Request Forgery tricks a signed-in browser into sending a request the user did not intend. Enable the built-in CSRF filter for browser forms that change data. CSRF checks cover POST, PUT, PATCH, and DELETE requests.

In `app/Config/Filters.php`, keep the existing configuration and enable the CSRF filter in the global before list:

```php
public $globals = [
    'before' => [
        'csrf',
    ],
];
```

A form can include the token with the security helper:

```php
<form action="/articles" method="post">
    <?= csrf_field() ?>

    <label for="title">Title</label>
    <input id="title" name="title" value="<?= old('title') ?>">

    <button type="submit">Save</button>
</form>
```

If the Form Helper's `form_open()` is used with the CSRF filter enabled, it inserts the hidden token. Do not add a second token field to the same form. Do not exempt a route just to silence a failed check. If an external webhook cannot send a browser token, exempt only that exact endpoint and verify its signature or shared secret independently.

Use explicit HTTP verbs in routes and keep legacy auto-routing disabled when relying on method-based security rules. A CSRF token does not replace authentication, authorization, or input validation.

## Escape values when rendering HTML

Validation checks whether input is acceptable. Escaping prevents a value from being interpreted as markup or script when displayed. Escape user-controlled text at the output boundary:

```php
<h1><?= esc($article['title']) ?></h1>
<p><?= esc($article['summary']) ?></p>
```

For a value inserted into an HTML attribute, use the HTML context:

```php
<input
    name="title"
    value="<?= esc(old('title', $article['title'] ?? ''), 'attr') ?>"
>
```

Do not print raw request values, database values originally supplied by users, or exception messages into HTML. Avoid building HTML by concatenating untrusted strings. If a field intentionally accepts rich HTML, sanitize it with a well-maintained allowlist before saving or rendering; `esc()` is not an HTML sanitizer.

## Validate uploaded files on the server

The browser's `accept` attribute helps people choose a file, but it is not a security check. Enforce required status, size, and allowed content types on the server. Uploaded-file validation uses file-specific rules such as `uploaded`, `max_size`, `mime_in`, and `is_image`.

For an image form, use multipart encoding:

```php
<form action="/profile/avatar" method="post" enctype="multipart/form-data">
    <?= csrf_field() ?>
    <input type="file" name="avatar" accept="image/jpeg,image/png,image/webp">
    <button type="submit">Upload</button>
</form>
```

A controller can validate the upload and store it under a generated name:

```php
<?php

namespace App\\Controllers;

class AvatarController extends BaseController
{
    public function store()
    {
        $rules = [
            'avatar' => [
                'label' => 'Avatar',
                'rules' => [
                    'uploaded[avatar]',
                    'is_image[avatar]',
                    'mime_in[avatar,image/jpeg,image/png,image/webp]',
                    'max_size[avatar,2048]',
                    'max_dims[avatar,2000,2000]',
                ],
            ],
        ];

        if (! $this->validateData([], $rules)) {
            return redirect()->back()
                ->withInput()
                ->with('errors', $this->validator->getErrors());
        }

        $file = $this->request->getFile('avatar');

        if (! $file->isValid() || $file->hasMoved()) {
            return redirect()->back()
                ->withInput()
                ->with('error', 'The image could not be uploaded.');
        }

        $storedName = $file->getRandomName();
        $file->move(WRITEPATH . 'uploads/avatars', $storedName);

        // Save $storedName against the signed-in user's record.
        return redirect()->to('/profile')
            ->with('message', 'Avatar uploaded.');
    }
}
```

The first argument to `validateData()` is empty because the file rules read the upload from the request. If the form also has ordinary fields, validate those separately or pass their values along with the file rules. Show validation messages with escaped output.

CodeIgniter stores the file under `writable/uploads` in this example, outside the public document root. This is suitable for private files. For public files, use a deliberate serving route or storage location that cannot execute uploaded scripts. Never trust the submitted filename or extension for a storage path. Use generated names, enforce size and type limits, and consider malware scanning for files that other users can download.

## Apply a security checklist

- Keep framework and PHP versions supported, and install dependencies from trusted sources.
- Keep secrets in environment configuration and out of Git.
- Validate every request on the server, including API and upload requests.
- Authorize access to each record, not only to the page or route.
- Use CSRF protection for browser requests that change state.
- Escape values for their output context.
- Limit upload size and type, generate stored names, and keep private files outside the public directory.
- Return generic error messages to users and write diagnostic details to protected logs.
- Use HTTPS in production and set secure cookie attributes.

## Common mistakes

- Treating a hidden field, disabled button, or route name as authorization.
- Accepting a file because its name ends in `.png`.
- Saving an uploaded file under its original client-provided name.
- Exempting a broad path from CSRF checks.
- Printing database text without escaping it.
- Replacing the entire filter aliases array and accidentally removing built-in aliases.
- Storing private uploads inside `public/` and exposing them directly.

## Official references

- [Controller Filters](https://codeigniter.com/user_guide/incoming/filters.html)
- [Security and CSRF](https://codeigniter.com/user_guide/libraries/security.html)
- [Working with Uploaded Files](https://codeigniter.com/user_guide/libraries/uploaded_files.html)
- [Validation rules](https://codeigniter.com/user_guide/libraries/validation.html)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Sessions, cookies, and flash data](./10-sessions-cookies-and-flash-data.md) | [Notes index](../README.md) | [Next: Building REST APIs](./12-building-rest-apis.md) |
