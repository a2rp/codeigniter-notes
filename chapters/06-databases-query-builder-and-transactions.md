# 6. Databases, Query Builder, and transactions

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Views, layouts, and helpers](./05-views-layouts-and-helpers.md) | [Notes index](../README.md) | [Next: Models and CRUD](./07-models-and-crud.md) |

## Configure a database connection

CodeIgniter keeps database connection settings in `app/Config/Database.php`. Store environment-specific credentials in the untracked `.env` file:

```ini
database.default.hostname = localhost
database.default.database = ci_notes
database.default.username = app_user
database.default.password = local_password
database.default.DBDriver = MySQLi
database.default.charset = utf8mb4
database.default.DBCollat = utf8mb4_general_ci
```

Use the keys and capitalization from the project's config class. Set up a local database and user with only the permissions the app needs. Keep production credentials outside Git.

Connect to the configured default group when needed:

```php
$db = ConfigDatabase::connect();
```

In controllers and models, prefer the framework's model or service patterns when they fit the task. Direct connection code is useful for learning the database layer and for operations that do not belong to a single model.

## Read rows with Query Builder

Query Builder composes a query and handles values in a safer, database-aware way:

```php
$db = ConfigDatabase::connect();

$articles = $db->table('articles')
    ->select('id, title, created_at')
    ->where('status', 'published')
    ->orderBy('created_at', 'DESC')
    ->get()
    ->getResultArray();
```

The table and selected columns are part of the application code. The filter value is passed separately to `where()`. Select only the columns the page needs and add ordering when the display requires it.

## Insert and update rows

Pass an associative array to insert or update:

```php
$builder = $db->table('articles');

$builder->insert([
    'title' => $title,
    'status' => 'draft',
]);

$articleId = $db->insertID();

$db->table('articles')
    ->where('id', $articleId)
    ->update([
        'title' => $newTitle,
    ]);
```

The Query Builder escapes supplied values. Keep table and column names fixed or choose them from an explicit allowlist. Do not build identifiers by concatenating request input.

## Bind values in SQL

When handwritten SQL is the clearest option, bind values separately:

```php
$query = $db->query(
    'SELECT id, title FROM articles WHERE status = ? AND author_id = ?',
    ['published', $authorId],
);

$articles = $query->getResultArray();
```

Bindings help keep values separate from SQL syntax. They do not make an untrusted table or column name safe. Allowlist any dynamic identifier before constructing a query.

## Use a transaction for related changes

A transaction keeps related database operations together. If one write fails, CodeIgniter can roll back the transaction instead of leaving only part of the change.

```php
$db->transStart();

$db->table('orders')->insert([
    'customer_id' => $customerId,
    'total' => $total,
]);

$db->table('inventory')
    ->where('product_id', $productId)
    ->set('quantity', 'quantity - 1', false)
    ->update();

$db->transComplete();

if ($db->transStatus() === false) {
    throw new RuntimeException('The order could not be saved.');
}
```

The raw expression in `set()` is application-owned SQL, not request data. Check affected rows when a missing inventory row must fail the operation. Use a transaction-safe table engine such as InnoDB when using MySQL.

## Query Builder safety and limits

Query Builder escapes values in generated queries, but it is not a substitute for application validation or authorization. Keep a user from requesting another person's record by checking ownership in the query or domain logic. Do not assume a safe query means the requested action is allowed.

## Practice

1. Configure a local database through `.env`.
2. Read a published article list with an explicit field selection and order.
3. Insert a row and store the generated id.
4. Use a transaction for two related writes.
5. Try an invalid record id and handle the missing result.

## References

- [Database configuration](https://codeigniter.com/user_guide/database/configuration.html)
- [Connecting to the database](https://codeigniter.com/user_guide/database/connecting.html)
- [Query Builder](https://codeigniter.com/user_guide/database/query_builder.html)
- [Query bindings](https://codeigniter.com/user_guide/database/queries.html)
- [Transactions](https://codeigniter.com/user_guide/database/transactions.html)
