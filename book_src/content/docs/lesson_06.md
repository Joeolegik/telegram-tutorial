---
title: "Урок 6. Собираем аналитику при помощи Botan (неактуально)"
type: docs
slug: "lesson_06"
BookToC: 3
---

{{< expand "⚠️ Важно!" >}}

Сервис Botan в середине 2018 года [был отключен](https://github.com/botanio/sdk) и больше не работает. Урок оставлен исключительно для истории.

{{< /expand >}}

Вы наверняка уже видели [новость](https://vk.com/wall-90229462_2725) о такой чудесной системе аналитики для ботов Telegram, как [Botan](https://github.com/botanio/sdk). Сам я ей пользуюсь уже несколько месяцев, поэтому могу с уверенностью сказать, что штука эта очень полезная и качественная. Сегодня мы научимся пользоваться Ботаном в наших Telegram-ботах. В данном уроке будет рассмотрено только подключение бота к Botan через веб-интерфейс, всё, что касается их бота, вы можете рассмотреть самостоятельно.

## Регистрируемся в Yandex AppMetrica

Первое, что необходимо сделать, это перейти по адресу https://appmetrica.yandex.com и войти при помощи своей Яндекс-почты. После того, как вы попадете в панель управления, нажимайте на "+ Добавить приложение". В появившемся окне нужно ввести ник бота (вместе с "собакой"), прочитать условия пользовательского соглашения и согласиться с ними, поставив галку напротив соответствующего пункта.

{{% img "l6_1.jpg" %}}Добавление приложения{{% /img %}}

После нажатия на "Добавить приложение", появится окошко, в котором нас интересует поле API Key. Скопируйте его куда-нибудь в надёжное место, скоро этот ключ нам понадобится.

{{% img "l6_2.jpg" %}}API Key{{% /img %}}

## Интегрируем аналитику в нашего бота

Теперь осталось начать сбор статистики по нашему боту! В качестве примера создадим ещё одного, в котором будет 2 команды: **/random** и **/yesorno**. Первая будет выводить случайное число от 1 до 10, а вторая - "да" или "нет".
Итак, для начала займемся файлом `config.py`. Помимо токена самого бота, создадим переменную `botan_key`, в которую запишем полученный ранее API Key.
Хорошо, с конфигом закончим. Создадим файл botan.py. Кто-то скажет, мол, можно же взять готовый из [соответствующего репозитория](https://github.com/botanio/sdk). Так-то да, но при работе с [pyTelegramBotAPI](https://github.com/eternnoir/pyTelegramBotAPI), мне пришлось внести в него некоторые дополнения. Сам файл довольно большой, чтобы приводить его полностью здесь, поэтому отсылаю вас к [своему репозиторию](https://github.com/MasterGroosha/telegram-tutorial/blob/master/lesson_06/botan.py), где его можно просмотреть.  
Функция `make_json` получает на вход объект `Message` и выбирает из него необходимые поля. В принципе, можно собирать статистику по чему угодно, т.е. можно смотреть, пользователь с каким ID или ником сколько раз вызвал ту или иную команду, в каком чате чаще используют бота и т.д.  
В данном примере я оставлю сбор различных параметров, но в коде будут указания, что делать, если нужно просто количество использований той или иной команды.

Создадим две функции:

{{< highlight none >}}
@bot.message_handler(commands=['random'])
def cmd_random(message):
    bot.send_message(message.chat.id, random.randint(1, 10))
    # Если не нужно собирать ничего, кроме количества использований, замените третий аргумент message на None
    botan.track(config.botan_key, message.chat.id, message, 'Случайное число')
    return


@bot.message_handler(commands=['yesorno'])
def cmd_yesorno(message):
    bot.send_message(message.chat.id, random.choice(strings))
    # Если не нужно собирать ничего, кроме количества использований, замените третий аргумент message на None
    botan.track(config.botan_key, message.chat.id, message, 'Да или нет')
    return
{{< /highlight >}}

Не забудьте импортировать стандартный модуль `random` и написать после него `random.seed()` для инициализации генератора!  
Также обратите внимание на строчку `botan.track(...)`. Именно эта строка и отвечает за отправку статистики. Первый аргумент - ключ, полученный при регистрации бота в Ботане, второй аргумент - chat.id, получаемый в сообщении. Третий аргумент - объект сообщения, из которого вытягивается необходимая информация, а последний - что-то типа метки, описывающая команду.  
В первой строке функции `cmd_yesorno` есть вызов `random.choice(strings)`, это выбор случайного элемента из массива, в нашем случае содержащего всего 2 элемента: строку "да" и строку "нет".

Собственно, всё. Дописываем нужные строки кода, как и раньше и запускаем бота! Несколько раз используем команды **/random** и **/yesorno**, подождем пару минут и заглянем в статистику, выбрав бота в Метрике, пункт "Приложение" - "События":

{{% img "l6_3.jpg" %}}Просмотр статистики{{% /img %}}

Как видите, всё прекрасно работает!

{{< btn_left relref="/docs/lesson_05" >}}Урок №5{{< /btn_left >}}
{{< btn_right relref="/docs/lesson_07" >}}Урок №7{{< /btn_right >}}