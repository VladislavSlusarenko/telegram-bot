# telegram-bot
import json
import telebot
import sqlite3
import requests
bot = telebot.TeleBot("6508339301:AAF-APNX1-jF_BroEWhzLEvBmJHRUIytG20")
API="f1965553ccae9b7f2fcae57534b7cce0"

name = None

@bot.message_handler(commands=['start'])
def start(message):
    conn = sqlite3.connect('vlad.sql')
    cur = conn.cursor()

    cur.execute('CREATE TABLE IF NOT EXISTS users(id int auto_increment primary key, name varchar (50), pass varchar (50))')
    conn.commit()
    cur.close()
    conn.close()

    bot.send_message(message.chat.id, ' Hello, we will register you now!Enter your name:')
    bot.register_next_step_handler(message, user_name)


def user_name(message):
    global name
    name = message.text.strip()
    bot.send_message(message.chat.id, ' Enter password:')
    bot.register_next_step_handler(message, user_pass)


def user_pass(message):
    password = message.text.strip()

    conn = sqlite3.connect('vlad.sql')
    cur = conn.cursor()

    cur.execute("INSERT INTO users(name, pass) VALUES ('%s','%s')" % (name,password))
    conn.commit()
    cur.close()
    conn.close()

    markup = telebot.types.InlineKeyboardMarkup()
    bot.send_message(message.chat.id, ' User registered!''Write city name ',reply_markup=markup)
    bot.register_next_step_handler(message, Посмотреть_погоду)

@bot.message_handler( callback_data= ["View_weather"])
def Посмотреть_погоду(message):
    bot.send_message(message.chat.id,"Sorry, write the name of the city again")



@bot.message_handler(content_types=['text'])
def get_weather(message):
    city = message.text.strip().lower()
    res = requests.get(f" https://api.openweathermap.org/data/2.5/weather?q={city }&appid={API }&units=metric")
    if res.status_code ==200:
        data = json.loads(res.text)
        temp = data["main"]["temp"]
        bot.reply_to(message,f"Now weather:{temp}")

        image = "зображеня2.png" if temp > 5.0 else "зображеня .png"
        file = open('./' + image, 'rb')
        bot.send_photo(message.chat.id,file)



    else:
        bot.reply_to(message,"The city is not correct")



bot.polling(none_stop=True)

