import telebot
import speech_recognition as sr
import wikipedia

# توکن ربات که از BotFather دریافت کردید
TOKEN = '8079579280:AAGiRGVqnyfXQ-BzXJElLwq367yeENOXQzU'
bot = telebot.TeleBot(TOKEN)

# پردازش صدای دریافت شده از کاربر
recognizer = sr.Recognizer()

def recognize_audio(audio_file):
    with sr.AudioFile(audio_file) as source:
        audio = recognizer.record(source)
    try:
        text = recognizer.recognize_google(audio)
        return text
    except sr.UnknownValueError:
        return "متوجه نشدم!"
    except sr.RequestError:
        return "مشکلی در اتصال به سرویس شناسایی صدا پیش آمد."

# جستجو در ویکی‌پدیا
def search_wikipedia(query):
    try:
        result = wikipedia.summary(query, sentences=1)
        return result
    except wikipedia.exceptions.DisambiguationError as e:
        return f"پیشنهادات: {e.options}"

# وقتی کاربر پیامی ارسال می‌کنه
@bot.message_handler(func=lambda message: True)
def handle_message(message):
    if message.content_type == 'text':
        bot.reply_to(message, "پیامی دریافت کردم: " + message.text)
    
    elif message.content_type == 'voice':
        file_info = bot.get_file(message.voice.file_id)
        downloaded_file = bot.download_file(file_info.file_path)
        
        # پردازش صدای ارسال‌شده و تبدیل آن به متن
        recognized_text = recognize_audio(downloaded_file)
        
        # جستجو در ویکی‌پدیا
        search_result = search_wikipedia(recognized_text)
        
        # ارسال نتیجه جستجو به کاربر
        bot.reply_to(message, f"نتیجه جستجو برای '{recognized_text}':\n{search_result}")

# شروع به دریافت و پاسخ دادن به پیام‌ها
bot.polling()