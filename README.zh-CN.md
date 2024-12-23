# 币安交易机器人
> 自动化加密货币交易机器人

[English](README.md) | [简体中文](README.zh-CN.md) | [Español](README.es.md) | [العربية](README.ar.md) | [हिन्दी](README.hi.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [বাংলা](README.bn.md) | [Français](README.fr.md) | [Indonesia](README.id.md) | [Türkçe](README.tr.md) | [اردو](README.ur.md) | [Українська](README.uk.md)

![github](https://img.shields.io/github/workflow/status/edeng23/binance-trade-bot/binance-trade-bot)
![docker](https://img.shields.io/docker/pulls/edeng23/binance-trade-bot)
[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/edeng23/binance-trade-bot)

[![Deploy to DO](https://mp-assets1.sfo2.digitaloceanspaces.com/deploy-to-do/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/coinbookbrasil/binance-trade-bot/tree/master&refcode=a076ff7a9a6a)

## 在Twitter上关注我 :)

[![Twitter](https://img.shields.io/twitter/url/https/twitter.com/cloudposse.svg?style=social&label=Follow%20%400xedeng)](https://twitter.com/0xedeng)

## 为什么要开发这个机器人？

这个项目的灵感来自于这样一个观察:所有加密货币的行为基本上都是相似的。当一个上涨时,它们都会上涨,当一个下跌时,它们都会下跌。而且,所有的币种都追随比特币的领导;区别只在于它们的相位偏移。

因此,如果币种基本上是相对于彼此振荡的,那么用上涨的币换取下跌的币,然后在比率反转时再换回来似乎是明智的。

## 工作原理

交易在币安市场平台上进行,当然,并不是每个山寨币对都有市场。解决这个问题的方法是使用桥接货币来补充缺失的交易对。默认的桥接货币是泰达币(USDT),它在设计上是稳定的,并且与平台上几乎每个币种都兼容。

<p align="center">
  币种 A → USDT → 币种 B
</p>

机器人利用观察到的行为模式,总是从"强势"币种转换到"弱势"币种,假设在某个时刻形势会逆转。然后它会返回到原始币种,最终持有比原来更多的数量。这个过程会考虑到交易手续费。

<div align="center">
  <p><b>币种 A</b> → USDT → 币种 B</p>
  <p>币种 B → USDT → 币种 C</p>
  <p>...</p>
  <p>币种 C → USDT → <b>币种 A</b></p>
</div>

机器人在一组配置好的币种之间跳转交易,条件是只有在相对于上次持有量有盈利时才会返回到某个币种。这意味着我们永远不会最终持有更少的某个币种。风险在于其中一个币种可能会相对于其他币种突然暴跌,这会触发我们的反向贪婪算法。

## 币安设置

- 创建一个[币安账户](https://www.binance.com/en/register?ref=129471815)(包含我的推荐链接,如果您使用它我将非常感激)。
- 启用双重认证。
- 创建一个新的 API 密钥。
- 获取一个加密货币。如果它的符号不在默认列表中,请添加它。

## 工具设置

### 安装 Python 依赖

在终端中运行以下命令:`pip install -r requirements.txt`。

### 创建用户配置

基于`.user.cfg.example`创建一个名为`user.cfg`的配置文件,然后添加您的 API 密钥和当前币种。

**配置文件包含以下字段:**

- **api_key** - 在币安账户设置阶段生成的币安 API 密钥。
- **api_secret_key** - 在币安账户设置阶段生成的币安密钥。
- **current_coin** - 这是您选择的起始币种。应该是您支持的币种列表中的一个。如果您想从桥接货币开始,请将此字段留空 - 机器人将从您支持的币种列表中随机选择一个币种并购买它。
- **bridge** - 您选择的桥接货币。请注意,不同的桥接货币会允许不同的支持币种集合。例如,可能存在某个币种/USDT交易对,但没有该币种/BUSD交易对。
- **tld** - 根据您的地区选择'com'或'us'。默认为'com'。
- **hourToKeepScoutHistory** - 控制在数据库中保留侦察值的小时数。超过指定时间后,信息将被删除。
- **scout_sleep_time** - 控制每次侦察之间等待的秒数。
- **use_margin** - 'yes'使用scout_margin。'no'使用scout_multiplier。
- **scout_multiplier** - 控制当前币种比率状态与之前比率状态之差的乘数。对于较大的值,机器人将等待更大的利润空间才进行交易。
- **scout_margin** - 每笔交易的最小币种收益百分比。0.8相当于在0.1%手续费时的scout_multiplier为5。
- **strategy** - 使用的交易策略。查看 [`binance_trade_bot/strategies`](binance_trade_bot/strategies/README.md) 获取更多信息。
- **buy_timeout/sell_timeout** - 控制在取消限价订单(买入/卖出)并返回到"侦察"模式之前等待的分钟数。0表示订单永远不会被提前取消。
- **scout_sleep_time** - 控制机器人分析当前价格之间等待的秒数。由于机器人现在使用websockets运行,这个值应该设置得较低(比如1),只有在观察到机器人CPU使用率较高或收到API请求权重限制错误时才需要设置更高的值。

#### 环境变量

`user.cfg`中提供的所有选项也可以使用环境变量进行配置。

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

### 使用BNB支付手续费
您可以[使用BNB支付币安平台上的任何费用](https://www.binance.com/en/support/faq/115000583311-Using-BNB-to-Pay-for-Fees),这将减少所有费用25%。为了支持这个优势,机器人将始终执行以下操作:
- 自动检测您是否启用了BNB手续费支付。
- 确保您的账户中有足够的BNB来支付检查交易的手续费。
- 在计算交易阈值时考虑折扣。

### 使用Apprise通知

Apprise允许机器人向所有最流行的通知服务发送通知,如:Telegram、Discord、Slack、Amazon SNS、Gotify等。

要设置此功能,您需要在config目录中创建一个apprise.yml文件。

有一个示例版本的文件可以帮助您入门。

如果您对运行Telegram机器人感兴趣,可以在[Telegram官方文档](https://core.telegram.org/bots)中找到更多信息。

### 运行

```shell
python -m binance_trade_bot
```

### Docker

官方镜像可在[这里](https://hub.docker.com/r/edeng23/binance-trade-bot)获取,每次有新的更改时都会更新。

```shell
docker-compose up
```

如果您只想启动SQLite浏览器

```shell
docker-compose up -d sqlitebrowser
```

## 回测

您可以在历史数据上测试机器人的表现。

```shell
python backtest.py
```

您可以自由修改该文件以测试和比较不同的设置和时间段。

## 开发

为了确保您的代码在提出拉取请求之前格式正确,请记住安装[pre-commit](https://pre-commit.com/):

```shell
pip install pre-commit
pre-commit install
```

侦察算法不太可能被更改。如果您想贡献替代方法,[添加一个新策略](binance_trade_bot/strategies/README.md)。

## 相关项目

感谢一群才华横溢的开发者,现在有了一个[用于远程管理此项目的Telegram机器人](https://github.com/lorcalhost/BTB-manager-telegram)。

## 支持项目

<a href="https://www.buymeacoffee.com/edeng" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>

## 加入聊天

- **Discord**: [邀请链接](https://discord.gg/m4TNaxreCN)

## 常见问题

在我们的discord服务器的相应频道中可以找到对最常见问题的回答列表。

<p align="center">
  <img src = "https://usercontent2.hubstatic.com/6061829.jpg">
</p>

## 免责声明

本项目仅供参考。您不应将此类信息或其他材料理解为法律、税务、投资、财务或其他建议。本文中的任何内容均不构成我或任何第三方服务提供商对在本或任何其他司法管辖区购买或出售任何证券或其他金融工具的请求、建议、认可或要约,如果在该司法管辖区根据证券法进行此类招揽或要约将是非法的。

如果您计划使用真实资金,请自行承担风险。

在任何情况下,我都不会对任何索赔、损害、损失、费用、成本或责任负责,包括但不限于任何直接或间接的利润损失。

</rewritten_file> 