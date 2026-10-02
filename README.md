# Android Cihaza Hermes Agent Kurulumu — Termux Türkçe Rehberi

> **Güncel resmî durum:** Termux paketi şu anda çalışmıyor. Düzeltme hazırlanıyor; aşağıdaki adımlar başarısız olabilir veya çalışmayan bir paket kurabilir. Bu nedenle kritik kullanım için düzeltme yayımlanana kadar bekleyin.

Android telefonda standart Termux uygulamasıyla Hermes Agent'ın resmî, imzalı APT paketi üzerinden kurulumu.

## Destek ve uyumluluk

- Yalnızca standart Termux uygulamasının varsayılan öneki (`/data/data/com.termux/files/usr`) desteklenir.
- Yalnızca aarch64 (`arm64-v8a`) Android cihazlar desteklenir; paket hedefi `android_24_arm64_v8a`dır.
- Yeniden adlandırılmış Termux uygulama paketleri ve başka mimariler desteklenmez.
- Android/Termux için masaüstü/sunucu `install.sh` betiğini veya glibc Linux arşivini **kullanmayın**.

## 1. Termux'u güvenli kaynaktan kur

Termux, root gerektirmeden çalışan Android terminal emülatörü ve Linux ortamıdır. Uygulamayı ve eklentilerini tek bir kaynaktan kurun; farklı kaynakları karıştırmayın.

- Resmî Termux sitesi: <https://termux.dev>
- F-Droid: <https://f-droid.org/packages/com.termux/>
- GitHub yayınları: <https://github.com/termux/termux-app/releases>

## 2. Resmî Hermes APT deposunu ekle

Önce depo kurulumu için gerekli araçları yükleyin:

```bash
pkg install curl gnupg
```

Anahtarlık dizinini oluşturup açık anahtarı indirin:

```bash
mkdir -p "$PREFIX/etc/apt/keyrings"
curl -fsSL \
  https://hermes-assets.nousresearch.com/releases/termux/stable/key.asc \
  -o "$PREFIX/etc/apt/keyrings/hermes-agent.asc"
```

Anahtarın birincil parmak izini doğrulayın:

```bash
gpg --show-keys --with-fingerprint "$PREFIX/etc/apt/keyrings/hermes-agent.asc"
```

Beklenen parmak izi şudur:

```text
C572 B5FD D1A2 9CCF A9A9 12B6 840B 0848 E139 156D
```

Parmak izi farklıysa durun; imza doğrulamasını devre dışı bırakmayın.

Depoyu ekleyin:

```bash
printf '%s\n' \
  "deb [signed-by=$PREFIX/etc/apt/keyrings/hermes-agent.asc] https://hermes-assets.nousresearch.com/releases/termux/stable hermes-stable main" \
  > "$PREFIX/etc/apt/sources.list.d/hermes-agent.list"
```

## 3. Hermes'i kur ve başlat

```bash
pkg update
pkg install hermes-agent
hermes setup
hermes --tui
```

Paket; Python, Node.js, npm, uv, ripgrep, ffmpeg ve çalışma zamanı kütüphanelerini içerir. `hermes`, `hermes-agent` ve `hermes-acp` paketli çalışma zamanlarını kullanır; Termux'un `python` veya `nodejs` paketlerini ayrıca kurmanız gerekmez.

## 4. Dosyalar ve güncelleme

| İçerik | Konum |
| --- | --- |
| Paket dosyaları | `$PREFIX/lib/hermes-agent/` |
| Komut bağlantıları | `$PREFIX/bin/hermes`, `$PREFIX/bin/hermes-agent`, `$PREFIX/bin/hermes-acp` |
| Yapılandırma ve kullanıcı verisi | `~/.hermes/`, veya seçilmiş `HERMES_HOME` |

APT ile kurulmuş Hermes'i yalnızca APT ile güncelleyin:

```bash
pkg update
pkg upgrade hermes-agent
```

`hermes update`, APT'nin sahip olduğu kurulumu değiştirmeyi reddeder ve paket yöneticisi komutunu gösterir.

### Canary kanalı

Kararlı sürüm yerine ön sürümleri izlemek için depo URL'sindeki `stable` değerini `canary`, APT suite değerindeki `hermes-stable` ifadesini `hermes-canary` yapın. İki kanal aynı anahtarla imzalanır. Kanallar arasında geçerken `hermes-agent.list` içindeki hem kanal yolunu hem suite'i değiştirin; ardından şunu çalıştırın:

```bash
pkg update && pkg upgrade hermes-agent
```

## 5. Gateway ve cron

Bu APT kurulumu `systemd`, `launchd` veya Windows Scheduled Tasks kullanmaz. Gateway'i Termux oturumunda çalıştırın:

```bash
hermes gateway run
```

Arka plan süreci için:

```bash
mkdir -p "${HERMES_HOME:-$HOME/.hermes}/logs"
nohup hermes gateway run >> "${HERMES_HOME:-$HOME/.hermes}/logs/gateway.log" 2>&1 &
```

Android, arka plandaki Termux süreçlerini askıya alabilir veya sonlandırabilir. Pil optimizasyonu istisnası ve `termux-wake-lock` yardımcı olabilir; kalıcılığı garanti etmez. Kritik otomasyonlarda ikinci kanal veya sunucu kullanın.

## 6. Sınırlar ve sorun giderme

- Paket `nemo-relay` exporter'ını, Electron'ı, yerel Chromium'u, masaüstü computer-use araçlarını veya yerel Docker daemon'unu içermez.
- Termux:API mikrofon/pano adaptörleri bu paket yolunda sağlanmaz. CLI/TUI'nin çalışması, yerel ses veya wake-word desteği olduğu anlamına gelmez.
- Android'de `sys.platform == "android"` döner; yalnızca `linux` için koşullandırılmış bağımlılık veya skill otomatik olarak kullanılamaz.
- `Package not found`: depo satırını doğrulayın, sonra `pkg update` çalıştırın.
- İmza hatasında anahtar parmak izini doğrulayın; unsigned depo kullanmayın ve hatayı atlamayın.
- Komut bulunamıyorsa `$PREFIX/bin` yolunun `PATH` içinde olduğunu doğrulayın veya paketi yeniden kurun.
- Eksik kütüphane/TUI bundle durumunda `hermes --version` ve tam hatayla bildirim yapın; çekirdek paket yerel yeniden derleme gerektirmemelidir.
- Gateway ekran kapandığında duruyorsa Android pil ve arka plan süreç sınırlarını inceleyin.

Genel tanı için:

```bash
hermes doctor
```

## 7. Kaldırma ve güvenlik

Paketi kaldırmak komut bağlantılarını siler; yapılandırmanızı, oturumlarınızı, skill'lerinizi ve memory'lerinizi korur:

```bash
pkg uninstall hermes-agent
```

- API anahtarlarını yalnızca `~/.hermes/.env` içinde tutun; sohbete, Git deposuna veya ortak depolamaya yazmayın.
- Anahtar parmak izi uyuşmuyorsa devam etmeyin; imza kontrolünü atlamayın.
- `HERMES_HOME` kullanıyorsanız güvenli ve erişilebilir bir konuma işaret ettiğini doğrulayın.

## Kaynaklar

- [Hermes Agent — Android / Termux](https://hermes-agent.nousresearch.com/docs/getting-started/termux)
- [Hermes Agent — Kurulum](https://hermes-agent.nousresearch.com/docs/getting-started/installation)
- [Hermes Agent — Güncelleme](https://hermes-agent.nousresearch.com/docs/getting-started/updating)
- [Hermes Agent — Cron](https://hermes-agent.nousresearch.com/docs/user-guide/features/cron)
- [Termux resmî sitesi](https://termux.dev)
- [Termux GitHub yayınları](https://github.com/termux/termux-app/releases)

## Lisans ve atıf

Bu rehberdeki özgün Türkçe açıklamalar [CC BY 4.0](LICENSE) ile yayımlanır. Hermes Agent, Termux ve diğer üçüncü taraf kaynaklara ait adlar, komutlar ve alıntılanan materyaller kendi sahiplerinin koşullarına tabidir; bu rehber onlara ilişkin ek lisans hakkı vermez.
