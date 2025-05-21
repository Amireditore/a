import zipfile
import os

# ساخت پوشه موقت برای فایل‌ها
os.makedirs("/mnt/data/amir_test_bot", exist_ok=True)

# کد اصلی ربات
bot_code = '''\
import logging
import requests
from telegram import Update, ReplyKeyboardMarkup, InputMediaPhoto
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes

BOT_TOKEN = "7986506143:AAHYJ-FQ4idpiq5pywGsnxsg9l9s95xPPKs"
AUDD_API_TOKEN = "9dffe2081af72af735eaa794bf3876e1"

logging.basicConfig(level=logging.INFO)

keyboard = ReplyKeyboardMarkup(
    [["راهنما", "درباره ما"], ["پشتیبانی"]],
    resize_keyboard=True
)

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    welcome_text = (
        "سلام! من Amir Test هستم.\n"
        "کافیه یه فایل صوتی یا voice برام بفرستی تا اسم آهنگ رو برات پیدا کنم.\n"
        "برای دیدن راهنما دکمه راهنما رو بزن."
    )
    await update.message.reply_text(welcome_text, reply_markup=keyboard)

async def handle_audio(update: Update, context: ContextTypes.DEFAULT_TYPE):
    file = None
    if update.message.audio:
        file = await update.message.audio.get_file()
    elif update.message.voice:
        file = await update.message.voice.get_file()
    else:
        await update.message.reply_text("لطفاً فقط فایل صوتی یا ویس بفرست.")
        return

    file_path = await file.download_to_drive()

    files = {'file': open(file_path, 'rb')}
    data = {'api_token': AUDD_API_TOKEN, 'return': 'apple_music,spotify'}
    response = requests.post('https://api.audd.io/', data=data, files=files)
    result = response.json()

    if result['status'] == 'success' and result['result']:
        song = result['result']
        title = song.get('title', 'نامشخص')
        artist = song.get('artist', 'نامشخص')
        album = song.get('album', 'نامشخص')
        release_date = song.get('release_date', 'نامشخص')

        spotify_link = song.get('spotify', {}).get('external_urls', {}).get('spotify', None)
        apple_music_link = song.get('apple_music', {}).get('url', None)
        cover_url = None

        if spotify_link:
            cover_url = song.get('spotify', {}).get('album', {}).get('images', [{}])[0].get('url', None)
        if not cover_url and apple_music_link:
            cover_url = song.get('apple_music', {}).get('artwork', {}).get('url', None)

        response_text = (
            f"اسم آهنگ: {title}\\n"
            f"خواننده: {artist}\\n"
            f"آلبوم: {album}\\n"
            f"تاریخ انتشار: {release_date}\\n"
        )

        if spotify_link:
            response_text += f"Spotify: {spotify_link}\\n"
        if apple_music_link:
            response_text += f"Apple Music: {apple_music_link}\\n"

        if cover_url:
            await update.message.reply_photo(photo=cover_url, caption=response_text)
        else:
            await update.message.reply_text(response_text)
    else:
        await update.message.reply_text("متاسفانه نتونستم آهنگ رو شناسایی کنم. لطفاً فایل واضح‌تری بفرست.")

async def handle_text(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text

    if text == "راهنما":
        help_text = (
            "کافیه یه فایل صوتی یا voice برام بفرستی.\\n"
            "من اسم آهنگ، خواننده و لینک‌های مربوطه رو پیدا می‌کنم."
        )
        await update.message.reply_text(help_text)

    elif text == "درباره ما":
        about_text = (
            "ربات Amir Test توسط یک برنامه‌نویس حرفه‌ای ساخته شده.\\n"
            "هدفش کمک به شما برای شناسایی آهنگ‌هاست."
        )
        await update.message.reply_text(about_text)

    elif text == "پشتیبانی":
        support_text = (
            "برای پشتیبانی با آدرس ایمیل زیر تماس بگیر:\\n"
            "amir.test.support@example.com"
        )
        await update.message.reply_text(support_text)
    else:
        await update.message.reply_text("لطفاً یکی از دکمه‌های موجود را انتخاب کن.")

def main():
    app = ApplicationBuilder().token(BOT_TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.AUDIO | filters.VOICE, handle_audio))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_text))

    print("ربات Amir Test اجرا شد.")
    app.run_polling()

if __name__ == "__main__":
    main()
'''

# ذخیره کد در فایل
code_path = "/mnt/data/amir_test_bot/amir_test_bot.py"
with open(code_path, "w", encoding="utf-8") as f:
    f.write(bot_code)

# فایل requirements.txt
requirements = '''\
python-telegram-bot==20.0
requests
'''

requirements_path = "/mnt/data/amir_test_bot/requirements.txt"
with open(requirements_path, "w", encoding="utf-8") as f:
    f.write(requirements)

# فایل Procfile برای هاست
procfile_content = "worker: python amir_test_bot.py"
procfile_path = "/mnt/data/amir_test_bot/Procfile"
with open(procfile_path, "w", encoding="utf-8") as f:
    f.write(procfile_content)

# ساخت فایل ZIP
zip_path = "/mnt/data/amir_test_bot.zip"
with zipfile.ZipFile(zip_path, 'w') as zipf:
    zipf.write(code_path, arcname="amir_test_bot.py")
    zipf.write(requirements_path, arcname="requirements.txt")
    zipf.write(procfile_path, arcname="Procfile")

zip_path

