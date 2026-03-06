#Спринт #Тг-бот #Музыка #try #except #retry
### Реализация retry для fetch_json
```python
import asyncio

MAX_RETRIES = 3
RETRY_DELAY = 2  # секунды

async def fetch_json(url: str, params: dict | None = None):
    for attempt in range(1, MAX_RETRIES + 1):
        try:
            async with session.get(
                url,
                params=params,
                timeout=aiohttp.ClientTimeout(total=JSON_TIMEOUT)
            ) as resp:
                if resp.status != 200:
                    raise Exception(f"HTTP статус {resp.status}")
                return await resp.json(content_type=None)

        except Exception as e:
            print(f"Fetch attempt {attempt} failed: {e}")
            if attempt < MAX_RETRIES:
                await asyncio.sleep(RETRY_DELAY)
            else:
                return None

```

### Retry для send_preview
```python
async def send_preview(message: Message, track: dict):
    url = track.get("previewUrl")
    if not url:
        return

    for attempt in range(1, MAX_RETRIES + 1):
        try:
            await message.answer_audio(
                audio=url,
                title=track.get("trackName", "Unknown"),
                performer=track.get("artistName", "Unknown"),
                caption="🎵 Вот твой трек"
            )
            return  # успешно — выходим
        except Exception as e:
            print(f"Send preview attempt {attempt} failed: {e}")
            if attempt < MAX_RETRIES:
                await asyncio.sleep(RETRY_DELAY)
            else:
                print(f"Не удалось отправить трек: {track.get('trackName')}")
```


Если iTunes временно не отвечает - бот автоматически попробует ещё раз
Telegram-ошибки при отправке трека (например, медленная сеть) тоже обрабатываются
Ограниченное число повторов предотвращает бесконечные циклы.

Потом думаю сделаю:

Обернуть все функции поиска (get_random_tracks, get_top_tracks) через fetch_json с retry
Можно добавить экспоненциальную задержку (2, 4, 8 сек) для ещё большей устойчивости.