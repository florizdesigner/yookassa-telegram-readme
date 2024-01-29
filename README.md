# ЮKassa: интеграция с ботом в Telegram
### Предисловие:
#### Есть два варианта интеграции ЮKassa в бот Telegram: 
* **через [собственное решение](https://yookassa.ru/docs/support/payments/onboarding/integration/cms-module/telegram)**: нативное решение используя API Telegram, магазин в ЮKassa должен быть на e-mail протоколе;
* через самостоятельную интеграцию (обычно используется [протокол API](https://yookassa.ru/developers/payment-acceptance/getting-started/quick-start))

#### Наше решение, плюсы и минусы:
* Нет цепочки редиректов, используется встроенный интерфейс, который понятен пользователю.
* Простая и понятная интеграция.
* Только один способ оплаты, который не поддерживает холдирование, рекуррентные платежи.
* Нет обработки неуспешных платежей, в отличие от API (стандартная ошибка Telegram).

#### Интеграция по API ЮKassa, плюсы и минусы:
* Все доступные способы оплаты, холдирование, рекуррентные платежи.
* Поддержка ФФД 1.2 для тех, у кого подключена интеграции с онлайн-кассой.
* Есть много готовых конструкторов ботов с интеграцией по нашему API.
* Не совсем удобный редирект, если передавать https://t.me (посмотреть редирект через диплинк tg://) **??? Пока не проверил ???**

### Схемы:
#### Интеграция через самостоятельную интеграцию:
![telegram-cloud-photo-size-2-5298704074109212772-y](https://github.com/florizdesigner/yookassa-telegram-readme/assets/56426989/043db79b-066c-4dec-a64a-6555e8be6b48)

#### Интеграция через наше решение:
![telegram-cloud-photo-size-2-5298704074109212784-y](https://github.com/florizdesigner/yookassa-telegram-readme/assets/56426989/46f6c325-a5a1-4361-bac6-710b4060fa8c)
> Схема реализации через метод **createInvoiceLink** осуществляется аналогичным образом, только отрисовка счета происходит при переходе по полученной ссылке

### Визуально:
#### Метод sendInvoice
https://github.com/florizdesigner/yookassa-telegram-readme/assets/56426989/106ca73c-7957-499e-a98a-41afee4f0821

#### Метод createInvoiceLink
https://github.com/florizdesigner/yookassa-telegram-readme/assets/56426989/d8227568-24bd-4c4f-8afe-6bcc57e3393b

#### Самостоятельная интеграция


## Примеры кода:

#### telebot (Python)
```
import json
import telebot

TELEGRAM_BOT_TOKEN = "YOUR_BOT_TOKEN"
TELEGRAM_PROVIDER_TOKEN = "YOUR_PAYMENT_TOKEN"

bot = telebot.TeleBot(TELEGRAM_BOT_TOKEN)

PROVIDER_DATA_WO_EMAIL = {
    "receipt": {
        "items": [{
            "description": "Товар А",
            "quantity": "1.00",
            "amount": {
                "value": "100.00",
                "currency": "RUB"
            },
            "vat_code": 1
        }]
    }
}

@bot.message_handler(commands=['invoice'])
def send_invoice_link(message):
    link = bot.create_invoice_link(
        title="Оплата за услугу #1",
        description="Услуга №1 предоставляется по e-mail",
        payload="order_id=11000",
        provider_token=TELEGRAM_PROVIDER_TOKEN,
        currency="RUB",
        prices=[telebot.types.LabeledPrice("Услуга 1", 10000)],
        need_email=True, 
        send_email_to_provider=True,
        provider_data=json.dumps(PROVIDER_DATA_WO_EMAIL)
    )

    bot.send_message(
        chat_id=message.chat.id,
        text=link
    )


@bot.message_handler(commands=['pay'])
def send_payment(message):
    bot.send_invoice(
        chat_id=message.chat.id,
        title="Оплата",
        description="Оплата_оплата",
        invoice_payload="123123123",
        provider_token=TELEGRAM_PROVIDER_TOKEN,
        currency="RUB",
        prices=[telebot.types.LabeledPrice("Услуга 1", 10000)],
        need_email=True, 
        send_email_to_provider=True,
        provider_data=json.dumps(PROVIDER_DATA_WO_EMAIL)
        )


@bot.pre_checkout_query_handler(lambda query: True)
def pre_checkout_query(pre_checkout_q: telebot.types.PreCheckoutQuery):
    print(pre_checkout_q)
    bot.answer_pre_checkout_query(pre_checkout_q.id, ok=True)

@bot.message_handler(content_types=['successful_payment'])
def successful_payment(message):#: Message):
    bot.send_message(message.chat.id, f"Thanks! Payment was successful, id: {message.successful_payment.provider_payment_charge_id}")

bot.infinity_polling()
```

node
