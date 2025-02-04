import pandas as pd
import random
import asyncio
from aiogram import Bot, Dispatcher, types
from aiogram.types import Message
from aiogram.filters import Command
import os

TOKEN = os.getenv("TELEGRAM_BOT_TOKEN")  # Используем переменные окружения
if not TOKEN:
    raise ValueError("Отсутствует токен бота! Установите TELEGRAM_BOT_TOKEN в переменных окружения.")

bot = Bot(token=TOKEN)
dp = Dispatcher()

def load_price_database():
    """Загружает базу данных с ценами номеров."""
    data = {
        "Номер": ["А777АА 77", "М555ММ 99", "О001ОО 77", "Т222ТТ 78", "К888КК 97"],
        "Категория": ["Очень красивый", "Очень красивый", "Элитный", "Красивый", "Красивый"],
        "Цена (руб.)": [5000000, 3500000, 7000000, 1500000, 2000000]
    }
    return pd.DataFrame(data)

def generate_price(number):
    """Генерирует случайную цену для стандартных номеров."""
    base_price = random.randint(10000, 100000)  # Базовая стоимость обычного номера
    variation = random.randint(-5000, 5000)  # Небольшая вариация в цене
    return max(1000, base_price + variation)  # Минимальная цена - 1000 руб.

def analyze_number(number, db):
    """Анализирует номер и возвращает его цену и категорию."""
    result = db[db["Номер"] == number]
    if not result.empty:
        return result.iloc[0].to_dict()
    else:
        return {"Номер": number, "Категория": "Обычный", "Цена (руб.)": generate_price(number)}

@dp.message(Command("start"))
async def cmd_start(message: Message):
    await message.answer("Привет! Отправь мне номер автомобиля, и я скажу его примерную стоимость 🚗💰")

@dp.message()
async def handle_message(message: Message):
    db = load_price_database()
    user_input = message.text.strip()
    result = analyze_number(user_input, db)
    response = f"Номер: {result['Номер']}\nКатегория: {result['Категория']}\nЦена: {result['Цена (руб.)']} руб."
    await message.answer(response)

async def main():
    print("Бот запущен 🚀")
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
