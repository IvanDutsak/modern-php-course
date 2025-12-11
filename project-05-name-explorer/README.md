# Проект 5: Дослідник імен

## Опис проекту

Проект представляє собою веб-додаток для дослідження та пошуку особистих імен. Програма дозволяє користувачам переглядати статистику найпопулярніших імен, досліджувати імена за першою літерою алфавіту та отримувати детальну інформацію про конкретне ім'я.

## Технології

- PHP
- MySQL/MariaDB
- PDO
- MVC Architecture (базова)
- Пагінація
- HTML/CSS

## Основні функції

1. **Топ-10 імен** - відображення найпопулярніших імен
2. **Алфавітна навігація** - перегляд імен за першою літерою (A-Z)
3. **Пагінація** - 15 імен на сторінку
4. **Детальна інформація** - історичні дані про використання імені
5. **Статистика** - кількість реєстрацій по роках

## Структура бази даних

### Таблиця `names`

| Поле | Тип | Опис |
|------|-----|------|
| id | INTEGER | PRIMARY KEY |
| name | VARCHAR(100) | Ім'я |
| year | INTEGER | Рік реєстрації |
| count | INTEGER | Кількість використань |

## Структура проекту

```
src/
├── index.php            # Головна сторінка (топ-10)
├── letter.php           # Імена за літерою
├── name.php             # Детальна інформація про ім'я
├── db-connect.inc.php   # Підключення до БД
├── styles/
│   └── simple.css       # Стилі
└── views/
    ├── header.php       # Шапка з навігацією A-Z
    └── footer.php       # Підвал
```

## Як запустити

1. Створіть базу даних та імпортуйте дані:
```sql
CREATE DATABASE names_db;
USE names_db;

CREATE TABLE names (
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    year INTEGER NOT NULL,
    count INTEGER NOT NULL,
    INDEX idx_name (name),
    INDEX idx_year (year)
);
```

2. Налаштуйте підключення в `db-connect.inc.php`
3. Відкрийте `http://localhost/index.php`

## Ключові концепції

### Топ-10 найпопулярніших імен

```php
$stmt = $pdo->query('
    SELECT name, SUM(count) as total 
    FROM names 
    GROUP BY name 
    ORDER BY total DESC 
    LIMIT 10
');
```

### Імена за літерою з пагінацією

```php
$letter = $_GET['letter'] ?? 'A';
$page = $_GET['page'] ?? 1;
$perPage = 15;
$offset = ($page - 1) * $perPage;

$stmt = $pdo->prepare('
    SELECT DISTINCT name 
    FROM names 
    WHERE name LIKE ? 
    ORDER BY name 
    LIMIT ? OFFSET ?
');
$stmt->execute([$letter . '%', $perPage, $offset]);
```

### Історія імені

```php
$stmt = $pdo->prepare('
    SELECT year, count 
    FROM names 
    WHERE name = ? 
    ORDER BY year ASC
');
$stmt->execute([$name]);
```

## Особливості реалізації

- ✅ Алфавітна навігація з підсвіткою активної літери
- ✅ Ефективна пагінація для великих наборів даних
- ✅ Індекси БД для швидкого пошуку
- ✅ Чиста структура URL (letter.php?letter=A&page=2)
- ✅ Історичні графіки популярності імен


## Автор

**Іван Дуцак**  
Виконано в рамках курсу Modern PHP
