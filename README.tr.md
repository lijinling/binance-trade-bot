# Binance Trading Bot
> Otomatik kripto para ticaret botu

[English](README.md) | [简体中文](README.zh-CN.md) | [Español](README.es.md) | [العربية](README.ar.md) | [हिन्दी](README.hi.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [বাংলা](README.bn.md) | [Français](README.fr.md) | [Indonesia](README.id.md) | [Türkçe](README.tr.md) | [اردو](README.ur.md) | [Українська](README.uk.md)

![github](https://img.shields.io/github/workflow/status/edeng23/binance-trade-bot/binance-trade-bot)
![docker](https://img.shields.io/docker/pulls/edeng23/binance-trade-bot)
[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/edeng23/binance-trade-bot)

[![Deploy to DO](https://mp-assets1.sfo2.digitaloceanspaces.com/deploy-to-do/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/coinbookbrasil/binance-trade-bot/tree/master&refcode=a076ff7a9a6a)

## Twitter'da beni takip edin :)

[![Twitter](https://img.shields.io/twitter/url/https/twitter.com/cloudposse.svg?style=social&label=Follow%20%400xedeng)](https://twitter.com/0xedeng)

## Neden?

Bu proje, tüm kripto paraların neredeyse aynı şekilde davrandığı gözleminden ilham aldı. Biri yükseldiğinde hepsi yükselir, biri düştüğünde hepsi düşer. _Neredeyse_. Dahası, tüm coinler Bitcoin'in liderliğini takip eder; fark, faz kaymalarındadır.

Dolayısıyla, coinler temelde birbirlerine göre salınım yapıyorsa, yükselen coini düşen coinle takas etmek ve oran tersine döndüğünde geri takas etmek akıllıca görünüyor.

## Nasıl?

İşlemler Binance piyasa platformunda yapılır, ki elbette her altcoin çifti için piyasalar yoktur. Bunun çözümü, eksik çiftleri tamamlayacak bir köprü para birimi kullanmaktır. Varsayılan köprü para birimi Tether (USDT)'dir, tasarım gereği stabildir ve platformdaki neredeyse her coinle uyumludur.

<p align="center">
  Coin A → USDT → Coin B
</p>

Botun gözlemlenen davranıştan yararlanma şekli, her zaman "güçlü" coinden "zayıf" coine geçiş yapmaktır, bir noktada durumun tersine döneceği varsayımıyla. Daha sonra orijinal coine geri dönecek, sonunda başlangıçta olduğundan daha fazlasına sahip olacaktır. Bu, işlem ücretleri göz önünde bulundurularak yapılır.

<div align="center">
  <p><b>Coin A</b> → USDT → Coin B</p>
  <p>Coin B → USDT → Coin C</p>
  <p>...</p>
  <p>Coin C → USDT → <b>Coin A</b></p>
</div>

Bot, son tutulan miktara göre kârlı olmadıkça bir coine geri dönmeme şartıyla yapılandırılmış bir coin seti arasında zıplar. Bu, asla belirli bir coinden daha azına sahip olmayacağımız anlamına gelir. Risk, coinlerden birinin diğerlerine göre aniden serbest düşüşe geçmesi ve tersine açgözlü algoritmamızı çekmesidir.

## Binance Kurulumu

- Bir [Binance hesabı](https://www.binance.com/en/register?ref=129471815) oluşturun (Referans linkimi içerir, kullanırsanız çok minnettar olurum).
- İki faktörlü kimlik doğrulamayı etkinleştirin.
- Yeni bir API anahtarı oluşturun.
- Bir kripto para alın. Sembolü varsayılan listede yoksa ekleyin.

## Araç Kurulumu

### Python bağımlılıklarını yükleyin

Terminalde şu satırı çalıştırın: `pip install -r requirements.txt`.

### Kullanıcı yapılandırması oluşturun

`.user.cfg.example` temelinde `user.cfg` adında bir .cfg dosyası oluşturun, ardından API anahtarlarınızı ve mevcut coininizi ekleyin.

**Yapılandırma dosyası aşağıdaki alanlardan oluşur:**

- **api_key** - Binance hesap kurulum aşamasında oluşturulan Binance API anahtarı.
- **api_secret_key** - Binance hesap kurulum aşamasında oluşturulan Binance gizli anahtarı.
- **current_coin** - Bu sizin başlangıç coin seçiminizdir. Desteklenen coin listenizden biri olmalıdır. Köprü para biriminizden başlamak istiyorsanız, bu alanı boş bırakın - bot desteklenen coin listenizden rastgele bir coin seçecek ve satın alacaktır.
- **bridge** - Seçtiğiniz köprü para birimi. Farklı köprülerin farklı desteklenen coin setlerine izin vereceğini unutmayın. Örneğin, belirli-coin/USDT çifti olabilir ama belirli-coin/BUSD çifti olmayabilir.
- **tld** - Bölgenize bağlı olarak 'com' veya 'us'. Varsayılan 'com'dur.
- **hourToKeepScoutHistory** - Veritabanında kaç saat scout değerlerinin tutulacağını kontrol eder. Belirtilen süre geçtikten sonra bilgiler silinecektir.
- **scout_sleep_time** - Her scout arasında kaç saniye bekleneceğini kontrol eder.
- **use_margin** - scout_margin kullanmak için 'yes'. scout_multiplier kullanmak için 'no'.
- **scout_multiplier** - Coin oranlarının mevcut durumu ile oranların önceki durumu arasındaki farkın çarpılacağı değeri kontrol eder. Daha büyük değerler için, bot işlem yapmadan önce daha büyük marjlar bekleyecektir.
- **scout_margin** - İşlem başına minimum coin kazanç yüzdesi. 0.8, %0.1 ücretle 5 scout çarpanına dönüşür.
- **strategy** - Kullanılacak ticaret stratejisi. Daha fazla bilgi için [`binance_trade_bot/strategies`](binance_trade_bot/strategies/README.md) bakın.
- **buy_timeout/sell_timeout** - Limit emrini (alış/satış) iptal etmeden ve "scout" moduna dönmeden önce kaç dakika bekleneceğini kontrol eder. 0, emrin asla erken iptal edilmeyeceği anlamına gelir.
- **scout_sleep_time** - Botun mevcut fiyat analizleri arasında kaç saniye beklemesi gerektiğini kontrol eder. Bot artık websockets üzerinde çalıştığından, bu değer düşük bir şeye (1 gibi) ayarlanmalıdır, 1'in üzerine ayarlamanın nedenleri, botun yüksek CPU kullanımını gözlemlediğinizde veya istek ağırlık limiti hakkında API hataları aldığınızda olur.

#### Çevre Değişkenleri

`user.cfg`'de sağlanan tüm seçenekler çevre değişkenleri kullanılarak da yapılandırılabilir.

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

### BNB ile Ücret Ödeme
[Binance platformundaki herhangi bir ücreti ödemek için BNB kullanabilirsiniz](https://www.binance.com/en/support/faq/115000583311-Using-BNB-to-Pay-for-Fees), bu tüm ücretleri %25 azaltacaktır. Bu avantajı desteklemek için, bot her zaman aşağıdaki işlemleri gerçekleştirecektir:
- BNB ücret ödemesinin etkin olup olmadığını otomatik olarak tespit eder.
- İncelenen işlemin ücretini ödemek için hesabınızda yeterli BNB olduğundan emin olur.
- İşlem eşiğini hesaplarken indirimi dikkate alır.

### Apprise ile Bildirimler

Apprise botun tüm en popüler mevcut bildirim servislerine bildirim göndermesine olanak tanır, örneğin: Telegram, Discord, Slack, Amazon SNS, Gotify, vb.

Bunu ayarlamak için, config dizininde bir apprise.yml dosyası oluşturmanız gerekir.

Başlamanıza yardımcı olmak için bu dosyanın örnek bir sürümü vardır.

Telegram botu çalıştırmakla ilgileniyorsanız, daha fazla bilgi [Telegram'ın resmi belgelerinde](https://core.telegram.org/bots) bulunabilir.

### Çalıştırma

```shell
python -m binance_trade_bot
```

### Docker

Resmi imaj [burada](https://hub.docker.com/r/edeng23/binance-trade-bot) mevcuttur ve her yeni değişiklikle güncellenecektir.

```shell
docker-compose up
```

Sadece SQLite tarayıcısını başlatmak istiyorsanız

```shell
docker-compose up -d sqlitebrowser
```

## Backtesting

Botun nasıl davrandığını görmek için tarihsel veriler üzerinde test edebilirsiniz.

```shell
python backtest.py
```

Farklı ayarları ve zaman periyotlarını test etmek ve karşılaştırmak için bu dosyayı değiştirmekten çekinmeyin.

## Geliştirme

Kodunuzun bir pull request yapmadan önce düzgün biçimlendirildiğinden emin olmak için,
[pre-commit](https://pre-commit.com/) kurmayı unutmayın:

```shell
pip install pre-commit
pre-commit install
```

Scout algoritması muhtemelen değiştirilmeyecektir. Alternatif bir yöntemle katkıda bulunmak istiyorsanız, [yeni bir strateji ekleyin](binance_trade_bot/strategies/README.md).

## İlgili Projeler

Yetenekli geliştiriciler grubu sayesinde, artık [bu projeyi uzaktan yönetmek için bir Telegram botu](https://github.com/lorcalhost/BTB-manager-telegram) var.

## Projeyi Destekleyin

<a href="https://www.buymeacoffee.com/edeng" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>

## Sohbete Katılın

- **Discord**: [Davet Linki](https://discord.gg/m4TNaxreCN)

## SSS

En sık sorulan sorular gibi görünen sorulara verilen yanıtların bir listesi Discord sunucumuzda, ilgili kanalda bulunabilir.

<p align="center">
  <img src = "https://usercontent2.hubstatic.com/6061829.jpg">
</p>

## Sorumluluk Reddi

Bu proje yalnızca bilgilendirme amaçlıdır. Herhangi bir
bilgiyi veya diğer materyali yasal, vergi, yatırım, finansal veya
diğer tavsiyeler olarak yorumlamamalısınız. Burada yer alan hiçbir şey benim veya herhangi bir üçüncü taraf hizmet sağlayıcısının
bu veya başka bir yargı alanında herhangi bir menkul kıymet veya diğer finansal araçları satın almak veya satmak için
talep, öneri, onay veya teklifi teşkil etmez, böyle bir talep veya teklifin o yargı alanının
menkul kıymetler yasaları kapsamında yasa dışı olacağı yerlerde.

Gerçek para kullanmayı planlıyorsanız, RİSK SİZE AİTTİR.

Hiçbir koşulda herhangi bir
iddia, zarar, kayıp, gider, maliyet veya yükümlülükten sorumlu olmayacağım, bunlara
sınırlama olmaksızın, kâr kaybı için herhangi bir doğrudan veya dolaylı zarar dahildir. 