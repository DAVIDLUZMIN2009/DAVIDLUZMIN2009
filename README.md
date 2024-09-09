import telebot
from telebot import types

# Створення бота з токеном
bot = telebot.TeleBot('7288720778:AAFOfEDWgUAkxE49tlbm97gBl1tCWUG-dfE')

# Словник для зберігання балансу користувачів
user_balances = {}

# Словник для зберігання цін на рефералів
referral_prices = {
    "Hamster Kombat": 30,
    "City": 30,
    "Lost Dogs": 20,
    "Blum": 18,
    "Major": 15,
    "Musk Empeire": 15,
    "1 win Token": 10,
    "Інше": 9
}

# Змінні для зберігання інформації про замовлення
order_info = {}

# Команда старт
@bot.message_handler(commands=['start'])
def start_command(message):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    profile_button = types.KeyboardButton("👤 Профіль")
    deposit_button = types.KeyboardButton("💳 Поповнення")
    order_button = types.KeyboardButton("🛍️ Замовити")
    support_button = types.KeyboardButton("👨‍💻 Підтримка")

    markup.add(profile_button, deposit_button, order_button, support_button)
    bot.send_message(message.chat.id, "Привіт! Наш бот допоможе вам легко та зручно купувати рефералів для різних платформ. За доступною ціною💰", reply_markup=markup)

# Обробка кнопок
@bot.message_handler(func=lambda message: True)
def menu_handler(message):
    if message.text == "👤 Профіль":
        user_id = message.from_user.id
        balance = user_balances.get(user_id, 0)

        markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
        main_menu_button = types.KeyboardButton("👈 Головне меню")
        markup.add(main_menu_button)

        bot.send_message(message.chat.id, f"Ваш ID: {user_id}\nВаш баланс: {balance} UAH", reply_markup=markup)

    elif message.text == "💳 Поповнення":
        markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
        main_menu_button = types.KeyboardButton("👈 Головне меню")
        markup.add(main_menu_button)

        bot.send_message(message.chat.id, "Введіть суму поповнення:", reply_markup=markup)
        bot.register_next_step_handler(message, process_deposit)

    elif message.text == "🛍️ Замовити":
        process_referral_order(message)

    elif message.text == "👨‍💻 Підтримка":
        markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
        main_menu_button = types.KeyboardButton("👈 Головне меню")
        markup.add(main_menu_button)

        bot.send_message(message.chat.id, "Щоб зв'язатися з підтримкою, напишіть нам, і ми найближчим часом зв'яжемося з вами\n@Dav_i_dK", reply_markup=markup)

    elif message.text == "👈 Головне меню":
        start_command(message)

def process_deposit(message):
    try:
        deposit_amount = int(message.text)
        user_id = message.from_user.id

        markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
        pay_button = types.KeyboardButton("💸 Оплатити")
        main_menu_button = types.KeyboardButton("👈 Головне меню")
        markup.add(pay_button, main_menu_button)

        bot.send_message(message.chat.id, f"Сума поповнення: {deposit_amount} UAH", reply_markup=markup)
        bot.register_next_step_handler(message, lambda msg: confirm_payment(msg, deposit_amount))

    except ValueError:
        bot.send_message(message.chat.id, "Будь ласка, введіть дійсну суму.")

def confirm_payment(message, deposit_amount):
    if message.text == "💸 Оплатити":
        bot.send_message(message.chat.id, f"Сума поповнення: {deposit_amount} UAH\nНомер рахунку: 5375 4114 3033 5070\nВідправте чек з оплатою в чат.")
        bot.register_next_step_handler(message, lambda msg: handle_receipt(msg, deposit_amount))
    else:
        start_command(message)

def handle_receipt(message, deposit_amount):
    if message.photo:
        admin_id = 6603912642

        # Відправка чеку адміністратору
        bot.send_photo(admin_id, message.photo[-1].file_id, caption=f"Користувач: {message.from_user.id}\nСума поповнення: {deposit_amount} UAH")

        # Створення кнопок для адміністратора
        markup = types.InlineKeyboardMarkup()
        confirm_button = types.InlineKeyboardButton("✅ Підтвердити", callback_data=f"confirm_{message.from_user.id}_{deposit_amount}")
        cancel_button = types.InlineKeyboardButton("❌ Скасувати", callback_data=f"cancel_{message.from_user.id}_{deposit_amount}")
        markup.add(confirm_button, cancel_button)

        # Відправка адміністратору повідомлення з кнопками
        bot.send_message(admin_id, "Виберіть дію:", reply_markup=markup)

        # Відправка користувачу повідомлення з подякою
        bot.send_message(message.chat.id, "Дякуємо! Очікуйте підтвердження депозиту. Вам повідомлять, коли депозит буде підтверджено.")
    else:
        bot.send_message(message.chat.id, "Будь ласка, відправте фото чека.")

def process_referral_order(message):
    markup = types.InlineKeyboardMarkup()
    for option, price in referral_prices.items():
        callback_data = f"referral_{option}"
        markup.add(types.InlineKeyboardButton(f"{option} - {price} UAH", callback_data=callback_data))

    bot.send_message(message.chat.id, "Виберіть тип реферала:", reply_markup=markup)

def calculate_order(message, referral_type):
    try:
        quantity = int(message.text)
        price_per_referral = referral_prices[referral_type]
        total_amount = price_per_referral * quantity

        # Збереження інформації про замовлення
        order_info[message.from_user.id] = {
            'referral_type': referral_type,
            'quantity': quantity,
            'total_amount': total_amount
        }

        markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
        pay_button = types.KeyboardButton("💸 Оплатити")
        main_menu_button = types.KeyboardButton("👈 Головне меню")
        markup.add(pay_button, main_menu_button)

        bot.send_message(message.chat.id, f"Сума за одного реферала: {price_per_referral} UAH\nКількість рефералів: {quantity}\nСума замовлення: {total_amount} UAH", reply_markup=markup)
        bot.register_next_step_handler(message, lambda msg: process_payment(msg, total_amount))

    except ValueError:
        bot.send_message(message.chat.id, "Будь ласка, введіть дійсну кількість.")

def process_payment(message, total_amount):
    if message.text == "💸 Оплатити":
        bot.send_message(message.chat.id, f"Відправте ваше реферальне посилання на бота.")
        bot.register_next_step_handler(message, lambda msg: handle_order_payment(msg, total_amount))
    else:
        start_command(message)

def handle_order_payment(message, total_amount):
    user_id = message.from_user.id

    if message.text:
        user_balance = user_balances.get(user_id, 0)
        if user_balance >= total_amount:
            user_balances[user_id] -= total_amount

            admin_id = 6603912642
            # Отримання інформації про замовлення
            order = order_info.get(user_id, {})
            referral_type = order.get('referral_type')
            quantity = order.get('quantity')

            bot.send_message(admin_id, f"Замовлення:\nКористувач: {user_id}\nСума замовлення: {total_amount} UAH\nТип: {referral_type}\nКількість: {quantity}\nПосилання на оплату: {message.text}")

            bot.send_message(message.chat.id, "Очікуйте на зарахування рефералів.")
        else:
            bot.send_message(message.chat.id, "На вашому рахунку недостатньо коштів. Поповніть рахунок.")
            start_command(message)
    else:
        bot.send_message(message.chat.id, "Будь ласка, відправте посилання на оплату.")

@bot.callback_query_handler(func=lambda call: True)
def callback_query(call):
    data = call.data.split('_')
    if data[0] == "referral":
        referral_type = data[1]
        markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
        main_menu_button = types.KeyboardButton("👈 Головне меню")
        markup.add(main_menu_button)

        bot.send_message(call.message.chat.id, f"Введіть кількість рефералів для {referral_type}:", reply_markup=markup)
        bot.register_next_step_handler(call.message, lambda msg: calculate_order(msg, referral_type))
    elif data[0] == "confirm":
        user_id = int(data[1])
        deposit_amount = int(data[2])
        user_balances[user_id] = user_balances.get(user_id, 0) + deposit_amount

        # Оновлення повідомлення у адміністратора, щоб видалити кнопки
        bot.edit_message_text(chat_id=call.message.chat.id, message_id=call.message.message_id,
                              text=f"Підтверджено поповнення на {deposit_amount} UAH.")

        bot.send_message(user_id, f"Ваш депозит на {deposit_amount} UAH підтверджено!")
    elif data[0] == "cancel":
        user_id = int(data[1])
        deposit_amount = int(data[2])

        # Оновлення повідомлення у адміністратора, щоб видалити кнопки
        bot.edit_message_text(chat_id=call.message.chat.id, message_id=call.message.message_id,
                              text=f"Скасовано поповнення на {deposit_amount} UAH.")

        bot.send_message(user_id, f"Ваш депозит на {deposit_amount} UAH скасовано.")

# Запуск бота
bot.polling(none_stop=True)
