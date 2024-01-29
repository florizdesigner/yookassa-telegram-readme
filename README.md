# ЮKassa: интеграция с ботом в Telegram
> **Полезные ссылки:**
> - [Документация нового API ЮKassa](https://yookassa.ru/developers/payment-acceptance/getting-started/quick-start)
> - [Инструкция по подключению нативного решения в Telegram (через методы sendInvoice, createInvoiceLink)](https://yookassa.ru/docs/support/payments/onboarding/integration/cms-module/telegram)
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

### Схемы:
#### Интеграция через самостоятельную интеграцию:
![image](https://github.com/florizdesigner/yookassa-telegram-readme/assets/56426989/eeef7c20-4119-4a70-8a9b-ebeb04e751ba)

#### Интеграция через наше решение:
![telegram-cloud-photo-size-2-5298704074109212784-y](https://github.com/florizdesigner/yookassa-telegram-readme/assets/56426989/46f6c325-a5a1-4361-bac6-710b4060fa8c)
> Схема реализации через метод **createInvoiceLink** осуществляется аналогичным образом, только отрисовка счета происходит при переходе по полученной ссылке

### Процесс нативной интеграции:
Процесс:

1. Отправляем sendInvoice
2. Бот отправляет инвойс клиенту, в ответ возвращает запись следующего вида:
```
{
  message_id: 1330,
  from: {
    id: 123123,
    is_bot: true,
    first_name: 'your_telegram_bot',
    username: 'your_telegram_bot'
  },
  chat: {
    id: 123123,
    first_name: 'first',
    last_name: 'name',
    username: 'username',
    type: 'private'
  },
  date: 1687843481,
  invoice: {
    title: 'Title1',
    description: 'Description1',
    start_parameter: '',
    currency: 'RUB',
    total_amount: 10000
  },
  reply_markup: { inline_keyboard: [ [Array] ] }
}
```
3. Юзер вводит данные карты, если данные корректны и ограничений нет - окно закрывается и необходимо нажать на кнопку "Оплатить". После её нажатия в бот возвращается preCheckoutQuery:
```
{
  id: '123123123123123123',
  from: {
    id: 123123123,
    is_bot: false,
    first_name: 'firstname',
    last_name: 'lastname',
    username: 'username',
    language_code: 'ru'
  },
  currency: 'RUB',
  total_amount: 10000,
  invoice_payload: 'Payload1'
}
```
4. Вы отвечаете методом answerPreCheckoutQuery используя id из запроса выше и начинается процесс оплаты.
5. Если оплата прошла успешно, в бот придёт вебхук SuccessfulPayment (на этом этапе можно реализовать выдачу товара / услуги клиенту):
```
{
  message_id: 1333,
  from: {
    id: 123123123,
    is_bot: false,
    first_name: 'first',
    last_name: 'last',
    username: 'username',
    language_code: 'ru'
  },
  chat: {
    id: 123123,
    first_name: 'first',
    last_name: 'last',
    username: 'username',
    type: 'private'
  },
  date: 1687843710,
  successful_payment: {
    currency: 'RUB',
    total_amount: 10000,
    invoice_payload: 'Payload1',
    telegram_payment_charge_id: '1234567899_123456789_470137_7249233419076009880
',
    provider_payment_charge_id: '22223333-000f-5000-a000-12f6527400d9'
  }
}
```
### Визуально:
#### Метод sendInvoice
https://github.com/florizdesigner/yookassa-telegram-readme/assets/56426989/106ca73c-7957-499e-a98a-41afee4f0821

#### Метод createInvoiceLink
https://github.com/florizdesigner/yookassa-telegram-readme/assets/56426989/d8227568-24bd-4c4f-8afe-6bcc57e3393b

#### Самостоятельная интеграция (один из вариантов, наше видение)ч
https://github.com/florizdesigner/yookassa-telegram-readme/assets/56426989/0155bc25-91d3-4e76-9d52-9f11830ad057

## Примеры кода:

#### telebot (Python) и нативная интеграция (методы: createInvoiceLink, sendInvoice)
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

#### Telegraf.js (JavaScript) и нативная интеграция (метод sendInvoice)
```
const { Telegraf } = require('telegraf')
require('dotenv').config()

const bot = new Telegraf(process.env.TELEGRAM_BOT_TOKEN)

bot.on('pre_checkout_query', ctx => ctx.answerPreCheckoutQuery(true, 'error'))
bot.on('successful_payment', ctx => ctx.reply(`Thanks! Payment is successful, id: ${ctx.message.successful_payment.provider_payment_charge_id}`))

bot.command('payment', ctx => {
    ctx.replyWithInvoice({
        title: "invoice",
        payload: "payload",
        description: "description",
        currency: "RUB",
        need_email: true,
        send_email_to_provider: true,
        prices: [{label: "123123", amount: 10000}],
        provider_token: process.env.TELEGRAM_PROVIDER_TOKEN,
        provider_data: {
            "receipt": {
                "items": [{
                    "description": "123123",
                    "quantity": "1.00",
                    "amount": {
                        "value": "100.00",
                        "currency": "RUB"
                    },
                    "vat_code": 1
                }]
            }
        }
    }, )
})

bot.launch()
```
#### Пример самописной интеграции на JavaScript:

Ссылка на проект — https://github.com/florizdesigner/yookassa_telegram_bot

---                                                                
                                                                                            © tech-support yookassa.ru
