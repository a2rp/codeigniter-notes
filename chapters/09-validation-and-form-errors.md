# 9. Validation and form errors

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Migrations and seeders](./08-migrations-and-seeders.md) | [Notes index](../README.md) | [Next: Sessions, cookies, and flash data](./10-sessions-cookies-and-flash-data.md) |

## Validate on the server

Browser validation improves the form experience, but it does not protect the application. A request can be sent without the browser form, so validate every submitted value on the server before saving it or using it in a sensitive operation.

Validation checks data against rules. It does not clean or rewrite the data. Escape values when displaying them and use the validated result when continuing the request.

## Validate only the expected POST fields

A controller can collect the fields it expects and call `validateData()`:

```php
$data = [
    'title' => $this->request->getPost('title'),
    'body' => $this->request->getPost('body'),
];

$rules = [
    'title' => 'required|max_length[160]',
    'body' => 'required',
];

if (! $this->validateData($data, $rules)) {
    return view('articles/new', [
        'errors' => $this->validator->getErrors(),
        'values' => $data,
    ]);
}

$validData = $this->validator->getValidated();
```

Use `getPost()` when the endpoint expects form POST values. Avoid the legacy `getVar()` shortcut for new code because it can combine several input sources. `getValidated()` returns only the fields that had validation rules, which helps prevent unvalidated request values from reaching a write operation.

## Show clear field errors

A view can display an error next to its field:

```php
<label for="title">Title</label>
<input
    id="title"
    name="title"
    value="<?= esc($values['title'] ?? '') ?>"
    required
>

<?php if (isset($errors['title'])): ?>
    <p role="alert"><?= esc($errors['title']) ?></p>
<?php endif ?>
```

Give the user a clear message that describes how to correct the value. Escape both the submitted value and the message before placing them in HTML. A validation failure should keep valid fields populated so the user does not need to re-enter everything.

## Store reusable rules

When rules apply to multiple controller actions, place them in a group in `app/Config/Validation.php`:

```php
public array $article = [
    'title' => 'required|max_length[160]',
    'body' => 'required',
    'status' => 'required|in_list[draft,published]',
];
```

Then call `validateData()` with the group name:

```php
if (! $this->validateData($data, 'article')) {
    return view('articles/new', [
        'errors' => $this->validator->getErrors(),
        'values' => $data,
    ]);
}
```

For field-specific messages, define custom messages alongside the rules. Keep rule groups named for the data they validate.

## Choose useful rules

Common rule choices include:

- `required` for a value that must be present.
- `permit_empty` when an empty string is a valid choice.
- `max_length[160]` to cap text size.
- `valid_email` for email syntax.
- `in_list[draft,published]` to restrict a value to known options.
- `is_natural_no_zero` for a positive whole-number identifier.

Use strict rules for new projects. They avoid implicit type conversion and are important when validating non-string input such as decoded JSON. A rule should express the data the application expects, not just the input widget's constraints.

## Protect database invariants too

Validation gives a useful response to the user, but it does not replace database constraints. Use unique indexes, foreign keys, and required columns to protect invariants when concurrent requests reach the server.

For updates, validate the identifier itself and make sure the current user is permitted to update that record. Do not trust a hidden input or route parameter as proof of ownership.

## Practice

1. Validate an article title and body before saving.
2. Add a named rule group for a second form.
3. Show a specific error next to the invalid field.
4. Try a JSON request containing a number where a string is expected.
5. Confirm that only validated fields reach the model.

## References

- [Validation library](https://codeigniter.com/user_guide/libraries/validation.html)
- [Controller validation](https://codeigniter.com/user_guide/incoming/controllers.html#validating-data)
- [CodeIgniter first app: create items](https://codeigniter.com/user_guide/guides/first-app/create_news_items.html)
