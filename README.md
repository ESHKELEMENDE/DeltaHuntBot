# DeltaHunterBot

🚀 Telegram-бот для криптовалютного арбитража между биржами.

Сравнивает цены между:
- Binance
- Bybit
- OKX
- KuCoin
- Gate.io
- MEXC
- Coinbase
- HTX (ex-Huobi)
- Bitget
- Kraken

📊 Отправляет арбитражные сигналы пользователю.
📦 Язык: Python
DeltaHuntBot/
├── main.py
├── config.py
├── requirements.txt
└── utils/
    └── fetch_prices.py
    📁 Структура проекта
DeltaHunterBot
├── main.py
├── config.py
├── exchanges.py
├── arbitrage.py
├── subscription.py
├── utils.py
├── requirements.txt
└── .gitignore
======= .gitignore =======
pycache/ .env *.pyc
======= requirements.txt =======
python-telegram-bot==20.6 requests python-dotenv
======= config.py =======
import os from dotenv import load_dotenv
load_dotenv()
TELEGRAM_TOKEN = os.getenv("TELEGRAM_TOKEN") ARBITRAGE_THRESHOLD = 0.5 # В процентах
======= exchanges.py =======
import requests
def get_price_binance(symbol="BTCUSDT"): url = f"https://api.binance.com/api/v3/ticker/price?symbol={symbol}" return float(requests.get(url).json()['price'])
Можно добавить аналогичные функции для других бирж позже
======= arbitrage.py =======
def find_opportunity(prices: dict, threshold: float): results = [] exchanges = list(prices.keys()) for i in range(len(exchanges)): for j in range(i + 1, len(exchanges)): ex1, ex2 = exchanges[i], exchanges[j] p1, p2 = prices[ex1], prices[ex2] diff = abs(p1 - p2) / min(p1, p2) * 100 if diff >= threshold: results.append((ex1, ex2, round(diff, 2))) return results
======= subscription.py =======
subscribers = {}
def subscribe(user_id): subscribers[user_id] = True
def unsubscribe(user_id): if user_id in subscribers: del subscribers[user_id]
def is_subscribed(user_id): return subscribers.get(user_id, False)
======= utils.py =======
def format_opportunities(opps): if not opps: return "Нет выгодных пар сейчас." return "\n".join([f"{ex1} ↔ {ex2} | Δ {diff}%" for ex1, ex2, diff in opps])
======= main.py =======
from telegram import Update from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes from config import TELEGRAM_TOKEN, ARBITRAGE_THRESHOLD from exchanges import get_price_binance from arbitrage import find_opportunity from subscription import subscribe, unsubscribe, is_subscribed from utils import format_opportunities
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE): user_id = update.effective_user.id subscribe(user_id) await update.message.reply_text("Вы подписались на арбитражные сигналы 🟢")
async def stop(update: Update, context: ContextTypes.DEFAULT_TYPE): user_id = update.effective_user.id unsubscribe(user_id) await update.message.reply_text("Вы отписались от сигналов ❌")
async def check(update: Update, context: ContextTypes.DEFAULT_TYPE): user_id = update.effective_user.id if not is_subscribed(user_id): await update.message.reply_text("Подпишитесь через /start, чтобы получать сигналы") return
prices = { "Binance": get_price_binance(), # Добавь другие биржи сюда "FakeExchange": get_price_binance() * 1.01 # для демонстрации разницы } opps = find_opportunity(prices, ARBITRAGE_THRESHOLD) message = format_opportunities(opps) await update.message.reply_text(message) 
if name == 'main': app = ApplicationBuilder().token(TELEGRAM_TOKEN).build() app.add_handler(CommandHandler("start", start)) app.add_handler(CommandHandler("stop", stop)) app.add_handler(CommandHandler("check", check)) print("DeltaHunterBot запущен...") app.run_polling()
