# CodeIgniter 4 Study Notes

These are my personal study notes from learning and working with CodeIgniter 4. I use this repository to keep the core ideas, patterns, and PHP examples I want to revisit in one place.

The notes begin with a small working app and build toward routing, controllers, views, databases, validation, security, APIs, testing, and deployment. Each chapter explains the purpose of the feature, shows practical PHP examples, and points to the current official documentation for version-specific details.

## What I am covering

This is a focused set of notes for building and maintaining CodeIgniter 4 applications. It covers the framework's request flow, MVC structure, Spark commands, configuration, data access, and the checks needed before a production release.

The examples use PHP and Composer. CodeIgniter and PHP requirements can change, so check the official requirements page when setting up a new project.

## Chapters

01. [Installation and the first route](./chapters/01-installation-and-first-route.md)  
   Set up PHP and Composer, create a CodeIgniter 4 app starter, run Spark, and return a first response.

02. [Project structure and configuration](./chapters/02-project-structure-and-configuration.md)  
   Understand app, public, writable, environment settings, namespaces, autoloading, and configuration.

03. [Routing and route groups](./chapters/03-routing-and-route-groups.md)  
   Define explicit routes, HTTP verbs, route parameters, groups, filters, and named routes.

04. [Controllers, requests, and responses](./chapters/04-controllers-requests-and-responses.md)  
   Handle request data, return views or responses, use redirects, and separate HTTP work from domain logic.

05. [Views, layouts, and helpers](./chapters/05-views-layouts-and-helpers.md)  
   Build reusable view layouts, pass safe data, escape output, use helpers, and handle form values.

06. [Databases, Query Builder, and transactions](./chapters/06-databases-query-builder-and-transactions.md)  
   Configure database connections, compose safe queries, bind values, and protect multi-step changes.

07. [Models and CRUD](./chapters/07-models-and-crud.md)  
   Use CodeIgniter models, allowed fields, validation, pagination, and clear data access methods.

08. [Migrations and seeders](./chapters/08-migrations-and-seeders.md)  
   Version database schema changes, seed repeatable data, and run or roll back migrations.

09. [Validation and form errors](./chapters/09-validation-and-form-errors.md)  
   Validate server input with strict rules, return field errors, preserve safe form values, and handle JSON data.

10. [Sessions, cookies, and flash data](./chapters/10-sessions-cookies-and-flash-data.md)  
   Manage session state, use flash messages, configure cookies, and understand lifecycle and security limits.

11. [Filters, security, and file uploads](./chapters/11-filters-security-and-file-uploads.md)  
   Apply route filters, CSRF protection, output escaping, secure uploads, and access checks.

12. [Building REST APIs](./chapters/12-building-rest-apis.md)  
   Design resource routes, validate JSON, return consistent status codes, and format API responses.

13. [Services, events, and Spark commands](./chapters/13-services-events-and-spark-commands.md)  
   Use shared services, register events, create CLI commands, and keep application bootstrapping understandable.

14. [Errors, logging, and caching](./chapters/14-errors-logging-and-caching.md)  
   Configure environment-safe errors, record useful logs, and cache work that is safe to reuse.

15. [Testing controllers and models](./chapters/15-testing-controllers-and-models.md)  
   Write focused unit and feature tests, use test responses, isolate data, and run the test suite.

16. [Deployment and production checklist](./chapters/16-deployment-and-production-checklist.md)  
   Configure production settings, document server requirements, run Composer production installs, and verify releases.

## Reference chapters

- [All code samples](./chapters/98-all-code-samples.md) collects the complete examples from the chapters.
- [Complete questions and answers](./chapters/99-complete-q-and-a.md) gathers review questions and clear answers.

## Using these notes

Read in order for a guided study path, or use the chapter links to revisit one topic. Try each example in a local app, inspect the request and response, and test changes with your own data. Keep credentials out of source control and use environment-specific settings for local and production systems.

## Main references

- [CodeIgniter 4 user guide](https://codeigniter.com/user_guide/)
- [CodeIgniter 4 installation](https://codeigniter.com/user_guide/installation/index.html)
- [CodeIgniter 4 requirements](https://codeigniter.com/user_guide/intro/requirements.html)
- [CodeIgniter 4 source repository](https://github.com/codeigniter4/CodeIgniter4)

## License

These notes are available under the [MIT License](./LICENSE).

## Links

- Portfolio: https://www.ashishranjan.net
- GitHub: https://github.com/a2rp
- CodePen: https://codepen.io/ash1198
- LinkedIn: https://www.linkedin.com/in/aashishranjan
- Facebook: https://www.facebook.com/theash.ashish/
- YouTube: https://www.youtube.com/@ashishranjan-ashz?sub_confirmation=1
- Email: mailto:ash.ranjan09@gmail.com

## Support

- Support: https://a2rp-donation-page.netlify.app/
- Buy Me a Coffee: https://buymeacoffee.com/ashishranjan
- Patreon: https://www.patreon.com/ashishranjan
