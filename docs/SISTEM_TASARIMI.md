# YouTube Filtre & YouTube Filter Store — Sistem Tasarımı

> **Kaynak:** Ekip liderinin tasarım dökümanı (2026-10-18, el yazısı notlar + metin). Bu dosya onun düzenlenmiş hâlidir; **1–4. bölümler liderin kurallarıdır, değiştirilmez.** (2026-10-22: bu bölümlere yalnızca açıklayıcı *Not* satırları eklendi, metin değiştirilmedi.)
> **6. bölüm** uygulama sırasında belirsiz kalan noktaların nasıl yorumlandığını ve hangilerinin liderden teyit beklediğini listeler.
> **5 ve 7. bölümler** YouTube Filter Store'un topluluk oylama, rol, anonim oy ve moderasyon kurallarını tanımlar; site deposunda tutulur.
> Kodda karşılığı: `src/core/decision.js` (algoritma), `src/core/taxonomy.js` (ayar şeması), `src/core/engine.js` (eylem çözümleme), `https://github.com/servi-team/yt-filter-store/blob/main/docs/db-sema.md` (veri sözleşmesi).

Bu belge, YouTube Filtre eklentisinin çalışma algoritmasını, filtreleme modlarını ve kullanıcı ayarlarını tanımlar; siteyle ortak veri sözleşmesine `https://github.com/servi-team/yt-filter-store/blob/main/docs/db-sema.md` üzerinden bağlanır.

> **Site bölümleri:** 5 ve 7. bölümler site deposuna taşındı: `https://github.com/servi-team/yt-filter-store/blob/main/docs/SISTEM_TASARIMI.md`. Bu dosya yalnızca eklenti kurallarını (1–4 ve 6) içerir.

---

## 1. Sözlük ve temel kavramlar

- **Konu:** Kanalın veya videonun genel kategorisi. Örnek: spor, teknoloji, haber, müzik, eğlence, oyun.
- **Etiket (flag):** Uyarı nitelikli işaretler. Örnek: şiddet, küfür, argo, zararlı, spam, olumsuz örnek, AI ile oluşturulmuş.
  *Kullanıcı bunları engellemek zorunda değildir; kendi tercihine göre izlemeyi seçebilir.*
- **Bilinmeyen (Unknown):** Videoya hiçbir şekilde etiket ve konu bilgisi girilmemiş olması.
  Bu durumda (kullanıcı ayarlarda kapatmadıysa) **kanala bakılır.** Kanal için de bilgi girilmemişse **veya** kullanıcı "kanal kontrol ayarı"nı kapatmışsa o video "bilinmeyen" kabul edilir.
- **Kararsız (Undecided):** Video veya kanal hakkında bilgi girilmiş, ancak topluluk oylamasında ("Bu video bu konuya/etikete mi ait?") net bir sonuç çıkmamış (toplam oyu 0 olan) konu ve etiketler.

## 2. Genel algoritma ve temel kurallar

### Kontrol sırası (her durumda)
1. Önce **videonun** konu ve etiketlerine bakılır.
2. Videoya dair **hiçbir veri girilmemişse** kanala bakılır (kanal kontrol ayarı açıksa).

### Her zaman geçerli kural (etiket önceliği)
**Engellenen bir etiket varsa her zaman engelle.** Kullanıcının engellediği bir etiket videoda bulunuyorsa seçili mod ne olursa olsun (Serbest, Orta, Katı fark etmez) video engellenir.
Örnek: kullanıcı "küfür" etiketini engelliyorsa, videonun konusu izinli olsa bile video engellenir.

## 3. Kullanıcı ayarları ve arayüz

### A. İlk kurulum (Hoş geldiniz ekranı)
- **PIN seçimi:** Ayarların güvenliği için 4 haneli PIN belirlenir (`_ _ _ _`).
- **Bilinmeyen videolar tercihi:** "Henüz yeni bir eklentiyiz, bilinmeyen çok sayıda video ve kanal var." uyarısıyla başlangıç tercihi alınır: **Bilinmeyenler: İzin ver / Engelle.**

### B. Kanal kontrol ayarı
Ayarlara "Kanalın konu ve etiketlerine bakmayı devre dışı bırak" seçeneği eklenir. Bu ayar kapatıldığında, videosunda veri olmayan içerikler için kanal verisine bakılmaz; doğrudan "bilinmeyen" muamelesi yapılır.

### C. Engelleme görünümü ve filtreleme biçimi
Kullanıcı, engellenen videoların **nasıl** filtreleneceğini **konuya veya etikete göre ayrı ayrı** seçebilir (arayüzde anahtar/toggle ve yanında dropdown menü):
- **Gizle** — videoyu tamamen sayfadan kaldırır.
- **Bulanıklaştır** — videoyu blurlar.
- **Üstünü bir uyarı sayfasıyla kapat.**

Örnek: *Spor (filtreleniyor) → Bulanıklaştır*; *Küfür (filtreleniyor) → Gizle*.

## 4. Filtreleme modları
> Tüm modlarda video için veri girilmediyse ve kanal kontrol ayarı açıksa kanalın verilerine bakılır.

### 🟢 Serbest mod
- **Temel kural:** Videonun konularından **herhangi biri** izin verilenler listesindeyse (ve engellenen bir etiketi yoksa) **izin ver.**
- **Kararsız durumlar:** Kararsız kalınan konu ve etiketlere kullanıcı lehine ("işine geldiği gibi") izin verilir.
  - Örnek 1: "Küfür" etiketi kullanıcıda engelli. Video "kararsız olarak küfür" diye işaretlenmiş → **izin ver.**
  - Örnek 2: "Spor" konusuna kullanıcıda izin veriliyor. Video "kararsız olarak spor" diye işaretlenmiş → **izin ver.**

### 🟡 Orta mod
- **Temel kural:** Videonun **ana konusu** izin verilenler listesindeyse (ve her zamanki gibi engellenen bir etiketi yoksa) **izin ver.**
- **Ek seçenekler** (Orta mod kullanıcıya 2 yeni özelleştirme sunar):
  - **Bilinmeyenler:** İzin ver / Engelle (açılış ekranında kullanıcıya sorulur).
  - **Kararsızlar:** İzin ver (varsayılan) / Engelle.

### 🔴 Katı mod
- **Temel kural:** Videonun konularından (ana veya yan konular) **herhangi biri engelliyse engelle** (+ engellenen bir etiket varsa engelle).

---

## 6. Uygulamada netleştirilen noktalar (lider dökümanında açık olmayanlar)

Aşağıdakiler **bizim yorumumuzdur**. ✅ = kodlandı/test edildi veya onaylandı (site uygulayacak), ❓ = hâlâ liderden teyit bekliyor.

| # | Konu | Yorum / karar | Durum |
|---|---|---|---|
| 1 | **Ana konu ölçütü** | Parantezde "en çok upvote" yazıyor, ama örnekte Oyun 30 upvote alırken ana konu Teknoloji (25 upvote, +23 net). Örnek net puanla tutarlı; **net puan** esas alınır. Sitede "upvote" ibaresi silinmeli. | ✅ onaylandı (2026-10-21) |
| 2 | **Beraberlik** | "Listedeki ilki" = konu listesindeki (`taxonomy.KONULAR`) sıra. **Sıralama: (1) net toplam, (2) upvote sayısı fazla olan, (3) hâlâ eşitse `taxonomy.KONULAR` sırasındaki ilk.** Kazanamayan eşit konu yan konu olur. Sıra sabit olduğu için sonuç belirlidir. | ✅ onaylandı (2026-10-22; upvote adımı eklendi, site uygular) |
| 3 | **Kararsız = toplam 0 *ve* en az bir oy** | Hiç oy almamış (0/0/0) konu kararsız sayılmaz, kayıtta hiç yer almaz. 3 oy eşiği (#18) ile birlikte: kararsız = net toplam 0 **ve** toplam oy en az 3. Sitenin arayüzünde oy almamış konunun nasıl adlandırılacağı (ör. "oy yok") Faz 0'da netleşecek; `db.json`'a hiçbir durumda yazılmaz. | ✅ onaylandı (2026-10-21; site uygular) |
| 4 | **Kaynak seçimi** | Video verisi **varsa yalnızca video**, yoksa (ve kanal kontrol açıksa) yalnızca kanal değerlendirilir; ikisi asla birleştirilmez. Eski özet "video veya kanal" diyordu; bu belgeyle değiştirildi. | ✅ |
| 5 | **Yalnızca kararsız verisi olan video** | "Bilgi girilmiş" sayılır → **bilinmeyen değildir**, kanala bakılmaz. | ✅ |
| 6 | **Ana konusuz kayıt** | Oylamada hiçbir konu pozitif çıkmazsa (veya hiçbiri eşiği geçmezse) ana konu olmaz. Böyle kayıt geçerlidir (yalnızca etiket ve/veya kararsız bilgi taşıyabilir). Yan konu varsa ana konu zorunludur. Tamamen boş kayıt geçersizdir ("bilinmeyen"). Düzenleme sitesinde editör girişinde ana konu yine zorunlu kalır. | ✅ |
| 7 | **Ana konusuz kayıtta konu kuralı** | Kesin konu yoksa konu kuralı uygulanmaz; yalnızca etiketler ve kararsız politikası belirler. | ✅ |
| 8 | **Serbest + kararsız konu** | Kararsız bir konu "kullanıcı lehine" **izinli konu sayılır**: ana konusu engelli bir videoda kararsız bir konu varsa Serbest modda video açılır. | ❓ teyit |
| 9 | **Serbest + kararsız etiket** | Kararsız etiket hiçbir zaman engellemez. "Her zaman engelle" kuralı yalnızca **kesinleşmiş** etiketler için geçerlidir. | ✅ |
| 10 | **Orta + "Kararsızlar: Engelle"** | Kararsız bir konu/etiket, **kullanıcı onu zaten engelliyorsa** kesinleşmiş gibi sayılır ve engeller. Kullanıcının engellemediği kararsız şey engellemez. | ❓ teyit |
| 11 | **Katı + kararsız** | Dökümanda yok. Katı modda kararsızlar, **kullanıcı o konu/etiketi engelliyorsa** her zaman engeller (Orta + Engelle ile aynı). | ❓ teyit |
| 12 | **Bilinmeyen ayarı** | Her modda geçerli (hoş geldiniz ekranında ve Filtre sekmesinde). `null` (henüz seçilmedi) → engelle. | ✅ |
| 13 | **Listeler** | Dökümanda anılmaz; mevcut kural korunur: liste (video > kanal; beyaz > kara > özel) **her zaman** konu/etiketi geçer. | ✅ (önceki karar) |
| 14 | **"Uyarı sayfası" eylemi** | Kartlarda: kartın üstü sebebi yazan opak bir kapakla kapanır (tıklanamaz). İzleme/kanal sayfasında: tam sayfa engel banner'ı. "Gizle" izleme sayfasında da banner üretir (sayfayı gizlemek mümkün olmadığı için). | ✅ |
| 15 | **Eylem sertliği** | Birden çok sebepte en sert eylem kazanır: **Gizle > Uyarı sayfası > Bulanıklaştır.** | ✅ |
| 16 | **Konu/etiket dışı engellerin görünümü** | Liste, bilinmeyen ve "izinli konu yok" kaynaklı engeller tek bir **genel eylem** kullanır (Görünüm sekmesi → "Diğer Engeller"). | ✅ |
| 17 | **PIN** | Yeni kurulumda PIN önceden **atanmaz**; ilk açılışta tam 4 rakamlı PIN seçtirilir ("Şimdilik atla" mümkün). Mevcut kullanıcıların `1234` PIN'i ve değiştirme uyarısı aynen kalır. PIN hâlâ yalnızca arayüz kilididir. | ✅ |
| 18 | **Asgari oy eşiği** | **En az 3 oy** (2026-10-21 onaylandı, 2026-10-22 teyit edildi). Eşik **toplam oy** (upvote + kararsız + downvote) üzerinden uygulanır: 3'ün altındaki konu/etiket değerlendirilmez, ana konu adayı olamaz ve `db.json`'a yazılmaz. Kararsız oylar eşiğe sayılır, net toplama sayılmaz. Sonuç: yeni kayıtlar konu/etiketlerinde 3 oya ulaşana kadar eklentide "bilinmeyen" kalır. Eklenti veriyi hazır alır; eşiği site uygular. | ✅ onaylandı |
| 19 | **Beyaz liste ve etiket koruması** | Beyaz liste, veritabanındaki "kumar" etiketli kanalı bile açar (tasarım gereği "asla engellenmez"). Bu davranış **mutlak** kabul edildi; etiket-bazlı koruma ("ebeveyn kilidi") 2.0.0 kapsamı dışında, ileride ayrı ayar olarak değerlendirilecek. | ✅ onaylandı (2026-10-21) |

### Ayar şemasındaki karşılıkları (`ayar`)
| Döküman | Alan |
|---|---|
| Mod | `mod`: `serbest` \| `orta` \| `kati` |
| Bilinmeyenler: izin ver / engelle | `etiketlenmemis`: `ac` \| `engelle` \| `null` |
| Kanal kontrol ayarı | `kanalKontrol`: `true` (varsayılan) \| `false` |
| Kararsızlar (Orta) | `kararsiz`: `izinver` (varsayılan) \| `engelle` |
| Konu başına filtreleme biçimi | `konular[id]`: `izinli` \| `kaldir` (Gizle) \| `bulaniklastir` \| `uyari` |
| Etiket başına filtreleme biçimi | `etiketler[id]`: aynı değerler |
| Diğer engellerin biçimi | `engelEylemi`: `kaldir` \| `bulaniklastir` \| `uyari` |
