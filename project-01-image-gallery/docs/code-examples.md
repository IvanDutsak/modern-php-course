# Приклади коду - Проект 1: Галерея зображень

## Головний файл gallery.php

```php
<?php
include './inc/functions.inc.php';
include './inc/images.inc.php';
?>
<?php include './views/header.php'; ?>

<div class="gallery-container">
    <?php foreach($imageTitles AS $src => $title): ?>
    <a href="image.php?<?php echo http_build_query(['image' => $src]); ?>" class="gallery-item">
        <h3><?php echo e($title); ?></h3>
        <img src="./images/<?php echo rawurlencode($src); ?>" alt="<?php echo e($title); ?>" />
    </a>
<?php endforeach; ?>
</div>

<?php include './views/footer.php'; ?>
```

## Пояснення коду

### 1. Підключення файлів

```php
include './inc/functions.inc.php';  // Допоміжні функції (e() для екранування)
include './inc/images.inc.php';     // Масив з даними про зображення
```

### 2. Цикл foreach для відображення галереї

```php
foreach($imageTitles AS $src => $title)
```
- `$src` - назва файлу зображення (ключ масиву)
- `$title` - назва зображення (значення масиву)

### 3. Безпечна робота з URL

```php
http_build_query(['image' => $src])  // Створює query string: image=filename.jpg
rawurlencode($src)                   // Кодує назву файлу для URL
```

### 4. Захист від XSS

```php
e($title)  // Функція екранування HTML-символів (htmlspecialchars)
```

## Файл з даними images.inc.php

```php
<?php
$imageTitles = [
    'IMG_0933.jpg' => 'Mountain Landscape',
    'IMG_3280.jpg' => 'Sunset at Beach',
    'IMG_4944.jpg' => 'City Skyline',
    'IMG_9066.jpg' => 'Forest Path'
];
```

## Функції безпеки functions.inc.php

```php
<?php
function e(string $value): string {
    return htmlspecialchars($value, ENT_QUOTES, 'UTF-8');
}
```

## Результат виконання

При відкритті `gallery.php` користувач бачить:
- Сітку зображень з назвами
- Кожне зображення є посиланням
- При кліку відкривається `image.php?image=IMG_0933.jpg`
- URL безпечно закодовані
- HTML-символи в назвах екрановані

## Ключові концепції

1. **Асоціативні масиви** - зберігання пар ключ-значення
2. **GET-параметри** - передача даних через URL
3. **URL encoding** - безпечна робота з спеціальними символами
4. **XSS захист** - екранування виводу
5. **Модульність** - розділення коду на файли
