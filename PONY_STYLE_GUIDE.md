## Pony Language Code Style Guide

от 13.03.2026 г.

### 1. Именование

**Правила:**
* **CamelCase** для типов, акторов, классов, трейтов, интерфейсов: `MyActor`, `HttpClient`, `Serializable`.
* **snake_case** для методов, функций, полей, параметров: `send_request`, `connection_count`, `parse_body(data: Array[U8])`.
* **Заглавные буквы** для констант: `MAX_CONNECTIONS = 1000`, `DEFAULT_PORT = 8080`.
* **Префиксы** для специальных случаев:
    * `_` для приватных полей: `_buffer`, `_is_connected`;
    * `is_`, `has_`, `can_` для булевых методов: `is_connected()`, `has_data()`, `can_write()`.

**Примеры:**
```pony
actor WebServer
  var _connection_count: USize = 0
  let MAX_REQUEST_SIZE: USize = 65536

  be start(port: U16) =>
    // ...

  fun is_running(): Bool =>
    _connection_count > 0
```

### 2. Форматирование кода

**Отступы:** 2 пробела (не табуляции).
**Длина строки:** максимум 80 символов. При превышении — перенос с отступом:
```pony
be send_request(
  url: String,
  headers: Map[String, String] iso,
  body: Array[U8] iso)
=>
  // ...
```
**Пустые строки:**
* 1 пустая строка между методами;
* 2 пустые строки между крупными блоками (акторы, классы);
* не более 2 пустых строк подряд.

### 3. Организация кода в файлах

**Порядок объявлений:**
1. `use` директивы.
2. Константы верхнего уровня.
3. Типы (трейты, интерфейсы, классы, акторы).
4. Функции верхнего уровня (если нужны).

**Пример:**
```pony
use "net"
use "json"

const DEFAULT_PORT: U16 = 8080
const MAX_CONNECTIONS: USize = 1000

trait Serializable
  fun serialize(): String

actor Server
  // ...

fun parse_config(data: String): Map[String, String] =>
  // ...
```

### 4. Комментарии и документация

**Виды комментариев:**
* `#` — однострочный комментарий для пояснений:
```pony
# Оптимизированный буфер для чтения
let _buffer: Array[U8] = Array[U8](4096)
```
* `"""` — многострочные комментарии или документация:
```pony
"""
Обрабатывает HTTP-запрос и отправляет ответ.

Параметры:
- request: полный HTTP-запрос
- conn: соединение для отправки ответа
"""
be handle_request(request: String, conn: TCPConnection ref) =>
  // ...
```

**Требования к документации:**
* документируйте все публичные методы и типы;
* описывайте параметры и возвращаемые значения;
* указывайте побочные эффекты (например, изменение состояния).

### 5. Работа с типами и ссылками

**Рекомендации:**
* явно указывайте типы везде, где это улучшает читаемость;
* используйте `iso` и `trn` для изменяемых данных, передаваемых между акторами;
* избегайте `ref` в публичных API — это может привести к гонкам данных;
* предпочитайте иммутабельные типы (`val`, `box`) там, где это возможно.

**Пример:**
```pony
be process_data(data: Array[U8] iso) =>  # Правильно: владение передаётся
  // ...

be process_data_bad(data: Array[U8] ref) => # Опасно: общий доступ
  // ...
```

### 6. Обработка ошибок

**Подход:** используйте `try`/`else` для явной обработки ошибок, избегайте паники (`error`).

**Пример:**
```pony
try
  conn.write(response.array())?
  conn.flush()?
else
  _out.print("Failed to send response")
  conn.close()
end
```

### 7. Структура акторов

**Рекомендуемый порядок членов:**
1. поля (`let`, `var`);
2. конструктор (`new`);
3. поведения (`be`);
4. методы (`fun`).

**Пример:**
```pony
actor Worker
  let _out: OutStream
  var _requests_handled: USize = 0

  new create(out: OutStream) =>
    _out = out

  be handle_connection(conn: TCPConnection iso) =>
    // ...

  fun get_stats(): USize =>
    _requests_handled
```

---

## Структура проекта

Рекомендуемая структура для нового Pony‑проекта:

```
my-pony-project/
├── src/
│   ├── main.pony                  # Точка входа приложения
│   ├── config/                  # Конфигурация
│   │   ├── config.pony          # Загрузка и парсинг конфигурации
│   │   └── defaults.pony        # Значения по умолчанию
│   ├── http/                  # HTTP-функциональность
│   │   ├── server.pony          # Основной HTTP-сервер
│   │   ├── router.pony          # Маршрутизация запросов
│   │   └── response.pony        # Формирование ответов
│   ├── models/                # Модели данных
│   │   ├── request.pony         # Модель HTTP-запроса
│   │   └── user.pony           # Модель пользователя
│   ├── utils/                 # Вспомогательные функции
│   │   ├── logger.pony         # Логирование
│   │   └── helpers.pony       # Общие утилиты
│   └── services/              # Бизнес-логика
│       ├── auth.pony           # Аутентификация
│       └── database.pony      # Работа с БД
├── tests/
│   ├── unit/                  # Юнит-тесты
│   │   ├── http_test.pony
│   │   └── utils_test.pony
│   └── integration/           # Интеграционные тесты
│       └── server_test.pony
├── build/
│   └── release/               # Собранные бинарники
├── docs/                    # Документация
│   ├── api.md               # API reference
│   └── architecture.md       # Архитектура системы
├── scripts/               # Вспомогательные скрипты
│   ├── build.sh            # Сборка проекта
│   └── deploy.sh           # Деплой
├── .gitignore
├── ponyproject.json       # Конфигурация проекта
└── README.md              # Описание проекта
```

### Содержимое ключевых файлов

**1. `ponyproject.json`**
```json
{
  "name": "my-pony-app",
  "version": "0.1.0",
  "description": "Моё приложение на Pony",
  "main": "src/main.pony",
  "dependencies": [
    {"name": "pony-test", "version": "0.6.0"}
  ]
}
```

**2. `src/main.pony`**
```pony
use "env"

actor Main
  new create(env: Env) =>
    env.out.print("Starting my Pony application...")
    let server = HTTP.Server(env, 8080)
```

**3. `.gitignore`**
```
build/
*.o
*.so
*.dylib
*.dll
*.exe
.DS_Store
__pycache__
```

**4. `README.md`**
```markdown
# My Pony Application

Высокопроизводительное приложение на языке Pony.

## Сборка
```bash
ponyc src/
```

## Запуск
```bash
./my-pony-app
```

## Тесты
```bash
ponyc tests/
./my-pony-app-tests
```
```

---

## Дополнительные рекомендации

1. **Модульность:** разбивайте код на небольшие модули по функциональности.
2. **Единообразие:** придерживайтесь одних и тех же соглашений во всём проекте.
3. **Документация:** документируйте публичные API и сложные алгоритмы.
4. **Тестирование:** покрывайте код тестами, особенно критичные участки.
5. **Рефакторинг:** регулярно пересматривайте и улучшайте код согласно стилю.