# Android Cihaza Hermes Agent Kurulumu — Termux Türkçe Rehberi

Android telefonda Termux kullanarak Hermes Agent kurulumunu sıfırdan anlatan Türkçe rehber.

> **Platform durumu:** Android/Termux, Hermes için best-effort / Tier 2 platformdur. Telefonunda çalışan bir CLI ve Telegram gateway kurabilirsin; ancak masaüstü/sunucu özelliklerinin tamamı garanti edilmez.

## Bu rehber kimin için?

- Android telefonda Hermes'i terminalden kullanmak isteyenler
- Telegram botu veya zamanlanmış görev çalıştırmak isteyenler
- Root kullanmadan yerel bir geliştirme/otomasyon ortamı isteyenler

## 1. Termux'u güvenli kaynaktan kur

Termux, root gerektirmeden çalışan Android terminal emülatörü ve Linux ortamıdır. Uygulamayı **tek bir kaynaktan** kur; farklı kaynaklardaki Termux uygulaması ve eklentilerini karıştırma.

- Resmî Termux sitesi: <https://termux.dev>
- F-Droid: <https://f-droid.org/packages/com.termux/>
- GitHub yayınları: <https://github.com/termux/termux-app/releases>

Uygulamayı aç ve Android'in istediği depolama/bildirim izinlerini yalnızca ihtiyacın varsa ver.

## 2. Termux'u güncelle

```bash
pkg update && pkg upgrade
```

Paket aynası yavaş veya bozuksa önce şunu çalıştırıp bir ayna seç:

```bash
termux-change-repo
```

## 3. İsteğe bağlı: Telefon depolamasına erişim

Dosyalarına Termux üzerinden erişmen gerekiyorsa:

```bash
termux-setup-storage
```

Android izin penceresini onayla. Bu işlem `~/storage` altında ortak depolama kısayolları oluşturur. Gizli anahtarları veya Hermes `~/.hermes/.env` dosyasını ortak depolamada tutma.

## 4. Hermes'i resmî kurulum betiğiyle yükle

Önce Git'in varlığını kontrol et; yoksa kur:

```bash
git --version || pkg install git
```

Ardından resmî Hermes installer'ını çalıştır:

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

Kurulum, Termux'ta gerekli paketleri ve Python ortamını hazırlamayı dener; `hermes` komutunu PATH'e bağlar. Yeni terminal aç veya kabuk ayarını yeniden yükle:

```bash
source ~/.bashrc
```

## 5. Kurulumu doğrula ve Hermes'i başlat

```bash
hermes --version
hermes doctor
hermes
```

İlk açılışta model/sağlayıcı seçimi istenir. Sonradan değiştirmek için:

```bash
hermes model
```

Nous Portal hesabıyla kurulum yapmak istersen:

```bash
hermes setup --portal
```

## 6. Telegram ve zamanlanmış işler

Telegram gateway için önce interaktif yapılandırmayı başlat:

```bash
hermes gateway setup
```

Zamanlanmış işler Hermes'in cron sistemiyle çalışır. Ancak Android, arka plandaki Termux işlemlerini pil tasarrufu için durdurabilir. Sürekli açık gateway veya cron görevleri için:

- Termux'u Android pil optimizasyonundan çıkar.
- Uygulamayı zorla kapatma.
- Uzun süreli, kritik otomasyonlarda VPS/sunucu kullanmayı düşün.

## 7. Termux'taki bilinen sınırlar

| Özellik | Durum |
| --- | --- |
| Hermes CLI, cron, PTY/arka plan terminali, Telegram gateway | Test edilmiş Termux yolunda desteklenir |
| Docker tabanlı terminal yalıtımı | Termux içinde yok |
| Yerel sesli yazıya döküm (`faster-whisper`) | Android wheel'i olmadığı için test edilen yolda yok |
| Browser / Playwright otomatik kurulumu | Installer tarafından atlanır; deneysel kabul edilir |
| Arka plan gateway sürekliliği | Android tarafından durdurulabilir; best-effort |

## 8. Sık karşılaşılan sorunlar

### `hermes: command not found`

Yeni bir terminal aç. Devam ederse:

```bash
source ~/.bashrc
hermes doctor
```

### Python sürümü desteklenmiyor

Resmî Termux sayfasına göre Hermes Python `>=3.11,<3.14` gerektirir. Termux'taki güncel Python bu aralığın dışındaysa installer desteklenen yorumlayıcıyı bulmaya çalışır. Manuel kurulum yapıyorsan resmî Termux sayfasındaki `python3.13` yönergesini izle.

### Bağımlılık derleme hatası

Manuel kaynak kurulumunda Android derleme araçları gerekir:

```bash
pkg install clang rust make pkg-config libffi openssl
```

Ardından Termux'a özel kurulum yolunu kullan:

```bash
python -m pip install -e '.[termux]' -c constraints-termux.txt
```

### `ANDROID_API_LEVEL` / `jiter` / `maturin` hatası

Kaynak dizininde API seviyesini ayarlayıp tekrar dene:

```bash
export ANDROID_API_LEVEL="$(getprop ro.build.version.sdk)"
```

### Voice veya `[all]` kurulumu başarısız

`faster-whisper` bağımlılığı olan `ctranslate2`, Android wheel yayımlamaz. `.[all]` yerine resmî belgelerdeki `.[termux]` veya `.[termux-all]` yolunu tercih et.

## 9. Hızlı native APT paketi alternatifi

Cihazda Python/Rust bağımlılıklarını derlemek istemiyorsan topluluk tarafından işletilen native APT paketi seçeneği vardır. Bu **NousResearch'ün resmî dağıtımı değildir**; imzalama anahtarına ve depo işletmecisine güvenmen gerekir.

Paketli seçenek, güncelleme politikası ve teknik ayrıntılar için şu rehbere git:

➡️ **[Termux için Hermes Agent — Türkçe teknik rehber](https://github.com/mogihermes/hermes-termux-tr)**

APT ile kurulmuş Hermes sürümleri `hermes update` yerine şu komutla güncellenir:

```bash
pkg update
pkg upgrade hermes-agent
```

## Güvenlik notları

- API anahtarlarını yalnızca `~/.hermes/.env` içinde tut; sohbete, Git deposuna veya ortak depolamaya yazma.
- `curl | bash` yalnızca bildiğin ve doğruladığın alan adlarından çalıştırılmalıdır. Bu rehberdeki resmî installer alan adı: `hermes-agent.nousresearch.com`.
- Topluluk APT paketi, resmî installer ile aynı güven zincirine sahip değildir. Seçmeden önce teknik rehberdeki anahtar parmak izini ve kaynak depoyu incele.

## Kaynaklar

- [Hermes Agent — Android / Termux](https://hermes-agent.nousresearch.com/docs/getting-started/termux)
- [Hermes Agent — Kurulum](https://hermes-agent.nousresearch.com/docs/getting-started/installation)
- [Hermes Agent — Güncelleme](https://hermes-agent.nousresearch.com/docs/getting-started/updating)
- [Termux resmî sitesi](https://termux.dev)
- [Termux GitHub yayınları](https://github.com/termux/termux-app/releases)
- [İleri seviye Termux Hermes rehberi](https://github.com/mogihermes/hermes-termux-tr)
