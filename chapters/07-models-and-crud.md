# 7. Models and CRUD

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Databases, Query Builder, and transactions](./06-databases-query-builder-and-transactions.md) | [Notes index](../README.md) | [Next: Migrations and seeders](./08-migrations-and-seeders.md) |

## Give a model responsibility for one table

CodeIgniter models provide common database operations and can apply validation before writing data. Models are usually kept in `app/Models/` and extend `CodeIgniter\Model`.

Create `app/Models/ArticleModel.php`:

```php
<?php

namespace App\Models;

use CodeIgniter\Model;

class ArticleModel extends Model
{
    protected $table = 'articles';
    protected $primaryKey = 'id';
    protected $allowedFields = ['title', 'body', 'status'];
    protected $useTimestamps = true;

    protected $validationRules = [
        'title' => 'required|max_length[160]',
        'body' => 'required',
        'status' => 'required|in_list[draft,published]',
    ];
}
```

The table and primary key identify the database table and row key. The `allowedFields` list is a mass-assignment safety boundary: extra keys passed to model writes are discarded. Do not add the primary key or sensitive ownership fields unless the application truly allows the caller to set them.

When `useTimestamps` is enabled, the table must contain the configured timestamp fields, usually `created_at` and `updated_at`.

## Read records

Use the model for ordinary table operations:

```php
$model = model(ArticleModel::class);

$article = $model->find($id);
$recentArticles = $model
    ->where('status', 'published')
    ->orderBy('created_at', 'DESC')
    ->findAll(10);
```

Handle a missing row before using it. For private records, include the current user's ownership condition in the query or apply an authorization check before returning the row.

## Validate and create a record

A model can validate the fields it is configured to accept:

```php
$data = [
    'title' => $this->request->getPost('title'),
    'body' => $this->request->getPost('body'),
    'status' => 'draft',
];

$model = model(ArticleModel::class);

if ($model->save($data) === false) {
    return view('articles/new', [
        'errors' => $model->errors(),
        'values' => $data,
    ]);
}

return redirect()
    ->to('/articles')
    ->with('message', 'Draft saved.');
```

Only proceed after a successful write. Show the model validation errors next to the form fields and preserve safe values so the user can correct a mistake without entering everything again.

For security-sensitive workflows, validate request data explicitly in the controller, then pass only the validated allowlisted fields to the model. The validation chapter covers this pattern in more detail.

## Update and delete by primary key

Pass the record id explicitly when updating or deleting:

```php
$model->update($id, [
    'title' => $validatedData['title'],
    'body' => $validatedData['body'],
]);

$model->delete($id);
```

Before changing or deleting a record, verify that the current user can perform that action on that record. Knowing an id is not proof of authorization.

Soft deletes are an option when the model and table are configured with a deleted timestamp field. They keep a row out of ordinary model reads while retaining it in the database. Use them only when the product needs recovery or an audit trail, and define how long deleted data should remain.

## Paginate a list

Avoid loading an unbounded table into a page. The model's `paginate()` method fetches a page and prepares the pager:

```php
$model = model(ArticleModel::class);
$articles = $model
    ->where('status', 'published')
    ->orderBy('created_at', 'DESC')
    ->paginate(10);

return view('articles/index', [
    'articles' => $articles,
    'pager' => $model->pager,
]);
```

In the view, render the pager links with the configured pagination view. Check the official pagination guide when you need custom page groups or multiple paginators.

## Model design guidelines

- Keep one model centered on one primary table.
- Set `allowedFields` explicitly.
- Use validation rules that match the data being saved.
- Use a model for common CRUD and Query Builder for a more custom query.
- Check ownership and permission separately from input validation.
- Use pagination for lists that can grow over time.
- Do not skip model validation just to make a failing save succeed.

## Practice

1. Create an article table with id, title, body, status, and timestamp columns.
2. Add an `ArticleModel` with allowed fields and validation rules.
3. List only published articles and order them newest first.
4. Add pagination and test a missing record.
5. Confirm a caller cannot set a field that is absent from `allowedFields`.

## References

- [Using CodeIgniter's Model](https://codeigniter.com/user_guide/models/model.html)
- [Model validation](https://codeigniter.com/user_guide/models/model.html#validation)
- [Pagination](https://codeigniter.com/user_guide/libraries/pagination.html)
- [CodeIgniter first app: create items](https://codeigniter.com/user_guide/guides/first-app/create_news_items.html)
