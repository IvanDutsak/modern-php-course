# Приклади коду - Проект 8: CMS System

## IoC Container (Dependency Injection Container)

### Клас Container.php

```php
<?php

namespace App\Support;

class Container {
    private array $instances = [];  // Кеш створених об'єктів
    private array $recipes = [];    // Рецепти створення об'єктів

    // Реєстрація рецепту створення об'єкта
    public function bind(string $what, \Closure $recipe) {
        $this->recipes[$what] = $recipe;
    }
    
    // Отримання об'єкта (з кешем)
    public function get($what) {
        // Якщо об'єкт вже створений - повертаємо з кешу
        if (empty($this->instances[$what])) {
            // Якщо рецепту немає - помилка
            if (empty($this->recipes[$what])) {
                echo "Could not build: {$what}.\n";
                die();
            }
            // Створюємо об'єкт за рецептом і кешуємо
            $this->instances[$what] = $this->recipes[$what]();
        }
        return $this->instances[$what];
    }
}
```

### Використання Container

```php
<?php
require __DIR__ . '/inc/all.inc.php';

use App\Support\Container;
use App\Repository\PagesRepository;
use App\Frontend\Controller\PagesController;

$container = new Container();

// Реєстрація PDO
$container->bind('pdo', function() {
    return new PDO(
        'mysql:host=localhost;dbname=cms_db;charset=utf8mb4',
        'root',
        '',
        [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
    );
});

// Реєстрація Repository
$container->bind('pagesRepository', function() use ($container) {
    return new PagesRepository($container->get('pdo'));
});

// Реєстрація Controller
$container->bind('pagesController', function() use ($container) {
    return new PagesController($container->get('pagesRepository'));
});

// Використання
$controller = $container->get('pagesController');
$controller->index();
```

## Repository Pattern

### PagesRepository.php

```php
<?php

namespace App\Repository;

use App\Model\PageModel;
use PDO;

class PagesRepository {
    
    public function __construct(
        private PDO $pdo
    ) {}
    
    // Отримати всі сторінки
    public function findAll(): array {
        $stmt = $this->pdo->query('SELECT * FROM `pages` ORDER BY `created_at` DESC');
        return $stmt->fetchAll(PDO::FETCH_CLASS, PageModel::class);
    }
    
    // Знайти сторінку за slug
    public function findBySlug(string $slug): PageModel|false {
        $stmt = $this->pdo->prepare('SELECT * FROM `pages` WHERE `slug` = :slug');
        $stmt->execute(['slug' => $slug]);
        $stmt->setFetchMode(PDO::FETCH_CLASS, PageModel::class);
        return $stmt->fetch();
    }
    
    // Знайти сторінку за ID
    public function findById(int $id): PageModel|false {
        $stmt = $this->pdo->prepare('SELECT * FROM `pages` WHERE `id` = :id');
        $stmt->execute(['id' => $id]);
        $stmt->setFetchMode(PDO::FETCH_CLASS, PageModel::class);
        return $stmt->fetch();
    }
    
    // Створити нову сторінку
    public function create(PageModel $page): bool {
        $stmt = $this->pdo->prepare('
            INSERT INTO `pages` (`title`, `slug`, `content`) 
            VALUES (:title, :slug, :content)
        ');
        return $stmt->execute([
            'title' => $page->title,
            'slug' => $page->slug,
            'content' => $page->content
        ]);
    }
    
    // Оновити сторінку
    public function update(PageModel $page): bool {
        $stmt = $this->pdo->prepare('
            UPDATE `pages` 
            SET `title` = :title, `slug` = :slug, `content` = :content 
            WHERE `id` = :id
        ');
        return $stmt->execute([
            'title' => $page->title,
            'slug' => $page->slug,
            'content' => $page->content,
            'id' => $page->id
        ]);
    }
    
    // Видалити сторінку
    public function delete(int $id): bool {
        $stmt = $this->pdo->prepare('DELETE FROM `pages` WHERE `id` = :id');
        return $stmt->execute(['id' => $id]);
    }
    
    // Перевірка унікальності slug
    public function slugExists(string $slug, ?int $excludeId = null): bool {
        if ($excludeId) {
            $stmt = $this->pdo->prepare('
                SELECT COUNT(*) FROM `pages` 
                WHERE `slug` = :slug AND `id` != :id
            ');
            $stmt->execute(['slug' => $slug, 'id' => $excludeId]);
        } else {
            $stmt = $this->pdo->prepare('SELECT COUNT(*) FROM `pages` WHERE `slug` = :slug');
            $stmt->execute(['slug' => $slug]);
        }
        return $stmt->fetchColumn() > 0;
    }
}
```

## MVC Controller

### PagesAdminController.php

```php
<?php

namespace App\Admin\Controller;

use App\Repository\PagesRepository;
use App\Model\PageModel;
use App\Support\CsrfHelper;

class PagesAdminController extends AbstractAdminController {
    
    public function __construct(
        private PagesRepository $pagesRepository
    ) {}
    
    // Список всіх сторінок
    public function index(): void {
        $pages = $this->pagesRepository->findAll();
        require __DIR__ . '/../../../views/admin/pages/index.view.php';
    }
    
    // Форма створення нової сторінки
    public function create(): void {
        if ($_SERVER['REQUEST_METHOD'] === 'POST') {
            CsrfHelper::requireValidToken();
            
            $page = new PageModel();
            $page->title = $_POST['title'] ?? '';
            $page->slug = $_POST['slug'] ?? '';
            $page->content = $_POST['content'] ?? '';
            
            // Валідація
            if (empty($page->title) || empty($page->slug) || empty($page->content)) {
                $error = 'All fields are required';
            } elseif ($this->pagesRepository->slugExists($page->slug)) {
                $error = 'Slug already exists';
            } else {
                $this->pagesRepository->create($page);
                header('Location: index.php?route=admin/pages');
                exit;
            }
        }
        
        require __DIR__ . '/../../../views/admin/pages/create.view.php';
    }
    
    // Редагування сторінки
    public function edit(int $id): void {
        $page = $this->pagesRepository->findById($id);
        
        if (!$page) {
            header('Location: index.php?route=admin/pages');
            exit;
        }
        
        if ($_SERVER['REQUEST_METHOD'] === 'POST') {
            CsrfHelper::requireValidToken();
            
            $page->title = $_POST['title'] ?? '';
            $page->slug = $_POST['slug'] ?? '';
            $page->content = $_POST['content'] ?? '';
            
            // Валідація
            if (empty($page->title) || empty($page->slug) || empty($page->content)) {
                $error = 'All fields are required';
            } elseif ($this->pagesRepository->slugExists($page->slug, $page->id)) {
                $error = 'Slug already exists';
            } else {
                $this->pagesRepository->update($page);
                header('Location: index.php?route=admin/pages');
                exit;
            }
        }
        
        require __DIR__ . '/../../../views/admin/pages/edit.view.php';
    }
    
    // Видалення сторінки
    public function delete(int $id): void {
        CsrfHelper::requireValidToken();
        $this->pagesRepository->delete($id);
        header('Location: index.php?route=admin/pages');
        exit;
    }
}
```

## CSRF Protection

### CsrfHelper.php

```php
<?php

namespace App\Support;

class CsrfHelper {
    
    // Генерація токену
    public static function generateToken(): string {
        if (empty($_SESSION['csrf_token'])) {
            $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
        }
        return $_SESSION['csrf_token'];
    }
    
    // Валідація токену
    public static function validateToken(string $token): bool {
        return !empty($_SESSION['csrf_token']) 
            && hash_equals($_SESSION['csrf_token'], $token);
    }
    
    // Вимагати валідний токен або завершити скрипт
    public static function requireValidToken(): void {
        if (empty($_POST['csrf_token']) || !self::validateToken($_POST['csrf_token'])) {
            die('CSRF token validation failed');
        }
    }
}
```

### Використання в формі

```php
<form method="POST">
    <input type="hidden" name="csrf_token" value="<?php echo CsrfHelper::generateToken(); ?>">
    
    <label>Title:</label>
    <input type="text" name="title" required>
    
    <label>Slug:</label>
    <input type="text" name="slug" required>
    
    <label>Content:</label>
    <textarea name="content" required></textarea>
    
    <button type="submit">Save</button>
</form>
```

## Authentication Service

### AuthService.php

```php
<?php

namespace App\Admin\Support;

class AuthService {
    
    private array $credentials = [
        'admin' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi' // admin123
    ];
    
    // Вхід в систему
    public function login(string $username, string $password): bool {
        if (!isset($this->credentials[$username])) {
            return false;
        }
        
        if (!password_verify($password, $this->credentials[$username])) {
            return false;
        }
        
        $_SESSION['user'] = $username;
        return true;
    }
    
    // Вихід з системи
    public function logout(): void {
        unset($_SESSION['user']);
        session_destroy();
    }
    
    // Перевірка автентифікації
    public function isAuthenticated(): bool {
        return !empty($_SESSION['user']);
    }
    
    // Вимагати автентифікацію
    public function requireAuth(): void {
        if (!$this->isAuthenticated()) {
            header('Location: index.php?route=admin/login');
            exit;
        }
    }
}
```

## Routing (index.php)

```php
<?php
session_start();
require __DIR__ . '/inc/all.inc.php';

use App\Support\Container;

$container = new Container();

// Реєстрація всіх сервісів...
// (код реєстрації з прикладу вище)

$route = $_GET['route'] ?? 'home';

// Маршрутизація
match($route) {
    // Публічні маршрути
    'home' => $container->get('pagesController')->index(),
    'page' => $container->get('pagesController')->show($_GET['slug'] ?? ''),
    
    // Адмін-маршрути
    'admin/login' => $container->get('loginController')->login(),
    'admin/logout' => $container->get('loginController')->logout(),
    'admin/pages' => $container->get('pagesAdminController')->index(),
    'admin/pages/create' => $container->get('pagesAdminController')->create(),
    'admin/pages/edit' => $container->get('pagesAdminController')->edit((int)$_GET['id']),
    'admin/pages/delete' => $container->get('pagesAdminController')->delete((int)$_GET['id']),
    
    // 404
    default => $container->get('notFoundController')->show()
};
```

## Ключові концепції

### 1. IoC Container (Dependency Injection)
- Централізоване управління залежностями
- Lazy loading - об'єкти створюються тільки коли потрібні
- Singleton pattern - один екземпляр на весь додаток

### 2. Repository Pattern
- Абстракція доступу до даних
- Легке тестування (можна замінити на Mock)
- Відокремлення бізнес-логіки від SQL

### 3. MVC Architecture
- **Model** - структура даних (PageModel)
- **View** - шаблони відображення
- **Controller** - обробка запитів та логіка

### 4. Security
- **CSRF Tokens** - захист від підробки запитів
- **Password Hashing** - `password_hash()` та `password_verify()`
- **Prepared Statements** - захист від SQL-ін'єкцій
- **Session Management** - безпечна автентифікація

### 5. Namespaces
```php
namespace App\Admin\Controller;
use App\Repository\PagesRepository;
use App\Model\PageModel;
```

## Структура БД

```sql
CREATE TABLE `pages` (
    `id` INTEGER PRIMARY KEY AUTO_INCREMENT,
    `title` VARCHAR(255) NOT NULL,
    `slug` VARCHAR(255) NOT NULL UNIQUE,
    `content` LONGTEXT NOT NULL,
    `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    `updated_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX `idx_slug` (`slug`)
);
```

## Приклади URL

- Публічна частина: `index.php?route=page&slug=about-us`
- Адмін-панель: `index.php?route=admin/pages`
- Створення: `index.php?route=admin/pages/create`
- Редагування: `index.php?route=admin/pages/edit&id=5`
