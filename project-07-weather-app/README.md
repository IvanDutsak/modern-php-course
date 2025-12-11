# Проект 7: Додаток погоди

## Опис проекту

Це веб-додаток для відображення інформації про погоду в обраному місті. Проект розроблений на PHP з використанням принципів об'єктно-орієнтованого програмування та паттерну Strategy через інтерфейси. Додаток надає зручний інтерфейс для перегляду поточної температури та умов погоди з відповідним динамічним оформленням сторінки залежно від типу погоди.

## Технології

- PHP (OOP)
- Interfaces
- Strategy Pattern
- Namespaces
- API Integration (опціонально)
- HTML/CSS (динамічне оформлення)

## Основні функції

1. **Пошук погоди** - введення назви міста
2. **Відображення температури** - поточна температура в °C
3. **Умови погоди** - сонячно, хмарно, дощ, сніг
4. **Динамічний дизайн** - зміна оформлення залежно від погоди
5. **Гнучка архітектура** - легка зміна джерела даних

## Структура проекту (OOP + Strategy Pattern)

```
src/
├── index.php                           # Головна сторінка
├── Interfaces/
│   └── WeatherProviderInterface.php    # Інтерфейс провайдера погоди
├── Providers/
│   ├── ApiWeatherProvider.php          # Реальний API
│   ├── RandomWeatherProvider.php       # Випадкові дані
│   └── TestWeatherProvider.php         # Тестові дані
├── Models/
│   └── Weather.php                     # Модель погоди
├── Services/
│   └── WeatherService.php              # Сервіс для роботи з погодою
├── Helpers/
│   └── TemperatureHelper.php           # Конвертація температури
├── styles/
│   ├── sunny.css                       # Стилі для сонячної погоди
│   ├── cloudy.css                      # Стилі для хмарної погоди
│   ├── rainy.css                       # Стилі для дощової погоди
│   └── snowy.css                       # Стилі для снігової погоди
└── views/
    ├── header.php                      # Шапка
    └── footer.php                      # Підвал
```

## Як запустити

1. Розмістіть файли в директорії веб-сервера
2. (Опціонально) Налаштуйте API ключ в `ApiWeatherProvider.php`
3. Виберіть провайдера погоди в `index.php`
4. Відкрийте `http://localhost/index.php`

## Ключові концепції

### Strategy Pattern через інтерфейси

```php
interface WeatherProviderInterface {
    public function getWeather(string $city): Weather;
}
```

### Реалізація провайдерів

```php
class ApiWeatherProvider implements WeatherProviderInterface {
    private string $apiKey;
    
    public function __construct(string $apiKey) {
        $this->apiKey = $apiKey;
    }
    
    public function getWeather(string $city): Weather {
        $url = "https://api.openweathermap.org/data/2.5/weather?q={$city}&appid={$this->apiKey}";
        $response = file_get_contents($url);
        $data = json_decode($response, true);
        
        return new Weather(
            $city,
            $data['main']['temp'] - 273.15, // Kelvin to Celsius
            $data['weather'][0]['main']
        );
    }
}

class RandomWeatherProvider implements WeatherProviderInterface {
    public function getWeather(string $city): Weather {
        $conditions = ['Sunny', 'Cloudy', 'Rainy', 'Snowy'];
        return new Weather(
            $city,
            rand(-10, 35),
            $conditions[array_rand($conditions)]
        );
    }
}

class TestWeatherProvider implements WeatherProviderInterface {
    public function getWeather(string $city): Weather {
        return new Weather($city, 22, 'Sunny');
    }
}
```

### Модель Weather

```php
class Weather {
    public function __construct(
        public string $city,
        public float $temperature,
        public string $condition
    ) {}
    
    public function getTemperatureInFahrenheit(): float {
        return ($this->temperature * 9/5) + 32;
    }
    
    public function getStylesheet(): string {
        return match(strtolower($this->condition)) {
            'sunny' => 'sunny.css',
            'cloudy' => 'cloudy.css',
            'rainy', 'rain' => 'rainy.css',
            'snowy', 'snow' => 'snowy.css',
            default => 'sunny.css'
        };
    }
}
```

### Використання сервісу

```php
// Вибір провайдера
$provider = new RandomWeatherProvider();
// $provider = new ApiWeatherProvider('your_api_key');
// $provider = new TestWeatherProvider();

$service = new WeatherService($provider);
$weather = $service->getWeatherForCity($_GET['city'] ?? 'Kyiv');
```

### Динамічне підключення стилів

```php
<link rel="stylesheet" href="styles/<?= htmlspecialchars($weather->getStylesheet()) ?>">
```

## Переваги Strategy Pattern

- ✅ Легка зміна джерела даних без зміни коду
- ✅ Тестування без реальних API запитів
- ✅ Розширюваність - легко додати нові провайдери
- ✅ Дотримання принципу Open/Closed (SOLID)
- ✅ Dependency Injection

## Умови погоди та стилі

| Умова | CSS файл | Колір фону | Іконка |
|-------|----------|------------|--------|
| Sunny | sunny.css | Жовтий/Помаранчевий | ☀️ |
| Cloudy | cloudy.css | Сірий | ☁️ |
| Rainy | rainy.css | Синій | 🌧️ |
| Snowy | snowy.css | Білий/Блакитний | ❄️ |

## Скріншоти

Скріншоти роботи додатку доступні в папці `screenshots/`

## Автор

**Іван Дуцак**  
Виконано в рамках курсу Modern PHP
