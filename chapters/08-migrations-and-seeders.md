# 8. Migrations and seeders

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Models and CRUD](./07-models-and-crud.md) | [Notes index](../README.md) | [Next: Validation and form errors](./09-validation-and-form-errors.md) |

## Why migrations matter

A migration records a database schema change in source control. It gives the app a repeatable way to create or alter tables in local, test, and production environments. Keep schema changes in migrations rather than asking each developer to edit a database manually.

Generate one from the project root:

```sh
php spark make:migration CreateArticlesTable
```

The command creates a timestamped file in `app/Database/Migrations/`. The class name and namespace are generated for you.

## Create a table

A migration uses Forge to define fields and keys:

```php
<?php

namespace App\Database\Migrations;

use CodeIgniter\Database\Migration;

class CreateArticlesTable extends Migration
{
    public function up()
    {
        $this->forge->addField([
            'id' => [
                'type' => 'INT',
                'constraint' => 10,
                'unsigned' => true,
                'auto_increment' => true,
            ],
            'title' => [
                'type' => 'VARCHAR',
                'constraint' => 160,
            ],
            'body' => [
                'type' => 'TEXT',
            ],
            'status' => [
                'type' => 'VARCHAR',
                'constraint' => 20,
            ],
            'created_at' => [
                'type' => 'DATETIME',
                'null' => true,
            ],
            'updated_at' => [
                'type' => 'DATETIME',
                'null' => true,
            ],
        ]);

        $this->forge->addPrimaryKey('id');
        $this->forge->createTable('articles');
    }

    public function down()
    {
        $this->forge->dropTable('articles');
    }
}
```

The `up()` method applies the change. The `down()` method describes how to reverse it. Keep those operations paired and review the rollback before running it against a database with data.

## Run and inspect migrations

Apply pending migrations:

```sh
php spark migrate
```

Inspect migration status:

```sh
php spark migrate:status
```

In a development database, a rollback can return to an earlier state:

```sh
php spark migrate:rollback
```

A rollback can remove a table or column and its data. Confirm the selected database group and backup important data before applying schema changes to a shared or production system. Avoid using refresh commands casually because they roll the schema back and apply it again.

## Add development data with a seeder

A seeder inserts sample or reference data that is separate from schema definition. Generate a file:

```sh
php spark make:seeder ArticleSeeder
```

A minimal seeder can add records through the database connection:

```php
<?php

namespace App\Database\Seeds;

use CodeIgniter\Database\Seeder;

class ArticleSeeder extends Seeder
{
    public function run()
    {
        $this->db->table('articles')->insertBatch([
            [
                'title' => 'First study note',
                'body' => 'A short example record.',
                'status' => 'published',
            ],
            [
                'title' => 'Draft study note',
                'body' => 'Another example record.',
                'status' => 'draft',
            ],
        ]);
    }
}
```

Run the seeder by class name:

```sh
php spark db:seed ArticleSeeder
```

A simple insert seeder may add duplicate rows when run more than once. For repeatable seed data, use a stable unique key and an intentional update or upsert strategy, or reset only a disposable development database.

## Migration and seed responsibilities

- Migrations define schema and structural changes.
- Seeders provide known data for development, tests, or reference tables.
- Production secrets and personal data do not belong in seed files.
- Rollbacks and refreshes can delete data, so confirm the target database first.
- Keep the schema compatible with the model's field names and timestamp settings.

## Practice

1. Add a migration for an articles table.
2. Run it on a local database and inspect the table.
3. Add a small sample seeder and run it.
4. Verify the seeder's behavior if run twice.
5. Review the migration's `down()` method before trying a rollback.

## References

- [Database migrations](https://codeigniter.com/user_guide/dbmgmt/migration.html)
- [Database Forge](https://codeigniter.com/user_guide/dbmgmt/forge.html)
- [Database seeders](https://codeigniter.com/user_guide/dbmgmt/seeds.html)
- [Creating the database and model](https://codeigniter.com/user_guide/guides/api/database-setup.html)
