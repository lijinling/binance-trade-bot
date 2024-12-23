# Bot de Trading Binance
> Bot automatizado de trading de criptomoedas

[English](README.md) | [简体中文](README.zh-CN.md) | [Español](README.es.md) | [العربية](README.ar.md) | [हिन्दी](README.hi.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [বাংলা](README.bn.md) | [Français](README.fr.md) | [Indonesia](README.id.md)

![github](https://img.shields.io/github/workflow/status/edeng23/binance-trade-bot/binance-trade-bot)
![docker](https://img.shields.io/docker/pulls/edeng23/binance-trade-bot)
[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/edeng23/binance-trade-bot)

[![Deploy to DO](https://mp-assets1.sfo2.digitaloceanspaces.com/deploy-to-do/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/coinbookbrasil/binance-trade-bot/tree/master&refcode=a076ff7a9a6a)

## Siga-me no Twitter :)

[![Twitter](https://img.shields.io/twitter/url/https/twitter.com/cloudposse.svg?style=social&label=Follow%20%400xedeng)](https://twitter.com/0xedeng)

## Por quê?

Este projeto foi inspirado pela observação de que todas as criptomoedas se comportam praticamente da mesma maneira. Quando uma sobe, todas sobem, e quando uma cai, todas caem. _Praticamente_. Além disso, todas as moedas seguem a liderança do Bitcoin; a diferença está no seu deslocamento de fase.

Então, se as moedas estão basicamente oscilando em relação umas às outras, parece inteligente trocar a moeda em alta pela moeda em baixa, e depois trocar de volta quando a proporção se inverter.

## Como?

O trading é feito na plataforma Binance, que, é claro, não tem mercados para todos os pares de altcoins. A solução para isso é usar uma moeda ponte que complementará os pares ausentes. A moeda ponte padrão é o Tether (USDT), que é estável por design e compatível com praticamente todas as moedas na plataforma.

<p align="center">
  Moeda A → USDT → Moeda B
</p>

A maneira como o bot aproveita o comportamento observado é sempre fazer o downgrade da moeda "forte" para a moeda "fraca", assumindo que em algum momento as mesas vão virar. Ele então retornará à moeda original, tendo finalmente mais do que tinha originalmente. Isso é feito levando em consideração as taxas de trading.

<div align="center">
  <p><b>Moeda A</b> → USDT → Moeda B</p>
  <p>Moeda B → USDT → Moeda C</p>
  <p>...</p>
  <p>Moeda C → USDT → <b>Moeda A</b></p>
</div>

O bot pula entre um conjunto configurado de moedas com a condição de que não retorna a uma moeda a menos que seja lucrativo em relação à quantidade mantida por último. Isso significa que nunca terminaremos tendo menos de uma determinada moeda. O risco é que uma das moedas pode entrar em queda livre em relação às outras de repente, atraindo nosso algoritmo ganancioso reverso.

## Configuração da Binance

- Crie uma [conta Binance](https://www.binance.com/en/register?ref=129471815) (Inclui meu link de referência, ficarei muito grato se você usá-lo).
- Ative a autenticação de dois fatores.
- Crie uma nova chave API.
- Obtenha uma criptomoeda. Se seu símbolo não estiver na lista padrão, adicione-o.

## Configuração da Ferramenta

### Instalar dependências Python

Execute a seguinte linha no terminal: `pip install -r requirements.txt`.

### Criar configuração do usuário

Crie um arquivo .cfg chamado `user.cfg` baseado em `.user.cfg.example`, depois adicione suas chaves API e moeda atual.

**O arquivo de configuração consiste nos seguintes campos:**

- **api_key** - Chave API Binance gerada na etapa de configuração da conta Binance.
- **api_secret_key** - Chave secreta Binance gerada na etapa de configuração da conta Binance.
- **current_coin** - Esta é sua moeda inicial de escolha. Deve ser uma das moedas da sua lista de moedas suportadas. Se você quiser começar da sua moeda ponte, deixe este campo vazio - o bot selecionará uma moeda aleatória da sua lista de moedas suportadas e a comprará.
- **bridge** - Sua moeda ponte de escolha. Note que diferentes pontes permitirão diferentes conjuntos de moedas suportadas. Por exemplo, pode haver um par moeda-específica/USDT mas nenhum par moeda-específica/BUSD.
- **tld** - 'com' ou 'us', dependendo da sua região. O padrão é 'com'.
- **hourToKeepScoutHistory** - Controla quantas horas de valores de scout são mantidas no banco de dados. Após o tempo especificado, a informação será excluída.
- **scout_sleep_time** - Controla quantos segundos são esperados entre cada scout.
- **use_margin** - 'yes' para usar scout_margin. 'no' para usar scout_multiplier.
- **scout_multiplier** - Controla o valor pelo qual a diferença entre o estado atual das proporções de moedas e o estado anterior das proporções é multiplicada. Para valores maiores, o bot esperará margens maiores antes de fazer um trade.
- **scout_margin** - Porcentagem mínima de ganho de moeda por trade. 0.8 se traduz em um multiplicador de scout de 5 com taxa de 0.1%.
- **strategy** - A estratégia de trading a ser usada. Veja [`binance_trade_bot/strategies`](binance_trade_bot/strategies/README.md) para mais informações.
- **buy_timeout/sell_timeout** - Controla quantos minutos esperar antes de cancelar uma ordem limite (compra/venda) e retornar ao modo "scout". 0 significa que a ordem nunca será cancelada prematuramente.
- **scout_sleep_time** - Controla quantos segundos o bot deve esperar entre análises de preços atuais. Como o bot agora opera em websockets, este valor deve ser definido como algo baixo (como 1), as razões para defini-lo acima de 1 são quando você observa alto uso de CPU pelo bot ou recebe erros de API sobre limite de peso de requisições.

#### Variáveis de Ambiente

Todas as opções fornecidas em `user.cfg` também podem ser configuradas usando variáveis de ambiente.

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

### Pagando Taxas com BNB
Você pode [usar BNB para pagar quaisquer taxas na plataforma Binance](https://www.binance.com/en/support/faq/115000583311-Using-BNB-to-Pay-for-Fees), o que reduzirá todas as taxas em 25%. Para suportar este benefício, o bot sempre executará as seguintes operações:
- Detectar automaticamente se você tem o pagamento de taxa BNB habilitado.
- Garantir que você tenha BNB suficiente em sua conta para pagar a taxa da negociação inspecionada.
- Levar em consideração o desconto ao calcular o limite de trading.

### Notificações com Apprise

Apprise permite que o bot envie notificações para todos os serviços de notificação mais populares disponíveis como: Telegram, Discord, Slack, Amazon SNS, Gotify, etc.

Para configurar isso, você precisa criar um arquivo apprise.yml no diretório config.

Existe uma versão de exemplo deste arquivo para ajudá-lo a começar.

Se você estiver interessado em executar um bot Telegram, mais informações podem ser encontradas na [documentação oficial do Telegram](https://core.telegram.org/bots).

### Executar

```shell
python -m binance_trade_bot
```

### Docker

A imagem oficial está disponível [aqui](https://hub.docker.com/r/edeng23/binance-trade-bot) e será atualizada com cada nova alteração.

```shell
docker-compose up
```

Se você quiser iniciar apenas o navegador SQLite

```shell
docker-compose up -d sqlitebrowser
```

## Backtesting

Você pode testar o bot em dados históricos para ver como ele se comporta.

```shell
python backtest.py
```

Sinta-se à vontade para modificar esse arquivo para testar e comparar diferentes configurações e períodos de tempo.

## Desenvolvimento

Para garantir que seu código esteja formatado corretamente antes de fazer um pull request,
lembre-se de instalar [pre-commit](https://pre-commit.com/):

```shell
pip install pre-commit
pre-commit install
```

O algoritmo de scout provavelmente não será alterado. Se você quiser contribuir com um método alternativo, [adicione uma nova estratégia](binance_trade_bot/strategies/README.md).

## Projetos Relacionados

Graças a um grupo de desenvolvedores talentosos, agora há um [bot Telegram para gerenciar este projeto remotamente](https://github.com/lorcalhost/BTB-manager-telegram).

## Apoie o Projeto

<a href="https://www.buymeacoffee.com/edeng" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>

## Junte-se ao Chat

- **Discord**: [Link de Convite](https://discord.gg/m4TNaxreCN)

## FAQ

Uma lista de respostas para o que parecem ser as perguntas mais frequentemente feitas pode ser encontrada em nosso servidor Discord, no canal correspondente.

<p align="center">
  <img src = "https://usercontent2.hubstatic.com/6061829.jpg">
</p>

## Aviso Legal

Este projeto é apenas para fins informativos. Você não deve interpretar qualquer
informação ou outro material como aconselhamento jurídico, tributário, de investimento, financeiro ou
outro. Nada contido aqui constitui uma solicitação, recomendação,
endosso ou oferta minha ou de qualquer prestador de serviços terceirizado para comprar ou vender
quaisquer valores mobiliários ou outros instrumentos financeiros nesta ou em qualquer outra
jurisdição onde tal solicitação ou oferta seria ilegal sob as
leis de valores mobiliários dessa jurisdição.

Se você planeja usar dinheiro real, USE POR SUA PRÓPRIA CONTA E RISCO.

Em nenhuma circunstância serei responsável por quaisquer
reclamações, danos, perdas, despesas, custos ou responsabilidades de qualquer tipo, incluindo,
sem limitação, quaisquer danos diretos ou indiretos por perda de lucros. 