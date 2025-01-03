# Торговий бот Binance
> Автоматизований бот для торгівлі криптовалютою

[English](README.md) | [简体中文](README.zh-CN.md) | [Español](README.es.md) | [العربية](README.ar.md) | [हिन्दी](README.hi.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [বাংলা](README.bn.md) | [Français](README.fr.md) | [Indonesia](README.id.md) | [Türkçe](README.tr.md) | [اردو](README.ur.md) | [Українська](README.uk.md)

![github](https://img.shields.io/github/workflow/status/edeng23/binance-trade-bot/binance-trade-bot)
![docker](https://img.shields.io/docker/pulls/edeng23/binance-trade-bot)
[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/edeng23/binance-trade-bot)

[![Deploy to DO](https://mp-assets1.sfo2.digitaloceanspaces.com/deploy-to-do/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/coinbookbrasil/binance-trade-bot/tree/master&refcode=a076ff7a9a6a)

## Слідкуйте за мною у Twitter :)

[![Twitter](https://img.shields.io/twitter/url/https/twitter.com/cloudposse.svg?style=social&label=Follow%20%400xedeng)](https://twitter.com/0xedeng)

## Чому?

Цей проект був натхненний спостереженням, що всі криптовалюти поводяться практично однаково. Коли одна зростає, зростають усі, і коли одна падає, падають усі. _Практично_. Більше того, всі монети слідують за лідерством Bitcoin; різниця в їх фазовому зсуві.

Тому, якщо монети в основному коливаються відносно одна одної, здається розумним обмінювати зростаючу монету на падаючу, а потім обмінювати назад, коли співвідношення змінюється на протилежне.

## Як?

Торгівля здійснюється на платформі Binance, яка, звичайно, не має ринків для кожної пари альткоїнів. Рішенням є використання мостової валюти, яка доповнить відсутні пари. За замовчуванням мостовою валютою є Tether (USDT), який стабільний за дизайном і сумісний практично з усіма монетами на платформі.

<p align="center">
  Монета A → USDT → Монета B
</p>

Спосіб, яким бот використовує спостережувану поведінку, полягає в тому, щоб завжди переходити від "сильної" монети до "слабкої", припускаючи, що в якийсь момент ситуація зміниться. Потім він повернеться до вихідної монети, маючи в підсумку більше, ніж було спочатку. Це робиться з урахуванням торгових комісій.

<div align="center">
  <p><b>Монета A</b> → USDT → Монета B</p>
  <p>Монета B → USDT → Монета C</p>
  <p>...</p>
  <p>Монета C → USDT → <b>Монета A</b></p>
</div>

Бот перестрибує між налаштованим набором монет за умови, що він не повертається до монети, поки це не стане вигідним відносно останньої утримуваної кількості. Це означає, що ми ніколи не закінчимо з меншою кількістю певної монети. Ризик полягає в тому, що одна з монет може раптово обвалитися відносно інших, приваблюючи наш зворотний жадібний алгоритм.

## Налаштування Binance

- Створіть [обліковий запис Binance](https://www.binance.com/en/register?ref=129471815) (Включає моє реферальне посилання, буду дуже вдячний, якщо ви його використаєте).
- Увімкніть двофакторну автентифікацію.
- Створіть новий API ключ.
- Отримайте криптовалюту. Якщо її символ не входить до списку за замовчуванням, додайте його.

## Налаштування інструменту

### Встановлення залежностей Python

Виконайте наступний рядок у терміналі: `pip install -r requirements.txt`.

### Створення користувацької конфігурації

Створіть файл .cfg з назвою `user.cfg` на основі `.user.cfg.example`, потім додайте ваші API ключі та поточну монету.

**Файл конфігурації містить наступні поля:**

- **api_key** - API ключ Binance, згенерований на етапі налаштування облікового запису Binance.
- **api_secret_key** - Секретний ключ Binance, згенерований на етапі налаштування облікового запису Binance.
- **current_coin** - Це ваша початкова монета за вибором. Вона повинна бути однією з монет з вашого списку підтримуваних монет. Якщо ви хочете почати з вашої мостової валюти, залиште це поле порожнім - бот вибере випадкову монету з вашого списку підтримуваних монет і купить її.
- **bridge** - Ваша мостова валюта за вибором. Зверніть увагу, що різні мости дозволять використовувати різні набори підтримуваних монет. Наприклад, може існувати пара конкретна-монета/USDT, але не існувати пари конкретна-монета/BUSD.
- **tld** - 'com' або 'us', залежно від вашого регіону. За замовчуванням 'com'.
- **hourToKeepScoutHistory** - Контролює, скільки годин значень розвідки зберігається в базі даних. Після вказаного часу інформація буде видалена.
- **scout_sleep_time** - Контролює, скільки секунд очікування між кожною розвідкою.
- **use_margin** - 'yes' для використання scout_margin. 'no' для використання scout_multiplier.
- **scout_multiplier** - Контролює значення, на яке множиться різниця між поточним станом співвідношень монет і попереднім станом співвідношень. Для більших значень бот буде чекати більшої маржі перед здійсненням угоди.
- **scout_margin** - Мінімальний відсоток прибутку монети за угоду. 0.8 перекладається в множник розвідки 5 при комісії 0.1%.
- **strategy** - Використовувана торгова стратегія. Див. [`binance_trade_bot/strategies`](binance_trade_bot/strategies/README.md) для отримання додаткової інформації.
- **buy_timeout/sell_timeout** - Контролює, скільки хвилин чекати перед скасуванням лімітного ордера (купівля/продаж) і поверненням в режим "розвідки". 0 означає, що ордер ніколи не буде скасований передчасно.
- **scout_sleep_time** - Контролює, скільки секунд бот повинен чекати між аналізами поточних цін. Оскільки бот тепер працює на websockets, це значення має бути встановлене на щось низьке (наприклад, 1), причини встановити його вище 1 - коли ви спостерігаєте високе використання CPU ботом або отримуєте помилки API про ліміт ваги запитів.

#### Змінні середовища

Всі опції, надані в `user.cfg`, також можуть бути налаштовані за допомогою змінних середовища.

```
CURRENT_COIN_SYMBOL:
SUPPORTED_COIN_LIST: "XLM TRX ICX EOS IOTA ONT QTUM ETC ADA XMR DASH NEO ATOM DOGE VET BAT OMG BTT"
BRIDGE_SYMBOL: USDT
API_KEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
API_SECRET_KEY: NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j
SCOUT_MULTIPLIER: 5
SCOUT_SLEEP_TIME: 1
TLD: com
STRATEGY: default
BUY_TIMEOUT: 0
SELL_TIMEOUT: 0
```

### Оплата комісій за допомогою BNB
Ви можете [використовувати BNB для оплати будь-яких комісій на платформі Binance](https://www.binance.com/en/support/faq/115000583311-Using-BNB-to-Pay-for-Fees), що зменшить всі комісії на 25%. Для підтримки цієї переваги бот завжди буде виконувати наступні операції:
- Автоматично визначати, чи у вас увімкнена оплата комісій BNB.
- Переконатися, що у вас достатньо BNB на рахунку для оплати комісії перевіреної угоди.
- Враховувати знижку при розрахунку торгового порогу.

### Сповіщення з Apprise

Apprise дозволяє боту надсилати сповіщення у всі найпопулярніші доступні сервіси сповіщень, такі як: Telegram, Discord, Slack, Amazon SNS, Gotify тощо.

Для налаштування цього вам потрібно створити файл apprise.yml у директорії config.

Існує приклад версії цього файлу, щоб допомогти вам почати.

Якщо ви зацікавлені в запуску бота Telegram, додаткову інформацію можна знайти в [офіційній документації Telegram](https://core.telegram.org/bots).

### Запуск

```shell
python -m binance_trade_bot
```

### Docker

Офіційний образ доступний [тут](https://hub.docker.com/r/edeng23/binance-trade-bot) і буде оновлюватися з кожною новою зміною.

```shell
docker-compose up
```

Якщо ви хочете запустити тільки браузер SQLite

```shell
docker-compose up -d sqlitebrowser
```

## Бектестинг

Ви можете протестувати бота на історичних даних, щоб побачити, як він себе поводить.

```shell
python backtest.py
```

Не соромтеся модифікувати цей файл для тестування та порівняння різних налаштувань і періодів часу.

## Розробка

Щоб переконатися, що ваш код правильно відформатований перед створенням pull request,
не забудьте встановити [pre-commit](https://pre-commit.com/):

```shell
pip install pre-commit
pre-commit install
```

Алгоритм розвідки навряд чи буде змінений. Якщо ви хочете внести свій внесок альтернативним методом, [додайте нову стратегію](binance_trade_bot/strategies/README.md).

## Пов'язані проекти

Завдяки групі талановитих розробників, тепер є [бот Telegram для віддаленого управління цим проектом](https://github.com/lorcalhost/BTB-manager-telegram).

## Підтримайте проект

<a href="https://www.buymeacoffee.com/edeng" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>

## Приєднуйтесь до чату

- **Discord**: [Посилання на запрошення](https://discord.gg/m4TNaxreCN)

## FAQ

Список відповідей на питання, які здаються найбільш часто задаваними, можна знайти на нашому сервері Discord у відповідному каналі.

<p align="center">
  <img src = "https://usercontent2.hubstatic.com/6061829.jpg">
</p>

## Відмова від відповідальності

Цей проект призначений тільки для інформаційних цілей. Ви не повинні розглядати будь-яку
інформацію або інший матеріал як юридичну, податкову, інвестиційну, фінансову або
іншу консультацію. Ніщо з того, що міститься тут, не є пропозицією, рекомендацією,
схваленням або пропозицією від мене або будь-якого стороннього постачальника послуг купувати або продавати
будь-які цінні папери або інші фінансові інструменти в цій або будь-якій іншій
юрисдикції, де така пропозиція була б незаконною відповідно до
законів про цінні папери цієї юрисдикції.

Якщо ви плануєте використовувати реальні гроші, ВИКОРИСТОВУЙТЕ ЇХ НА СВІЙ РИЗИК.

За жодних обставин я не буду нести відповідальність за будь-які
претензії, збитки, шкоду, витрати, витрати або зобов'язання будь-якого роду, включаючи,
без обмежень, будь-які прямі або непрямі збитки за втрату прибутку. 