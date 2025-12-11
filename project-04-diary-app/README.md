# Проект 4: Додаток-щоденник

## Опис проекту

PHP Diary - це веб-застосунок, який дозволяє користувачам вести персональний щоденник записів з можливістю додавання тексту, дати та зображень. Проект розроблений на PHP з використанням бази даних MySQL та забезпечує зручний інтерфейс для управління записами щоденника.

## Технології

- PHP
- MySQL/MariaDB
- PDO (PHP Data Objects)
- HTML/CSS
- Форми та валідація
- Завантаження файлів

## Основні функції

1. **Створення записів** - додавання нових записів з текстом, датою та зображенням
2. **Перегляд записів** - відображення всіх записів у хронологічному порядку
3. **Пагінація** - розбиття записів на сторінки (3 записи на сторінку)
4. **Завантаження зображень** - можливість прикріплення фото до запису

## Структура бази даних

### Таблиця `entries`

| Поле | Тип | Опис |
|------|-----|------|
| id | INTEGER | PRIMARY KEY, AUTO_INCREMENT |
| title | VARCHAR(255) | Назва запису |
| date | DATE | Дата створення |
| message | LONGTEXT | Текст запису |
| image | VARCHAR(255) | Назва файлу зображення |

## Структура проекту

```
src/
├── index.php            # Головна сторінка зі списком записів
├── form.php             # Форма створення нового запису
├── create-entry.php     # Обробка створення запису
├── db-connect.inc.php   # Підключення до БД
├── uploads/             # Папка для завантажених зображень
├── styles/
│   └── simple.css       # Стилі
└── views/
    ├── header.php       # Шапка
    └── footer.php       # Підвал
```

## Як запустити

1. Створіть базу даних MySQL:
```sql
CREATE DATABASE diary_db;
USE diary_db;

CREATE TABLE entries (
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    date DATE NOT NULL,
    message LONGTEXT NOT NULL,
    image VARCHAR(255) NULL
);
```

2. Налаштуйте підключення в `db-connect.inc.php`
3. Розмістіть файли в директорії веб-сервера
4. Відкрийте `http://localhost/index.php`

## Ключові концепції

### Підключення до БД через PDO

```php
$pdo = new PDO(
    'mysql:host=localhost;dbname=diary_db;charset=utf8mb4',
    'username',
    'password'
);
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
```

### Підготовлені запити

```php
$stmt = $pdo->prepare('INSERT INTO entries (title, date, message, image) VALUES (?, ?, ?, ?)');
$stmt->execute([$title, $date, $message, $imageName]);
```

### Пагінація

```php
$page = $_GET['page'] ?? 1;
$perPage = 3;
$offset = ($page - 1) * $perPage;

$stmt = $pdo->prepare('SELECT * FROM entries ORDER BY date DESC LIMIT ? OFFSET ?');
```

### Завантаження файлів

```php
if (isset($_FILES['image']) && $_FILES['image']['error'] === UPLOAD_ERR_OK) {
    $uploadDir = 'uploads/';
    $fileName = uniqid() . '_' . $_FILES['image']['name'];
    move_uploaded_file($_FILES['image']['tmp_name'], $uploadDir . $fileName);
}
```

## Безпека

- ✅ Використання підготовлених запитів (prepared statements)
- ✅ Валідація даних форми
- ✅ Перевірка типів файлів при завантаженні
- ✅ Унікальні імена файлів для запобігання конфліктів

## Скріншоти

Скріншоти роботи додатку доступні в папці `screenshots/`

## Автор

**Іван Дуцак**  
Виконано в рамках курсу Modern PHP
