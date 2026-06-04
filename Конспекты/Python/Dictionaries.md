
## Что такое dictionary

Dictionary (`dict`) - это встроенная структура данных в Python, которая хранит данные в формате ключ - значение (key-value pairs).  
Она используется для быстрого поиска, хранения и изменения данных.

Пример:

```
person = {    "name": "Alex",    "age": 25,    "city": "London"}
```

Здесь:

- `"name"` - ключ
- `"Alex"` - значение
# Основные свойства dictionaries

## 1. Изменяемость (mutable)

Словарь можно изменять после создания:

```
person["age"] = 26
```

Можно:
- добавлять элементы
- изменять
- удалять
## 2. Уникальные ключи

Ключи не могут повторяться:

```
data = {    "a": 1,    "a": 2}print(data)# {'a': 2}
```
## 3. Ключи должны быть hashable

Ключами могут быть:
- строки
- числа
- tuple

Нельзя использовать:
- list
- set
- другие mutable объекты

Пример ошибки:

```
data = {    [1, 2]: "test"}
```
## 4. Быстрый доступ к данным

Python dictionaries реализованы через hash table / hashmap, поэтому поиск по ключу работает очень быстро - примерно O(1).
## 5. Сохранение порядка
# Создание dictionaries

### Через `{}`

```
user = {    "name": "Alice",    "age": 30}
```
### Через `dict()`

```
user = dict(name="Alice", age=30)
```
### Пустой словарь

```
data = {}
```
или
```
data = dict()
```

# Получение значений

### Через `[]`

```
print(user["name"])
```
Минус:
если ключа нет - `KeyError`
### Через `.get()`

```
print(user.get("name"))print(user.get("country", "Unknown"))
```

Плюсы:
- безопасно
- можно задать значение по умолчанию
# Добавление и изменение данных

```
user["city"] = "Paris"user["age"] = 31
```
# Удаление элементов

### del

```
del user["age"]
```

### .pop()

```
city = user.pop("city")
```

# Полезные методы

### .keys()

Возвращает все ключи:

```
user.keys()
```
### .values()

Возвращает значения:

```
user.values()
```
### .items()

Возвращает пары ключ-значение:

```
user.items()
```
# Перебор dictionary

### По ключам

```
for key in user:    print(key)
```
### По значениям

```
for value in user.values():    print(value)
```
### По ключу и значению

```
for key, value in user.items():    print(key, value)
```
### Проверка существования ключа

```
if "name" in user:    print("Exists")
```
### Dictionary comprehension

```
squares = {x: x*x for x in range(5)}
```

Результат:

```
{    0: 0,    1: 1,    2: 4,    3: 9,    4: 16}
```
### Вложенные dictionaries

Словари могут содержать другие словари:

```
student = {    "name": "Tom",    "grades": {        "math": 90,        "python": 100    }}
```

Доступ:

```
student["grades"]["python"]
```
# Dictionaries vs Lists

### List

Доступ по индексу:

```
data[0]
```
### Dictionary

Доступ по ключу:

```
data["name"]
```
# Где используются dictionaries

### JSON

JSON очень похож на Python dict:

```
{    "name": "Alex",    "age": 25}
```
### API

Ответы API обычно приходят как dictionaries.
### Конфигурации

```
settings = {    "theme": "dark",    "language": "en"}
```
### Подсчет данных

```
counter = {}for char in text:    counter[char] = counter.get(char, 0) + 1
```

# Что я понял

Dictionaries - одна из самых важных структур данных в Python.  
Они позволяют удобно организовывать данные и быстро получать к ним доступ.  
Практически любое реальное Python-приложение использует dictionaries:

- backend
- APIs
- JSON
- базы данных
- конфиги
- обработка данных