# 2. Project structure and configuration

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Installation and the first route](./01-installation-and-first-route.md) | [Notes index](../README.md) | [Next: Routing and route groups](./03-routing-and-route-groups.md) |

## Know where each kind of file belongs

A Composer app starter separates application code, web-accessible files, framework dependencies, and runtime data.

```text
app/
  Config/
  Controllers/
  Models/
  Views/
public/
  index.php
  assets/
writable/
  cache/
  logs/
  session/
vendor/
  codeigniter4/
.env
env
spark
```

- `app/` contains application classes and configuration.
- `public/` is the web document root and contains the front controller and public assets.
- `writable/` stores files the running app needs to change, such as logs, cache, and sessions.
- `vendor/` contains Composer dependencies and should be recreated with Composer after a fresh clone.
- `spark` is the command-line entry point.
- `env` is the sample environment file. A local `.env` contains values for one environment.

Point the web server at `public/`, not the project root. This keeps application classes, dependencies, and configuration outside the directly served directory.

## Follow the application namespace

The app starter maps the `App` namespace to the `app/` directory. A controller at `app/Controllers/Welcome.php` begins with:

```php
<?php

namespace App\Controllers;

class Welcome extends BaseController
{
    public function index(): string
    {
        return 'Welcome';
    }
}
```

The namespace and directory must match the autoload mapping. File name case matters on many production servers, even when development happens on a case-insensitive Windows file system.

Inspect the active namespace mappings with:

```sh
php spark namespaces
```

Use PSR-4 namespaces for application classes rather than including files manually.

## Use configuration classes for app-wide settings

Framework settings are organized into classes under `app/Config/`. Use these files for stable application configuration such as routes, filters, database defaults, and autoload mappings.

Values that vary by environment belong in environment variables. This lets local and production environments use different database credentials or base URLs without changing application source.

## Keep environment values private

The root `.env` file is loaded automatically. It is based on the tracked `env` template and should remain untracked.

```ini
CI_ENVIRONMENT = development
app.baseURL = 'http://localhost:8080/'
database.default.hostname = localhost
database.default.database = ci_notes
database.default.username = app_user
database.default.password = local_password
database.default.DBDriver = MySQLi
```

Use the keys supported by the project's own `env` template and config classes. Never commit real passwords, API keys, or production credentials. Confirm that `.env` appears in `.gitignore` before publishing the repository.

You can confirm the active environment from the app configuration or set it with Spark:

```sh
php spark env
php spark env development
```

Do not expose detailed development errors or environment dumps on a public production site. They can reveal credentials and internal paths.

## Add an application class

For example, place a small service class at `app/Services/WelcomeMessage.php`:

```php
<?php

namespace App\Services;

class WelcomeMessage
{
    public function text(): string
    {
        return 'A message from an application service.';
    }
}
```

The app autoloader can locate it by its namespace and path. A controller can use it like this:

```php
<?php

namespace App\Controllers;

use App\Services\WelcomeMessage;

class Welcome extends BaseController
{
    public function index(): string
    {
        $message = new WelcomeMessage();

        return $message->text();
    }
}
```

In larger applications, use constructor injection or a configured service when you need to swap implementations or share lifecycle-managed dependencies.

## Configuration checklist

- Keep public files in `public/`.
- Store logs and runtime data in `writable/`.
- Match namespace names and paths exactly.
- Keep local `.env` files out of Git.
- Use production-safe error settings on public deployments.
- Reinstall Composer dependencies after cloning instead of tracking `vendor/`.

## Practice

1. Find the controller namespace mapping in `app/Config/Autoload.php`.
2. Inspect the route, filter, and database config files.
3. Add a non-secret setting to `.env` and read it through configuration.
4. Confirm Git ignores the local `.env` file.

## References

- [Application configuration](https://codeigniter.com/user_guide/general/configuration.html)
- [Autoloading and namespaces](https://codeigniter.com/user_guide/concepts/autoloader.html)
- [Managing applications](https://codeigniter.com/user_guide/general/managing_apps.html)
- [Environment configuration](https://codeigniter.com/user_guide/general/environments.html)
