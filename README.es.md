# Bot de Trading de Binance
> Bot automatizado de trading de criptomonedas

[English](README.md) | [简体中文](README.zh-CN.md) | [Español](README.es.md) | [العربية](README.ar.md) | [हिन्दी](README.hi.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [Deutsch](README.de.md) | [Français](README.fr.md) | [Indonesia](README.id.md)

![github](https://img.shields.io/github/workflow/status/edeng23/binance-trade-bot/binance-trade-bot)
![docker](https://img.shields.io/docker/pulls/edeng23/binance-trade-bot)
[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/edeng23/binance-trade-bot)

[![Deploy to DO](https://mp-assets1.sfo2.digitaloceanspaces.com/deploy-to-do/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/coinbookbrasil/binance-trade-bot/tree/master&refcode=a076ff7a9a6a)

## ¡Sígueme en Twitter! :)

[![Twitter](https://img.shields.io/twitter/url/https/twitter.com/cloudposse.svg?style=social&label=Follow%20%400xedeng)](https://twitter.com/0xedeng)

## ¿Por qué?

Este proyecto fue inspirado por la observación de que todas las criptomonedas se comportan prácticamente de la misma manera. Cuando una sube, todas suben, y cuando una cae, todas caen. _Prácticamente_. Además, todas las monedas siguen el liderazgo de Bitcoin; la diferencia está en su desfase.

Entonces, si las monedas básicamente oscilan una respecto a la otra, parece inteligente intercambiar la moneda que está subiendo por la que está bajando, y luego volver a intercambiar cuando la relación se invierte.

## ¿Cómo?

El trading se realiza en la plataforma de mercado de Binance, que por supuesto, no tiene mercados para todos los pares de altcoins. La solución para esto es usar una moneda puente que complementará los pares faltantes. La moneda puente predeterminada es Tether (USDT), que es estable por diseño y compatible con casi todas las monedas en la plataforma.

<p align="center">
  Moneda A → USDT → Moneda B
</p>

La forma en que el bot aprovecha el comportamiento observado es siempre degradar de la moneda "fuerte" a la moneda "débil", bajo la suposición de que en algún momento las tornas cambiarán. Luego volverá a la moneda original, teniendo finalmente más de lo que tenía originalmente. Esto se hace teniendo en cuenta las comisiones de trading.

<div align="center">
  <p><b>Moneda A</b> → USDT → Moneda B</p>
  <p>Moneda B → USDT → Moneda C</p>
  <p>...</p>
  <p>Moneda C → USDT → <b>Moneda A</b></p>
</div>

El bot salta entre un conjunto configurado de monedas con la condición de que no vuelve a una moneda a menos que sea rentable con respecto a la cantidad mantenida por última vez. Esto significa que nunca terminaremos teniendo menos de una determinada moneda. El riesgo es que una de las monedas puede caer en picado en relación con las otras de repente, atrayendo nuestro algoritmo inverso codicioso.

## Configuración de Binance

- Crea una [cuenta de Binance](https://www.binance.com/en/register?ref=129471815) (Incluye mi enlace de referido, estaré muy agradecido si lo usas).
- Habilita la autenticación de dos factores.
- Crea una nueva clave API.
- Obtén una criptomoneda. Si su símbolo no está en la lista predeterminada, agrégalo.

## Configuración de la Herramienta

### Instalar dependencias de Python

Ejecuta la siguiente línea en la terminal: `pip install -r requirements.txt`.

### Crear configuración de usuario

Crea un archivo .cfg llamado `user.cfg` basado en `.user.cfg.example`, luego agrega tus claves API y moneda actual.

**El archivo de configuración consta de los siguientes campos:**

- **api_key** - Clave API de Binance generada en la etapa de configuración de la cuenta de Binance.
- **api_secret_key** - Clave secreta de Binance generada en la etapa de configuración de la cuenta de Binance.
- **current_coin** - Esta es tu moneda inicial de elección. Debe ser una de las monedas de tu lista de monedas soportadas. Si quieres comenzar desde tu moneda puente, deja este campo vacío - el bot seleccionará una moneda aleatoria de tu lista de monedas soportadas y la comprará.
- **bridge** - Tu moneda puente de elección. Ten en cuenta que diferentes puentes permitirán diferentes conjuntos de monedas soportadas. Por ejemplo, puede haber un par moneda-particular/USDT pero no un par moneda-particular/BUSD.
- **tld** - 'com' o 'us', dependiendo de tu región. Por defecto es 'com'.
- **hourToKeepScoutHistory** - Controla cuántas horas de valores de exploración se mantienen en la base de datos. Después de que haya pasado el tiempo especificado, la información será eliminada.
- **scout_sleep_time** - Controla cuántos segundos se espera entre cada exploración.
- **use_margin** - 'yes' para usar scout_margin. 'no' para usar scout_multiplier.
- **scout_multiplier** - Controla el valor por el cual se multiplica la diferencia entre el estado actual de las relaciones de monedas y el estado anterior de las relaciones. Para valores más grandes, el bot esperará márgenes más grandes antes de hacer un trade.
- **scout_margin** - Porcentaje mínimo de ganancia de moneda por trade. 0.8 se traduce en un multiplicador de exploración de 5 con una comisión del 0.1%.
- **strategy** - La estrategia de trading a usar. Ver [`binance_trade_bot/strategies`](binance_trade_bot/strategies/README.md) para más información.
- **buy_timeout/sell_timeout** - Controla cuántos minutos esperar antes de cancelar una orden límite (compra/venta) y volver al modo "exploración". 0 significa que la orden nunca se cancelará prematuramente.
- **scout_sleep_time** - Controla cuántos segundos debe esperar el bot entre análisis de precios actuales. Como el bot ahora opera con websockets, este valor debe establecerse en algo bajo (como 1), las razones para establecerlo por encima de 1 son cuando observas un alto uso de CPU por parte del bot o recibes errores de API sobre el límite de peso de las solicitudes.

#### Variables de Entorno

Todas las opciones proporcionadas en `user.cfg` también se pueden configurar usando variables de entorno.

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

### Pagar Comisiones con BNB
Puedes [usar BNB para pagar cualquier comisión en la plataforma Binance](https://www.binance.com/en/support/faq/115000583311-Using-BNB-to-Pay-for-Fees), lo que reducirá todas las comisiones en un 25%. Para aprovechar este beneficio, el bot siempre realizará las siguientes operaciones:
- Detectar automáticamente si tienes habilitado el pago de comisiones con BNB.
- Asegurarse de que tienes suficiente BNB en tu cuenta para pagar la comisión de la operación inspeccionada.
- Tener en cuenta el descuento al calcular el umbral de trading.

### Notificaciones con Apprise

Apprise permite que el bot envíe notificaciones a todos los servicios de notificación más populares disponibles como: Telegram, Discord, Slack, Amazon SNS, Gotify, etc.

Para configurar esto, necesitas crear un archivo apprise.yml en el directorio config.

Hay una versión de ejemplo de este archivo para ayudarte a empezar.

Si estás interesado en ejecutar un bot de Telegram, puedes encontrar más información en la [documentación oficial de Telegram](https://core.telegram.org/bots).

### Ejecutar

```shell
python -m binance_trade_bot
```

### Docker

La imagen oficial está disponible [aquí](https://hub.docker.com/r/edeng23/binance-trade-bot) y se actualizará con cada nuevo cambio.

```shell
docker-compose up
```

Si solo quieres iniciar el navegador SQLite

```shell
docker-compose up -d sqlitebrowser
```

## Backtesting

Puedes probar el bot con datos históricos para ver cómo se comporta.

```shell
python backtest.py
```

Siéntete libre de modificar ese archivo para probar y comparar diferentes configuraciones y períodos de tiempo.

## Desarrollo

Para asegurarte de que tu código esté formateado correctamente antes de hacer una solicitud de extracción,
recuerda instalar [pre-commit](https://pre-commit.com/):

```shell
pip install pre-commit
pre-commit install
```

Es poco probable que se cambie el algoritmo de exploración. Si deseas contribuir con un método alternativo, [agrega una nueva estrategia](binance_trade_bot/strategies/README.md).

## Proyectos Relacionados

Gracias a un grupo de desarrolladores talentosos, ahora hay un [bot de Telegram para administrar este proyecto de forma remota](https://github.com/lorcalhost/BTB-manager-telegram).

## Apoya el Proyecto

<a href="https://www.buymeacoffee.com/edeng" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>

## Únete al Chat

- **Discord**: [Enlace de Invitación](https://discord.gg/m4TNaxreCN)

## Preguntas Frecuentes

Puedes encontrar una lista de respuestas a las preguntas más frecuentes en nuestro servidor de Discord, en el canal correspondiente.

<p align="center">
  <img src = "https://usercontent2.hubstatic.com/6061829.jpg">
</p>

## Descargo de Responsabilidad

Este proyecto es solo para fines informativos. No debes interpretar ninguna
información u otro material como asesoramiento legal, fiscal, de inversión, financiero u
otro. Nada de lo contenido aquí constituye una solicitud, recomendación,
respaldo u oferta mía o de cualquier proveedor de servicios tercero para comprar o vender
cualquier valor u otro instrumento financiero en esta o en cualquier otra
jurisdicción en la que dicha solicitud u oferta sería ilegal según las
leyes de valores de dicha jurisdicción.

Si planeas usar dinero real, ÚSALO BAJO TU PROPIO RIESGO.

Bajo ninguna circunstancia seré responsable de ningún
reclamo, daño, pérdida, gasto, costo o responsabilidad de ningún tipo, incluidos,
sin limitación, cualquier daño directo o indirecto por pérdida de beneficios.