# 98. All code samples

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Deployment and production checks](./16-deployment-and-production-checklist.md) | [Notes index](../README.md) | [Next: Complete questions and answers](./99-complete-q-and-a.md) |

This chapter gathers the fenced code examples from the core chapters so they can be reviewed in one place. The source chapter is listed above each group.

## 1. Installation and the first route

### Example 1

```sh
php --version
composer --version
php -m
```

### Example 2

```sh
composer create-project codeigniter4/appstarter ci-notes-playground
cd ci-notes-playground
```

### Example 3

```sh
cp env .env
```

### Example 4

```powershell
Copy-Item -LiteralPath env -Destination .env
```

### Example 5

```ini
app.baseURL = 'http://localhost:8080/'
```

### Example 6

```sh
php spark serve
```

### Example 7

```php
$routes->get('hello', static function () {
    return 'Hello from CodeIgniter';
});
```

### Example 8

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

### Example 9

```php
$routes->get('welcome', 'Welcome::index');
```

## 2. Project structure and configuration

### Example 10

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

### Example 11

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

### Example 12

```sh
php spark namespaces
```

### Example 13

```ini
CI_ENVIRONMENT = development
app.baseURL = 'http://localhost:8080/'
database.default.hostname = localhost
database.default.database = ci_notes
database.default.username = app_user
database.default.password = local_password
database.default.DBDriver = MySQLi
```

### Example 14

```sh
php spark env
php spark env development
```

### Example 15

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

### Example 16

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

## 3. Routing and route groups

### Example 17

```php
$routes->get('/', 'Home::index');
$routes->get('articles', 'Articles::index');
$routes->get('articles/(:num)', 'Articles::show/$1');
$routes->post('articles', 'Articles::create');
$routes->put('articles/(:num)', 'Articles::update/$1');
$routes->delete('articles/(:num)', 'Articles::delete/$1');
```

### Example 18

```php
$routes->get('articles', 'Articles::index');
$routes->get('articles/(:num)', 'Articles::show/$1');
$routes->post('articles', 'Articles::create');
$routes->patch('articles/(:num)', 'Articles::update/$1');
$routes->delete('articles/(:num)', 'Articles::delete/$1');
```

### Example 19

```php
$routes->group('admin', static function ($routes) {
    $routes->get('reports', 'Reports::index');
    $routes->get('reports/(:num)', 'Reports::show/$1');
    $routes->post('reports', 'Reports::create');
});
```

### Example 20

```php
$routes->get('articles/(:num)', 'Articles::show/$1', [
    'as' => 'article_show',
]);
```

### Example 21

```php
$url = url_to('article_show', $articleId);
```

### Example 22

```php
$routes->resource('photos');
```

### Example 23

```sh
php spark routes
```

## 4. Controllers, requests, and responses

### Example 24

```php
<?php

namespace App\Controllers;

use App\Models\ArticleModel;

class Articles extends BaseController
{
    public function index(): string
    {
        $articles = model(ArticleModel::class)->findAll();

        return view('articles/index', [
            'articles' => $articles,
        ]);
    }
}
```

### Example 25

```php
$search = $this->request->getGet('q');
$title  = $this->request->getPost('title');
$method = $this->request->getMethod();
```

### Example 26

```php
$payload = $this->request->getJSON(true);
```

### Example 27

```php
public function show(int $id): string
{
    $article = model(ArticleModel::class)->find($id);

    if ($article === null) {
        throw CodeIgniterExceptionsPageNotFoundException::forPageNotFound();
    }

    return view('articles/show', [
        'article' => $article,
    ]);
}
```

### Example 28

```php
public function create(): CodeIgniter\HTTP\ResponseInterface
{
    $payload = $this->request->getJSON(true);

    if (! is_array($payload)) {
        return $this->response
            ->setStatusCode(400)
            ->setJSON(['error' => 'A JSON object is required.']);
    }

    return $this->response
        ->setStatusCode(201)
        ->setJSON(['message' => 'Request accepted.']);
}
```

### Example 29

```php
return redirect()
    ->to('/articles')
    ->with('message', 'Article created.');
```

## 5. Views, layouts, and helpers

### Example 30

```php
return view('articles/show', [
    'title' => $article['title'],
    'article' => $article,
]);
```

### Example 31

```php
<h1><?= esc($title) ?></h1>
<p><?= esc($article['body']) ?></p>
```

### Example 32

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

### Example 33

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

### Example 34

```php
<?= $this->include('shared/navigation') ?>
```

### Example 35

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

### Example 36

```php
helper(['form', 'url']);
```

## 6. Databases, Query Builder, and transactions

### Example 37

```ini
database.default.hostname = localhost
database.default.database = ci_notes
database.default.username = app_user
database.default.password = local_password
database.default.DBDriver = MySQLi
database.default.charset = utf8mb4
database.default.DBCollat = utf8mb4_general_ci
```

### Example 38

```php
$db = ConfigDatabase::connect();
```

### Example 39

```php
$db = ConfigDatabase::connect();

$articles = $db->table('articles')
    ->select('id, title, created_at')
    ->where('status', 'published')
    ->orderBy('created_at', 'DESC')
    ->get()
    ->getResultArray();
```

### Example 40

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

### Example 41

```php
$query = $db->query(
    'SELECT id, title FROM articles WHERE status = ? AND author_id = ?',
    ['published', $authorId],
);

$articles = $query->getResultArray();
```

### Example 42

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

## 7. Models and CRUD

### Example 43

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

### Example 44

```php
$model = model(ArticleModel::class);

$article = $model->find($id);
$recentArticles = $model
    ->where('status', 'published')
    ->orderBy('created_at', 'DESC')
    ->findAll(10);
```

### Example 45

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

### Example 46

```php
$model->update($id, [
    'title' => $validatedData['title'],
    'body' => $validatedData['body'],
]);

$model->delete($id);
```

### Example 47

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

## 8. Migrations and seeders

### Example 48

```sh
php spark make:migration CreateArticlesTable
```

### Example 49

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

### Example 50

```sh
php spark migrate
```

### Example 51

```sh
php spark migrate:status
```

### Example 52

```sh
php spark migrate:rollback
```

### Example 53

```sh
php spark make:seeder ArticleSeeder
```

### Example 54

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

### Example 55

```sh
php spark db:seed ArticleSeeder
```

## 9. Validation and form errors

### Example 56

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

### Example 57

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

### Example 58

```php
public array $article = [
    'title' => 'required|max_length[160]',
    'body' => 'required',
    'status' => 'required|in_list[draft,published]',
];
```

### Example 59

```php
if (! $this->validateData($data, 'article')) {
    return view('articles/new', [
        'errors' => $this->validator->getErrors(),
        'values' => $data,
    ]);
}
```

## 10. Sessions, cookies, and flash data

### Example 60

```php
$session = session();

$session->set('user_id', $user->id);
$userId = $session->get('user_id');

$session->remove('temporary_filter');
```

### Example 61

```php
$session = session();
$session->regenerate(true);
$session->set('user_id', $user->id);
```

### Example 62

```php
return redirect()
    ->to('/profile')
    ->with('message', 'Profile updated.');
```

### Example 63

```php
<?php $message = session()->getFlashdata('message'); ?>

<?php if ($message !== null): ?>
    <p role="status"><?= esc($message) ?></p>
<?php endif ?>
```

### Example 64

```php
$this->response->setCookie(
    'display_mode',
    'dark',
    60 * 60 * 24 * 30,
    '',
    '/',
    '',
    true,
    true,
    'Lax',
);
```

### Example 65

```php
$displayMode = $this->request->getCookie('display_mode');
```

## 11. Filters, security, and file uploads

### Example 66

```php
$routes->get('account', 'AccountController::index', [
    'filter' => 'session-auth',
]);
```

### Example 67

```php
<?php

namespace App\\Filters;

use CodeIgniter\\Filters\\FilterInterface;
use CodeIgniter\\HTTP\\RequestInterface;
use CodeIgniter\\HTTP\\ResponseInterface;

class SessionAuth implements FilterInterface
{
    public function before(RequestInterface $request, $arguments = null)
    {
        if (session()->get('userId') === null) {
            return redirect()->to('/login');
        }
    }

    public function after(
        RequestInterface $request,
        ResponseInterface $response,
        $arguments = null
    ) {
    }
}
```

### Example 68

```php
public $aliases = [
    // Keep the aliases supplied by the starter project.
    'session-auth' => \\App\\Filters\\SessionAuth::class,
];
```

### Example 69

```php
public $globals = [
    'before' => [
        'csrf',
    ],
];
```

### Example 70

```php
<form action="/articles" method="post">
    <?= csrf_field() ?>

    <label for="title">Title</label>
    <input id="title" name="title" value="<?= old('title') ?>">

    <button type="submit">Save</button>
</form>
```

### Example 71

```php
<h1><?= esc($article['title']) ?></h1>
<p><?= esc($article['summary']) ?></p>
```

### Example 72

```php
<input
    name="title"
    value="<?= esc(old('title', $article['title'] ?? ''), 'attr') ?>"
>
```

### Example 73

```php
<form action="/profile/avatar" method="post" enctype="multipart/form-data">
    <?= csrf_field() ?>
    <input type="file" name="avatar" accept="image/jpeg,image/png,image/webp">
    <button type="submit">Upload</button>
</form>
```

### Example 74

```php
<?php

namespace App\\Controllers;

class AvatarController extends BaseController
{
    public function store()
    {
        $rules = [
            'avatar' => [
                'label' => 'Avatar',
                'rules' => [
                    'uploaded[avatar]',
                    'is_image[avatar]',
                    'mime_in[avatar,image/jpeg,image/png,image/webp]',
                    'max_size[avatar,2048]',
                    'max_dims[avatar,2000,2000]',
                ],
            ],
        ];

        if (! $this->validateData([], $rules)) {
            return redirect()->back()
                ->withInput()
                ->with('errors', $this->validator->getErrors());
        }

        $file = $this->request->getFile('avatar');

        if (! $file->isValid() || $file->hasMoved()) {
            return redirect()->back()
                ->withInput()
                ->with('error', 'The image could not be uploaded.');
        }

        $storedName = $file->getRandomName();
        $file->move(WRITEPATH . 'uploads/avatars', $storedName);

        // Save $storedName against the signed-in user's record.
        return redirect()->to('/profile')
            ->with('message', 'Avatar uploaded.');
    }
}
```

## 12. Building REST APIs

### Example 75

```php
$routes->group('api', ['namespace' => 'App\\Controllers\\Api'], static function ($routes) {
    $routes->get('articles', 'Articles::index');
    $routes->get('articles/(:num)', 'Articles::show/$1');
    $routes->post('articles', 'Articles::create');
    $routes->put('articles/(:num)', 'Articles::update/$1');
    $routes->delete('articles/(:num)', 'Articles::delete/$1');
});
```

### Example 76

```php
<?php

namespace App\\Controllers\\Api;

use App\\Controllers\\BaseController;
use App\\Models\\ArticleModel;
use CodeIgniter\\API\\ResponseTrait;

class Articles extends BaseController
{
    use ResponseTrait;

    protected $format = 'json';

    public function index()
    {
        $articles = model(ArticleModel::class)
            ->orderBy('created_at', 'DESC')
            ->findAll(20);

        return $this->respond([
            'data' => $articles,
        ]);
    }

    public function show(int $id)
    {
        $article = model(ArticleModel::class)->find($id);

        if ($article === null) {
            return $this->failNotFound('Article not found.');
        }

        return $this->respond([
            'data' => $article,
        ]);
    }
}
```

### Example 77

```php
public function create()
{
    $data = $this->request->getJSON(true) ?? [];

    $rules = [
        'title' => 'required|min_length[3]|max_length[160]',
        'body'  => 'required',
    ];

    if (! $this->validateData($data, $rules)) {
        return $this->failValidationErrors($this->validator->getErrors());
    }

    $values = $this->validator->getValidated();
    $id = model(ArticleModel::class)->insert($values);

    if ($id === false) {
        return $this->failServerError('The article could not be saved.');
    }

    $article = model(ArticleModel::class)->find($id);

    return $this->respondCreated([
        'data' => $article,
    ]);
}
```

### Example 78

```php
$page = max(1, (int) $this->request->getGet('page'));
$limit = (int) $this->request->getGet('limit');

if ($limit < 1) {
    $limit = 20;
}

$limit = min($limit, 100);
$offset = ($page - 1) * $limit;

$rows = model(ArticleModel::class)
    ->orderBy('created_at', 'DESC')
    ->findAll($limit, $offset);
```

### Example 79

```sh
curl -i http://localhost:8080/api/articles
```

### Example 80

```sh
curl -i -X POST http://localhost:8080/api/articles \
  -H "Content-Type: application/json" \
  -d '{"title":"First article","body":"A short example."}'
```

## 13. Services, events, and Spark commands

### Example 81

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

### Example 82

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

### Example 83

```php
$publisher = service('articlePublisher');
$publisher->publish($articleId);
```

### Example 84

```php
<?php

use CodeIgniter\\Events\\Events;

Events::on('article.published', static function (int $articleId): void {
    log_message('info', 'Article published: {id}', ['id' => $articleId]);
});
```

### Example 85

```php
use CodeIgniter\\Events\\Events;

if ($publisher->publish($articleId)) {
    Events::trigger('article.published', $articleId);
}
```

### Example 86

```sh
php spark
php spark list
php spark help migrate
php spark routes
php spark migrate
php spark db:seed ArticleSeeder
```

### Example 87

```sh
php spark make:command SendDigest
```

### Example 88

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

### Example 89

```cron
0 7 * * * cd /var/www/app && /usr/bin/php spark email:send-digest
```

## 14. Errors, logging, and caching

### Example 90

```php
log_message('info', 'Article {id} published by user {userId}', [
    'id' => $articleId,
    'userId' => $currentUserId,
]);

try {
    $publisher->publish($articleId);
} catch (\\Throwable $exception) {
    log_message('error', 'Publishing article {id} failed: {message}', [
        'id' => $articleId,
        'message' => $exception->getMessage(),
    ]);

    throw $exception;
}
```

### Example 91

```php
$key = 'homepage:published-articles';

$articles = cache($key);

if ($articles === null) {
    $articles = model(ArticleModel::class)
        ->where('status', 'published')
        ->orderBy('created_at', 'DESC')
        ->findAll(10);

    cache()->save($key, $articles, 300);
}
```

### Example 92

```php
cache()->delete('homepage:published-articles');
```

### Example 93

```php
public function index()
{
    $this->cachePage(60);

    return view('home', [
        'articles' => $this->articles->publishedForHome(),
    ]);
}
```

### Example 94

```sh
php spark
php spark routes
```

## 15. Testing controllers and models

### Example 95

```sh
composer require --dev phpunit/phpunit
vendor/bin/phpunit
```

### Example 96

```php
<?php

namespace Tests\\Unit;

use CodeIgniter\\Test\\CIUnitTestCase;

final class ArticleTitleTest extends CIUnitTestCase
{
    public function testTitleIsTrimmedBeforeItIsShown(): void
    {
        $title = trim('  Notes about routing  ');

        $this->assertSame('Notes about routing', $title);
    }
}
```

### Example 97

```php
<?php

namespace Tests\\Feature;

use CodeIgniter\\Test\\CIUnitTestCase;
use CodeIgniter\\Test\\FeatureTestTrait;

final class HomePageTest extends CIUnitTestCase
{
    use FeatureTestTrait;

    public function testHomePageLoads(): void
    {
        $result = $this->get('/');

        $result->assertOK();
        $result->assertSee('Welcome');
    }
}
```

### Example 98

```php
$result = $this->withBodyFormat('json')
    ->post('/api/articles', [
        'title' => 'A test article',
        'body'  => 'A short body.',
    ]);

$result->assertStatus(201);
$result->assertJSONFragment([
    'title' => 'A test article',
]);
```

### Example 99

```php
<?php

namespace Tests\\Feature;

use CodeIgniter\\Test\\CIUnitTestCase;
use CodeIgniter\\Test\\DatabaseTestTrait;
use CodeIgniter\\Test\\FeatureTestTrait;

final class ArticleApiTest extends CIUnitTestCase
{
    use DatabaseTestTrait;
    use FeatureTestTrait;

    protected $migrate = true;
    protected $migrateOnce = false;
    protected $refresh = true;

    public function testUnknownArticleReturnsNotFound(): void
    {
        $result = $this->get('/api/articles/999999');

        $result->assertStatus(404);
    }
}
```

## 16. Deployment and production checks

### Example 100

```sh
composer install --no-dev --optimize-autoloader
```

### Example 101

```sh
php spark migrate
php spark migrate:status
```

### Example 102

```sh
php spark phpini:check
php spark routes
```

## Source chapters

- [1. Installation and the first route](./01-installation-and-first-route.md)
- [2. Project structure and configuration](./02-project-structure-and-configuration.md)
- [3. Routing and route groups](./03-routing-and-route-groups.md)
- [4. Controllers, requests, and responses](./04-controllers-requests-and-responses.md)
- [5. Views, layouts, and helpers](./05-views-layouts-and-helpers.md)
- [6. Databases, Query Builder, and transactions](./06-databases-query-builder-and-transactions.md)
- [7. Models and CRUD](./07-models-and-crud.md)
- [8. Migrations and seeders](./08-migrations-and-seeders.md)
- [9. Validation and form errors](./09-validation-and-form-errors.md)
- [10. Sessions, cookies, and flash data](./10-sessions-cookies-and-flash-data.md)
- [11. Filters, security, and file uploads](./11-filters-security-and-file-uploads.md)
- [12. Building REST APIs](./12-building-rest-apis.md)
- [13. Services, events, and Spark commands](./13-services-events-and-spark-commands.md)
- [14. Errors, logging, and caching](./14-errors-logging-and-caching.md)
- [15. Testing controllers and models](./15-testing-controllers-and-models.md)
- [16. Deployment and production checks](./16-deployment-and-production-checklist.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Deployment and production checks](./16-deployment-and-production-checklist.md) | [Notes index](../README.md) | [Next: Complete questions and answers](./99-complete-q-and-a.md) |