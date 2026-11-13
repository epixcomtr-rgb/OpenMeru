# OpenMeru Mail — Türkçe sürüm (fork)

[OpenMeru Mail](https://github.com/WeslleyHarakawa/OpenMeru), kendi alan adınızda çalıştırdığınız açık kaynak (MIT) bir webmail'dir: gönderim Brevo (veya herhangi bir SMTP aktarıcısı) ile, alım Cloudflare Email Routing + küçük bir Email Worker ile yapılır. Postfix, IMAP veya 25 numaralı port gerekmez.

Bu fork, upstream koda **Türkçe arayüzü (11. dil)** ekler. Başka hiçbir davranış değiştirilmez; upstream ile uyumlu kalır.

- Upstream: <https://github.com/WeslleyHarakawa/OpenMeru>
- Bu fork: <https://github.com/epixcomtr-rgb/OpenMeru>
- Canlı kullanım: <https://mail.epix.tr> (epix.tr)

---

## Bu forkta ne var?

| Konu | Durum |
|---|---|
| Arayüz dilleri | 10 → **11** (Türkçe eklendi) |
| Türkçe çeviri kapsamı | `lang/en.json` içindeki **572 anahtarın tamamı** |
| Doğrulama mesajları | `lang/tr/validation.php` (Türkçe) |
| Yerelleştirme kodu | `pt_BR` deseniyle aynı: `tr` / `tr-TR` |
| Bonus hata düzeltmesi | Sayfalama metnindeki `:total` bozulması (bkz. aşağıda) |

### Eklenen / değişen dosyalar

| Dosya | Ne yapıldı |
|---|---|
| `lang/tr.json` | **Yeni.** 572 arayüz metninin Türkçesi. `en.json` ile birebir aynı anahtarlar, aynı sıra; `:email`, `:count`, `:total` gibi yer tutucular ve HTML etiketleri korundu. |
| `lang/tr/validation.php` | **Yeni.** Laravel form doğrulama mesajları. |
| `app/Support/Locales.php` | `'tr' => ['Türkçe', 'tr-TR']` eklendi; dil listesi artık 11. |
| `lang/en.json` | "10 languages" → "11 languages", dil listesine Türkçe eklendi. |
| `app/Http/Controllers/SeoController.php`<br>`app/Http/Controllers/McpController.php` | `llms.txt` ve MCP açıklamalarındaki dil sayısı 11'e güncellendi. |
| `resources/views/partials/lang-menu.blade.php` | Yorum güncellendi (davranış aynı; menü `Locales::ALL` üzerinden otomatik). |
| `tests/Unit/LangParityTest.php` | Test listesine `tr` eklendi — yeni bir anahtar eklendiğinde Türkçe'nin de tamamlanması zorunlu. |
| `public_root/assets/js/mail.js` | Dil-bağımsız hata düzeltmesi (aşağıya bakın). |
| `README.md` | Dil sayısı ve Türkçe bağlantısı. |
| `README.tr.md` | **Yeni.** Bu dosya. |

### Düzeltilen hata (tüm diller için geçerli)

`public_root/assets/js/mail.js` içinde:

```js
// önce (bozuk): ":to" önce değiştiği için ":total" bozuluyordu
t("admin.showing").replace(":from", 1).replace(":to", items.length).replace(":total", state.total)

// sonra (doğru): en uzun anahtar önce
t("admin.showing").replace(":total", state.total).replace(":from", 1).replace(":to", items.length)
```

Sonuç: gelen kutusunda "2 kayıttan 1–2 arası gösteriliyor" yerine "2**tal** kayıttan 1–**:to** arası…" görünüyordu.

---

## Türkçe dil nasıl seçilir?

Dil seçimi **kullanıcı bazlıdır**; global bir "site dili" ayarı yoktur.

1. Sağ üstteki 🌐 (Dil) düğmesi → **Türkçe**
2. veya bağlantıyla: `https://alanadiniz.com/?lang=tr`

Seçim `openmeru_lang` çerezinde (1 yıl) tutulur ve giriş yapılmışsa `users.locale` kolonuna da yazılır.

Öncelik sırası: `?lang=` → çerez → tarayıcının `Accept-Language` başlığı → English.

Türkçe dosya eklemek/çeviri yapmak isteyenler için: `lang/en.json` ile `lang/tr.json` anahtar setleri **aynı olmak zorunda** (`tests/Unit/LangParityTest.php` bunu denetler):

```bash
php artisan test --filter=LangParityTest
```

---

## Kurulum

Upstream ile aynıdır; bu fork'u klonlamak için:

```bash
git clone https://github.com/epixcomtr-rgb/OpenMeru.git openmeru-app
cd openmeru-app
php artisan openmeru:install
```

Kurulum sihirbazı PHP/Composer kontrolü, bağımlılıklar, `.env`, anahtarlar, tablolar ve Cloudflare/Brevo değerlerini adım adım halleder. Ayrıntılı rehber: upstream `README.md` ve <https://openmeru.com/#install>.

Web sunucusunun belge kökü **`public_root`** klasörü olmalıdır. PHP 8.5, MySQL 8 / MariaDB gerekir.

---

## Güncelleme akışı (upstream'i takip etmek)

Bu fork upstream'in bir dalıdır, bu yüzden güncellemeler upstream'den çekilir:

```bash
git remote add upstream https://github.com/WeslleyHarakawa/OpenMeru.git   # ilk seferde
git fetch upstream
git merge upstream/main        # Türkçe değişiklikler korunur, çakışma olursa çözün
git push origin main
```

Yerleşik güncelleyici (`php artisan openmeru:update` / panelde **Şimdi güncelle**) çalışma ağacı kirliyse durur ve şunu söyler: *"Local changes would be overwritten by the update"*. Sunucuda yerel değişiklik varsa:

```bash
git stash push -u -m tr-lang
php artisan openmeru:update
git stash pop
```

> **Not:** Bu forkta `VERSION` dosyası bilinçli olarak değiştirilmedi. Sürüm numarası upstream yayınlarıyla karşılaştırıldığı için yükseltmek güncelleme denetimini yanıltır.

### Sunucu notu: `storage/framework/views` sahipliği

Blade dosyalarının mtime'ı değiştiğinde Laravel derlenmiş görünümü `touch()` eder; dosya web kullanıcısına ait değilse `touch(): Utime failed: Operation not permitted` ile **HTTP 500** oluşur. Derlenmiş görünüm önbelleği web kullanıcısına ait olmalı:

```bash
sudo chown -R www-data:www-data storage/framework/views
```

---

## Lisans ve teşekkür

- Lisans: **MIT** (`LICENSE`)
- Özgün proje ve tüm tasarım: **Weslley Harakawa** — <https://github.com/WeslleyHarakawa/OpenMeru>
- Türkçe çeviri: [epixcomtr-rgb](https://github.com/epixcomtr-rgb) (epix.tr)
