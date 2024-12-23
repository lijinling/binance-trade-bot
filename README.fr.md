# Bot de Trading Binance
> Bot de trading automatisé de cryptomonnaies

[English](README.md) | [简体中文](README.zh-CN.md) | [Español](README.es.md) | [العربية](README.ar.md) | [हिन्दी](README.hi.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [বাংলা](README.bn.md) | [Français](README.fr.md) | [Indonesia](README.id.md)

![github](https://img.shields.io/github/workflow/status/edeng23/binance-trade-bot/binance-trade-bot)
![docker](https://img.shields.io/docker/pulls/edeng23/binance-trade-bot)
[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/edeng23/binance-trade-bot)

[![Deploy to DO](https://mp-assets1.sfo2.digitaloceanspaces.com/deploy-to-do/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/coinbookbrasil/binance-trade-bot/tree/master&refcode=a076ff7a9a6a)

## Suivez-moi sur Twitter :)

[![Twitter](https://img.shields.io/twitter/url/https/twitter.com/cloudposse.svg?style=social&label=Follow%20%400xedeng)](https://twitter.com/0xedeng)

## Pourquoi ?

Ce projet a été inspiré par l'observation que toutes les cryptomonnaies se comportent pratiquement de la même manière. Quand l'une monte, elles montent toutes, et quand l'une plonge, elles plongent toutes. _Pratiquement_. De plus, toutes les monnaies suivent le leadership du Bitcoin ; la différence réside dans leur déphasage.

Donc, si les monnaies oscillent fondamentalement les unes par rapport aux autres, il semble judicieux d'échanger la monnaie qui monte contre celle qui baisse, puis de revenir quand le rapport s'inverse.

## Comment ?

Le trading se fait sur la plateforme de marché Binance qui, bien sûr, n'a pas de marchés pour toutes les paires d'altcoins. La solution consiste à utiliser une monnaie de pont qui complétera les paires manquantes. La monnaie de pont par défaut est le Tether (USDT), qui est stable par conception et compatible avec presque toutes les monnaies de la plateforme.

<p align="center">
  Monnaie A → USDT → Monnaie B
</p>

La façon dont le bot tire parti du comportement observé est de toujours passer de la monnaie "forte" à la monnaie "faible", en supposant qu'à un moment donné la situation s'inversera. Il reviendra ensuite à la monnaie d'origine, en détenant finalement plus qu'initialement. Cela se fait en tenant compte des frais de trading.

<div align="center">
  <p><b>Monnaie A</b> → USDT → Monnaie B</p>
  <p>Monnaie B → USDT → Monnaie C</p>
  <p>...</p>
  <p>Monnaie C → USDT → <b>Monnaie A</b></p>
</div>

Le bot saute entre un ensemble configuré de monnaies à condition de ne pas revenir à une monnaie sauf si c'est rentable par rapport au montant détenu précédemment. Cela signifie que nous ne finirons jamais par avoir moins d'une certaine monnaie. Le risque est qu'une des monnaies puisse chuter brutalement par rapport aux autres soudainement, attirant notre algorithme glouton inverse.

## Configuration de Binance

- Créez un [compte Binance](https://www.binance.com/en/register?ref=129471815) (Inclut mon lien de parrainage, je vous serais très reconnaissant si vous l'utilisez).
- Activez l'authentification à deux facteurs.
- Créez une nouvelle clé API.
- Obtenez une cryptomonnaie. Si son symbole n'est pas dans la liste par défaut, ajoutez-le.

## Configuration de l'Outil

### Installer les dépendances Python

Exécutez la ligne suivante dans le terminal : `pip install -r requirements.txt`.

### Créer la configuration utilisateur

Créez un fichier .cfg nommé `user.cfg` basé sur `.user.cfg.example`, puis ajoutez vos clés API et votre monnaie actuelle.

**Le fichier de configuration comprend les champs suivants :**

- **api_key** - Clé API Binance générée lors de la configuration du compte Binance.
- **api_secret_key** - Clé secrète Binance générée lors de la configuration du compte Binance.
- **current_coin** - C'est votre monnaie de départ. Elle doit être l'une des monnaies de votre liste de monnaies supportées. Si vous voulez partir de votre monnaie de pont, laissez ce champ vide - le bot sélectionnera une monnaie aléatoire dans votre liste de monnaies supportées et l'achètera.
- **bridge** - Votre monnaie de pont. Notez que différents ponts permettront différents ensembles de monnaies supportées. Par exemple, il peut y avoir une paire monnaie-particulière/USDT mais pas de paire monnaie-particulière/BUSD.
- **tld** - 'com' ou 'us', selon votre région. Par défaut 'com'.
- **hourToKeepScoutHistory** - Contrôle combien d'heures de valeurs de scout sont conservées dans la base de données. Après le temps spécifié, l'information sera supprimée.
- **scout_sleep_time** - Contrôle combien de secondes sont attendues entre chaque scout.
- **use_margin** - 'yes' pour utiliser scout_margin. 'no' pour utiliser scout_multiplier.
- **scout_multiplier** - Contrôle la valeur par laquelle la différence entre l'état actuel des ratios de monnaies et l'état précédent des ratios est multipliée. Pour des valeurs plus grandes, le bot attendra des marges plus importantes avant de faire un trade.
- **scout_margin** - Pourcentage minimum de gain de monnaie par trade. 0.8 se traduit par un multiplicateur de scout de 5 avec des frais de 0.1%.
- **strategy** - La stratégie de trading à utiliser. Voir [`binance_trade_bot/strategies`](binance_trade_bot/strategies/README.md) pour plus d'informations.
- **buy_timeout/sell_timeout** - Contrôle combien de minutes attendre avant d'annuler un ordre limite (achat/vente) et de revenir en mode "scout". 0 signifie que l'ordre ne sera jamais annulé prématurément.
- **scout_sleep_time** - Contrôle combien de secondes le bot doit attendre entre les analyses des prix actuels. Comme le bot fonctionne maintenant sur websockets, cette valeur devrait être réglée assez bas (comme 1), les raisons de la régler au-dessus de 1 sont quand vous observez une utilisation élevée du CPU par le bot ou que vous recevez des erreurs d'API sur la limite de poids des requêtes.

#### Variables d'Environnement

Toutes les options fournies dans `user.cfg` peuvent également être configurées en utilisant des variables d'environnement.

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

### Payer les Frais avec BNB
Vous pouvez [utiliser BNB pour payer tous les frais sur la plateforme Binance](https://www.binance.com/en/support/faq/115000583311-Using-BNB-to-Pay-for-Fees), ce qui réduira tous les frais de 25%. Pour profiter de cet avantage, le bot effectuera toujours les opérations suivantes :
- Détecter automatiquement si vous avez activé le paiement des frais en BNB.
- S'assurer que vous avez suffisamment de BNB dans votre compte pour payer les frais de la transaction inspectée.
- Prendre en compte la réduction lors du calcul du seuil de trading.

### Notifications avec Apprise

Apprise permet au bot d'envoyer des notifications à tous les services de notification les plus populaires disponibles tels que : Telegram, Discord, Slack, Amazon SNS, Gotify, etc.

Pour configurer cela, vous devez créer un fichier apprise.yml dans le répertoire config.

Il existe une version exemple de ce fichier pour vous aider à démarrer.

Si vous êtes intéressé par l'exécution d'un bot Telegram, plus d'informations peuvent être trouvées dans la [documentation officielle de Telegram](https://core.telegram.org/bots).

### Exécuter

```shell
python -m binance_trade_bot
```

### Docker

L'image officielle est disponible [ici](https://hub.docker.com/r/edeng23/binance-trade-bot) et sera mise à jour à chaque nouveau changement.

```shell
docker-compose up
```

Si vous voulez seulement démarrer le navigateur SQLite

```shell
docker-compose up -d sqlitebrowser
```

## Backtesting

Vous pouvez tester le bot sur des données historiques pour voir comment il se comporte.

```shell
python backtest.py
```

N'hésitez pas à modifier ce fichier pour tester et comparer différents paramètres et périodes.

## Développement

Pour vous assurer que votre code est correctement formaté avant de faire une pull request,
n'oubliez pas d'installer [pre-commit](https://pre-commit.com/) :

```shell
pip install pre-commit
pre-commit install
```

L'algorithme de scout ne sera probablement pas modifié. Si vous souhaitez contribuer avec une méthode alternative, [ajoutez une nouvelle stratégie](binance_trade_bot/strategies/README.md).

## Projets Connexes

Grâce à un groupe de développeurs talentueux, il existe maintenant un [bot Telegram pour gérer ce projet à distance](https://github.com/lorcalhost/BTB-manager-telegram).

## Soutenez le Projet

<a href="https://www.buymeacoffee.com/edeng" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>

## Rejoignez le Chat

- **Discord** : [Lien d'Invitation](https://discord.gg/m4TNaxreCN)

## FAQ

Une liste de réponses aux questions qui semblent être les plus fréquemment posées peut être trouvée sur notre serveur Discord, dans le canal correspondant.

<p align="center">
  <img src = "https://usercontent2.hubstatic.com/6061829.jpg">
</p>

## Avertissement

Ce projet est uniquement à titre informatif. Vous ne devez pas interpréter toute
information ou autre matériel comme un conseil juridique, fiscal, d'investissement, financier ou
autre. Rien de ce qui est contenu ici ne constitue une sollicitation, recommandation,
endossement ou offre de ma part ou de tout fournisseur de services tiers d'acheter ou de vendre
des titres ou autres instruments financiers dans cette juridiction ou toute autre
juridiction dans laquelle une telle sollicitation ou offre serait illégale en vertu des
lois sur les valeurs mobilières de cette juridiction.

Si vous prévoyez d'utiliser de l'argent réel, UTILISEZ-LE À VOS PROPRES RISQUES.

En aucun cas je ne serai tenu responsable de toute
réclamation, dommage, perte, dépense, coût ou responsabilité de quelque nature que ce soit, y compris,
sans limitation, tout dommage direct ou indirect pour perte de profits.