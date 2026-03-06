#Спринт #Тг-бот #Музыка #Таймаут
```python
import asyncio
import random
import aiohttp

from aiogram import Bot, Dispatcher
from aiogram.filters import Command
from aiogram.types import Message


# НАСТРОЙКИ

TOKEN = "8205786674:AAF0JYnQBU7F6-hXQ0eqjYoJZyxAFhlxKsA"
SEARCH_LIMIT = 5
JSON_TIMEOUT = 10

bot = Bot(token="8205786674:AAF0JYnQBU7F6-hXQ0eqjYoJZyxAFhlxKsA")
dp = Dispatcher()

session: aiohttp.ClientSession | None = None


# HTTP HELPER

async def fetch_json(url: str, params: dict | None = None):
    try:
        async with session.get(
            url,
            params=params,
            timeout=aiohttp.ClientTimeout(total=JSON_TIMEOUT)
        ) as resp:

            if resp.status != 200:
                return None

            return await resp.json(content_type=None)

    except Exception as e:
        print("Fetch error:", e)
        return None


# ПОИСК

async def search_music(query, limit=SEARCH_LIMIT):
    url = "https://itunes.apple.com/search"
    params = {
        "term": query,
        "entity": "song",
        "limit": limit,
        "country": "RU"
    }

    data = await fetch_json(url, params)
    if not data:
        return []

    results = data.get("results", [])
    return [r for r in results if r.get("previewUrl")]


async def search_by_artist(artist, limit=SEARCH_LIMIT):
    return await search_music(artist, limit)


async def search_by_track(track, limit=SEARCH_LIMIT):
    return await search_music(track, limit)


async def get_random_tracks(limit=SEARCH_LIMIT):
    terms = ["love", "pop", "rock", "dance", "rap", "hit"]
    results = await search_music(random.choice(terms), limit=20)

    if not results:
        return []

    return random.sample(results, min(limit, len(results)))


async def get_top_tracks(limit=SEARCH_LIMIT):
    results = await search_music("top hits 2025", limit=20)

    if not results:
        return []

    return results[:limit]


# ОТПРАВКА (БЕЗ СКАЧИВАНИЯ)

async def send_preview(message: Message, track: dict):
    url = track.get("previewUrl")
    if not url:
        return

    try:
        await message.answer_audio(
            audio=url,
            title=track.get("trackName", "Unknown"),
            performer=track.get("artistName", "Unknown"),
            caption="🎵 Вот твой трек"
        )
    except Exception as e:
        print("Send preview error:", e)


async def send_tracks_parallel(message: Message, tracks: list):
    tasks = [send_preview(message, t) for t in tracks]
    await asyncio.gather(*tasks)


# КОМАНДЫ 
@dp.message(Command("start"))
async def start(message: Message):
    await message.answer(
        "🎵 Привет!\n\n"
        "Напиши название трека или исполнителя.\n\n"
        "Команды:\n"
        "/track <название>\n"
        "/artist <имя>\n"
        "/random\n"
        "/top\n"
        "/help"
    )


@dp.message(Command("help"))
async def help_cmd(message: Message):
    await message.answer(
        " Команды:\n\n"
        "/track <название>\n"
        "/artist <имя>\n"
        "/random\n"
        "/top\n\n"
        "Можно просто написать текст для поиска."
    )


@dp.message(Command("track"))
async def track_cmd(message: Message):
    args = message.text.split(maxsplit=1)
    if len(args) < 2:
        await message.answer("Укажи название после /track")
        return

    tracks = await search_by_track(args[1])
    if not tracks:
        await message.answer("Ничего не найдено")
        return

    await send_tracks_parallel(message, tracks)


@dp.message(Command("artist"))
async def artist_cmd(message: Message):
    args = message.text.split(maxsplit=1)
    if len(args) < 2:
        await message.answer("Укажи исполнителя после /artist")
        return

    tracks = await search_by_artist(args[1])
    if not tracks:
        await message.answer("Исполнитель не найден")
        return

    await send_tracks_parallel(message, tracks)


@dp.message(Command("random"))
async def random_cmd(message: Message):
    tracks = await get_random_tracks()
    if not tracks:
        await message.answer("Не удалось найти треки")
        return

    await send_tracks_parallel(message, tracks)


@dp.message(Command("top"))
async def top_cmd(message: Message):
    tracks = await get_top_tracks()
    if not tracks:
        await message.answer("Топ временно недоступен")
        return

    await send_tracks_parallel(message, tracks)


@dp.message()
async def text_search(message: Message):
    query = message.text.strip()
    if not query:
        return

    tracks = await search_music(query)
    if not tracks:
        await message.answer("Ничего не найдено")
        return

    await send_tracks_parallel(message, tracks)


# ЗАПУСК 

async def main():
    global session
    session = aiohttp.ClientSession()

    print(" Музыкальный бот запущен ")

    try:
        await dp.start_polling(bot)
    finally:
        await session.close()


if __name__ == "__main__":
    asyncio.run(main())
```

Я сделал таймаут пока что частично, потому что он только задаёт ограничение времени для запроса, но полностью не обрабатывается. В коде нет отдельной обработки ошибки таймаута, она попадает в общий except Exception, поэтому нельзя точно определить, что произошёл именно таймаут.