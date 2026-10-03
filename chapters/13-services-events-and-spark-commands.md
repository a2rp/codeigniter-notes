# 13. Services, events, and Spark commands

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Building REST APIs](./12-building-rest-apis.md) | [Notes index](../README.md) | [Next: Errors, logging, and caching](./14-errors-logging-and-caching.md) |

## In this chapter

Use services to create shared dependencies, events to announce things that happened, and Spark commands for scheduled or operator-run tasks.

## Services provide dependencies

A service is a named way to create or retrieve an object. CodeIgniter uses services for core objects such as the router and logger. A service can hide the construction details of an application class and provide a shared instance when that is useful.

Keep a service focused on creating the dependency. Put its actual work in a normal class so it can be tested directly:

```php
<?php

namespace App\\Services;

use App\\Models\\ArticleModel;

class ArticlePublisher
{
    public function __construct(
        private ?ArticleModel $articles = null
    ) {
        $this->articles ??= model(ArticleModel::class);
    }

    public function publish(int $id): bool
    {
        return $this->articles->update($id, ['status' => 'published']);
    }
}
```

Register the factory in `app/Config/Services.php`, keeping the existing framework methods intact:

```php
<?php

namespace Config;

use App\\Services\\ArticlePublisher;
use CodeIgniter\\Config\\BaseService;

class Services extends BaseService
{
    public static function articlePublisher(bool $getShared = true): ArticlePublisher
    {
        if ($getShared) {
            return static::getSharedInstance('articlePublisher');
        }

        return new ArticlePublisher();
    }
}
```

Retrieve the service where it is needed:

```php
$publisher = service('articlePublisher');
$publisher->publish($articleId);
```

Since CodeIgniter 4.5, the global `service()` function is recommended when retrieving a service without parameters. Use a shared instance when the dependency is safe to reuse. Avoid putting request-specific state inside a shared service. A factory that accepts runtime arguments should be created with the appropriate parameters rather than hidden behind a shared instance.

## Events announce completed work

An event lets one part of the app announce that something happened, while listeners handle optional side effects. Register listeners in `app/Config/Events.php`:

```php
<?php

use CodeIgniter\\Events\\Events;

Events::on('article.published', static function (int $articleId): void {
    log_message('info', 'Article published: {id}', ['id' => $articleId]);
});
```

Trigger the event after the primary change succeeds:

```php
use CodeIgniter\\Events\\Events;

if ($publisher->publish($articleId)) {
    Events::trigger('article.published', $articleId);
}
```

Events can also run at framework lifecycle points. A custom event name such as `article.published` belongs to the application; choose a clear name and agree on its arguments with each listener.

Do not move a required database update into an optional listener. If the listener fails, the primary action may already have completed. For important multi-step work, use a transaction or a durable job queue. Event listeners are useful for decoupled side effects such as logging, analytics, or notification requests.

## Spark runs framework commands

Spark runs from the project root:

```sh
php spark
php spark list
php spark help migrate
php spark routes
php spark migrate
php spark db:seed ArticleSeeder
```

The available command list depends on the installed framework and project packages. Check `php spark help <command>` before using flags, especially when following notes written for a different version.

Create a custom command:

```sh
php spark make:command SendDigest
```

A command class extends `BaseCommand` and implements `run()`:

```php
<?php

namespace App\\Commands;

use CodeIgniter\\CLI\\BaseCommand;
use CodeIgniter\\CLI\\CLI;

class SendDigest extends BaseCommand
{
    protected $group = 'Email';
    protected $name = 'email:send-digest';
    protected $description = 'Send the daily article digest.';
    protected $usage = 'email:send-digest [--dry-run]';

    public function run(array $params)
    {
        $dryRun = in_array('--dry-run', $params, true);

        // Call an application service to build and send the digest.
        CLI::write(
            $dryRun ? 'Digest preview completed.' : 'Digest sent.',
            'green'
        );
    }
}
```

Keep the command thin. Parse command arguments, call a service, and report the result. Put query and delivery behavior in classes that can also be tested outside the CLI. Avoid exposing sensitive values in terminal output or logs.

## Schedule commands outside web requests

A server scheduler can run a Spark command at a fixed time. For example, a Linux cron entry can change to the project directory and invoke PHP:

```cron
0 7 * * * cd /var/www/app && /usr/bin/php spark email:send-digest
```

Use the correct project path, PHP executable, timezone, and system account for the server. Keep secrets out of the command line because process lists and logs may reveal arguments. Make scheduled tasks safe to run again after a timeout, and record enough information to diagnose failures.

## Common mistakes

- Using a service as a large container for unrelated application logic.
- Keeping user or request-specific values in a shared service.
- Triggering a side effect before the database write succeeds.
- Depending on an event listener for a change that must happen atomically.
- Putting all command behavior in the command class.
- Scheduling commands with an environment that differs from the app's PHP version or working directory.
- Printing tokens, passwords, or private data in command output.

## Official references

- [Services](https://codeigniter.com/user_guide/concepts/services.html)
- [Events](https://codeigniter.com/user_guide/extending/events.html)
- [Spark commands](https://codeigniter.com/user_guide/cli/spark_commands.html)
- [Creating Spark commands](https://codeigniter.com/user_guide/cli/cli_commands.html)
- [CLI overview](https://codeigniter.com/user_guide/cli/cli_overview.html)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Building REST APIs](./12-building-rest-apis.md) | [Notes index](../README.md) | [Next: Errors, logging, and caching](./14-errors-logging-and-caching.md) |
