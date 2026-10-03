# 1. Installation and the first route

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| Notes index | [Notes index](../README.md) | [Next: Project structure and configuration](./02-project-structure-and-configuration.md) |

## Check the environment

CodeIgniter 4 requires PHP 8.2 or newer in the current release line. The required PHP extensions include `intl` and `mbstring`. Composer 2.0.14 or newer is required for the Composer installation method.

Check the installed tools:

```sh
php --version
composer --version
php -m
```

In the PHP module list, confirm `intl` and `mbstring` are enabled. If they are missing, enable the matching extensions in the PHP configuration used by your terminal and web server.

## Create an app starter

Composer's app starter gives you a ready project structure with the framework dependency:

```sh
composer create-project codeigniter4/appstarter ci-notes-playground
cd ci-notes-playground
```

Copy the sample environment file and edit the application URL:

```sh
cp env .env
```

On Windows PowerShell, the equivalent copy command is:

```powershell
Copy-Item -LiteralPath env -Destination .env
```

Open `.env`, find the application URL setting, and set it for the local server:

```ini
app.baseURL = 'http://localhost:8080/'
```

Keep local secrets and environment-specific values out of tracked source files. The `.env` file is intended for the current environment.

## Run the local server

From the project root, start CodeIgniter's development server:

```sh
php spark serve
```

Open `http://localhost:8080/` in a browser. The `spark` script is CodeIgniter's command-line entry point. It can start the local server and run many framework commands.

## Add an explicit route

Open `app/Config/Routes.php` and add a route:

```php
$routes->get('hello', static function () {
    return 'Hello from CodeIgniter';
});
```

Visit `http://localhost:8080/hello`. The `get` method matches an HTTP GET request for the `hello` path. CodeIgniter 4 does not enable legacy automatic routing by default, so explicit routes make the public request map easier to review.

## Return a controller response

As the route grows, move its work into a controller. Create `app/Controllers/Welcome.php`:

```php
<?php

namespace App\Controllers;

class Welcome extends BaseController
{
    public function index(): string
    {
        return 'Welcome to my CodeIgniter notes app.';
    }
}
```

Register it in `app/Config/Routes.php`:

```php
$routes->get('welcome', 'Welcome::index');
```

A controller action handles the matched request and returns a response. Keep the first action small. Later chapters separate request handling, view rendering, and data access.

## Understand the first request

The browser sends a request for a URL. CodeIgniter matches it against the route table, calls the selected controller or closure, then sends the returned response back to the browser. The `public` directory is the web document root, while application code and writable runtime data belong elsewhere.

## Common setup problems

- **PHP reports the wrong version:** check which PHP executable your shell finds with `php --version`.
- **An extension is missing:** enable `intl` or `mbstring` in the active PHP configuration and restart the server.
- **Composer cannot create the project:** confirm Composer is current and the terminal has network access.
- **The route returns a 404:** confirm the route is in the active routes file and the request path and HTTP method match.
- **The URL is wrong:** set `app.baseURL` in `.env` and ensure the value ends with a slash.

## Practice

1. Add a `good-morning` route that returns a short message.
2. Add a controller action for a `contact` path.
3. Change the response and refresh the page.
4. Run `php spark routes` to inspect registered routes.

## References

- [CodeIgniter 4 requirements](https://codeigniter.com/user_guide/intro/requirements.html)
- [Composer installation](https://codeigniter.com/user_guide/installation/installing_composer.html)
- [Running the app](https://codeigniter.com/user_guide/installation/running.html)
- [URI routing](https://codeigniter.com/user_guide/incoming/routing.html)
