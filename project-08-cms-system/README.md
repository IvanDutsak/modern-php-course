# Проект 8: Система керування контентом (CMS)

## Опис проекту

Це мікро-CMS система на PHP з розділенням на адміністративну і публічну частини. Проект реалізований з використанням принципів MVC (Model-View-Controller), IoC контейнера (Inversion of Control) та репозиторіїв для роботи з даними. Система побудована на PDO для роботи з MySQL БД та включає повноцінну систему автентифікації.

## Технології

- PHP (Advanced OOP)
- MVC Architecture
- IoC Container (Dependency Injection)
- Repository Pattern
- Authentication & Sessions
- CSRF Protection
- Password Hashing
- MySQL/MariaDB
- PDO
- Namespaces & Autoloading

## Основні функції

### Публічна частина
1. **Перегляд сторінок** - відображення опублікованих сторінок
2. **ЧПУ (SEO-friendly URLs)** - використання slug для URL
3. **404 сторінка** - обробка неіснуючих сторінок

### Адміністративна частина
1. **Автентифікація** - вхід/вихід з системи
2. **Управління сторінками** - створення, редагування, видалення
3. **CRUD операції** - повний цикл роботи з контентом
4. **Валідація** - перевірка унікальності slug
5. **Безпека** - CSRF токени, хешування паролів

## Структура бази даних

### Таблиця `pages`

| Поле | Тип | Опис |
|------|-----|------|
| id | INTEGER | PRIMARY KEY, AUTO_INCREMENT |
| title | VARCHAR(255) | Заголовок сторінки |
| slug | VARCHAR(255) | URL-friendly назва (унікальний) |
| content | LONGTEXT | Вміст сторінки |
| created_at | TIMESTAMP | Дата створення |
| updated_at | TIMESTAMP | Дата оновлення |

### Таблиця `users`

| Поле | Тип | Опис |
|------|-----|------|
| id | INTEGER | PRIMARY KEY, AUTO_INCREMENT |
| username | VARCHAR(100) | Логін (унікальний) |
| password | VARCHAR(255) | Хеш пароля |
| email | VARCHAR(255) | Email |
| created_at | TIMESTAMP | Дата реєстрації |

## Структура проекту (MVC + IoC)

```
src/
├── index.php                           # Головний маршрутизатор
├── Controllers/
│   ├── PagesController.php             # Публічні сторінки
│   ├── PagesAdminController.php        # Адмін-панель сторінок
│   └── LoginController.php             # Автентифікація
├── Models/
│   ├── PageModel.php                   # Модель сторінки
│   └── UserModel.php                   # Модель користувача
├── Repositories/
│   ├── PagesRepository.php             # Репозиторій сторінок
│   └── UsersRepository.php             # Репозиторій користувачів
├── Services/
│   └── AuthService.php                 # Сервіс автентифікації
├── Helpers/
│   └── CsrfHelper.php                  # CSRF захист
├── Container/
│   └── Container.php                   # IoC контейнер
├── Database/
│   └── Connection.php                  # Підключення до БД
├── config/
│   └── database.php                    # Конфігурація БД
├── views/
│   ├── public/
│   │   ├── page.php                    # Шаблон публічної сторінки
│   │   └── 404.php                     # Шаблон 404
│   ├── admin/
│   │   ├── pages-list.php              # Список сторінок
│   │   ├── page-create.php             # Форма створення
│   │   ├── page-edit.php               # Форма редагування
│   │   └── login.php                   # Форма входу
│   ├── layouts/
│   │   ├── header.php                  # Шапка
│   │   └── footer.php                  # Підвал
└── styles/
    ├── public.css                      # Стилі публічної частини
    └── admin.css                       # Стилі адмін-панелі
```

## Як запустити

1. Створіть базу даних:
```sql
CREATE DATABASE cms_db;
USE cms_db;

CREATE TABLE pages (
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    slug VARCHAR(255) NOT NULL UNIQUE,
    content LONGTEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_slug (slug)
);

CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Створити адміністратора (пароль: admin123)
INSERT INTO users (username, password, email) 
VALUES ('admin', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', 'admin@example.com');
```

2. Налаштуйте підключення в `config/database.php`
3. Відкрийте `http://localhost/index.php`
4. Для входу в адмін-панель: `http://localhost/index.php?route=admin/login`

## Ключові концепції

### IoC Container

```php
class Container {
    private array $services = [];
    
    public function set(string $name, callable $resolver): void {
        $this->services[$name] = $resolver;
    }
    
    public function get(string $name) {
        if (!isset($this->services[$name])) {
            throw new Exception("Service {$name} not found");
        }
        return $this->services[$name]($this);
    }
}

// Реєстрація сервісів
$container = new Container();

$container->set('pdo', function() {
    return new PDO('mysql:host=localhost;dbname=cms_db', 'user', 'pass');
});

$container->set('pagesRepository', function($c) {
    return new PagesRepository($c->get('pdo'));
});

$container->set('pagesController', function($c) {
    return new PagesController($c->get('pagesRepository'));
});
```

### MVC Routing

```php
// index.php
$route = $_GET['route'] ?? 'home';

match($route) {
    'home' => $container->get('pagesController')->index(),
    'page' => $container->get('pagesController')->show($_GET['slug']),
    'admin/pages' => $container->get('pagesAdminController')->index(),
    'admin/pages/create' => $container->get('pagesAdminController')->create(),
    'admin/pages/edit' => $container->get('pagesAdminController')->edit($_GET['id']),
    'admin/pages/delete' => $container->get('pagesAdminController')->delete($_GET['id']),
    'admin/login' => $container->get('loginController')->login(),
    'admin/logout' => $container->get('loginController')->logout(),
    default => $container->get('pagesController')->notFound()
};
```

### Repository Pattern

```php
class PagesRepository {
    private PDO $pdo;
    
    public function __construct(PDO $pdo) {
        $this->pdo = $pdo;
    }
    
    public function findAll(): array {
        $stmt = $this->pdo->query('SELECT * FROM pages ORDER BY created_at DESC');
        return $stmt->fetchAll(PDO::FETCH_CLASS, PageModel::class);
    }
    
    public function findBySlug(string $slug): ?PageModel {
        $stmt = $this->pdo->prepare('SELECT * FROM pages WHERE slug = ?');
        $stmt->execute([$slug]);
        $stmt->setFetchMode(PDO::FETCH_CLASS, PageModel::class);
        return $stmt->fetch() ?: null;
    }
    
    public function create(PageModel $page): bool {
        $stmt = $this->pdo->prepare('INSERT INTO pages (title, slug, content) VALUES (?, ?, ?)');
        return $stmt->execute([$page->title, $page->slug, $page->content]);
    }
    
    public function update(PageModel $page): bool {
        $stmt = $this->pdo->prepare('UPDATE pages SET title = ?, slug = ?, content = ? WHERE id = ?');
        return $stmt->execute([$page->title, $page->slug, $page->content, $page->id]);
    }
    
    public function delete(int $id): bool {
        $stmt = $this->pdo->prepare('DELETE FROM pages WHERE id = ?');
        return $stmt->execute([$id]);
    }
    
    public function slugExists(string $slug, ?int $excludeId = null): bool {
        if ($excludeId) {
            $stmt = $this->pdo->prepare('SELECT COUNT(*) FROM pages WHERE slug = ? AND id != ?');
            $stmt->execute([$slug, $excludeId]);
        } else {
            $stmt = $this->pdo->prepare('SELECT COUNT(*) FROM pages WHERE slug = ?');
            $stmt->execute([$slug]);
        }
        return $stmt->fetchColumn() > 0;
    }
}
```

### Автентифікація

```php
class AuthService {
    private UsersRepository $usersRepository;
    
    public function __construct(UsersRepository $usersRepository) {
        $this->usersRepository = $usersRepository;
    }
    
    public function login(string $username, string $password): bool {
        $user = $this->usersRepository->findByUsername($username);
        
        if (!$user || !password_verify($password, $user->password)) {
            return false;
        }
        
        $_SESSION['user_id'] = $user->id;
        $_SESSION['username'] = $user->username;
        return true;
    }
    
    public function logout(): void {
        session_destroy();
    }
    
    public function isAuthenticated(): bool {
        return isset($_SESSION['user_id']);
    }
    
    public function requireAuth(): void {
        if (!$this->isAuthenticated()) {
            header('Location: index.php?route=admin/login');
            exit;
        }
    }
}
```

### CSRF Protection

```php
class CsrfHelper {
    public static function generateToken(): string {
        if (!isset($_SESSION['csrf_token'])) {
            $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
        }
        return $_SESSION['csrf_token'];
    }
    
    public static function validateToken(string $token): bool {
        return isset($_SESSION['csrf_token']) && hash_equals($_SESSION['csrf_token'], $token);
    }
    
    public static function requireValidToken(): void {
        if (!isset($_POST['csrf_token']) || !self::validateToken($_POST['csrf_token'])) {
            die('CSRF token validation failed');
        }
    }
}

// У формі
<input type="hidden" name="csrf_token" value="<?= CsrfHelper::generateToken() ?>">

// При обробці
CsrfHelper::requireValidToken();
```

## Безпека

- ✅ **Password Hashing** - `password_hash()` та `password_verify()`
- ✅ **CSRF Protection** - токени для всіх форм
- ✅ **Prepared Statements** - захист від SQL-ін'єкцій
- ✅ **Session Management** - безпечна робота з сесіями
- ✅ **Input Validation** - перевірка всіх даних
- ✅ **XSS Protection** - `htmlspecialchars()` для виводу

## Архітектурні переваги

- ✅ **MVC** - чітке розділення логіки, даних та представлення
- ✅ **IoC Container** - керування залежностями
- ✅ **Repository Pattern** - абстракція доступу до даних
- ✅ **SOLID принципи** - чистий та підтримуваний код
- ✅ **Dependency Injection** - гнучкість та тестованість
- ✅ **Namespaces** - організація коду


## Автор

**Іван Дуцак**  
Виконано в рамках курсу Modern PHP
