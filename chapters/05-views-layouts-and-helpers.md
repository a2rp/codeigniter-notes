# 5. Views, layouts, and helpers

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Controllers, requests, and responses](./04-controllers-requests-and-responses.md) | [Notes index](../README.md) | [Next: Databases, Query Builder, and transactions](./06-databases-query-builder-and-transactions.md) |

## Return data to a view

Views are PHP files in `app/Views/`. A controller can pass an associative array to the `view()` function:

```php
return view('articles/show', [
    'title' => $article['title'],
    'article' => $article,
]);
```

The keys become variables in the view. Use clear names and pass only the data the page needs. Views should format and display data rather than run database queries or decide application permissions.

## Escape values when writing HTML

PHP does not automatically escape values printed in a view. Escape untrusted values for the output context:

```php
<h1><?= esc($title) ?></h1>
<p><?= esc($article['body']) ?></p>
```

The default `esc()` context is HTML. For values placed inside JavaScript, CSS, or an HTML attribute, use the correct documented context. Do not print submitted or database text directly into a page.

## Create a shared layout

A layout contains the page structure that multiple views reuse. Create `app/Views/layouts/main.php`:

```php
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title><?= esc($this->renderSection('title')) ?></title>
</head>
<body>
    <header>
        <a href="<?= site_url('/') ?>">My application</a>
    </header>

    <main>
        <?= $this->renderSection('content') ?>
    </main>
</body>
</html>
```

The `renderSection()` calls mark locations where child views supply content. A child view extends the layout and defines matching sections:

```php
<?= $this->extend('layouts/main') ?>

<?= $this->section('title') ?>
Articles
<?= $this->endSection() ?>

<?= $this->section('content') ?>
<h1>Articles</h1>
<p>Browse the latest articles.</p>
<?= $this->endSection() ?>
```

Return the child view from a controller with `return view('articles/index')`. CodeIgniter detects that it extends the layout and combines the sections.

## Reuse a partial

Use partial views for repeated page pieces such as a navigation block or a small form. With the layout renderer, include a partial with:

```php
<?= $this->include('shared/navigation') ?>
```

Keep partials small and pass any required variables explicitly when the calling view does not already provide them.

## Use the Form Helper

Load the form helper where the form is rendered or configure it for autoloading:

```php
<?php helper('form'); ?>

<?= form_open('/articles') ?>
    <label for="title">Title</label>
    <input
        id="title"
        name="title"
        value="<?= set_value('title') ?>"
        required
    >

    <button type="submit">Save article</button>
<?= form_close() ?>
```

When the CSRF filter is enabled for the form page, `form_open()` can add its hidden token. For a manually written form, use `csrf_field()` when required by the filter configuration. Do not print a second token if the helper already inserted it.

`set_value()` restores a submitted value after validation errors and escapes the value by default. Keep labels connected to their inputs with matching `for` and `id` values.

## Use helpers for repeated framework work

Helpers are collections of common functions, such as URL, form, or filesystem utilities. Load only what the code needs:

```php
helper(['form', 'url']);
```

Use `site_url()` or `url_to()` when building application links instead of hard-coding a host name. Helpers keep common operations consistent, but application rules belong in controllers, models, or services.

## View guidelines

- Escape dynamic text with `esc()`.
- Keep data access out of views.
- Use one layout for repeated document structure.
- Give every form control a visible label.
- Reuse partials for repeated markup, not for large hidden application flows.
- Keep CSRF configuration aligned with how the form is rendered.

## Practice

1. Add a layout with a title and content section.
2. Create a page view that extends the layout.
3. Pass a user-provided title and print it through `esc()`.
4. Add a form helper input and confirm the value returns after validation fails.

## References

- [View layouts](https://codeigniter.com/user_guide/outgoing/view_layouts.html)
- [View renderer and escaping](https://codeigniter.com/user_guide/outgoing/view_renderer.html)
- [Form Helper](https://codeigniter.com/user_guide/helpers/form_helper.html)
- [Helper functions](https://codeigniter.com/user_guide/general/helpers.html)
