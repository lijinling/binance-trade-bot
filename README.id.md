# Bot Trading Binance
> Bot trading cryptocurrency otomatis

[English](README.md) | [简体中文](README.zh-CN.md) | [Español](README.es.md) | [العربية](README.ar.md) | [हिन्दी](README.hi.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [বাংলা](README.bn.md) | [Français](README.fr.md) | [Indonesia](README.id.md)

![github](https://img.shields.io/github/workflow/status/edeng23/binance-trade-bot/binance-trade-bot)
![docker](https://img.shields.io/docker/pulls/edeng23/binance-trade-bot)
[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/edeng23/binance-trade-bot)

[![Deploy to DO](https://mp-assets1.sfo2.digitaloceanspaces.com/deploy-to-do/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/coinbookbrasil/binance-trade-bot/tree/master&refcode=a076ff7a9a6a)

## Ikuti saya di Twitter :)

[![Twitter](https://img.shields.io/twitter/url/https/twitter.com/cloudposse.svg?style=social&label=Follow%20%400xedeng)](https://twitter.com/0xedeng)

## Mengapa?

Proyek ini terinspirasi dari pengamatan bahwa semua cryptocurrency berperilaku hampir sama. Ketika satu naik, semuanya naik, dan ketika satu turun, semuanya turun. _Hampir_. Selain itu, semua koin mengikuti kepemimpinan Bitcoin; perbedaannya ada pada pergeseran fasenya.

Jadi, jika koin-koin pada dasarnya berosilasi satu sama lain, tampaknya cerdas untuk menukar koin yang naik dengan koin yang turun, dan kemudian menukar kembali ketika rasionya terbalik.

## Bagaimana?

Trading dilakukan di platform pasar Binance, yang tentu saja, tidak memiliki pasar untuk setiap pasangan altcoin. Solusinya adalah menggunakan mata uang jembatan yang akan melengkapi pasangan yang hilang. Mata uang jembatan default adalah Tether (USDT), yang stabil secara desain dan kompatibel dengan hampir setiap koin di platform.

<p align="center">
  Koin A → USDT → Koin B
</p>

Cara bot memanfaatkan perilaku yang diamati adalah selalu menurunkan dari koin "kuat" ke koin "lemah", dengan asumsi bahwa pada suatu titik keadaan akan berbalik. Kemudian akan kembali ke koin asli, akhirnya memiliki lebih banyak dari yang dimiliki sebelumnya. Ini dilakukan dengan mempertimbangkan biaya trading.

<div align="center">
  <p><b>Koin A</b> → USDT → Koin B</p>
  <p>Koin B → USDT → Koin C</p>
  <p>...</p>
  <p>Koin C → USDT → <b>Koin A</b></p>
</div>

Bot melompat di antara set koin yang dikonfigurasi dengan syarat bahwa tidak kembali ke koin kecuali menguntungkan sehubungan dengan jumlah yang terakhir dipegang. Ini berarti kita tidak akan pernah berakhir memiliki lebih sedikit dari koin tertentu. Risikonya adalah salah satu koin mungkin tiba-tiba jatuh bebas relatif terhadap yang lain, menarik algoritma serakah terbalik kita.

## Pengaturan Binance

- Buat [akun Binance](https://www.binance.com/en/register?ref=129471815) (Termasuk link referral saya, saya akan sangat berterima kasih jika Anda menggunakannya).
- Aktifkan otentikasi dua faktor.
- Buat kunci API baru.
- Dapatkan cryptocurrency. Jika simbolnya tidak ada dalam daftar default, tambahkan.

## Pengaturan Alat

### Instal dependensi Python

Jalankan baris berikut di terminal: `pip install -r requirements.txt`.

### Buat konfigurasi pengguna

Buat file .cfg bernama `user.cfg` berdasarkan `.user.cfg.example`, kemudian tambahkan kunci API dan koin saat ini Anda.

**File konfigurasi terdiri dari bidang-bidang berikut:**

- **api_key** - Kunci API Binance yang dihasilkan pada tahap pengaturan akun Binance.
- **api_secret_key** - Kunci rahasia Binance yang dihasilkan pada tahap pengaturan akun Binance.
- **current_coin** - Ini adalah koin awal pilihan Anda. Harus salah satu dari koin dalam daftar koin yang didukung Anda. Jika Anda ingin mulai dari mata uang jembatan Anda, biarkan bidang ini kosong - bot akan memilih koin acak dari daftar koin yang didukung Anda dan membelinya.
- **bridge** - Mata uang jembatan pilihan Anda. Perhatikan bahwa jembatan yang berbeda akan memungkinkan set koin yang didukung yang berbeda. Misalnya, mungkin ada pasangan koin-tertentu/USDT tetapi tidak ada pasangan koin-tertentu/BUSD.
- **tld** - 'com' atau 'us', tergantung wilayah Anda. Default adalah 'com'.
- **hourToKeepScoutHistory** - Mengontrol berapa jam nilai scout disimpan dalam database. Setelah waktu yang ditentukan berlalu, informasi akan dihapus.
- **scout_sleep_time** - Mengontrol berapa detik yang ditunggu antara setiap scout.
- **use_margin** - 'yes' untuk menggunakan scout_margin. 'no' untuk menggunakan scout_multiplier.
- **scout_multiplier** - Mengontrol nilai yang digunakan untuk mengalikan perbedaan antara keadaan saat ini dari rasio koin dan keadaan sebelumnya dari rasio. Untuk nilai yang lebih besar, bot akan menunggu margin yang lebih besar sebelum melakukan trade.
- **scout_margin** - Persentase minimum keuntungan koin per trade. 0.8 diterjemahkan ke pengali scout 5 pada biaya 0.1%.
- **strategy** - Strategi trading yang akan digunakan. Lihat [`binance_trade_bot/strategies`](binance_trade_bot/strategies/README.md) untuk informasi lebih lanjut.
- **buy_timeout/sell_timeout** - Mengontrol berapa menit untuk menunggu sebelum membatalkan order limit (beli/jual) dan kembali ke mode "scout". 0 berarti order tidak akan pernah dibatalkan secara prematur.
- **scout_sleep_time** - Mengontrol berapa detik bot harus menunggu antara analisis harga saat ini. Karena bot sekarang beroperasi pada websockets, nilai ini harus diatur ke sesuatu yang rendah (seperti 1), alasan untuk mengaturnya di atas 1 adalah ketika Anda mengamati penggunaan CPU yang tinggi oleh bot atau menerima error API tentang batas berat permintaan.

#### Variabel Lingkungan

Semua opsi yang disediakan dalam `user.cfg` juga dapat dikonfigurasi menggunakan variabel lingkungan.

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

### Membayar Biaya dengan BNB
Anda dapat [menggunakan BNB untuk membayar biaya apa pun di platform Binance](https://www.binance.com/en/support/faq/115000583311-Using-BNB-to-Pay-for-Fees), yang akan mengurangi semua biaya sebesar 25%. Untuk mendukung manfaat ini, bot akan selalu melakukan operasi berikut:
- Secara otomatis mendeteksi apakah Anda memiliki pembayaran biaya BNB yang diaktifkan.
- Memastikan Anda memiliki BNB yang cukup di akun Anda untuk membayar biaya perdagangan yang diperiksa.
- Mempertimbangkan diskon saat menghitung ambang batas trading.

### Notifikasi dengan Apprise

Apprise memungkinkan bot untuk mengirim notifikasi ke semua layanan notifikasi paling populer yang tersedia seperti: Telegram, Discord, Slack, Amazon SNS, Gotify, dll.

Untuk mengatur ini, Anda perlu membuat file apprise.yml di direktori config.

Ada versi contoh dari file ini untuk membantu Anda memulai.

Jika Anda tertarik untuk menjalankan bot Telegram, informasi lebih lanjut dapat ditemukan di [dokumentasi resmi Telegram](https://core.telegram.org/bots).

### Menjalankan

```shell
python -m binance_trade_bot
```

### Docker

Image resmi tersedia [di sini](https://hub.docker.com/r/edeng23/binance-trade-bot) dan akan diperbarui dengan setiap perubahan baru.

```shell
docker-compose up
```

Jika Anda hanya ingin memulai browser SQLite

```shell
docker-compose up -d sqlitebrowser
```

## Backtesting

Anda dapat menguji bot pada data historis untuk melihat bagaimana perilakunya.

```shell
python backtest.py
```

Jangan ragu untuk memodifikasi file tersebut untuk menguji dan membandingkan pengaturan dan periode waktu yang berbeda.

## Pengembangan

Untuk memastikan kode Anda diformat dengan benar sebelum membuat pull request,
ingatlah untuk menginstal [pre-commit](https://pre-commit.com/):

```shell
pip install pre-commit
pre-commit install
```

Algoritma scout kemungkinan tidak akan diubah. Jika Anda ingin berkontribusi dengan metode alternatif, [tambahkan strategi baru](binance_trade_bot/strategies/README.md).

## Proyek Terkait

Berkat sekelompok pengembang berbakat, sekarang ada [bot Telegram untuk mengelola proyek ini dari jarak jauh](https://github.com/lorcalhost/BTB-manager-telegram).

## Dukung Proyek

<a href="https://www.buymeacoffee.com/edeng" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>

## Bergabung dengan Chat

- **Discord**: [Link Undangan](https://discord.gg/m4TNaxreCN)

## FAQ

Daftar jawaban untuk apa yang tampaknya menjadi pertanyaan yang paling sering diajukan dapat ditemukan di server Discord kami, di saluran yang sesuai.

<p align="center">
  <img src = "https://usercontent2.hubstatic.com/6061829.jpg">
</p>

## Penafian

Proyek ini hanya untuk tujuan informasi. Anda tidak boleh menafsirkan
informasi atau materi lain sebagai nasihat hukum, pajak, investasi, keuangan, atau
lainnya. Tidak ada yang terkandung di sini yang merupakan permintaan, rekomendasi,
dukungan, atau penawaran dari saya atau penyedia layanan pihak ketiga mana pun untuk membeli atau menjual
sekuritas atau instrumen keuangan lainnya di yurisdiksi ini atau yurisdiksi lain mana pun
di mana permintaan atau penawaran tersebut akan melanggar hukum berdasarkan
undang-undang sekuritas yurisdiksi tersebut.

Jika Anda berencana menggunakan uang sungguhan, GUNAKAN DENGAN RISIKO ANDA SENDIRI.

Dalam keadaan apa pun saya tidak akan bertanggung jawab atas
klaim, kerusakan, kerugian, pengeluaran, biaya, atau kewajiban apa pun, termasuk,
tanpa batasan, kerusakan langsung atau tidak langsung apa pun atas kehilangan keuntungan. 