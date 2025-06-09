# Ozzy_codex
import requests
import telebot
from telebot import types
import uuid
from uuid import uuid4
import random
import string
import logging
import os

# Setup logging for error tracking
logging.basicConfig(level=logging.INFO)

def generate_user_agent():
    """Generate a random user agent"""
    agents = [
        "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
        "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
        "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36"
    ]
    return random.choice(agents)

def rest1(user_id, message):
    headers = {
        'authority': 'www.instagram.com',
        'accept': '*/*',
        'accept-language': 'en-US,en;q=0.9,en-GB;q=0.8',
        'content-type': 'application/x-www-form-urlencoded',
        'origin': 'https://www.instagram.com',
        'referer': 'https://www.instagram.com/accounts/password/reset/',
        'sec-ch-prefers-color-scheme': 'dark',
        'sec-ch-ua': '"Not A(Brand";v="8", "Chromium";v="132"',
        'sec-ch-ua-full-version-list': '"Not A(Brand";v="8.0.0.0", "Chromium";v="132.0.6961.0"',
        'sec-ch-ua-mobile': '?0',
        'sec-ch-ua-model': '""',
        'sec-ch-ua-platform': '"Linux"',
        'sec-ch-ua-platform-version': '""',
        'sec-fetch-dest': 'empty',
        'sec-fetch-mode': 'cors',
        'sec-fetch-site': 'same-origin',
        'user-agent': generate_user_agent(),
        'x-asbd-id': '129477',
        'x-csrftoken': 'MoKKhcy0MtQHyxInGtndUr',
        'x-ig-app-id': '936619743392459',
        'x-ig-www-claim': '0',
        'x-instagram-ajax': '1019919946',
        'x-mid': '1ujdauo3vgzs6u1r2s91ofgohk1h2rgm2kmhpc2132ncy3gny20f',
        'x-requested-with': 'XMLHttpRequest',
        'x-web-device-id': 'ECF42637-890D-492D-8B40-4CFA749DB5F5',
        'x-web-session-id': '7ncrkr:ntw37b:pp1uq6',
    }

    data = {
        'email_or_username': user_id,
    }

    try:
        res = requests.post(
            'https://www.instagram.com/api/v1/web/accounts/account_recovery_send_ajax/',
            headers=headers,
            data=data,
        ).json()

        if 'contact_point' in res:
            we = res['contact_point']
            ms = f'''📩 A password reset link has been sent to your Instagram account!
✉️ Associated email: 
[{we}]
            '''
            bot.reply_to(message, ms)
        else:
            gh = '''⚠️ The email or username is not found!
🛠️ Please check the data you entered.
            '''
            bot.reply_to(message, gh)
    except Exception as e:
        logging.error(f"Error in rest1: {e}")
        mn = '''❌ There was an error with the connection or the internet!
🌐 Please check your connection and try again.
        '''
        bot.reply_to(message, mn)

def rest(user_id, message):
    url = "https://i.instagram.com/api/v1/accounts/send_password_reset/"

    payload = {
        'ig_sig_key_version': "4",
        'user_email': user_id,
        'device_id': str(uuid.uuid4()),
    }

    headers = {
        'User-Agent': "Instagram 113.0.0.39.122 Android (30/11; 320dpi; 720x1339; realme; RMX3261; RMX3261; S19610AA1; en_CA)",
        'Connection': "Keep-Alive",
        'Accept-Encoding': "gzip",
        'Cookie2': "$Version=1",
        'Accept-Language': "en-CA, en-US",
        'X-IG-Connection-Type': "WIFI",
        'X-IG-Capabilities': "AQ==",
        'Cookie': "mid=Z4pqeQABAAHARa5XXmMPD5DG3OUA; csrftoken=Fz0IDmyOhyfPHAinkGtwy5RjqpwCfDcK"
    }

    try:
        res = requests.post(url, data=payload, headers=headers).json()

        if 'obfuscated_email' in res:
            se = res['obfuscated_email']
            ms = f'''📩 A password reset link has been sent to your Instagram account!
✉️ Associated email: 
[{se}]
            '''
            bot.reply_to(message, ms)
        elif 'rate_limit_error' in res:
            md = '''Please wait 20 minutes ⏳ and try again.
🕒 Patience is key!
'''
            bot.reply_to(message, md)
        else:
            rest1(user_id, message)
    except Exception as e:
        logging.error(f"Error in rest: {e}")
        rest1(user_id, message)

# Get bot token from environment variables
TOKEN = os.getenv('TELEGRAM_BOT_TOKEN')

# If not found in secrets, add your token here temporarily
if not TOKEN:
    TOKEN = "YOUR_BOT_TOKEN_HERE"  # Replace with your actual token
    
if not TOKEN or TOKEN == "YOUR_BOT_TOKEN_HERE":
    print("❌ Error: Please add your bot token!")
    print("Either set TELEGRAM_BOT_TOKEN in Secrets tab or replace 'YOUR_BOT_TOKEN_HERE' with your actual token")
    exit(1)

bot = telebot.TeleBot(TOKEN)

@bot.message_handler(commands=['start'])
def send_welcome(message):
    welcome_text = """
👋 Welcome to ~𝐎𝐳𝐳𝐲🧃

Ozzy helps you send password reset link for jacked instagram accounts.

• Start exploring by using /help

Made with ♡ by ~𝐎𝐳𝐳𝐲🧃
    """
    # Create menu keyboard
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True, one_time_keyboard=False)
    about_button = types.KeyboardButton("About bot")
    markup.add(about_button)
    
    bot.send_message(message.chat.id, welcome_text, parse_mode='Markdown', reply_markup=markup)

@bot.message_handler(commands=['help'])
def send_help(message):
    help_text = """
⚠️ Provide account username after /reset. Example: /reset meta
    """
    bot.send_message(message.chat.id, help_text, parse_mode='Markdown')

# Message handler for About bot button
@bot.message_handler(func=lambda message: message.text == "About bot")
def handle_about_bot(message):
    about_text = """
👋 Welcome to ~Ozzy🧃

Ozzy helps you send password reset links for jacked instagram accounts.

• Start exploring by using /help

boost your social plugging services using automated form fillers, bots, etc. Contact @m3xxo 

Made with ♡ by ~Mexo
    """
    bot.send_message(message.chat.id, about_text, parse_mode='Markdown')

# Command Handler for the /reset@username command
@bot.message_handler(commands=['reset'])
def handle_reset(message):
    try:
        # Parse username from command
        command_parts = message.text.split()
        if len(command_parts) < 2:
            bot.reply_to(message, "⚠️ Please provide a username. Example: `/reset username` or `/reset @username`", parse_mode='Markdown')
            return
        
        username = command_parts[1].strip()
        # Remove @ if present
        if username.startswith('@'):
            username = username[1:]
        
        if username:
            bot.reply_to(message, f"🔄 Processing reset request for: {username}")
            rest(username, message)
        else:
            bot.reply_to(message, "⚠️ Please provide a valid username. Example: `/reset username`")
    except Exception as e:
        logging.error(f"Error in handle_reset: {e}")
        bot.reply_to(message, "❌ Error occurred while processing request.")

# This handler ignores all non-command messages and keeps the bot quiet
@bot.message_handler(func=lambda message: True)
def handle_other_messages(message):
    bot.reply_to(message, "⚠️ Provide account username after /reset. Example: /reset meta")

print("🔐 Instagram Reset Bot starting...")
print("📚 Educational demonstration mode")
print("⚠️  For learning purposes only")
print("🤖 Bot running... Press Ctrl+C to stop")

# Delete webhook before polling (fixes conflict)
try:
    bot.delete_webhook(drop_pending_updates=True)
    print("✅ Webhook deleted successfully")
except Exception as e:
    logging.warning(f"Could not delete webhook: {e}")

bot.polling(none_stop=True)
