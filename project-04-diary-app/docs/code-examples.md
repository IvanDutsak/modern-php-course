# Приклади коду - Проект 4: Додаток-щоденник

## Головний файл index.php (з пагінацією)

```php
<?php
require __DIR__ . '/inc/functions.inc.php';
require __DIR__ . '/inc/db-connect.inc.php';

date_default_timezone_set('Europe/Berlin');

// Налаштування пагінації
$perPage = 3;
$page = (int) ($_GET['page'] ?? 1);
if ($page < 1) $page = 1;

// Розрахунок offset для SQL запиту
$offset = ($page - 1) * $perPage;

// Підрахунок загальної кількості записів
$stmtCount = $pdo->prepare('SELECT COUNT(*) AS `count` FROM `entries`');
$stmtCount->execute();
$count = $stmtCount->fetch(PDO::FETCH_ASSOC)['count'];

// Розрахунок кількості сторінок
$numPages = ceil($count / $perPage);

// Отримання записів для поточної сторінки
$stmt = $pdo->prepare('SELECT * FROM `entries` ORDER BY `date` DESC, `id` DESC LIMIT :perPage OFFSET :offset');
$stmt->bindValue('perPage', (int) $perPage, PDO::PARAM_INT);
$stmt->bindValue('offset', (int) $offset, PDO::PARAM_INT);
$stmt->execute();
$results = $stmt->fetchAll(PDO::FETCH_ASSOC);
?>

<!-- Відображення записів -->
<?php require __DIR__ . '/views/header.view.php'; ?>
<h1 class="main-heading">Entries</h1>
<?php foreach ($results AS $result): ?>
    <div class="card">
        <?php if (!empty($result['image'])): ?>
            <div class="card__image-container">
                <img class="card__image" src="uploads/<?php echo e($result['image']); ?>" alt="" />
            </div>
        <?php endif; ?>
        <div class="card__desc-container">
            <?php
                // Конвертація дати з формату YYYY-MM-DD
                $dateExploded = explode('-', $result['date']);
                $timestamp = mktime(12, 0, 0, $dateExploded[1], $dateExploded[2], $dateExploded[0]);
            ?>
            <div class="card__desc-time"><?php echo e(date('m/d/Y', $timestamp)); ?></div>
            <h2 class="card__heading"><?php echo e($result['title']); ?></h2>
            <p class="card__paragraph">
                <?php echo nl2br(e($result['message'])); ?>
            </p>
        </div>
    </div>
<?php endforeach; ?>
```

## Пояснення ключових концепцій

### 1. Підключення до бази даних (db-connect.inc.php)

```php
<?php
$host = 'localhost';
$dbname = 'diary_db';
$username = 'root';
$password = '';

try {
    $pdo = new PDO(
        "mysql:host=$host;dbname=$dbname;charset=utf8mb4",
        $username,
        $password,
        [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
        ]
    );
} catch (PDOException $e) {
    die('Database connection failed: ' . $e->getMessage());
}
```

### 2. Пагінація

```php
// Формула для offset:
// Сторінка 1: offset = (1 - 1) * 3 = 0  → записи 1-3
// Сторінка 2: offset = (2 - 1) * 3 = 3  → записи 4-6
// Сторінка 3: offset = (3 - 1) * 3 = 6  → записи 7-9

$offset = ($page - 1) * $perPage;

// Кількість сторінок
$numPages = ceil($count / $perPage);  // ceil(10 / 3) = 4 сторінки
```

### 3. Підготовлені запити з bindValue

```php
$stmt = $pdo->prepare('SELECT * FROM `entries` LIMIT :perPage OFFSET :offset');
$stmt->bindValue('perPage', (int) $perPage, PDO::PARAM_INT);
$stmt->bindValue('offset', (int) $offset, PDO::PARAM_INT);
$stmt->execute();
```

**Чому bindValue?**
- Захист від SQL-ін'єкцій
- Правильна типізація параметрів
- LIMIT та OFFSET вимагають INTEGER тип

### 4. Робота з датами

```php
// Дата в БД: '2025-11-30'
$dateExploded = explode('-', '2025-11-30');  // ['2025', '11', '30']
$timestamp = mktime(12, 0, 0, $dateExploded[1], $dateExploded[2], $dateExploded[0]);
$formatted = date('m/d/Y', $timestamp);  // '11/30/2025'
```

### 5. Відображення багаторядкового тексту

```php
nl2br(e($result['message']))
```
- `e()` - екранує HTML (захист від XSS)
- `nl2br()` - конвертує `\n` в `<br />`

## Створення нового запису (create-entry.php)

```php
<?php
require __DIR__ . '/inc/functions.inc.php';
require __DIR__ . '/inc/db-connect.inc.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $title = $_POST['title'] ?? '';
    $date = $_POST['date'] ?? '';
    $message = $_POST['message'] ?? '';
    $imageName = null;
    
    // Обробка завантаження зображення
    if (isset($_FILES['image']) && $_FILES['image']['error'] === UPLOAD_ERR_OK) {
        $uploadDir = __DIR__ . '/uploads/';
        $imageName = uniqid() . '_' . basename($_FILES['image']['name']);
        $uploadPath = $uploadDir . $imageName;
        
        // Перевірка типу файлу
        $allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
        if (in_array($_FILES['image']['type'], $allowedTypes)) {
            move_uploaded_file($_FILES['image']['tmp_name'], $uploadPath);
        }
    }
    
    // Вставка в БД
    $stmt = $pdo->prepare('
        INSERT INTO `entries` (`title`, `date`, `message`, `image`) 
        VALUES (:title, :date, :message, :image)
    ');
    
    $stmt->execute([
        'title' => $title,
        'date' => $date,
        'message' => $message,
        'image' => $imageName
    ]);
    
    // Перенаправлення на головну сторінку
    header('Location: index.php');
    exit;
}
```

### Пояснення завантаження файлів

```php
// Перевірка успішного завантаження
$_FILES['image']['error'] === UPLOAD_ERR_OK

// Унікальне ім'я файлу
$imageName = uniqid() . '_' . basename($_FILES['image']['name']);
// Результат: 6756abc123_photo.jpg

// Переміщення з тимчасової директорії
move_uploaded_file($_FILES['image']['tmp_name'], $uploadPath);
```

## Структура бази даних

```sql
CREATE TABLE `entries` (
    `id` INTEGER PRIMARY KEY AUTO_INCREMENT,
    `title` VARCHAR(255) NOT NULL,
    `date` DATE NOT NULL,
    `message` LONGTEXT NOT NULL,
    `image` VARCHAR(255) NULL,
    `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Приклад запису
INSERT INTO `entries` (`title`, `date`, `message`, `image`) 
VALUES (
    'My First Entry',
    '2025-11-30',
    'Today was a great day!\nI learned PHP.',
    '6756abc123_photo.jpg'
);
```

## Навігація пагінації

```php
<div class="pagination">
    <?php if ($page > 1): ?>
        <a href="?page=<?php echo $page - 1; ?>">← Previous</a>
    <?php endif; ?>
    
    <span>Page <?php echo $page; ?> of <?php echo $numPages; ?></span>
    
    <?php if ($page < $numPages): ?>
        <a href="?page=<?php echo $page + 1; ?>">Next →</a>
    <?php endif; ?>
</div>
```

## Ключові концепції проекту

1. **PDO** - сучасний спосіб роботи з БД
2. **Prepared Statements** - безпека від SQL-ін'єкцій
3. **Пагінація** - ефективна робота з великими даними
4. **File Upload** - завантаження та валідація файлів
5. **Date Handling** - робота з датами в PHP та MySQL
6. **CRUD операції** - Create, Read, Update, Delete
