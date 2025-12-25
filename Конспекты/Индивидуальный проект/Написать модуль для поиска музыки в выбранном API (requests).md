```python
import requests

class MusicSearcher:
    ITUNES_API_URL = "https://itunes.apple.com/search"

    def __init__(self, country='US', limit=5):
        self.country = country
        self.limit = limit

    def search(self, query):
        params = {
            'term': query,
            'country': self.country,
            'media': 'music',
            'limit': self.limit
        }
        try:
            response = requests.get(self.ITUNES_API_URL, params=params)
            response.raise_for_status()
        except requests.RequestException as e:
            print(f"Ошибка запроса: {e}")
            return []

        results = response.json().get('results', [])
        return [
            {
                'track': item.get('trackName'),
                'artist': item.get('artistName'),
                'album': item.get('collectionName'),
                'preview': item.get('previewUrl')
            } for item in results
        ]
```

Импортируем requests – это чтобы программа могла обращаться к интернету и получать данные.
    
Создаём класс MusicSearcher – как «поисковик музыки», который знает, куда отправлять запросы (iTunes API).
    
Конструктор `__init__` – задаёт параметры поиска: страну и сколько треков вернуть.
    
Метод `search` – это основная работа:
    - Берёт то, что ты хочешь найти (имя исполнителя или песню).
    - Формирует запрос к iTunes с нужными параметрами.    
    - Отправляет запрос в интернет.
    - Если сервер вернул ошибку, просто возвращает пустой список.
        
Обработка ответа – берём то, что пришло от iTunes, и выбираем только нужные поля: название песни, исполнителя, альбом и ссылку на прослушивание.
    - Всё это складываем в список словарей, чтобы с ним было удобно работать.

### Что она делает:
Она берёт название песни или имя исполнителя, которое ты ей говоришь.
    
Идёт в интернет (к iTunes API) и спрашивает: “Эй, покажи мне треки, которые подходят под этот запрос”.
    
Получает список песен от iTunes (это огромный JSON-файл с кучей информации).
    
Берёт из этого списка только важное: название песни, кто её исполняет, альбом, ссылку на прослушивание.
    
Складывает всё в удобный список, чтобы можно было легко показать или использовать.
    
Выводит результат на экран, если мы просим показать результат.