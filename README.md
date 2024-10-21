import telebot
from telebot import types

# Создаем объект бота с вашим API-токеном
bot = telebot.TeleBot('')

# Токен для оплаты, полученный от BotFather
PAYMENT_PROVIDER_TOKEN = 'YOUR_PAYMENT_PROVIDER_TOKEN'

# Каталог товаров с фото и информацией
catalog = {
    "naik": {
        "description": "naik. Он очень качественный.",
        "price": 2500,  # Цена в рублях
        "photo": "https://t.me/vertualnibazar/30"  # Ссылка на фото товара
    },
    "Свитер": {
        "description": "СВИТЕР. Еще лучше!",
        "price": 2199,
        "photo": "https://t.me/vertualnibazar/22"
    },
    "Tmall": {
        "description": "Tmall -Мужская обувь Tmall 2024, весна и лето, дышащие и универсальные низкие кроссовки на толстой подошве, модные однотонные повседневные кроссовки на меху на шнуровкеаРазмеры: 39-40-41  ",
        "price": 2200,
        "photo": "https://t.me/vertualnibazar/21"
    }
}


# Обрабатываем команду /start
@bot.message_handler(commands=['start'])
def handle_start(message):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    btn1 = types.KeyboardButton('Доставка')
    btn2 = types.KeyboardButton('Каталог')
    btn3 = types.KeyboardButton('Связаться с нами')
    markup.add(btn1, btn2,btn3)

    bot.send_message(message.chat.id, 'Добро пожаловать в наш магазин!\nВыберите действие:', reply_markup=markup)


# Обрабатываем текстовые сообщения для каталога
@bot.message_handler(content_types=['text'])
def handle_text(message):
    if message.text == 'Каталог':
        markup = types.InlineKeyboardMarkup()
        for item in catalog:
            btn = types.InlineKeyboardButton(text=item, callback_data=f"item_{item}")
            markup.add(btn)

        bot.send_message(message.chat.id, 'Выберите товар из каталога:', reply_markup=markup)

    elif message.text == 'Связаться с нами':
         bot.send_message(message.chat.id, 'https://t.me/vertualnibazarPay')

    elif message.text =='Доставка':
        bot.send_message(message.chat.id, 'Доставка по Росси и по Таджикистан.\n'
                                              'Доставка от 20 до25 дней\n'
                                              'Доставка по Росси отправляем через почту вы сами оплачиваете доставку!')

# Обрабатываем нажатие на кнопки товаров из каталога
@bot.callback_query_handler(func=lambda call: call.data.startswith("item_"))
def handle_catalog(call):
    item_name = call.data.split("item_")[1]  # Получаем название товара
    item = catalog.get(item_name, None)

    if item:
        # Отправляем фото и описание товара
        caption = f"Название: {item_name}\nОписание: {item['description']}\nЦена: {item['price']} руб."
        markup = types.InlineKeyboardMarkup()
        buy_btn = types.InlineKeyboardButton(text="Заказать", callback_data=f"buy_{item_name}")
        markup.add(buy_btn)
        bot.send_photo(call.message.chat.id, item['photo'], caption=caption, reply_markup=markup)
    else:
        bot.send_message(call.message.chat.id, "Извините, товар не найден.")
# Обрабатываем событие добавления нового участника
@bot.message_handler(content_types=['start'])
def welcome_new_member(message):
    for new_member in message.new_chat_members:
        # Приветствие для нового участника
        welcome_text = (f"Привет, {new_member.first_name}!\n"
                        "Добро пожаловать в наш магазин мужской одежды! Vertualnibazar🎉\n"
                        "Мы предлагаем качественные и стильные вещи по отличным ценам.\n"
                        "Если у вас есть вопросы, пишите нам в чат или смотрите ассортимент на нашем сайте.")


# Обрабатываем покупку товара
@bot.callback_query_handler(func=lambda call: call.data.startswith("buy_"))
def handle_buy(call):
    item_name = call.data.split("buy_")[1]
    item = catalog.get(item_name, None)

    if item:
        # Вместо отправки счета, отправляем ссылку на менеджера
        manager_url = 'https://t.me/vertualnibazarPay'  # Ссылка на менеджера или чат с менеджером

        # Сообщение с ссылкой на менеджера
        bot.send_message(
            call.message.chat.id,
            f"Чтобы приобрести {item_name}, свяжитесь с нашим менеджером по ссылке: {manager_url}"
        )
    else:
        bot.send_message(call.message.chat.id, "Извините, товар не найден.")

# Запускаем бота
bot.polling(none_stop=True)
