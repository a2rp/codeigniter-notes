# 16. Deployment and production checks

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Testing controllers and models](./15-testing-controllers-and-models.md) | [Notes index](../README.md) | [Next: All code samples](./98-all-code-samples.md) |

## In this chapter

Prepare a CodeIgniter app for a real server, keep private files out of the public web root, and release changes in a controlled order.

## Point the web server at public/

The project's `public/` directory is the web document root. Configure Apache or Nginx so requests cannot directly access `app/`, `writable/`, `.env`, or `vendor/`. Do not point the domain at the project root.

The public directory contains the front controller and public assets. The rest of the application should live outside the document root. If a hosting provider requires a `public_html` folder, follow its supported layout and move only public files into that directory while keeping application code and secrets in a protected location.

## Set production configuration

Use the production environment on the server. Keep the environment file and credentials private, and provide a valid `app.baseURL`, encryption key, database connection, and any mail or external service settings.

Do not commit `.env` or production credentials. Keep `.env.example` limited to variable names and safe placeholder values. Use the deployment platform's secret manager where available. Rotate any secret that was accidentally published.

Install production dependencies from the lock file:

```sh
composer install --no-dev --optimize-autoloader
```

Run the command in the release directory with the required PHP version and extensions. Commit `composer.lock` for an application so deployments use the dependency versions that were tested.

## Configure writable storage

The web server account must be able to write to `writable/` for logs, cache, sessions, and uploads. Grant only the access the app needs. Do not make the whole project writable and do not use broad permissions such as 777.

Confirm that upload and log directories are not publicly served. Keep private user files outside `public/` and deliver them through a route that checks authorization.

## Release database changes safely

Back up important data before applying schema changes. Review migrations for destructive operations and test them against a copy of production-like data. Apply migrations as a planned release step:

```sh
php spark migrate
php spark migrate:status
```

Run migrations with the deployment's production environment and database connection. Ensure that the application version and schema changes are compatible during rollout, especially when multiple app instances are running. Prefer expand-and-contract changes for a zero-downtime release: add compatible columns first, deploy code that can handle both states, migrate data, and remove old fields in a later release.

Never run `migrate:refresh` against production. That command rolls migrations back and reapplies them, which can destroy production data.

## Check PHP and the application

Run the PHP configuration check in the release environment:

```sh
php spark phpini:check
php spark routes
```

Then run the automated checks, open the site through HTTPS, and verify the most important user flows. Check authentication, a database read and write, a file upload if used, logs, redirects, and a not-found page. Confirm that production errors are generic and that no debug toolbar or development endpoint is exposed.

## Production release checklist

- The server uses a supported PHP version and required extensions.
- The web document root points to `public/`.
- The production environment and correct base URL are configured.
- Secret values are stored outside Git and are not web accessible.
- Composer dependencies match the committed lock file.
- The server account can write to `writable/` without broad permissions.
- HTTPS and secure cookie settings are enabled.
- Database backups and migration steps are ready.
- Tests pass with the intended PHP version and test database.
- Private routes check both identity and record-level permission.
- Uploads have size and type limits and are not executable.
- Logs are private, useful, and free of credentials.
- Monitoring and a rollback plan are available.

## Common mistakes

- Pointing the domain at the repository root.
- Copying a development `.env` file with real credentials into a public directory.
- Installing dependencies without the lock file or with development packages in production.
- Making every project file writable by the web server.
- Applying a destructive migration without a backup.
- Testing a deployment by running `migrate:refresh` on the live database.
- Enabling detailed errors to diagnose a production issue.
- Assuming that a successful deployment command means the app is healthy.

## Official references

- [Deployment](https://codeigniter.com/user_guide/installation/deployment.html)
- [Running your app](https://codeigniter.com/user_guide/installation/running.html)
- [Server requirements](https://codeigniter.com/user_guide/intro/requirements.html)
- [Application structure](https://codeigniter.com/user_guide/concepts/structure.html)
- [Composer installation](https://codeigniter.com/user_guide/installation/installing_composer.html)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Testing controllers and models](./15-testing-controllers-and-models.md) | [Notes index](../README.md) | [Next: All code samples](./98-all-code-samples.md) |
