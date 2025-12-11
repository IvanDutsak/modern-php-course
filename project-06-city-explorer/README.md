# Проект 6: Дослідник міст

## Опис проекту

Це веб-додаток на PHP, розроблений для перегляду та управління інформацією про міста по всьому світу. Додаток надає користувачам інтерактивний інтерфейс для навігації по базі даних світових міст, перегляду детальної інформації про кожне місто та виконання операцій редагування. Проект демонструє сучасні практики розробки на PHP, включаючи об'єктно-орієнтоване програмування та шаблон Repository.

## Технології

- PHP (OOP)
- MySQL/MariaDB
- PDO
- Repository Pattern
- Namespaces
- HTML/CSS
- Emoji прапори (ISO 3166-1 alpha-2)

## Основні функції

1. **Перегляд міст** - список міст з пагінацією (15 міст на сторінку)
2. **Детальна інформація** - повні дані про місто
3. **Редагування** - можливість оновлення інформації про місто
4. **Прапори країн** - візуальне відображення через emoji
5. **Пагінація** - зручна навігація між сторінками

## Структура бази даних

### Таблиця `cities`

| Поле | Тип | Опис |
|------|-----|------|
| id | INTEGER | PRIMARY KEY |
| name | VARCHAR(255) | Назва міста |
| country | VARCHAR(255) | Країна |
| country_code | CHAR(2) | ISO код країни |
| population | INTEGER | Населення |
| area | DECIMAL | Площа (км²) |

## Структура проекту (OOP)

```
src/
├── index.php                    # Головна сторінка
├── city.php                     # Детальна інформація
├── edit.php                     # Редагування міста
├── Models/
│   └── City.php                 # Модель міста
├── Repositories/
│   └── CityRepository.php       # Репозиторій для роботи з БД
├── Database/
│   └── Connection.php           # Підключення до БД
├── Helpers/
│   └── FlagHelper.php           # Генерація emoji прапорів
├── styles/
│   └── simple.css               # Стилі
└── views/
    ├── header.php               # Шапка
    └── footer.php               # Підвал
```

## Як запустити

1. Створіть базу даних:
```sql
CREATE DATABASE cities_db;
USE cities_db;

CREATE TABLE cities (
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    country VARCHAR(255) NOT NULL,
    country_code CHAR(2) NOT NULL,
    population INTEGER,
    area DECIMAL(10,2),
    INDEX idx_country (country)
);
```

2. Налаштуйте підключення в `Database/Connection.php`
3. Відкрийте `http://localhost/index.php`

## Ключові концепції

### Repository Pattern

```php
class CityRepository {
    private PDO $pdo;
    
    public function __construct(PDO $pdo) {
        $this->pdo = $pdo;
    }
    
    public function findAll(int $limit, int $offset): array {
        $stmt = $this->pdo->prepare('SELECT * FROM cities LIMIT ? OFFSET ?');
        $stmt->execute([$limit, $offset]);
        return $stmt->fetchAll(PDO::FETCH_CLASS, City::class);
    }
    
    public function findById(int $id): ?City {
        $stmt = $this->pdo->prepare('SELECT * FROM cities WHERE id = ?');
        $stmt->execute([$id]);
        $stmt->setFetchMode(PDO::FETCH_CLASS, City::class);
        return $stmt->fetch() ?: null;
    }
    
    public function update(City $city): bool {
        $stmt = $this->pdo->prepare('
            UPDATE cities 
            SET name = ?, country = ?, country_code = ?, population = ?, area = ? 
            WHERE id = ?
        ');
        return $stmt->execute([
            $city->name,
            $city->country,
            $city->country_code,
            $city->population,
            $city->area,
            $city->id
        ]);
    }
}
```

### Модель City

```php
class City {
    public int $id;
    public string $name;
    public string $country;
    public string $country_code;
    public ?int $population;
    public ?float $area;
    
    public function getFlag(): string {
        return FlagHelper::getEmojiFlag($this->country_code);
    }
}
```

### Генерація Emoji прапорів

```php
class FlagHelper {
    public static function getEmojiFlag(string $countryCode): string {
        $countryCode = strtoupper($countryCode);
        $offset = 127397;
        $flag = '';
        for ($i = 0; $i < strlen($countryCode); $i++) {
            $flag .= mb_chr(ord($countryCode[$i]) + $offset);
        }
        return $flag;
    }
}
```

## Переваги OOP підходу

- ✅ Розділення відповідальностей (Separation of Concerns)
- ✅ Легке тестування через Repository Pattern
- ✅ Повторне використання коду
- ✅ Чиста архітектура
- ✅ Простота розширення функціоналу


## Автор

**Іван Дуцак**  
Виконано в рамках курсу Modern PHP
