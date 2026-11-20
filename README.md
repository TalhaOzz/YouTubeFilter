# YouTube Filtreleyici

<p align="center">
  <img src="assets/readme/hero.svg" width="100%" alt="YouTube Filtreleyici: topluluk verisiyle çalışan, Chromium ve Firefox için konu ve etiket tabanlı yerel YouTube filtresi. Marka kalkanı spor, teknoloji, haber ve müzik etiketleriyle çevrilidir.">
</p>

**YouTube'un gürültüsünü topluluk verisiyle süz.** Konu ve etiket tabanlı filtreler Chromium ve Firefox'ta yerel çalışır. Veriyi YouTube Filter Store'daki anonim oylar üretir; uzantı filtre kararı için ağdan veri çekmez.

> **Site deposu:** YouTube Filter Store artık ayrı bir depoda tutulur: `https://github.com/servi-team/yt-filter-store`.

<p align="center">
  <sub><a href="../../releases">Sürüm notları ve paketler →</a></sub>
</p>

## Ne yapar?

YouTube'un ana sayfa, arama, kanal ve izleme sayfalarında çalışır. Her içerik önce video düzeyinde değerlendirilir; video verisi yoksa ve kanal kontrolü açıksa kanal verisine bakılır. Sonuç kullanıcıya göre **gizle**, **bulanıklaştır** veya **uyarı sayfası** olarak uygulanır.

| Uzantı | Site |
|---|---|
| YouTube sayfalarını tarayıcıda filtreler. Ayarlar, listeler ve PIN karması yerel kalır. | Anonim oyları toplar, editör/admin moderasyonunu yürütür ve sürümlü `db.json` yayımlar. |

## Uzantı

<p align="center">
  <img src="assets/readme/bolum-uzanti.svg" width="100%" alt="01 — Uzantı: tarayıcıda çalışır, veri yerelde kalır. Sağda tarayıcı paneli ve filtre anahtarları.">
</p>

Chromium ve Firefox için aynı kaynaktan üretilir. Filtre kararı tarayıcıda verilir; YouTube sayfasından ayrılmadan çalışır.

| Filtre | İstisnalar |
|:---:|:---:|
| <img src="docs/gorseller/eklenti-filtre.png" alt="YT Filtreleyici Filtre paneli: filtre modu, etiketlenmemiş içerik ve konu seçenekleri" width="320"> | <img src="docs/gorseller/eklenti-istisnalar.png" alt="YT Filtreleyici İstisnalar paneli: beyaz, kara ve özel listeler" width="320"> |
| <sub>Filtre modu ve konular</sub> | <sub>Beyaz, kara ve özel listeler</sub> |

<p align="center">
  <img src="docs/gorseller/eklenti-gorunum-koyu.png" alt="YT Filtreleyici Görünüm panelinin koyu tema görünümü" width="360">
</p>
<p align="center"><sub>Görünüm — arayüz temizleme ve koyu tema</sub></p>

### Öne çıkanlar

- **Üç mod:** Serbest, Orta ve Katı; konu ve etiket kurallarını farklı genişlikte uygular.
- **Esnek engelleme:** Konu ve etiketler için izin ver, gizle, bulanıklaştır veya uyarı sayfası.
- **İstisnalar:** Beyaz liste, kara liste ve özel listeler; video ya da kanalı tek tıkla listeye ekleme.
- **Hızlı karar bandı:** Geçerli sayfanın kararını ve sebebini gösterir; "Her zaman izin ver / engelle" aksiyonları sunar.
- **İsteğe bağlı hızlı ekleme:** Video ve kanal sayfalarında beyaz/kara listeye tek tıkla ekleme rayı.
- **Sadeleştirme:** Shorts, yorumlar, önerilen videolar ve hareketli küçük resim önizlemelerini ayrı ayrı kapatma.
- **İsteğe bağlı PIN kilidi:** PIN varsa ayarlar her açılışta istenir; PIN PBKDF2-SHA256 karması olarak saklanır, yazma işlemleri 10 dakikalık oturumla açılır.
- **Yerel yedekleme:** Liste paketleri, tam ayar yedeği ve isteğe bağlı `db.json` içe/dışa aktarma.

## Site

YouTube Filter Store, uzantının kullandığı veriyi üreten Next.js uygulamasıdır. Ziyaretçi hesap açmadan oy verir; editörler kayıt ve moderasyon yapar; adminler yayını yönetir. Site artık ayrı bir depoda tutulur: `https://github.com/servi-team/yt-filter-store`.

### Öne çıkanlar

- **Anonim oylama:** İmzalı HttpOnly çerez kimliğiyle hedef + konu/etiket başına tek oy; oy değiştirilebilir, hesap gerekmez.
- **Topluluk eşiği:** Sonuç için en az 3 oy gerekir. Ana konu net puana göre belirlenir; beraberlikte upvote ve taksonomi sırası kullanılır.
- **Rol tabanlı yönetim:** Ziyaretçi, editör ve admin rolleri. Herkese açık kayıt yoktur; hesapları admin oluşturur.
- **Moderasyon:** Oy geri alma, içerik kilitleme, çerez/ağ engeli, gerekçeli moderasyon kaydı ve bakım betiği.
- **Editör ve admin akışı:** Linkten içerik ekleme, YouTube metadata çözümleme, talep kuyruğu, arşivleme ve metadata düzeltme.
- **Yayın:** `db.json` önizlemesi, doğrulama, sürüm artırma, yayımlama ve indirme; geçmiş sürümler `?surum=N` ile sabit.
- **Kamusal deneyim:** Ana sayfa, keşfet, içerik sayfaları, gizlilik, koşullar, iletişim ve Türkçe hata sayfaları.

## Nasıl çalışır?

Uzantı veriyi ağdan çekmez. Konu/etiket kayıtlarını paketlenmiş `db.json` dosyasından veya kullanıcının içe aktardığı bir yedekten okur. Site oyları toplar, moderasyon sonucunu aynı sözleşmeye dönüştürür ve yalnızca yetkili adminin yayımlamasına izin verir.

<p align="center">
  <img src="assets/readme/karar-akisi.svg" width="100%" alt="Karar akışı: YouTube sayfası ve Filter Store'dan gelen db.json verisi YT Filtreleyici'de birleşir; tarayıcı içinde gizle, bulanıklaştır veya uyarı sayfası kararı verilir.">
</p>

| Sıra | Karar kuralı |
|---|---|
| 1 | Video verisi varsa yalnızca video değerlendirilir. |
| 2 | Video verisi yoksa ve kanal kontrolü açıksa kanal verisine bakılır. |
| 3 | Engellenen bir etiket her modda engeller. |

## Mimari

İki ayrı depo iki çalışma alanı barındırır: kararı tarayıcıda veren uzantı ve veriyi üreten Next.js sitesi (`https://github.com/servi-team/yt-filter-store`). İki taraf yalnızca sürümlü `db.json` sözleşmesinde buluşur; ham oylar ve moderasyon kayıtları sitede kalır.

<p align="center">
  <img src="assets/readme/mimari.svg" width="100%" alt="Mimari: tarayıcıdaki uzantı YouTube DOM'undan karar üretir ve yalnızca yerel depoyu kullanır; Next.js sitesi anonim oyları PostgreSQL'de toplar, moderasyondan geçirir ve sürümlü db.json yayımlar. Uzantı bu dosyayı paketli veya içe aktarılmış kopya olarak okur.">
</p>

| Katman | Görev | Sınır |
|---|---|---|
| **Uzantı** | YouTube DOM'unu izler, yerel kopyadan karar üretir ve gizle / bulanıklaştır / uyarı uygular. | Filtre için ağ isteği yapmaz; ayarlar, listeler ve PIN karması `storage.local` içinde kalır. |
| **Site** | Anonim oyları toplar, editör/admin moderasyonunu yürütür ve yayın adayını doğrular. | Ham oylar PostgreSQL'de kalır; dışa aktarılan dosya yalnızca sonuçları taşır. |
| **Köprü** | Sürümlü `db.json` (sema 1) paketlenir veya içe aktarılır. | `veriSurumu` büyük olan kazanır; eşitlikte paketli sürüm kullanılır. |

## Geliştirme

<p align="center">
  <img src="assets/readme/bolum-gelistirme.svg" width="100%" alt="03 — Geliştirme: iki depo, tek veri sözleşmesi. Uzantı testleri ve derlemesi; site ayrı depoda; terminalde npm run build.">
</p>

Uzantı kaynağı `src/` altındadır; site kaynağı ayrı depoda (`https://github.com/servi-team/yt-filter-store`) tutulur. `chromium/` ve `firefox/` derleme çıktısıdır; doğrudan düzenlenmez.

Gereksinim: **Node.js 20+**. Site kurulumu ve komutları için `https://github.com/servi-team/yt-filter-store` deposuna bakın.

```bash
# Uzantı
npm install
npm run build
npm test
npm run build:check
```

## Proje yapısı

```text
src/                  Uzantı kaynakları
manifests/            Chromium ve Firefox manifestleri
scripts/              Derleme, ikon ve paketleme betikleri
tests/                Node testleri + tests/smoke/ (jsdom)
chromium/             Derlenmiş Chromium uzantısı
firefox/              Derlenmiş Firefox uzantısı
docs/                 Eklenti kuralları, şemalar ve gizlilik belgeleri
docs/gorseller/       README ekran görüntüleri
assets/readme/        README hero, bölüm şeritleri ve mimari SVG'leri
```

## Belgeler

| Belge | Kapsam |
|---|---|
| [`docs/SISTEM_TASARIMI.md`](docs/SISTEM_TASARIMI.md) | Eklenti kuralları (site bölümleri site deposunda) |
| [Site deposu](https://github.com/servi-team/yt-filter-store) | `db.json` sözleşmesi, site kurulumu, rolleri, QA ve sunucu kurulumu |
| [`docs/GIZLILIK.md`](docs/GIZLILIK.md) | Uzantı gizlilik politikası |
| [`CHANGELOG.md`](CHANGELOG.md) | Değişiklik geçmişi |

## Gizlilik ve güvenlik

Uzantı hiçbir kişisel veri toplamaz, sunucuya göndermez ve satmaz. Ayarlar, listeler, PIN karması, sayfadan toplanan kanal adı/avatarı ve isteğe bağlı `db.json` kopyası yalnızca tarayıcının yerel deposunda tutulur.

Site, anonim oy için imzalı HttpOnly çerez kimliği ve `IP_HASH_SECRET` ile üretilmiş HMAC-SHA256 IP özeti kullanır; ham IP saklamaz. Analitik, reklam ve üçüncü taraf izleme yoktur. Ayrıntılar: [`docs/GIZLILIK.md`](docs/GIZLILIK.md) ve sitenin `/gizlilik` sayfası.

## Sürüm ve lisans

Son sürüm **2.1.0**. Paketler ve sürüm notları: [GitHub Releases](../../releases). Değişiklikler: [`CHANGELOG.md`](CHANGELOG.md).

[Apache License 2.0](LICENSE). Bu proje YouTube'un resmî bir ürünü değildir.

## Notlar

- Paketli `src/data/db.json` şu an örnek düzeyinde; yaygın YouTube içerikleri "bilinmeyen" olarak değerlendirilir. Kapsam, site yayını ve topluluk oylarıyla büyür.
- Gerçek YouTube üzerinde uçtan uca tarayıcı/ekran okuyucu QA'sı ve site deposundaki Docker/PostgreSQL/Caddy üretim denemesi bu makinede yapılmadı.
- Site içi "Bildir" formu yoktur; iletişim [GitHub Issues](../../issues) üzerindendir.
