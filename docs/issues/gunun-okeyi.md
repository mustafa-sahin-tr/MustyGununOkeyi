<!-- game: Günün Okeyi -->
<!-- repo: MustyGununOkeyi -->
# EPIC: Günün Okeyi — herkese aynı 14 taş, tek gönderim, en iyi diziliş
labels: epic, enhancement

## Neden
Sahibi (2026-10-14): Okey, Türkiye'de ve Almanya/Hollanda diasporasında 45+ kitlenin "ezbere bildiği" tek masa oyunu; mobildeki okey uygulamalarının tamamı çok oyunculu, sohbetli, zamanlı ve reklam yoğun. Tek kişilik, süresiz, günde bir el oynanan bir okey *bulmacası* yok. Wordle ritüeli (herkese aynı el, tek gönderim, "Ayşe 56" karşılaştırması) bu kitlede WhatsApp aile gruplarına doğrudan oturur: skor paylaşılır, çözüm paylaşılmaz, ertesi gün yine gelinir.

Araştırma raporu: Daily Drop epic'iyle aynı rapor (https://claude.ai/code/artifact/8ab0af47-8a63-4fac-8fd7-aad383f2fe81, Türkiye nişleri bölümü). Tasarım taslağı: "Günün Okeyi · Oyun" artboard'u (gösterge taşı, okey/sahte okey satırı, bulunan diziler çipleri, ahşap ıstakada 14 taş iki sıra, set üyelerinde renkli üst kenar, puan 41 / günün en iyisi 56 · Ayşe, Diziyi gönder, Yeniden diz, İpucu, reklam).

Portföydeki rolü: Türkiye nakit akışı oyunu (TR eCPM düşük ama kitle sadık ve DE/NL diasporası Avrupa eCPM'i getirir); Daily Drop'un günlük ritüel altyapısını (`DailySeed`, `DailyState`, `StreakService`, `ShareCardRenderer`, `FriendsService`) ikinci kez kullanan ve bunları `MustyCozyCore`'a taşıtan proje. Solo MVP 2–3 hafta.

## Hedef kitle ve konumlandırma
- 45+, Türkiye birincil; Almanya ve Hollanda'daki Türkçe konuşan diaspora ikincil; aile WhatsApp grupları dağıtım kanalı. Üçüncül: "rummikub puzzle" arayan EN kitle.
- Mağaza adı: **Günün Okeyi: Okey Bulmaca** (TR) / **Daily Okey: Rummy Tile Puzzle** (EN) / **Okey des Tages: Rummy Puzzle** (DE). Anahtar kelimeler: okey, okey bulmaca, günlük okey, okey oyunu tek kişilik, rummikub puzzle.
- Konumlandırma cümlesi: "Her gün herkese aynı 14 taş. Süre yok, rakip yok. En iyi dizilişi bul, puanını paylaş."
- Rakiplerden farkı: tek kişilik, süresiz, sohbetsiz; günde bir el; "en iyi mümkün puan" ile kendi kendine ölçülebilen bulmaca; büyük taş ve büyük yazı modu; oyun ekranında banner yok.

## Core loop ve kurallar (v1 kararları)
1. Oyuncu uygulamayı açar; "Günün Okeyi #N · <tarih>" kartı ve "Bugün herkes aynı taşları aldı" satırı görünür.
2. Taş seti: 1–13 × 4 renk (kırmızı, siyah, mavi, sarı) × 2 kopya = 104 taş + 2 sahte okey = 106. Gösterge: sahte okey dışındaki taşlardan biri (seed'den). Okey = göstergeyle aynı renkte, bir büyük sayı (gösterge 13 ise 1); iki kopyası da okeydir. Sahte okey: okey taşının yerine geçer, yani okeyin normal yüz değeri gibi oynanır (ör. gösterge kırmızı 5 → okey kırmızı 6 joker, sahte okey = kırmızı 6).
3. Günün eli: `seed = SHA256("gununokeyi:" + yyyy-MM-dd)` (oyuncunun yerel tarihi); gösterge çıkarıldıktan sonra kalan 105 taştan 14'ü. Çözücünün en iyi puanı 20'nin altındaysa el reddedilir ve `seed + 1` ile yeniden çekilir (her gün oynanabilir bir el).
4. Diziler: **seri** = aynı renk, ardışık ≥ 3 taş; 1 hem 13'ün ardından (12-13-1) hem başta (1-2-3) kullanılabilir, ama aynı seride iki rolde olamaz (13-1-2 geçersiz). **Per** = aynı sayı, farklı renk, 3–4 taş. Okey her iki dizide de herhangi bir taşın yerine geçer; bir dizide en fazla 2 okey. Çiftler modu v1'de yok.
5. Puan: dizideki her taşın sayı değeri toplamı; okey, yerine geçtiği taşın değerini alır; sahte okey okeyin yüz değerini alır. **Tam el** (14 taşın hepsi dizide) +50. Dizisiz taşlar 0, ceza yok. Puan tam sayı, deterministik.
6. Istaka: 2 sıra × 8 yuva (16 yuva, 14 taş, 2 boş yuva); boş yuva ve satır sonu grup ayracıdır. Her bitişik taş grubu otomatik olarak dizilere ayrılır (grup içinde en yüksek puanlı bölümleme); dizi üyelerinin üst kenarı renkli, HUD puanı canlı güncellenir. Taş 42×58 px (390 referans), büyük taş modunda 3 sıra × 6 yuva, 56×78 px.
7. Tek gönderim: günde bir "Diziyi gönder". Gönderimden sonra sonuç ekranı: oyuncunun puanı, çözücünün en iyi mümkün puanı ve en iyi diziliş (açılabilir), günün en iyisi (P2: dünya / arkadaş), seri, paylaşım kartı.
8. Ödüllü video (oyuncunun seçimi, günde birer kez): **İpucu** — çözücüden, oyuncunun henüz kurmadığı tek bir dizi ıstakada vurgulanır; **İkinci gönderim** — aynı elle bir kez daha gönderim, tabloya yüksek olan girer, kartta "2. gönderim" etiketi.
9. Pratik modu: sınırsız rastgele el, puan tabloya gitmez; her 3 elde bir interstitial, günde en fazla 3; No Ads (2,99 USD) tüm interstitial'ı kapatır, ödüllü videoyu kapatmaz. Günün eli ekranında ve Home'da hiçbir reklam yok; oyun ekranında banner yok.
10. Kozmetik taş setleri (1,99 USD): Klasik (ücretsiz), Ahşap, Kemik, Neon. Yalnızca görsel.
11. Süre yok; hiçbir ekranda sayaç yok. Reduce Motion, büyük yazı modu, Android geri tuşu, ana menü düğmesi; TR birincil, EN ve DE birinci sınıf çeviri.

## Kapsam dışı (v1)
- Çok oyunculu, sohbet, gerçek zamanlı masa; 101 Okey ve çiftler varyantları; tavla bulmacası (SONRA).
- Coin, enerji, booster envanteri; haftalık turnuva.
- Günün eli ekranında interstitial veya banner; yalnızca pratik modunda kotalı interstitial.
- iOS çıkışı MVP'de yok (Android soft launch sonrası).

## Başarı ölçütleri (kapılar)
- Soft launch (TR, DE, NL; Android; 14 gün): D1 ≥ %35, D7 ≥ %15, günlük gönderim oranı (gönderen / Home gören) ≥ %60, ortalama oturum ≥ 4 dk, çökme oranı < %0,5. Bilgi metriği: TR dışı kullanıcı payı (hedef ≥ %20; diasporanın gerçekten geldiğinin kanıtı), ipucu ödüllü video izlenme oranı.
- Kapı geçilirse: iOS build, No Ads fiyat testi, arkadaş ligi, Canlı.
- Kapı geçilmezse: iki hafta iterasyon (ıstaka hissi, otomatik algılama netliği, sonuç ekranı), yine geçilmezse rafa.

## Temalar ve story'ler (sıra)
**P1 · MVP**
- [ ] Günün Okeyi 1: Core paketini bağla ve proje iskeleti
- [ ] Günün Okeyi 2: Taş seti, gösterge/okey kuralı ve seed'li günlük el
- [ ] Günün Okeyi 3: Dizi doğrulama: seri, per, okey ikamesi ve 13-1 dönüşü
- [ ] Günün Okeyi 4: Puanlama ve tam el bonusu
- [ ] Günün Okeyi 5: Çözücü: en iyi mümkün puan ve diziliş
- [ ] Günün Okeyi 6: Istaka: sürükle-sırala, boşlukla gruplama ve yeniden diz
- [ ] Günün Okeyi 7: Otomatik dizi algılama, renkli kenar ve canlı puan
- [ ] Günün Okeyi 8: Günün eli durumu, tek gönderim ve gün değişimi
- [ ] Günün Okeyi 9: Sonuç ekranı: puanın, en iyi mümkün ve günün en iyisi
- [ ] Günün Okeyi 10: Paylaşım kartı ve seri
- [ ] Günün Okeyi 11: Ödüllü videoyla ipucu ve ikinci gönderim
- [ ] Günün Okeyi 12: Pratik modu ve interstitial kotası
- [ ] Günün Okeyi 13: İlk gün deneyimi ve kurallar ekranı
- [ ] Günün Okeyi 14: Analitik olay şeması ve kapı raporu
- [ ] Günün Okeyi 15: Mağaza sayfası, ASO ve soft launch yapılandırması
**P2 · Soft launch**
- [ ] Günün Okeyi 16: Dünya tablosu ve arkadaş ligi (FriendsService Core'a)
- [ ] Günün Okeyi 17: No Ads ve kozmetik taş setleri
- [ ] Günün Okeyi 18: Erişilebilirlik ve ayarlar: büyük taş modu, renk körü sembolleri
**P3 · Canlı**
- [ ] Günün Okeyi 19: Günlük hatırlatma bildirimi ve değerlendirme isteği
**SONRA**
- [ ] Günün Okeyi 20: 101 Okey varyantı
- [ ] Günün Okeyi 21: Çiftler modu
- [ ] Günün Okeyi 22: Haftalık turnuva
- [ ] Günün Okeyi 23: Tavla bulmacası (ayrı oyun, ortak çekirdek)

## Riskler ve açık sorular
- Kural varyantları: her masada farklı oynanan ayrıntılar (13-1-2 geçerli mi, sahte okeyin değeri, bir dizide iki okey) itiraz üretir. Kurallar `docs/okey-rules.md`'de tek yerde yazılır, kurallar ekranında gösterilir; "varyant tartışması" issue'su açık tutulur (çiftler, 101 puanlama).
- Otomatik dizi algılama "benim niyetimi anlamadı" hissi verebilir (K5-K6-K7-S7-M7 gibi çakışmalar). Çözüm: grup içinde en yüksek puanlı bölümleme + boşlukla elle ayırma; Play modunda 10 gerçek 45+ kullanıcıyla test.
- Kitle 45+: dokunma hedefleri ≥ 44 px, sürükleme toleransı yüksek, hiçbir sayaç yok; büyük taş modu ilk açılışta tek soruyla sunulur.
- TR eCPM düşük; gelir diasporadan ve No Ads'ten gelir. TR dışı pay %10'un altında kalırsa DE/NL mağaza metni ve ASO yeniden ele alınır.
- Yerel tarih seed'i: saat dilimi istismarı Daily Drop 9 ile aynı korumayla (sunucu saati farkı > 6 saat) sınırlanır.
- "Okey" adı ve taş görünümü: Okey taşları kamuya ait tasarım; mevcut uygulamaların logolarına benzememeli (ahşap ıstaka + krem taş + dört gerçek renk).

## Done when
P1 story'leri kapalı, soft launch 14 gün tamamlanmış, kapı raporu epic'e yorum olarak eklenmiş ve karar (Canlı / iterasyon / raf) yazılmış; `docs/okey-rules.md` kurallar ekranıyla birebir.

## STORY: Günün Okeyi 1: Core paketini bağla ve proje iskeleti
labels: tema:backend, P1, efor:M
milestone: MVP

**Günün Okeyi** epic'inin **1. story'si** (epic #EPIC). Tema: Altyapı.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** Unity projesi, `MustyCozyCore` UPM bağımlılığı, sahne düzeni, UGS kurulumu, Daily Drop'tan Core'a taşınan sınıflar

### Kullanıcı hikâyesi
Bir geliştirici olarak Günün Okeyi'nin ortak paketi (reklam, IAP, UGS, yerelleştirme, ayarlar, günlük ritüel sınıfları) ilk günden kullanmasını isterim, böylece Daily Drop'ta yazdığım hiçbir altyapıyı ikinci kez yazmam.

### Kabul kriterleri
- [ ] **Senaryo: Temiz kurulum** · Verilen boş bir Unity 2022 LTS projesi · Eğer `MustyCozyCore` paketi git URL ile eklenirse · O zaman proje derlenir ve `Bootstrap` sahnesi açılınca UGS oturumu (anonim Player Accounts) kurulur.
- [ ] **Senaryo: Sahne akışı** · Verilen `Bootstrap → Home → Play → Result` sahneleri · Eğer uygulama açılırsa · O zaman 2 sn içinde Home görünür; geri tuşu Home'da uygulamadan çıkar, diğer sahnelerde bir geri gider; Play'de geri tuşu eli bozmaz (Home'a döner, diziliş korunur).
- [ ] **Senaryo: Yerelleştirme** · Verilen TR, EN ve DE tabloları · Eğer cihaz dili TR, EN veya DE ise · O zaman tüm metinler o dilde, aksi halde EN; dil Ayarlar'dan değiştirilebilir.
- [ ] **Senaryo: Core'a taşınan sınıflar** · Verilen Daily Drop'taki `DailySeed`, `DailyState`, `MidnightWatcher`, `StreakService`, `ShareCardRenderer`, `ServerClock` · Eğer Core'a taşınıp oyun adıyla parametrelenirse · O zaman iki oyun da aynı sürümü derler ve Daily Drop'un Edit Mode testleri geçer.
- [ ] **Senaryo: Build** · Verilen Android build ayarları · Eğer `release` build alınırsa · O zaman paket adı `com.mtafasahin.gununokeyi`, min SDK 24, 64-bit, boyut < 50 MB.
- [ ] **Senaryo: Remote Config** · Verilen UGS Remote Config · Eğer `min_solver_score` anahtarı 20'den 25'e çekilirse · O zaman yeni oturumda değer uygulanır (yalnızca sonraki günün eline etki eder).

### Teknik görevler (Unity/C#)
- [ ] Repo iskeleti: `Assets/_Game/{Scripts,Prefabs,Art,Audio,Data}`, `.gitignore`, `docs/game-design.md` (bu epic'in kuralları), `docs/okey-rules.md` (kural varyantları ve v1 kararları), `docs/analytics.md`.
- [ ] `MustyCozyCore` bağımlılığı (UPM git URL, sürüm etiketi); Core'a taşıma PR'ı: `DailySeed(gameKey)`, `DailyState<TSnapshot>`, `MidnightWatcher`, `StreakService`, `ShareCardRenderer`, `ServerClock`, `AttemptPolicy`.
- [ ] Sahne yöneticisi: `SceneFlow` (Core, async yükleme, geçiş perdesi), `AppState` (ScriptableObject tabanlı oturum durumu).
- [ ] UGS: Player Accounts anonim giriş, Cloud Save, Analytics, Remote Config, Leaderboards projesi ve ortamları (`dev`, `prod`).
- [ ] Remote Config şeması: `min_solver_score`, `rack_rows`, `rack_cols`, `hint_rewarded_enabled`, `second_submit_enabled`, `practice_interstitial_every`, `practice_interstitial_daily_cap`, `show_par_in_game`.
- [ ] Yerelleştirme tabloları: TR kaynak, EN ve DE çeviri; okey terimleri sözlüğü (`docs/glossary.md`: gösterge/indicator/Anzeiger, per/set/Satz, seri/run/Reihe, sahte okey/false joker/falscher Okey).
- [ ] CI: GitHub Actions ile Android debug build; artefakt olarak APK.
- [ ] Kod stili ve analiz: `.editorconfig`, Roslyn analizörleri, `Assembly Definition` dosyaları (Game, Game.Rules (saf C#, Unity bağımsız), Game.Editor, Tests).

### Analitik
- `app_open` {first_open: bool, locale: string, country: string, build: int} — her açılışta
- `session_start` {day_index: int} — Home görünür olduğunda

### Notlar
- Karar: kural motoru (`Game.Rules` asmdef) Unity'ye bağımlı olmayan saf C#; aynı kod ileride Cloud Code doğrulamasında ve 101 Okey varyantında (SONRA) kullanılır.
- Core'a taşıma Daily Drop'u kırmamalı; taşıma PR'ı önce Daily Drop testleriyle doğrulanır. Core henüz bu sınıfları içermiyorsa bu story'nin eforu L olur.
- DE çevirisi: yerelleştirme ajansı değil, tablo çevirisi + diaspora test kullanıcısı okuması.

## STORY: Günün Okeyi 2: Taş seti, gösterge/okey kuralı ve seed'li günlük el
labels: tema:cekirdek, P1, efor:M
milestone: MVP

**Günün Okeyi** epic'inin **2. story'si** (epic #EPIC). Tema: Kurallar ve motor.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `Tile`, `TileSet`, `OkeyRule`, `DailyDeal`, `DayIndex`, HUD gösterge satırı

### Kullanıcı hikâyesi
Bir oyuncu olarak bugün dünyadaki herkesle aynı 14 taşı ve aynı göstergeyi aldığımı, okeyin hangi taş olduğunu ve sahte okeyin neye denk geldiğini net görmek isterim, böylece karşılaştırma adil ve kural açık olsun.

### Kabul kriterleri
- [ ] **Senaryo: Taş seti** · Verilen `TileSet.CreateStandard()` · Eğer sayılırsa · O zaman 106 taş vardır: 4 renk × 13 sayı × 2 kopya = 104 + 2 sahte okey; her taşın `(color, number, copy)` kimliği benzersizdir.
- [ ] **Senaryo: Okey belirleme** · Verilen gösterge kırmızı 5 · Eğer `OkeyRule.From(indicator)` çağrılırsa · O zaman okey = kırmızı 6 (iki kopya da), sahte okeyin yüz değeri kırmızı 6; gösterge siyah 13 ise okey siyah 1.
- [ ] **Senaryo: Gösterge sahte okey olamaz** · Verilen 10.000 farklı seed · Eğer gösterge üretilirse · O zaman hiçbiri sahte okey değildir ve gösterge taşı ele dağıtılmaz (el 105 taştan çekilir).
- [ ] **Senaryo: Aynı gün aynı el** · Verilen iki cihaz, aynı yerel tarih · Eğer ikisi de Play'e girerse · O zaman gösterge ve 14 taş (kopya numarası dahil) birebir aynıdır; sıralama da aynıdır (ıstakada ilk görünüm).
- [ ] **Senaryo: Farklı gün farklı el** · Verilen ardışık iki tarih · Eğer eller karşılaştırılırsa · O zaman en az 8 taş farklıdır.
- [ ] **Senaryo: Oynanabilir el garantisi** · Verilen çözücünün (5. story) en iyi puanı < `min_solver_score` (20) · Eğer el üretilirse · O zaman `seed + 1` ile yeniden çekilir (en fazla 20 deneme) ve HUD'da fark edilmez.
- [ ] **Senaryo: HUD gösterge satırı** · Verilen Play açıldı · Eğer HUD çizilirse · O zaman gösterge taşı (küçük, "Gösterge" etiketi), yanında "Okey: kırmızı 6" taş görseli ve "Sahte okey = kırmızı 6" satırı görünür; metin büyük yazı modunda kırpılmaz.
- [ ] **Senaryo: Gün numarası** · Verilen lansman tarihi `2026-11-23` · Eğer bugün `2026-11-24` ise · O zaman gün numarası #2 ve Home kartında "#2 · 24 Kasım" görünür.

### Teknik görevler (Unity/C#)
- [ ] `Tile` (readonly struct): `Color {Red, Black, Blue, Yellow}`, `Number 1–13`, `Copy 0/1`, `IsFalseJoker`; `TileId` (0–105) ve `ToString()` ("K6", "S13", "SO").
- [ ] `TileSet.CreateStandard()` ve `TileSet.Shuffle(IRandom)`; `OkeyRule {Indicator, OkeyFace, FalseJokerFace}` ve `IsOkey(Tile)`, `EffectiveFace(Tile)`.
- [ ] `DailyDeal.For(DateOnly, OkeyRule? ...)`: `DailySeed.For("gununokeyi", d)` → xoshiro256** (Core `Xoshiro256`, Daily Drop ile aynı sınıf) → Fisher-Yates karıştırma → gösterge (ilk sahte olmayan taş) → kalan 105'ten 14 taş; `rejectBelow` ile çözücü eşiği ve `seed + attempt` döngüsü.
- [ ] `DayIndex`: lansman tarihi sabiti; `DateTime.Now.Date` (yerel); gece yarısı geçişi 8. story'de.
- [ ] HUD `IndicatorRow` prefab: gösterge taşı 32×44 px, okey taşı görseli, sahte okey satırı; TR/EN/DE metin anahtarları.
- [ ] Edit Mode testleri (`Game.Rules.Tests`): set boyutu, okey belirleme (13 → 1), gösterge sahte değil, aynı tarih aynı el, farklı tarih farklı el, 10.000 el için ortalama okey sayısı ≈ 0,27 ± 0,02.
- [ ] `docs/okey-rules.md`: gösterge/okey/sahte okey kararları ve varyant notu.

### Analitik
- `daily_start` {day_index: int, seed_hash: string(8), okey_count: int, attempt: int} — günün eli ilk kez ıstakaya dizildiğinde

### Notlar
- Yerel tarih kararı Daily Drop ile aynı; saat dilimi istismarı 8. story'de sunucu saatiyle sınırlanır.
- Reddedilen el sayısı `seed + attempt` ile deterministik kaldığı için iki cihaz aynı sonuca varır; `min_solver_score` Remote Config'ten gün içinde değişse bile o günün eli değişmez (değer gün başında dondurulur ve `DailyState`'e yazılır).
- Açık soru (docs/okey-rules.md): sahte okeyin puanı okeyin yüz değeri mi (v1: evet) yoksa 0 mı?

## STORY: Günün Okeyi 3: Dizi doğrulama: seri, per, okey ikamesi ve 13-1 dönüşü
labels: tema:cekirdek, P1, efor:M
milestone: MVP

**Günün Okeyi** epic'inin **3. story'si** (epic #EPIC). Tema: Kurallar ve motor.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `Meld`, `MeldValidator`, `MeldKind`, `Game.Rules` testleri

### Kullanıcı hikâyesi
Bir oyuncu olarak ıstakaya koyduğum bir taş grubunun geçerli bir seri ya da per olup olmadığının masadaki kurallarla birebir değerlendirilmesini isterim, böylece "bu masada böyle oynanmaz" itirazı olmasın.

### Kabul kriterleri
- [ ] **Senaryo: Seri** · Verilen aynı renk ardışık 3+ taş (K4 K5 K6) · Eğer doğrulanırsa · O zaman `MeldKind.Run`, geçerli; K4 K6 K7 geçersiz (boşluk); K4 K5 K5 geçersiz (tekrar); 2 taş geçersiz; en fazla 13 taş.
- [ ] **Senaryo: Per** · Verilen aynı sayı farklı renk 3 veya 4 taş (K7 S7 M7) · Eğer doğrulanırsa · O zaman `MeldKind.Set`, geçerli; K7 K7 S7 geçersiz (aynı renk tekrar); 5 taş geçersiz.
- [ ] **Senaryo: 13-1 dönüşü** · Verilen K12 K13 K1 · Eğer doğrulanırsa · O zaman geçerli seri (1, 13'ün ardından 14 gibi); K13 K1 K2 geçersiz; K1 K2 K3 geçerli; K11 K12 K13 K1 geçerli; K12 K13 K1 K2 geçersiz (1 iki rolde olamaz).
- [ ] **Senaryo: Okey ikamesi** · Verilen okey = kırmızı 6 · Eğer K4 OKEY K6 doğrulanırsa · O zaman okey K5'in yerine geçer ve `Substitutions` listesinde `(okey, K5)` döner; K4 OKEY OKEY K7 geçerli (iki okey, K5 ve K6); OKEY OKEY K7 geçerli (3 taş, 2 okey); elde en fazla 2 okey olduğundan 3 okeyli dizi oluşamaz (doğrulayıcı yine de `TooManyJokers` döner).
- [ ] **Senaryo: Okey belirsizliği** · Verilen K4 K5 OKEY · Eğer doğrulanırsa · O zaman okey K6 veya K3'ün yerine geçebilir; doğrulayıcı en yüksek değerli ikameyi seçer (K6); 4. story bu kararı kullanır.
- [ ] **Senaryo: Okey taşı kendisi olarak** · Verilen okey kırmızı 6, grup K5 K6(okey) K7 · Eğer doğrulanırsa · O zaman okey yine joker sayılır ve en yüksek değerli ikameye gider (K6, çünkü doğal konumu); sonuç geçerli ve puan aynı.
- [ ] **Senaryo: Sahte okey** · Verilen sahte okey, okey kırmızı 6 · Eğer K5 SAHTE K7 doğrulanırsa · O zaman sahte okey K6 olarak değerlendirilir, geçerli; S5 SAHTE S7 geçersiz (sahte okey sarı 6 olamaz); M6 S6 SAHTE per geçerli (mavi, siyah, kırmızı 6); K6 S6 SAHTE per geçersiz (kırmızı 6 iki kez).
- [ ] **Senaryo: Performans** · Verilen 14 taşlık rastgele gruplar · Eğer 100.000 doğrulama yapılırsa · O zaman toplam süre < 200 ms (Edit Mode, Release) ve GC alloc 0 (span tabanlı).

### Teknik görevler (Unity/C#)
- [ ] `Meld` (readonly struct): `ReadOnlySpan<Tile>` veya `Tile[]` + `MeldKind {Run, Set}` + `Substitution[]` (okey → yerine geçtiği `Face`).
- [ ] `MeldValidator.TryValidate(ReadOnlySpan<Tile>, OkeyRule, out Meld)`: normalizasyon (sahte okey → yüz değeri, okey → joker), per yolu (renk kümesi + sayı eşitliği), seri yolu (renk eşitliği + sıralı doldurma; 1'i hem 1 hem 14 olarak deneyip geçerli ve en yüksek puanlı olanı seç).
- [ ] Joker yerleştirme: seri için boşlukları doldur, kalan jokerleri uçlara en yüksek değerli yere ekle (üst sınır 13, sonra 14=1 değil: 13'ten sonra 1 gelir, değeri 1); per için eksik renklerden herhangi biri (değer sayı ile aynı).
- [ ] `MeldKind`/`Face` tipleri; `Face` = (color, number) kopya bağımsız.
- [ ] Edit Mode testleri: tablo tabanlı (`[TestCase]`) en az 40 vaka; `docs/okey-rules.md`'deki her kural için bir test.
- [ ] Hata türleri: `MeldError {TooShort, Gap, Duplicate, MixedColors, TooManyJokers, WrapMisuse}` — 7. story kullanıcıya kısa ipucu gösterirken kullanır.
- [ ] Alloc'suz yol: `stackalloc` ve `ArrayPool<Tile>`; Profiler'da doğrulama.

### Analitik
- Yok (saf kural motoru; kullanım 7. story'de ölçülür).

### Notlar
- Karar (v1): 13-1 dönüşünde 1 yalnızca 13'ün ardından ve yalnızca bir kez; 13-1-2 geçersiz. Bu ayrıntı masalara göre değişir; `docs/okey-rules.md` "varyant tartışması" bölümüne yazıldı, kurallar ekranında (13. story) açıkça belirtilir.
- Karar (v1): bir dizide en fazla 2 okey (elde zaten en fazla 2 okey var; sahte okey joker değil).
- Doğrulayıcı tek grup için çalışır; 14 taşın tamamını bölümleme 5. (çözücü) ve 7. (otomatik algılama) story'lerindedir.

## STORY: Günün Okeyi 4: Puanlama ve tam el bonusu
labels: tema:cekirdek, P1, efor:S
milestone: MVP

**Günün Okeyi** epic'inin **4. story'si** (epic #EPIC). Tema: Kurallar ve motor.

**Öncelik:** P1 · **Efor:** S · **Milestone:** MVP · **Dokunduğu yer:** `Scorer`, `ScoreBreakdown`, HUD puan sayacı, `RackLayout` puan köprüsü

### Kullanıcı hikâyesi
Bir oyuncu olarak her dizinin kaç puan getirdiğini ve 14 taşın hepsini dizdiğimde tam el bonusunu anında görmek isterim, böylece hangi taşı nereye koymanın daha değerli olduğunu hissedeyim.

### Kabul kriterleri
- [ ] **Senaryo: Seri puanı** · Verilen K4 K5 K6 · Eğer puanlanırsa · O zaman 15; K11 K12 K13 K1 → 37 (1 yüz değeriyle).
- [ ] **Senaryo: Per puanı** · Verilen K7 S7 M7 Sa7 · Eğer puanlanırsa · O zaman 28.
- [ ] **Senaryo: Okey puanı** · Verilen K4 OKEY K6 · Eğer puanlanırsa · O zaman 15 (okey = K5 = 5); K4 K5 OKEY → 15 (okey K6 seçilir, K3 değil).
- [ ] **Senaryo: Sahte okey puanı** · Verilen okey kırmızı 6, K5 SAHTE K7 · Eğer puanlanırsa · O zaman 18 (sahte = 6).
- [ ] **Senaryo: Tam el** · Verilen 14 taşın hepsi geçerli dizilerde (ör. 3+3+4+4) · Eğer puanlanırsa · O zaman dizi toplamı + 50 ve `ScoreBreakdown.FullHand = true`; 13 taş dizide ise bonus yok.
- [ ] **Senaryo: Dizisiz taşlar** · Verilen 5 taş dizi dışında · Eğer puanlanırsa · O zaman bu taşlar 0 katkı, ceza yok; `Breakdown.LooseTiles = 5`.
- [ ] **Senaryo: Deterministik** · Verilen aynı diziliş · Eğer iki kez puanlanırsa · O zaman aynı int; float yok.
- [ ] **Senaryo: HUD** · Verilen puan 41'den 56'ya çıktı · Eğer HUD güncellenirse · O zaman sayaç 250 ms'de sayar, %105 pop; tam elde "Tam el +50" rozeti ve Medium haptik; Reduce Motion'da anında.

### Teknik görevler (Unity/C#)
- [ ] `Scorer.Score(IReadOnlyList<Meld>, int totalTiles)` → `ScoreBreakdown {MeldScores[], MeldTotal, FullHandBonus, Total, LooseTiles}`; `ScoreTable` SO (`fullHandBonus` 50, Remote Config `full_hand_bonus`).
- [ ] Okey değeri `Meld.Substitutions`'tan okunur (3. story); sahte okey `OkeyRule.FalseJokerFace.Number`.
- [ ] HUD `ScoreView` (sayaç tween, pop), `FullHandBadge`; Core `Haptics.Medium`.
- [ ] Edit Mode testleri: tablo tabanlı 15 vaka; tam el sınırı (13 vs 14 taş).
- [ ] `docs/okey-rules.md` puanlama bölümü ve 101 puanlama farkı notu (SONRA 20).

### Analitik
- `score_update` yok (gürültü); `submit` olayında `score`, `meld_count`, `full_hand`, `loose_tiles` gider (8. story).

### Notlar
- Tam el bonusu 50: 14 taşlık elde tipik dizi toplamı 40–90; bonus "hepsini dizmeyi" hedef yapar ama tek dizi toplamını ezmez. Play modunda ve soft launch dağılımıyla ayarlanır.
- Ceza yok kararı: 45+ kitlede "eksi puan" cesaret kırar; 101 varyantında (SONRA) ceza vardır.

## STORY: Günün Okeyi 5: Çözücü: en iyi mümkün puan ve diziliş
labels: tema:cekirdek, P1, efor:L
milestone: MVP

**Günün Okeyi** epic'inin **5. story'si** (epic #EPIC). Tema: Kurallar ve motor.

**Öncelik:** P1 · **Efor:** L · **Milestone:** MVP · **Dokunduğu yer:** `Solver`, `SolverResult`, `DailyDeal` el eşiği, Result "en iyi mümkün", ipucu (11. story)

### Kullanıcı hikâyesi
Bir oyuncu olarak gönderimden sonra bu elden en fazla kaç puan çıkabileceğini ve nasıl dizileceğini görmek isterim, böylece bulmacayı "çözdüm mü" hissi olsun; geliştirici olarak da aynı çözücüyle oynanabilir el garantisi ve ipucu üretmek isterim.

### Kabul kriterleri
- [ ] **Senaryo: Doğruluk** · Verilen `docs/solver-cases.md`'deki 30 el (elle çözülmüş) · Eğer `Solver.Solve` çalıştırılırsa · O zaman her biri için en iyi puan ve geçerli bir diziliş döner; hiçbir vakada daha yüksek puanlı diziliş bulunamaz (kaba kuvvet çapraz kontrol testi ≤ 10 taş için).
- [ ] **Senaryo: Süre** · Verilen 1.000 rastgele el (okeyli/okeysiz, 14 taş) · Eğer çözülürse · O zaman her biri < 100 ms, ortalama < 15 ms (Snapdragon 680 sınıfı cihazda Release build); en kötü el için `docs/solver-cases.md`'ye not.
- [ ] **Senaryo: Okey ikamesi** · Verilen 2 okeyli el · Eğer çözülürse · O zaman okeyler en yüksek toplamı veren yere yerleşir ve `SolverResult.Melds[i].Substitutions` doludur.
- [ ] **Senaryo: Tam el tercihi** · Verilen tam el mümkün olan el · Eğer çözülürse · O zaman bonus dahil karşılaştırma yapılır (dizi toplamı düşük ama tam el +50 ile daha yüksekse tam el seçilir).
- [ ] **Senaryo: Eşitlik** · Verilen aynı puanlı iki diziliş · Eğer çözülürse · O zaman daha az dizi (daha büyük gruplar) tercih edilir; sonuç deterministiktir (cihazdan bağımsız aynı diziliş).
- [ ] **Senaryo: Ana iş parçacığı** · Verilen Result açılışı · Eğer çözücü çalışırsa · O zaman UI donmaz (arka plan `Task` ya da el başında önbellek); sonuç `DailyState.parScore` olarak kalıcıdır.
- [ ] **Senaryo: Kısmi çözüm (ipucu)** · Verilen oyuncunun mevcut dizileri · Eğer `Solver.SuggestOne(hand, currentMelds)` çağrılırsa · O zaman en iyi dizilişte olup oyuncunun henüz kurmadığı tek bir dizi döner; yoksa `null`.

### Teknik görevler (Unity/C#)
- [ ] `Solver` (`Game.Rules`, saf C#): el → aday dizi üretimi (tüm geçerli seriler ve perler, okeyli varyantlarıyla; 14 taş için tipik aday sayısı 20–120) → bitmask (14 bit) üzerinde DP: `best[mask] = max(best[mask], best[mask ^ meldMask] + meldScore)`; tam el bonusu `mask == full` için eklenir; geri izleme ile diziliş.
- [ ] Okey ikamesi: adaylar üretilirken okey(ler) her olası yüze yerleştirilir (aynı aday için farklı ikameler farklı puan → en yüksek olan tutulur; ikame puanı `Meld.Substitutions` üzerinden).
- [ ] Budama: aday dizi aynı taş kümesiyle iki kez üretilmez (`HashSet<ushort>`); DP tablo boyutu 2^14 = 16.384 × int.
- [ ] `SolverResult {BestScore, Melds, FullHand, ElapsedMs}`; `SuggestOne` (en iyi dizilişten, oyuncunun dizileriyle kesişmeyen ilk dizi; kesişiyorsa oyuncunun en düşük puanlı dizisini bozmayan alternatif).
- [ ] `DailyDeal` entegrasyonu: el üretiminde çözücü eşiği (2. story); sonuç `DailyState.parScore` ve `parMelds` olarak yazılır.
- [ ] Kaba kuvvet referans çözücü (test projesinde, ≤ 10 taş) ve 30 elle çözülmüş vaka; benchmark testi (BenchmarkDotNet değil, basit `Stopwatch` + Release Edit Mode).
- [ ] Profiler: alloc ölçümü, `ArrayPool`; çözüm `Task.Run` ile arka planda, `CancellationToken` ile gün değişiminde iptal.

### Analitik
- `solver_run` {day_index: int, best_score: int, elapsed_ms: int, candidates: int} — günün eli için ilk çözümde (cihaz başına bir kez)

### Notlar
- Risk: taş kümesi aynı olup okey ikamesi farklı olan adaylarda puan farkı çıkabilir (K4 K5 OKEY: 15 vs 12); aday üretiminde kümeye en iyi ikame yazılır, bu nedenle DP doğruluğu korunur. Edge: iki okey + 13-1 dönüşü kombinasyonları test vakalarında.
- Kaba kuvvet çapraz kontrol ≤ 10 taşla sınırlı; 14 taş için 30 elle çözülmüş vaka yeterli sayılır.
- "En iyi mümkün" oyunda gösterilmez (`show_par_in_game` false); yalnızca gönderimden sonra. Oyun içinde gösterilmesi A/B'ye açık.

## STORY: Günün Okeyi 6: Istaka: sürükle-sırala, boşlukla gruplama ve yeniden diz
labels: tema:cekirdek, P1, efor:L
milestone: MVP

**Günün Okeyi** epic'inin **6. story'si** (epic #EPIC). Tema: Istaka ve kontrol.

**Öncelik:** P1 · **Efor:** L · **Milestone:** MVP · **Dokunduğu yer:** `Play` sahnesi, `RackView`, `RackSlot`, `TileView`, `DragController`, `RackLayout` modeli

### Kullanıcı hikâyesi
Bir oyuncu olarak taşları gerçek ıstakadaki gibi parmağımla kaydırıp sıralamak, araya boşluk bırakarak grupları ayırmak ve istersem baştan dizmek isterim, böylece düşünmek yerine dokunarak bulmaca çözeyim.

### Kabul kriterleri
- [ ] **Senaryo: Istaka düzeni** · Verilen 390×844 referans ekran · Eğer Play açılırsa · O zaman ahşap ıstaka 2 sıra × 8 yuva (16 yuva), 14 taş + 2 boş yuva; taş 42×58 px, yuva aralığı 4 px; ıstaka ekranın alt %40'ında, HUD üstte.
- [ ] **Senaryo: İlk diziliş** · Verilen günün eli · Eğer ilk kez açılırsa · O zaman taşlar `DailyDeal` sırasıyla (karışık) dizilir; boş yuvalar her sıranın sonunda.
- [ ] **Senaryo: Sürükleme** · Verilen bir taşa basıldı · Eğer parmak 8 px hareket ederse · O zaman taş kalkar (%110 ölçek, gölge), parmağı izler; bırakınca en yakın yuvaya oturur, aradaki taşlar kayar (ekleme, değiştirme değil); 150 ms tween, Reduce Motion'da anında.
- [ ] **Senaryo: Boşluk ayracı** · Verilen K4 K5 K6 _ S7 M7 Sa7 · Eğer dizilirse · O zaman boş yuva iki grubu ayırır; bir taş boş yuvaya bırakılınca boşluk kaymaz, yuva dolar; boşluk kalmadıysa satır sonu tek ayraçtır.
- [ ] **Senaryo: Boş yuva taşıma** · Verilen 2 boş yuva · Eğer oyuncu boş yuvaya uzun basıp sürüklerse · O zaman boşluk taşınır (grup ayracını yeniden konumlandırma); kısa basmada hiçbir şey olmaz.
- [ ] **Senaryo: Satırlar arası** · Verilen üst satır dolu · Eğer taş alt satıra sürüklenirse · O zaman alt satırda yer açılır ve taşan taş üst satırın sonuna geçmez (toplam yuva 16 sabit; kaydırma hedef satır içinde kalır, hedef satır doluysa bırakma iptal ve taş geri döner).
- [ ] **Senaryo: Yeniden diz** · Verilen değiştirilmiş ıstaka · Eğer "Yeniden diz" basılırsa · O zaman iki seçenek: "Renge göre" (renk, sonra sayı) ve "Sayıya göre"; 300 ms yeniden akış; "Geri al" 1 adım (son yeniden diz veya son sürükleme).
- [ ] **Senaryo: Dokunma toleransı** · Verilen 45+ kullanıcı · Eğer parmak taşın 6 px dışına basarsa · O zaman taş yine yakalanır (dokunma alanı 54×70 px); sürükleme sırasında UI düğmeleri tepki vermez.
- [ ] **Senaryo: Kaldığı yer** · Verilen yarım diziliş, uygulama kapatıldı · Eğer 2 saat sonra açılırsa · O zaman 16 yuvanın içeriği birebir geri gelir.

### Teknik görevler (Unity/C#)
- [ ] `RackLayout` modeli (`Game.Rules`): `TileId?[16]` (null = boş), `Move(from, to)` (ekleme semantiği, satır içi kaydırma), `MoveGap`, `SortByColor`, `SortByNumber`, `Groups()` → ardışık dolu yuva grupları (satır sonu ve boşluk ayraç); Edit Mode testleri (20 vaka).
- [ ] `RackView` (uGUI + TextMeshPro): 16 `RackSlot`, `GridLayout` yerine elle konumlandırma (tween için); `TileView` prefab (sprite + sayı + renk; set üyesi üst kenar 7. story).
- [ ] `DragController`: Unity Input System `Pointer`; durum {Idle, Pressed, Dragging, Dropping}; 8 px eşik; en yakın yuva hesabı (`Vector2.Distance`, satır önce); uzun basma 400 ms boşluk taşıma; `EventSystem.IsPointerOverGameObject` ile UI önceliği.
- [ ] Yuvadaki taşlar "kayar": hedef yuva doluysa, kaynak ile hedef arasındaki taşlar bir yuva kayar (satır içinde); satır sınırı aşılırsa bırakma iptal.
- [ ] `UndoStack` (1 adım, Remote Config `undo_depth`), "Yeniden diz" sheet'i (2 seçenek + Geri al).
- [ ] Kalıcılık: `RackLayout` `DailyState.snapshot` içinde (Core `DailyState<TSnapshot>`); her sürükleme sonunda ve `OnApplicationPause` kayıt (küçük JSON, < 200 B).
- [ ] Ses ve haptik: taş kalkma (tık), oturma (tahta "tak"), Core `Haptics.Light`; sesler gerçek okey taşı kaydı ailesinden.
- [ ] Play Mode testi: sürükle-bırak ile `RackLayout` değişimini doğrulayan otomasyon (InputTestFixture).

### Analitik
- `tile_move` {day_index: int, n: int} — yalnızca ilk 10 hamle ve her 10'da bir (hacim sınırı)
- `rack_sort` {mode: "color"|"number"} · `undo` {} — her kullanımda

### Notlar
- Karar: ekleme semantiği (araya sokma), değiştirme değil; gerçek ıstaka hissi. Değiştirme semantiği Play modunda alternatif olarak denenir (Remote Config `drag_mode`).
- Büyük taş modu (3 sıra × 6 yuva = 18 yuva, 56×78 px) 18. story'de; `RackLayout` satır/sütun parametrik yazılır (`rack_rows`, `rack_cols`).
- Tüm ölçüler Play modunda ayarlanır; 45+ kullanıcıyla ilk dokunma testi bu story kapanmadan.

## STORY: Günün Okeyi 7: Otomatik dizi algılama, renkli kenar ve canlı puan
labels: tema:cekirdek, P1, efor:M
milestone: MVP

**Günün Okeyi** epic'inin **7. story'si** (epic #EPIC). Tema: Istaka ve kontrol.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `MeldDetector`, `RackLayout.Groups`, `TileView` kenar vurgusu, HUD "Bulunan diziler" çipleri, `ScoreView`

### Kullanıcı hikâyesi
Bir oyuncu olarak taşları yan yana koyduğumda dizinin oluştuğunu renkli kenardan anında görmek ve puanın canlı değişmesini isterim, böylece "gönder" demeden önce ne kadar topladığımı bileyim.

### Kabul kriterleri
- [ ] **Senaryo: Algılama** · Verilen K4 K5 K6 yan yana · Eğer üçüncü taş oturursa · O zaman 100 ms içinde üç taşın üst kenarı yeşil (seri rengi), HUD'da "Seri 15" çipi belirir ve puan 15 artar.
- [ ] **Senaryo: Grup içi bölümleme** · Verilen tek grup K4 K5 K6 K7 S7 M7 · Eğer algılanırsa · O zaman en yüksek puanlı bölümleme seçilir: K4 K5 K6 (15) + K7 S7 M7 (21) = 36; K4 K5 K6 K7 (22) + 2 boş (0) = 22 değil; farklı diziler farklı kenar rengi (seri yeşil, per mavi), aynı gruptaki ardışık diziler arasında 2 px kenar boşluğu.
- [ ] **Senaryo: Okey vurgusu** · Verilen K4 OKEY K6 · Eğer algılanırsa · O zaman okey taşının üstünde küçük "K5" etiketi (neyin yerine geçtiği) görünür.
- [ ] **Senaryo: Geçersiz grup ipucu** · Verilen K4 K6 K7 (boşluklu) · Eğer grup oluşursa · O zaman kenar vurgusu yok; gruba dokununca 1,5 sn balon: "5 eksik" (MeldError.Gap); balon girdiyi kilitlemez.
- [ ] **Senaryo: Kısmi grup** · Verilen 5 taşlık grupta 3'ü dizi · Eğer algılanırsa · O zaman yalnızca 3 taş vurgulanır, kalan 2 nötr.
- [ ] **Senaryo: Tam el** · Verilen 14 taşın hepsi vurgulu · Eğer son dizi oluşursa · O zaman HUD "Tam el +50" rozeti, ıstaka kenarı altın parıltısı 600 ms (Reduce Motion'da yalnızca rozet).
- [ ] **Senaryo: Renk körü** · Verilen Ayarlar'da "Sembol ipuçları" açık · Eğer taşlar çizilirse · O zaman her renk için sayı yanında küçük sembol (kırmızı ●, siyah ■, mavi ▲, sarı ◆) ve kenar vurgusu desen farkıyla (seri düz, per kesikli).
- [ ] **Senaryo: Performans** · Verilen her sürükleme bırakması · Eğer algılama çalışırsa · O zaman < 5 ms ana iş parçacığı, GC alloc 0.

### Teknik görevler (Unity/C#)
- [ ] `MeldDetector.Detect(RackLayout, OkeyRule)` (`Game.Rules`): her grup için bitişik alt dizilerin (uzunluk 3–13) `MeldValidator` ile doğrulanması ve grup içi DP (`best[i] = max(best[i-1], best[j] + score(j..i))`, sıra korunur) → `DetectedMeld[] {slotRange, Meld, score}`; toplam `Scorer`'a gider.
- [ ] `TileView` kenar vurgusu: üst 4 px şerit (`Image`, renk + desen), okey etiketi (`TMP` 10 pt), geçiş 100 ms.
- [ ] HUD `MeldChips`: yatay kaydırmalı çipler ("Seri 15", "Per 21", "Tam el +50"), dokununca ilgili yuvalar 300 ms yanıp söner.
- [ ] `MeldError` → yerelleştirilmiş ipucu metinleri (TR/EN/DE), `HintBubble` (Core) 1,5 sn.
- [ ] Sembol ipuçları atlası (4 sembol), `shape_hints` ayarı (18. story bağlar; bu story'de bayrak okunur).
- [ ] Edit Mode testleri: bölümleme vakaları (çakışan seri/per, kısmi grup, okey etiketi); Play Mode: kenar vurgusu güncellemesi.

### Analitik
- `meld_detected` {kind: "run"|"set", size: int, with_okey: bool} — yeni dizi oluştuğunda (gün başına en fazla 30)
- `meld_hint_shown` {error: string} — geçersiz grup balonunda

### Notlar
- Risk: oyuncunun niyetiyle en yüksek puanlı bölümleme çelişirse (K5 K6 K7 S7 M7: seri mi per mi?) oyuncu boşlukla ayırarak zorlar; bu durum `meld_hint_shown` olmadan da `rack_sort`/`tile_move` desenlerinden izlenir. 45+ testinde "neden yanmadı" soruları bu story'nin kabul ölçüsüdür.
- Sıra korunur: algılama ıstakadaki fiziksel sırayı yeniden sıralamaz; taşları oyuncu dizer, motor yalnızca okur.

## STORY: Günün Okeyi 8: Günün eli durumu, tek gönderim ve gün değişimi
labels: tema:ritual, P1, efor:M
milestone: MVP

**Günün Okeyi** epic'inin **8. story'si** (epic #EPIC). Tema: Günlük el.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `DailyState` (Core), `SubmitPolicy`, Home kartı, `MidnightWatcher` (Core), `ServerClock` (Core), "Diziyi gönder" akışı

### Kullanıcı hikâyesi
Bir oyuncu olarak uygulamayı açınca bugünün elini gönderip göndermediğimi, göndermediysem kaldığım dizilişi, gönderdiysem puanımı ve yarına kalan süreyi görmek isterim; gönderimin günde bir olduğunu bilmek isterim, böylece puanım anlamlı olsun.

### Kabul kriterleri
- [ ] **Senaryo: Oynanmadı** · Verilen bugün hiç hamle yok · Eğer Home açılırsa · O zaman kart "Günün Okeyi #N · <tarih> · Oyna" durumunda, Oyna birincil, altında "Bugün herkes aynı taşları aldı".
- [ ] **Senaryo: Devam** · Verilen bugün yarım diziliş · Eğer Home açılırsa · O zaman kart "Devam et · 41 puan" ve Play aynı ıstakayla açılır.
- [ ] **Senaryo: Gönderim** · Verilen Play'de "Diziyi gönder" · Eğer basılırsa · O zaman onay sheet'i ("41 puan · 3 dizi · 5 taş dışarıda. Gönderilsin mi?"), onayda `status = Submitted`, `submittedScore` yazılır, Result açılır; iptalde hiçbir şey değişmez.
- [ ] **Senaryo: Gönderildi** · Verilen bugün gönderilmiş · Eğer Home açılırsa · O zaman kart puanı, "Sonucu gör" ve "Yarına 06:12:33" geri sayımını gösterir; Oyna devre dışı; Play yalnızca salt okunur açılır (diziliş görünür, sürükleme kapalı, "Gönderildi" rozeti).
- [ ] **Senaryo: İkinci gönderim yok** · Verilen gönderilmiş, ödüllü ikinci hak kullanılmış · Eğer Oyna denenirse · O zaman düğme devre dışı, "Yarın 00:00 yeni el" ipucu.
- [ ] **Senaryo: Gece yarısı Home'da** · Verilen uygulama açık, Home'da · Eğer yerel saat 00:00'ı geçerse · O zaman kart 1 sn içinde yeni güne döner (gün numarası +1, yeni el).
- [ ] **Senaryo: Gece yarısı Play'de** · Verilen gönderilmemiş diziliş · Eğer gün değişirse · O zaman diziliş kesilmez; gönderilince önceki güne yazılır; Result'ta "Dünkü el" rozeti; sonraki açılışta yeni el.
- [ ] **Senaryo: Saat ileri alma** · Verilen cihaz saati 1 gün ileri · Eğer Home açılırsa · O zaman sunucu saati (UGS) ile fark > 6 saat ise "Saat ayarı farklı, bugünün eli sunucu saatine göre" uyarısı ve `dayIndex` sunucu tarihinden hesaplanır (çevrimdışıysa yerel kabul).
- [ ] **Senaryo: Veri kaybı** · Verilen uygulama verisi silindi · Eğer yeniden açılırsa · O zaman Cloud Save'den bugünün durumu (status, submittedScore, rewarded bayrakları) gelir; snapshot yoksa ve `Submitted` ise tekrar oynanamaz.

### Teknik görevler (Unity/C#)
- [ ] `DailyState<RackSnapshot>` (Core): {dayIndex, status: NotPlayed|InProgress|Submitted, submitCount, submittedScore, bestScore, parScore, hintUsed, secondSubmitUsed, snapshot}; yerel JSON + Cloud Save anahtarı `daily/{dayIndex}`.
- [ ] `RackSnapshot`: `TileId?[16]`, okey kuralı özeti, hamle sayısı; kayıt her hamlede (debounce 500 ms) ve `OnApplicationPause`.
- [ ] `SubmitPolicy.CanSubmit(state)`: `MaxSubmits = 1 + (secondSubmitUsed ? 1 : 0)`; `SubmitFlow`: onay sheet'i, `MeldDetector` sonucu dondurma, `DailyState` yazımı, Result'a geçiş; Leaderboard gönderimi 16. story'de bu akışa bağlanır.
- [ ] `MidnightWatcher` (Core): her 30 sn yerel tarih karşılaştırması; `DayChanged` olayı; Play'de dinlenir ama eli bozmaz.
- [ ] `ServerClock` (Core): UGS Cloud Code `GetTime` veya HTTP `Date`; önbellek 6 saat; `clock_drift_s`.
- [ ] Home kartı UI: üç durum, geri sayım `TimeSpan`, gün numarası ve tarih (yerel biçim), "herkes aynı taşları aldı" satırı.
- [ ] Play salt okunur modu: `DragController.enabled = false`, "Gönderildi" rozeti, "Sonucu gör" düğmesi.

### Analitik
- `home_view` {status: string, day_index: int} — Home her açıldığında
- `submit` {day_index: int, attempt: int, score: int, par_score: int, meld_count: int, full_hand: bool, loose_tiles: int, moves: int, duration_s: int} — her gönderimde
- `clock_drift_detected` {drift_s: int} — eşik aşıldığında

### Notlar
- Kapı metriği: günlük gönderim oranı = `submit` (attempt 1) / `home_view` (günlük benzersiz) ≥ %60.
- Karar: yarım bırakılan el gün bitince `Submitted` olmaz; kart "Yarım kaldı" gösterir, seri kırılır (10. story), tabloya girmez.
- Cloud Save çakışması: cihazlar arasında en yüksek `submitCount` ve `bestScore` kazanır; snapshot cihazda kalır.

## STORY: Günün Okeyi 9: Sonuç ekranı: puanın, en iyi mümkün ve günün en iyisi
labels: tema:ritual, P1, efor:M
milestone: MVP

**Günün Okeyi** epic'inin **9. story'si** (epic #EPIC). Tema: Sonuç ve karşılaştırma.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `Result` sahnesi, `ResultView`, `ResultViewModel`, `ParRevealView`

### Kullanıcı hikâyesi
Bir oyuncu olarak gönderimden sonra puanımı, bu elden çıkabilecek en iyi puanı ve en iyi dizilişi, günün en iyisini ve serimi tek ekranda görmek isterim, böylece "ne kadar yaklaştım" hissiyle yarın yine gelmek isteyeyim.

### Kabul kriterleri
- [ ] **Senaryo: İçerik** · Verilen gönderilmiş el · Eğer Result açılırsa · O zaman başlık "Günün Okeyi #N", puan (büyük, "41"), altında "En iyi mümkün 56" ve yüzde çubuğu (41/56 = %73), Seri / En iyi puanın / Dizi sayısı kutuları, "En iyi dizilişi gör" (ikincil), Paylaş (birincil), ödüllü ikinci gönderim (hak varsa), günün en iyisi satırı ("Ayşe 56" — P2'de dolar, MVP'de "Dünya tablosu yakında"), Ana menü düğmesi.
- [ ] **Senaryo: Par'a eşit** · Verilen puan = en iyi mümkün · Eğer Result açılırsa · O zaman "Eli çözdün!" kurdelesi, Heavy haptik ve 1 sn konfeti (Reduce Motion'da yalnızca kurdele); yüzde çubuğu altın.
- [ ] **Senaryo: En iyi dizilişi gör** · Verilen "En iyi dizilişi gör" · Eğer basılırsa · O zaman salt okunur bir ıstaka çözücünün dizilişini gösterir; oyuncunun da kurduğu diziler yeşil tik, kaçırdıkları turuncu kenar; "Senin dizilişin" ile geçiş (toggle).
- [ ] **Senaryo: Giriş animasyonu** · Verilen Result açılışı · Eğer Reduce Motion kapalıysa · O zaman puan 0'dan 600 ms'de sayar, sonra "En iyi mümkün" 300 ms gecikmeyle belirir, kutular 80 ms arayla; açıksa anında.
- [ ] **Senaryo: Yeni rekor** · Verilen önceki en iyi puan · Eğer bugünkü daha yüksekse · O zaman "Yeni rekor" rozeti (ele bağlı olduğu için küçük; asıl ölçüt par yüzdesi: "En iyi yüzden" ayrıca saklanır).
- [ ] **Senaryo: Geri** · Verilen Result · Eğer geri tuşuna basılırsa · O zaman Home'a dönülür; Result Home kartından tekrar açılabilir.
- [ ] **Senaryo: Yarım kalan gün** · Verilen gün gönderilmeden bitti · Eğer Result açılırsa · O zaman "Yarım kaldı" başlığı, o anki puan ve en iyi mümkün gösterilir, paylaşım kapalı.
- [ ] **Senaryo: Sayı biçimi** · Verilen yüzde ve puanlar · Eğer gösterilirse · O zaman yerel biçim (%73 / 73%), TR/EN/DE.

### Teknik görevler (Unity/C#)
- [ ] `ResultViewModel` (DailyState + SolverResult + StreakService + RunStats {moves, durationS, meldCount, fullHand}); `parPercent = floor(100 × score / parScore)`.
- [ ] `ResultView` (uGUI + TextMeshPro): sayaç ve belirme tween'leri, Reduce Motion dalı; "Ana menü" her zaman görünür.
- [ ] `ParRevealView`: salt okunur `RackView` örneği; çözücü dizilişi `RackLayout`'a yerleştirme (`Solver.ToRackLayout`, diziler boşlukla ayrılmış, 16 yuvaya sığmazsa 3. satır görünümü); oyuncu dizileriyle küme karşılaştırması (taş kümesi eşitliği).
- [ ] Düğme durumları: ikinci gönderim yalnızca `submitCount == 1 && !secondSubmitUsed && rewardedAvailable` (11. story).
- [ ] Günün en iyisi satırı: placeholder; 16. story `LeaderboardService` ile doldurur.
- [ ] `BestParPercent` istatistiği (Cloud Save) ve "En iyi yüzden" kutusu.

### Analitik
- `result_view` {day_index: int, attempt: int, score: int, par_score: int, par_percent: int, full_hand: bool} — her açılışta
- `par_reveal` {day_index: int, missed_melds: int} — "En iyi dizilişi gör" dokunulduğunda

### Notlar
- Tasarım taslağı: "Günün Okeyi · Oyun" artboard'unun sonuç varyantı; Paylaş tek sarı vurgu, "En iyi dizilişi gör" ikincil.
- Karar: en iyi diziliş yalnızca gönderimden sonra; ikinci gönderim hakkı kullanılacaksa çözüm gösterilmez mi? v1: gösterilir ama ikinci gönderimde kartta "2. gönderim" etiketi ve tabloya ayrı işaret (16. story) — "çözümü gördükten sonra gönderdi" şeffaflığı.

## STORY: Günün Okeyi 10: Paylaşım kartı ve seri
labels: tema:paylasim, P1, efor:M
milestone: MVP

**Günün Okeyi** epic'inin **10. story'si** (epic #EPIC). Tema: Sonuç ve karşılaştırma.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `ShareCardRenderer` (Core), `ShareService` (Core), `StreakService` (Core), `CountdownView`, Result Paylaş düğmesi

### Kullanıcı hikâyesi
Bir oyuncu olarak tek dokunuşla puanımı ve "en iyi mümkün"e ne kadar yaklaştığımı WhatsApp aile grubuna göndermek isterim; kaç gündür üst üste oynadığımı görmek isterim, böylece grupta küçük bir yarış olsun ve yarın yine geleyim.

### Kabul kriterleri
- [ ] **Senaryo: Görsel kart** · Verilen Result · Eğer Paylaş'a basılırsa · O zaman 1080×1350 PNG üretilir: "Günün Okeyi #N · <tarih>", puan büyük, "En iyi mümkün 56 · %73", gösterge taşı görseli, dizi sayısı (taşlar değil; çözüm sızmaz), seri, mağaza kısa linki; oyuncu adı yok.
- [ ] **Senaryo: Metin** · Verilen paylaşım · Eğer sistem paylaşım sayfası açılırsa · O zaman metin: "Günün Okeyi #142 · 41 puan (en iyi 56, %73) · 3 dizi · 6 gün seri\n<link>" (TR/EN/DE); emoji yok (45+ kitle, WhatsApp'ta sade).
- [ ] **Senaryo: Çözüm sızmaz** · Verilen kart ve metin · Eğer incelenirse · O zaman eldeki taşlar ve diziler yoktur; yalnızca gösterge görünür (herkes zaten aynı göstergeyi görür).
- [ ] **Senaryo: Platform** · Verilen Android · Eğer paylaşılırsa · O zaman `ACTION_SEND` ile PNG + metin birlikte gider; WhatsApp hedefinde metin ayrıca panoya kopyalanır (WhatsApp görsel+metin birlikte her sürümde taşımaz).
- [ ] **Senaryo: Hız** · Verilen orta sınıf cihaz · Eğer Paylaş'a basılırsa · O zaman kart 300 ms içinde hazır.
- [ ] **Senaryo: Seri artışı** · Verilen dün gönderildi · Eğer bugün gönderilirse · O zaman seri +1; dün gönderilmediyse 1'den başlar; "En uzun seri" ayrı saklanır; yarım kalan gün seriyi kırar.
- [ ] **Senaryo: Seri gösterimi** · Verilen seri 7 · Eğer Home/Result görünürse · O zaman "Seri · 7 gün" ve 7, 30, 100. günde küçük kutlama (konfeti 1 sn, Reduce Motion'da yalnızca rozet).
- [ ] **Senaryo: Geri sayım** · Verilen bugün gönderildi · Eğer Home'da beklenirse · O zaman "Yarına 06:12:33" her saniye güncellenir ve 00:00'da yeni güne geçer.

### Teknik görevler (Unity/C#)
- [ ] Core `ShareCardRenderer` tüketimi: oyuna özgü kart şablonu prefab (koyu ahşap zemin, krem taş görseli, Nunito TR/DE karakter seti), `ShareCardData {dayIndex, date, score, par, percent, meldCount, streak, indicatorTile}`.
- [ ] Core `ShareService`: Android `FileProvider` + `Intent`; WhatsApp için pano yedeği.
- [ ] Kısa link: düz mağaza linki + UTM (`utm_source=share&utm_content=day142`); TR/DE/NL için aynı link.
- [ ] Core `StreakService` {current, best, lastSubmittedDayIndex}; `CountdownView` (Core).
- [ ] 7, 30, 100 gün rozet görselleri (basit, okey taşı siluetli).
- [ ] Hata yolu: paylaşım iptalinde `share_cancel`; izin reddinde yalnızca metin.
- [ ] Edit Mode testi: seri mantığı (ardışık gün, kırılma, yarım gün); paylaşım metni biçimi (3 dil).

### Analitik
- `share_tap` {day_index: int, score: int, par_percent: int} — düğmeye dokunma
- `share_done` {target: string|null} — sayfa kapandığında
- `streak_update` {current: int, best: int} — her gönderimde

### Notlar
- Bilgi metriği: paylaşım oranı = share_tap / result_view (kapı değil; TR kitlede WhatsApp paylaşımı kapı metriğinden bağımsız izlenir, hedef ≥ %8).
- Karta oyuncu adı koymama kararı Daily Drop ile aynı.
- Kart varyantı A/B (`share_card_variant`) SONRA; v1 tek şablon.

## STORY: Günün Okeyi 11: Ödüllü videoyla ipucu ve ikinci gönderim
labels: tema:monetizasyon, P1, efor:M
milestone: MVP

**Günün Okeyi** epic'inin **11. story'si** (epic #EPIC). Tema: Monetizasyon.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `HintFlow`, `SecondSubmitFlow`, `AdsService` (Core), Play "İpucu" düğmesi, Result ikincil düğme

### Kullanıcı hikâyesi
Bir oyuncu olarak takıldığımda bir reklam izleyip çözücüden tek bir dizi önerisi almak ve kötü bir gönderimden sonra bir reklamla aynı eli bir kez daha göndermek isterim, böylece gün boşa gitmesin; geliştirici olarak da bu iki yerleşimin oyuncunun seçimi, dürüst ve ölçülebilir olmasını isterim.

### Kabul kriterleri
- [ ] **Senaryo: İpucu teklifi** · Verilen Play'de gönderilmemiş el, reklam yüklü, ipucu kullanılmamış · Eğer "İpucu" basılırsa · O zaman sheet: "Bir dizi önerisi al · reklam izle" (No Ads sahibi için de reklam; ipucu No Ads kapsamında değil); reklam yüklü değilse düğme devre dışı ve "yükleniyor" etiketi, 10 sn'de bir yeniden dener.
- [ ] **Senaryo: İpucu ödülü** · Verilen reklam sonuna kadar izlendi · Eğer `OnRewarded` gelirse · O zaman `Solver.SuggestOne` sonucu ıstakada 3 taş 1,5 sn yanıp söner ve HUD'da "Öneri: K4 K5 K6" çipi gönderime kadar kalır; taşlar otomatik taşınmaz (oyuncu dizer); `hintUsed = true`.
- [ ] **Senaryo: Önerilecek dizi yok** · Verilen oyuncunun dizilişi zaten en iyi mümkün · Eğer "İpucu" basılırsa · O zaman reklam gösterilmeden "Daha iyisi yok, gönderebilirsin" mesajı (boş ödül yok).
- [ ] **Senaryo: Günde bir ipucu** · Verilen ipucu kullanıldı · Eğer "İpucu" basılırsa · O zaman düğme devre dışı, "Bugünlük bu kadar" ipucu; ikinci gönderimde ipucu hakkı yenilenmez.
- [ ] **Senaryo: İkinci gönderim teklifi** · Verilen ilk gönderim yapıldı, puan < en iyi mümkün, reklam yüklü · Eğer Result açılırsa · O zaman "Bir kez daha gönder · reklam izle" düğmesi görünür; puan = en iyi mümkün ise görünmez.
- [ ] **Senaryo: İkinci gönderim ödülü** · Verilen reklam izlendi · Eğer `OnRewarded` gelirse · O zaman `secondSubmitUsed = true`, Play aynı elle ve son dizilişle açılır (sıfırdan değil), gönderim sayacı 2; tabloya yüksek puan girer; kartta "2. gönderim" etiketi.
- [ ] **Senaryo: Yarıda kapatma** · Verilen reklam yarıda kapatıldı · Eğer geri dönülürse · O zaman hak kullanılmamış sayılır, düğme kalır; uygulama kapanırsa durum korunur.
- [ ] **Senaryo: Günde bir ikinci gönderim** · Verilen ikinci gönderim bitti · Eğer Result açılırsa · O zaman düğme yok; "Yarın yeni el".

### Teknik görevler (Unity/C#)
- [ ] `HintFlow` ve `SecondSubmitFlow`: durum {Offered, Loading, Showing, Rewarded, Failed}; `AdsService.ShowRewarded(placement: "hint")` ve `"second_submit"`.
- [ ] Core `AdsService`: AppLovin MAX ödüllü yerleşim, yükleme/yeniden deneme, `OnRewarded` güvenilirlik (ödül yalnızca SDK callback'inde); iki yerleşim kimliği.
- [ ] `DailyState.hintUsed`, `secondSubmitUsed`, `hintMeld` (çip için) kalıcı + Cloud Save.
- [ ] HUD "İpucu" düğmesi durumları (aktif, yükleniyor, kullanıldı); öneri vurgusu `TileView.Blink`; "Öneri" çipi `MeldChips` içinde.
- [ ] Result ikincil düğme; `SubmitPolicy` ile `MaxSubmits` güncellemesi (8. story).
- [ ] Remote Config: `hint_rewarded_enabled`, `second_submit_enabled`; test reklam kimlikleri `dev`, gerçek `prod`.
- [ ] Hile: ödül callback'i gelmeden ipucu/gönderim açılmaz; `SuggestOne` sonucu cihazda hesaplanır (çözücü zaten yerel).

### Analitik
- `hint_offered` {day_index: int, score: int, par_score: int} · `hint_ad_start` · `hint_rewarded` {meld_kind: string} · `hint_ad_failed` {reason: string} · `hint_no_better` {}
- `second_submit_offered` {day_index: int} · `second_submit_ad_start` · `second_submit_rewarded` · `second_submit_ad_failed` {reason: string}

### Notlar
- İki ödüllü yerleşim ilkesi: günün eli ekranında ve Home'da başka reklam yok. İpucu "çözümü satın alma" değil tek dizi; oyuncunun kendi dizilişine saygı (otomatik taşıma yok).
- Açık soru: ipucu No Ads'e dahil edilsin mi? v1: hayır (ödüllü video No Ads'ten etkilenmez; STYLE ilkesi). Soft launch'ta No Ads sahiplerinin ipucu kullanımı izlenir.
- TR eCPM düşük; ipucu izlenme oranı (hint_rewarded / daily_start) bilgi metriği, hedef ≥ %15.

## STORY: Günün Okeyi 12: Pratik modu ve interstitial kotası
labels: tema:monetizasyon, P1, efor:M
milestone: MVP

**Günün Okeyi** epic'inin **12. story'si** (epic #EPIC). Tema: Pratik modu.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `PracticeMode`, `PracticeDeal`, `InterstitialQuota` (Core), Home "Pratik" düğmesi, Play pratik rozeti

### Kullanıcı hikâyesi
Bir oyuncu olarak günün elini bitirdikten sonra da oynamaya devam etmek ve sınırsız rastgele el çözmek isterim; geliştirici olarak bu modun ölçülü interstitial ile gelir getirmesini ve günün eli ritüelini gölgelememesini isterim.

### Kabul kriterleri
- [ ] **Senaryo: Giriş** · Verilen Home · Eğer "Pratik" basılırsa · O zaman rastgele seed'li (gün bağımsız) 14 taş + gösterge ile Play açılır; HUD'da "Pratik" rozeti; günün eli durumu etkilenmez.
- [ ] **Senaryo: Gönderim** · Verilen pratik eli · Eğer "Diziyi gönder" basılırsa · O zaman onay yok, hemen pratik sonuç kartı (puan, en iyi mümkün, "Yeni el" birincil, "Ana menü"); tabloya ve seriye gitmez; sınırsız tekrar.
- [ ] **Senaryo: Interstitial kotası** · Verilen No Ads yok · Eğer 3., 6., 9. pratik eli gönderilirse · O zaman sonuç kartından önce interstitial gösterilir (`practice_interstitial_every` = 3); günde en fazla 3 (`practice_interstitial_daily_cap`); 4. interstitial hiç gösterilmez; günün eli akışında asla interstitial yok.
- [ ] **Senaryo: Interstitial yüklenmedi** · Verilen reklam hazır değil · Eğer kota anı gelirse · O zaman atlanır, sayaç ilerlemez (sonraki ele tekrar denenir); oyuncu bekletilmez.
- [ ] **Senaryo: No Ads** · Verilen No Ads satın alınmış · Eğer pratik oynanırsa · O zaman hiç interstitial yok; ipucu ödüllü videosu pratikte de çalışır (günde bir sınırı pratiğe uygulanmaz; her elde bir ipucu, reklamla).
- [ ] **Senaryo: Kaldığı yer** · Verilen yarım pratik eli · Eğer uygulama kapanıp açılırsa · O zaman Home'da "Pratiğe devam et" ikincil bağlantı, aynı el geri gelir; gün değişimi pratik elini bozmaz.
- [ ] **Senaryo: Oynanabilir el** · Verilen pratik seed'i · Eğer çözücü en iyi puanı < 20 ise · O zaman yeniden çekilir (2. story kuralı).
- [ ] **Senaryo: Zorluk etiketi** · Verilen pratik eli · Eğer HUD çizilirse · O zaman çözücü puanına göre "Kolay / Orta / Zor" etiketi (par < 45 kolay, 45–70 orta, > 70 zor); günün elinde etiket yok (spoiler).

### Teknik görevler (Unity/C#)
- [ ] `PracticeDeal`: `DailyDeal` ile aynı üretim, seed = `Xoshiro256` rastgele tohum (`Guid` tabanlı); `PracticeState {seed, snapshot, handsPlayedToday, interstitialsShownToday, lastDate}` yerel JSON.
- [ ] `PracticeMode` akışı: Play'e `mode = Practice` parametresi; `SubmitFlow` onaysız dal; pratik sonuç kartı prefab'ı (Result sahnesinin hafif varyantı).
- [ ] Core `InterstitialQuota`: `every`, `dailyCap`, gün sıfırlama (yerel tarih), `TryShow(placement: "practice_result")`; `AdsService.ShowInterstitial`; No Ads ile tamamen kapalı (`IapService.OwnsNoAds`).
- [ ] Home "Pratik" düğmesi (ikincil), "Pratiğe devam et" bağlantısı; Play HUD rozeti ve zorluk etiketi.
- [ ] Remote Config: `practice_interstitial_every` (3), `practice_interstitial_daily_cap` (3), `practice_difficulty_thresholds` (45, 70).
- [ ] İpucu akışının pratik dalı: `HintFlow(mode: Practice)` günlük sınır yerine el başına bir.
- [ ] Edit Mode testi: kota sayacı (3-6-9, günlük tavan, yüklenmemiş reklamda sayaç ilerlemez).

### Analitik
- `practice_start` {hands_today: int} · `practice_submit` {score: int, par_score: int, par_percent: int, difficulty: string}
- `interstitial_shown` {placement: "practice_result", n_today: int} · `interstitial_skipped` {reason: "not_loaded"|"cap"|"no_ads"}

### Notlar
- İlke: günün eli deneyimi reklamsız kalır; gelir pratik ve ödüllü videodan. Pratik kotası (3 elde bir, günde 3) STYLE ilkesiyle uyumlu; eCPM düşük TR'de agresifleşmek D7'yi bozar.
- "Pratiğe devam et" ile günün eli Oyna düğmesi aynı ekranda; günün eli her zaman üstte ve birincil.

## STORY: Günün Okeyi 13: İlk gün deneyimi ve kurallar ekranı
labels: tema:cekirdek, P1, efor:M
milestone: MVP

**Günün Okeyi** epic'inin **13. story'si** (epic #EPIC). Tema: Istaka ve kontrol.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** `TutorialDirector`, `RulesScreen`, Home ilk açılış, Play ipuçları, `docs/okey-rules.md`

### Kullanıcı hikâyesi
Bir yeni oyuncu olarak (okeyi bilsem de bu uygulamanın kurallarını bilmem) 30 saniyede, oynarken ve girdim kilitlenmeden öğrenmek isterim; kuralları istediğim an okuyabilmek isterim, böylece ilk gün puanım öğrenme kurbanı olmasın ve "masamızda böyle değil" itirazım yanıt bulsun.

### Kabul kriterleri
- [ ] **Senaryo: Büyük taş sorusu** · Verilen ilk açılış · Eğer Home ilk kez görünürse · O zaman tek soru: "Taşlar nasıl görünsün?" iki önizleme (Normal / Büyük); seçim Ayarlar'da değiştirilebilir; soru bir kez.
- [ ] **Senaryo: Üç ipucu** · Verilen ilk Play · Eğer el açılırsa · O zaman 1) "Taşı sürükle, yan yana koy" eli (ilk hamleye kadar), 2) ilk dizi oluşunca "Yeşil kenar: dizi oldu, puan geldi" balonu, 3) ilk boşluk bırakmada veya 20 sn sonra "Boşluk grupları ayırır" balonu; hiçbiri girdiyi kilitlemez, her biri bir kez.
- [ ] **Senaryo: Okey açıklaması** · Verilen ilk Play · Eğer HUD gösterge satırına ilk kez dokunulursa veya 10 sn geçerse · O zaman "Gösterge K5 → Okey K6 (her taşın yerine geçer). Sahte okey = K6." balonu 4 sn.
- [ ] **Senaryo: Gönder açıklaması** · Verilen ilk gönderim onayı · Eğer sheet açılırsa · O zaman "Günde bir gönderim. Herkes aynı eli oynadı; en iyi mümkün puanı gönderince göreceksin." satırı eklenir.
- [ ] **Senaryo: Kurallar ekranı** · Verilen Home veya Play'deki ⓘ · Eğer basılırsa · O zaman 5 bölümlük kaydırmalı ekran: Taşlar ve okey, Seri (13-1 örneğiyle), Per, Puan ve tam el, Günlük el ve gönderim; her bölümde taş görselli örnek; TR/EN/DE; "Bu masada böyle oynanır" notu (varyantlar: 13-1-2 yok, dizide en fazla 2 okey, ceza yok).
- [ ] **Senaryo: Tekrar yok** · Verilen ipuçları görüldü · Eğer ikinci gün oynanırsa · O zaman ipucu çıkmaz; Ayarlar'dan "İpuçlarını sıfırla" ile geri gelir.
- [ ] **Senaryo: Isınma eli** · Verilen ilk açılış · Eğer oyuncu "Önce bir pratik el" seçerse · O zaman sayılmayan bir pratik el (12. story) oynatılır, sonra günün eli; seçenek bir kez, varsayılan "Günün eline başla".

### Teknik görevler (Unity/C#)
- [ ] `TutorialFlags` (Cloud Save), `TutorialDirector` (olay dinleyici: FirstMove, FirstMeld, FirstGap/Timer, IndicatorTap, FirstSubmit).
- [ ] El animasyonu ve balon prefab'ı (Core `HintBubble`), büyük yazı modunda balon metni ölçeklenir.
- [ ] `RulesScreen`: 5 bölüm, `TileView` ile canlı örnekler (statik prefab'lar), TR/EN/DE; içerik `docs/okey-rules.md` ile aynı kaynaktan (yerelleştirme tablosu anahtarları `rules.*`).
- [ ] İlk açılış "Büyük taş" sorusu: `rack_rows/cols` ve taş boyutu ayarını yazar (18. story altyapısını kullanır; 18 kapanmadıysa yalnızca ayar yazılır).
- [ ] Isınma eli: `PracticeMode` çağrısı, dönüşte günün eline otomatik geçiş.

### Analitik
- `ftue_step` {step: string} — her ipucu gösteriminde
- `big_tile_choice` {big: bool} — ilk açılış seçimi
- `rules_view` {section: string, source: "home"|"play"} — kurallar ekranı açılış ve bölüm kaydırmada
- `warmup_choice` {chosen: bool}

### Notlar
- İlke: zorunlu, girdiyi kilitleyen eğitim yok (MustyBlockBlast ve Daily Drop ile aynı).
- Kurallar ekranı "varyant tartışması" issue'sunun kullanıcıya dönük yüzü; mağaza yorumlarındaki kural itirazları bu ekrana bağlanır.

## STORY: Günün Okeyi 14: Analitik olay şeması ve kapı raporu
labels: tema:analitik, P1, efor:S
milestone: MVP

**Günün Okeyi** epic'inin **14. story'si** (epic #EPIC). Tema: Analitik.

**Öncelik:** P1 · **Efor:** S · **Milestone:** MVP · **Dokunduğu yer:** UGS Analytics olay tanımları, `docs/analytics.md`, SQL raporları

### Kullanıcı hikâyesi
Bir geliştirici olarak soft launch kapısını (D1, D7, günlük gönderim oranı, oturum süresi, TR dışı pay) tek sorguyla görmek isterim, böylece karar veriyle verilir.

### Kabul kriterleri
- [ ] **Senaryo: Şema** · Verilen `docs/analytics.md` · Eğer okunursa · O zaman tüm olaylar (app_open, session_start, home_view, daily_start, solver_run, tile_move, rack_sort, undo, meld_detected, meld_hint_shown, submit, result_view, par_reveal, share_tap, share_done, streak_update, hint_*, second_submit_*, practice_*, interstitial_*, ftue_step, big_tile_choice, rules_view, warmup_choice, settings_change, clock_drift_detected) parametre tipleriyle listelidir ve UGS'de tanımlıdır.
- [ ] **Senaryo: Doğrulama** · Verilen dev build · Eğer bir gün oynanırsa (günün eli + 3 pratik + ipucu) · O zaman UGS Event Browser'da olaylar sırayla ve doğru parametrelerle görünür.
- [ ] **Senaryo: Kapı sorgusu** · Verilen UGS SQL · Eğer `gate_report.sql` çalıştırılırsa · O zaman D1, D7, günlük gönderim oranı (submit attempt 1 benzersiz / home_view benzersiz, gün bazında), ortalama oturum, çökme oranı, TR dışı kullanıcı payı (`app_open.country`), ipucu izlenme oranı tek tabloda gelir.
- [ ] **Senaryo: Huni** · Verilen `funnel_day.sql` · Eğer çalıştırılırsa · O zaman home_view → daily_start → ilk meld_detected → submit → result_view → share_tap adımları gün bazında yüzdeyle gelir.
- [ ] **Senaryo: Par dağılımı** · Verilen `par_distribution.sql` · Eğer çalıştırılırsa · O zaman gün bazında par_percent dağılımı (medyan, %90) ve "eli çözen" oranı gelir (el zorluğu ayarı için).
- [ ] **Senaryo: Gizlilik** · Verilen olaylar · Eğer incelenirse · O zaman kişisel veri (ad, e-posta, kesin konum) yoktur; `country` yalnızca ülke kodu.

### Teknik görevler (Unity/C#)
- [ ] Core `Analytics` sarmalayıcısı tüketimi: `Track(string, Dictionary<string, object>)`, kuyruk + çevrimdışı tampon; `country` için UGS'nin sağladığı alan veya `RegionInfo`.
- [ ] UGS Analytics custom event tanımları (dashboard'dan), parametre tipleri.
- [ ] `docs/analytics.md`, `docs/reports/gate_report.sql`, `funnel_day.sql`, `par_distribution.sql`, `practice_ads.sql` (interstitial_shown / practice_submit).
- [ ] Crashlytics (Firebase) veya UGS Crash raporlama: çökme oranı için.

### Analitik
- Bu story şemanın kendisidir.

### Notlar
- MustyBlockBlast'taki `docs/UGS SQL` yaklaşımı aynen uygulanır; Daily Drop raporlarıyla aynı şablon, oyuna özgü sütunlar.
- `par_distribution.sql` el zorluğu için: medyan par_percent < %50 ise eller çok zor (min_solver_score veya bonus ayarı), > %90 ise çok kolay.

## STORY: Günün Okeyi 15: Mağaza sayfası, ASO ve soft launch yapılandırması
labels: tema:magaza, P1, efor:M
milestone: MVP

**Günün Okeyi** epic'inin **15. story'si** (epic #EPIC). Tema: Mağaza ve çıkış.

**Öncelik:** P1 · **Efor:** M · **Milestone:** MVP · **Dokunduğu yer:** Play Console, mağaza metinleri (TR/EN/DE), ekran görüntüleri, kapalı test

### Kullanıcı hikâyesi
Bir geliştirici olarak oyunu Türkiye, Almanya ve Hollanda'da kapalı test ve ardından soft launch'a çıkarmak isterim, böylece hem Türkiye hem diaspora verisi gelsin.

### Kabul kriterleri
- [ ] **Senaryo: Başlık ve metin** · Verilen Play listesi · Eğer yayımlanırsa · O zaman TR başlık "Günün Okeyi: Okey Bulmaca" (30 karakter sınırı), EN "Daily Okey: Rummy Tile Puzzle", DE "Okey des Tages: Rummy Puzzle"; kısa açıklama ≤ 80 karakter, uzun tire yok; tam açıklamada TR: "okey", "okey bulmaca", "günlük okey", "okey oyunu tek kişilik", "süre yok"; EN: "rummikub puzzle", "daily puzzle", "okey"; DE: "Okey", "Rummikub Rätsel", "tägliches Puzzle" geçer.
- [ ] **Senaryo: Görseller** · Verilen ekran görüntüleri · Eğer yüklenirse · O zaman dil başına 8 görüntü (ıstaka, dizi vurgusu, sonuç ve en iyi mümkün, paylaşım kartı, seri, pratik, büyük taş modu, kurallar), 1 özellik grafiği (ahşap ıstaka, dört gerçek renk); mevcut okey uygulamalarının logo ve renk düzenine benzemez.
- [ ] **Senaryo: Kapalı test** · Verilen 12+ test kullanıcısı (en az 4'ü 45+, en az 2'si DE/NL) · Eğer 14 gün sürerse · O zaman Play üretim erişimi formu doldurulur (MustyBlockBlast #1326 deneyimi).
- [ ] **Senaryo: Ülkeler** · Verilen soft launch · Eğer yayımlanırsa · O zaman yalnızca TR, DE, NL; No Ads ve taş seti fiyatları yerel (TR'de yerel kademe, DE/NL EUR).
- [ ] **Senaryo: Mağaza dili eşlemesi** · Verilen cihaz dili Almanca, ülke DE · Eğer sayfa açılırsa · O zaman DE liste görünür; Türkçe cihaz dili DE ülkesinde TR liste görünür (dil bazlı liste).
- [ ] **Senaryo: Sürüm bilgisi** · Verilen build · Eğer yayımlanırsa · O zaman versionCode ve versionName CHANGELOG ile eşleşir; gizlilik politikası ve veri güvenliği formu dolu (reklam SDK'sı, analitik, Cloud Save).

### Teknik görevler (Unity/C#)
- [ ] Mağaza metinleri `docs/store/{tr,en,de}.md`; ASO anahtar kelime listesi; rakip başlık ve ikon taraması (TR okey uygulamaları, Rummikub, EN "rummy puzzle").
- [ ] Ekran görüntüsü şablonu (Figma/Design kanvası), 1080×1920, 3 dil; büyük yazı (45+ okunabilirlik: başlıklar ≥ 64 px).
- [ ] Veri güvenliği formu; `docs/privacy.md`; AppLovin MAX ve UGS veri beyanları.
- [ ] Play Console kurulumu: iç test → kapalı test → üretim (3 ülke); fiyat kademeleri.
- [ ] Sürümleme: `CHANGELOG.md`, `versionCode` CI'da otomatik.
- [ ] UTM'li paylaşım linki takibi (Play Console "Acquisition" ile eşleme).

### Analitik
- Mağaza: Play Console dönüşüm oranı ülke/dil bazında; UTM'li paylaşım linkleri; `app_open.country` ile TR dışı pay.

### Notlar
- Başlık testi: soft launch sırasında TR "Okey Bulmaca" ile "Günlük Okey" A/B (Play listing experiments); EN'de "Rummy Tile Puzzle" ile "Rummikub Puzzle" (marka adı kullanımı riski: "Rummikub" tescilli; yalnızca anahtar kelime alanında ve açıklamada "rummikub-style" ifadesi; başlıkta kullanılmaz).
- NL'de Türkçe konuşan kitle için TR liste; Hollandaca liste yok (v1).

## STORY: Günün Okeyi 16: Dünya tablosu ve arkadaş ligi (FriendsService Core'a)
labels: tema:backend, P2, efor:L
milestone: Soft launch

**Günün Okeyi** epic'inin **16. story'si** (epic #EPIC). Tema: Sosyal.

**Öncelik:** P2 · **Efor:** L · **Milestone:** Soft launch · **Dokunduğu yer:** UGS Leaderboards `okey_daily_{dayIndex}`, `LeaderboardService` (Core), `FriendsService` (Core'a taşınır), Result "günün en iyisi" ve arkadaş listesi, Home "Bugün" sekmesi

### Kullanıcı hikâyesi
Bir oyuncu olarak gönderimden sonra bugün dünyada ilk yüzde kaça girdiğimi ve arkadaşlarımın (aile grubumun) bugünkü puanlarını "Ayşe 56" gibi görmek isterim, böylece aynı eli oynayan herkesle ve en çok da ailemle kıyaslanayım.

### Kabul kriterleri
- [ ] **Senaryo: Gönderim** · Verilen tamamlanmış gönderim · Eğer Result açılırsa · O zaman puan `okey_daily_{dayIndex}` tablosuna gönderilir; ikinci gönderim daha yüksekse günceller ve `metadata.attempt = 2` taşır.
- [ ] **Senaryo: Yüzdelik ve günün en iyisi** · Verilen tablo yanıtı · Eğer gelirse · O zaman "Dünyada ilk %12" ve günün en iyisi satırı "Ayşe 56" (takma ad + puan; takma ad yoksa "Oyuncu 1234"); yanıt 2 sn'de gelmezse "—" ve arka planda güncellenir.
- [ ] **Senaryo: Par ile tutarlılık** · Verilen sunucuya gelen puan > o günün `parScore` · Eğer gönderilirse · O zaman Cloud Code reddeder (çözücü sonucu sunucuda da hesaplanır: `Game.Rules` saf C# → Cloud Code JS portu değil; sunucu, istemcinin gönderdiği `parScore`'u günün ilk 100 gönderiminin medyanıyla karşılaştırır ve üst sınır olarak kullanır; tam doğrulama SONRA).
- [ ] **Senaryo: Arkadaş kodu** · Verilen profil · Eğer "Arkadaş ekle" açılırsa · O zaman 6 karakterli kod ve paylaşılabilir link; kod girilince iki taraf birbirinin listesinde (karşılıklı onay yok; engelleme var); sınır 50.
- [ ] **Senaryo: Arkadaşların puanı** · Verilen 3 arkadaş · Eğer Result açılırsa · O zaman arkadaşların bugünkü puanları sıralı ("Ayşe 56 · Mehmet 44 · sen 41"), oynamayanlar "henüz oynamadı"; 2. gönderimle gelen puanın yanında küçük "2" rozeti.
- [ ] **Senaryo: Davet linki** · Verilen link tıklandı, uygulama yüklü değil · Eğer mağazadan yüklenip açılırsa · O zaman kod panoda hazır gelir (clipboard tabanlı attribution).
- [ ] **Senaryo: Core paylaşımı** · Verilen `FriendsService` Core'a taşındı · Eğer Daily Drop aynı sürümü alırsa · O zaman Daily Drop 21 testleri geçer ve iki oyunun arkadaş listeleri ayrı (oyun anahtarı ile izole).
- [ ] **Senaryo: Çevrimdışı** · Verilen bağlantı yok · Eğer gönderim yapılırsa · O zaman kuyruklanır, bağlantı gelince gider; Result "—" gösterir.

### Teknik görevler (Unity/C#)
- [ ] UGS Leaderboards: günlük tablo (`okey_daily_{dayIndex}`, Cloud Code ile oluşturma), tutma 30 gün; `metadata {attempt, parScore}`.
- [ ] Core `LeaderboardService.Submit(boardId, score, metadata)`, `GetPlayerRange`, `GetTop(1)`, kuyruk; oyun tarafında `boardId = "okey_daily_" + dayIndex`.
- [ ] Cloud Code `submitOkeyScore`: üst sınır (medyan par × 1,0 + tam el toleransı), gün kontrolü, oyuncu kimliği.
- [ ] Core `FriendsService` (Daily Drop 21'den taşıma): UGS Friends veya Cloud Save + Cloud Code liste; takma ad Player Accounts profilinde; davet kodu (base32, çakışma kontrolü); engelleme ve kaldırma; oyun anahtarı ile izolasyon.
- [ ] Result "günün en iyisi" satırı, arkadaş listesi UI, "Davet et" (10. story ShareService); Home "Bugün" sekmesi (ilk 50 + kendi sırası).
- [ ] Takma ad doğrulama (uygunsuz kelime listesi TR/EN/DE, 3–12 karakter).

### Analitik
- `leaderboard_submit` {day_index: int, score: int, attempt: int, percentile: float}
- `friend_invite_share` · `friend_added` {count: int} · `friends_result_view` {friends_played: int}

### Notlar
- Bu story kapı sonrası; günlük gönderim oranı düşükse önce ritüel iyileştirmesi.
- UGS Leaderboards limitleri soft launch hacminde ücretsiz katmanda kalmalı; günlük tablo sayısı 30 günde 30 (temizleme Cloud Code'da).
- "Ayşe 56" tasarım taslağında Result'ta; 45+ kitle için takma ad girişi isteğe bağlı, ilk arkadaş eklerken sorulur.

## STORY: Günün Okeyi 17: No Ads ve kozmetik taş setleri
labels: tema:monetizasyon, P2, efor:M
milestone: Soft launch

**Günün Okeyi** epic'inin **17. story'si** (epic #EPIC). Tema: Monetizasyon.

**Öncelik:** P2 · **Efor:** M · **Milestone:** Soft launch · **Dokunduğu yer:** `IapService` (Core), `NoAdsGate`, `TileSkinSet`, Mağaza ekranı (Core `ShopScreen`), IAP ürünleri

### Kullanıcı hikâyesi
Bir oyuncu olarak tek seferlik küçük bir ücretle pratikteki reklamlardan kurtulmak ve taşlarımın görünümünü (ahşap, kemik, neon) değiştirmek isterim, böylece oyun benim olsun; geliştirici olarak satın alımların geri yüklenebilir ve dürüst olmasını isterim.

### Kabul kriterleri
- [ ] **Senaryo: Mağaza ekranı** · Verilen Home'daki Mağaza · Eğer açılırsa · O zaman No Ads (2,99 USD, yerel fiyat) üstte: "Pratikte reklam yok. İpucu ve ikinci gönderim videoları senin seçimin, onlar kalır." açıklaması; altında 4 taş seti: Klasik (ücretsiz), Ahşap, Kemik, Neon (her biri 1,99 USD); önizleme canlı ıstaka parçası; "Satın alımları geri yükle".
- [ ] **Senaryo: No Ads satın alma** · Verilen Google Play Billing · Eğer No Ads alınırsa · O zaman `NoAdsGate.IsActive` true olur, pratikteki interstitial anında kapanır (12. story), ödüllü videolar çalışmaya devam eder; Cloud Save'de kalıcı.
- [ ] **Senaryo: Taş seti satın alma** · Verilen set · Eğer alınırsa · O zaman anında uygulanır, Ayarlar'dan değiştirilebilir, Cloud Save'de kalıcı, geri yüklenebilir.
- [ ] **Senaryo: Adalet ve okunabilirlik** · Verilen herhangi set · Eğer oynanırsa · O zaman taş boyutu, sayı yazı tipi ve dört renk aynıdır (yalnızca zemin dokusu ve kenar); büyük taş modunda ve sembol ipuçlarıyla uyumlu; kontrast ≥ 4,5:1.
- [ ] **Senaryo: Paylaşım kartı** · Verilen seçili set · Eğer kart üretilirse · O zaman gösterge taşı seçili setle çizilir.
- [ ] **Senaryo: Geri yükleme** · Verilen yeni cihaz · Eğer "Geri yükle" basılırsa · O zaman No Ads ve setler tanınır.
- [ ] **Senaryo: Satın alma başarısız** · Verilen ödeme iptali · Eğer dönülürse · O zaman hiçbir şey açılmaz, hata metni kısa, tekrar denenebilir.

### Teknik görevler (Unity/C#)
- [ ] Unity IAP tüketilemeyen ürünler: `no_ads`, `skin_wood`, `skin_bone`, `skin_neon`; Play Console tanımları ve yerel fiyat kademeleri (TR, DE, NL).
- [ ] Core `IapService` tüketimi; `NoAdsGate` (Core): makbuz doğrulama (Cloud Code ile Google Play Developer API), yerel önbellek; `InterstitialQuota` ile bağ.
- [ ] `TileSkinSet` SO (zemin sprite, kenar, gölge; 4 renk ve sayı atlası ortak), `TileView` ile bağ; önizleme prefab'ı.
- [ ] Core `ShopScreen` iskeleti bağlama; fiyat yerelleştirme (`metadata.localizedPrice`).
- [ ] Remote Config: `shop_order`, No Ads fiyat testi için iki ürün (`no_ads`, `no_ads_b`) ve `no_ads_sku`.

### Analitik
- `shop_view` {source: string} · `iap_purchase` {sku: string, price_local: string} · `iap_restore` {count: int} · `skin_equip` {id: string}

### Notlar
- Karar: No Ads tek seferlik 2,99 USD (abonelik değil; 45+ kitlede abonelik direnci). Fiyat testi: 2,99 ve 1,99 USD (Remote Config ile sku seçimi).
- Fluffy Drop dersi: "çalışmayan No Ads" en çok şikâyet edilen madde; satın alma sonrası interstitial'ın kapandığı Play Mode testiyle doğrulanır.
- Neon seti büyük taş modunda parlaklık düşürülmüş varyant kullanır (okunabilirlik).

## STORY: Günün Okeyi 18: Erişilebilirlik ve ayarlar: büyük taş modu, renk körü sembolleri
labels: tema:erisilebilirlik, P2, efor:M
milestone: Soft launch

**Günün Okeyi** epic'inin **18. story'si** (epic #EPIC). Tema: Erişilebilirlik ve ayarlar.

**Öncelik:** P2 · **Efor:** M · **Milestone:** Soft launch · **Dokunduğu yer:** `SettingsScreen` (Core), `RackLayout` boyut parametreleri, `RackView` yeniden akış, `TileView` sembol atlası, büyük yazı modu

### Kullanıcı hikâyesi
45+ bir oyuncu olarak taşları ve yazıları büyük görmek, ses/haptik/hareket/dil ayarlarını tek yerden yapmak isterim; renk körü bir oyuncu olarak dört rengi sembolle ayırt etmek isterim, böylece gözlüksüz de oynayabileyim.

### Kabul kriterleri
- [ ] **Senaryo: Ayarlar listesi** · Verilen Ayarlar · Eğer açılırsa · O zaman Büyük taş, Büyük yazı, Sembol ipuçları, Ses, Haptik, Reduce Motion, Dil (TR/EN/DE), Takma ad, Taş seti, Bildirim (P3), Kuralları gör, İpuçlarını sıfırla, Uygulamayı puanla, Gizlilik, Satın alımları geri yükle, Sürüm.
- [ ] **Senaryo: Büyük taş modu** · Verilen "Büyük taş" açık · Eğer Play açılırsa · O zaman ıstaka 3 sıra × 6 yuva (18 yuva, 14 taş, 4 boş), taş 56×78 px, sayı 28 pt; mevcut diziliş kayıpsız yeniden akar (gruplar korunur, satır sonları yeniden hesaplanır); HUD daralır (gösterge satırı tek satıra iner), ekran dışına taşma yok (390×844 ve 360×780 referans).
- [ ] **Senaryo: Mod değişiminde diziliş** · Verilen 2×8'de 3 grup · Eğer 3×6'ya geçilirse · O zaman gruplar sırayla yerleşir, grup arası en az bir boşluk; geri dönüşte 2×8'e sığmıyorsa (boşluk sayısı yetmiyorsa) satır sonu ayracı kullanılır, taş kaybı yok.
- [ ] **Senaryo: Büyük yazı** · Verilen "Büyük yazı" açık veya sistem yazı boyutu büyük · Eğer HUD ve Result çizilirse · O zaman metinler %125, kırpılmaz (TextMeshPro auto-size sınırları), düğmeler ≥ 56 px yükseklik.
- [ ] **Senaryo: Sembol ipuçları** · Verilen açık · Eğer taşlar çizilirse · O zaman 7. story sembolleri (● ■ ▲ ◆) sayının yanında, kenar vurgusu desen farkıyla; gösterge ve okey satırında da semboller.
- [ ] **Senaryo: Reduce Motion** · Verilen açık · Eğer oynanırsa · O zaman taş tween'leri anında, konfeti ve parıltı yok, yanıp sönme yerine sabit vurgu; fonksiyon kaybı yok.
- [ ] **Senaryo: Android geri** · Verilen Ayarlar açık · Eğer geri basılırsa · O zaman Home'a dönülür.
- [ ] **Senaryo: Kontrast** · Verilen dört taş rengi ve krem zemin · Eğer ölçülürse · O zaman sayı/zemin kontrastı ≥ 4,5:1 (sarı için koyu sarı #B8860B tonu); tüm taş setlerinde (17. story) aynı eşik.

### Teknik görevler (Unity/C#)
- [ ] Core `SettingsScreen` bağlama; oyuna özgü anahtarlar: `big_tiles`, `shape_hints`, `tile_skin`.
- [ ] `RackLayout.Resize(rows, cols)`: grupları koruyarak yeniden yerleştirme algoritması (gruplar sırayla, boşluk ayraçlarıyla; sığmazsa satır sonu); Edit Mode testleri (2×8 ↔ 3×6, boşluk yetersizliği, taş sayısı korunumu).
- [ ] `RackView` yeniden akış: yuva sayısı değişimi, taş boyutu, HUD daraltma; güvenli alan.
- [ ] Büyük yazı: TextMeshPro auto-size ve TR/DE karakter atlası; düğme min yükseklik.
- [ ] Sembol atlası ve gösterge satırı sembolleri; renk tonları (sarı koyu ton) `TileColors` SO.
- [ ] Reduce Motion bayrağının tüm tween ve efektlere bağlanması (Core `Motion.Reduced`).

### Analitik
- `settings_change` {key: string, value: string} — her değişimde
- `big_tiles_active` {active: bool} — Play açılışında (kitle payını ölçmek için)

### Notlar
- Kitle 45+: büyük taş modu ilk açılışta soruluyor (13. story); bu story modun tam uygulamasıdır. Soft launch'ta `big_tiles_active` payı ≥ %30 beklenir; düşükse varsayılan büyük yapılır.
- Hollandalı/Alman cihazlarda sistem yazı boyutu büyük kullanımı yaygın; auto-size sınırları 360 px genişlikte de test edilir.

## STORY: Günün Okeyi 19: Günlük hatırlatma bildirimi ve değerlendirme isteği
labels: tema:ritual, P3, efor:S
milestone: Canlı

**Günün Okeyi** epic'inin **19. story'si** (epic #EPIC). Tema: Günlük el.

**Öncelik:** P3 · **Efor:** S · **Milestone:** Canlı · **Dokunduğu yer:** `LocalNotifications` (Core), `ReminderScheduler`, Ayarlar bildirim saati, mağaza içi değerlendirme isteği

### Kullanıcı hikâyesi
Bir oyuncu olarak günün eli hazır olduğunda seçtiğim saatte hatırlatılmak isterim, böylece serim kırılmasın; geliştirici olarak memnun oyuncudan doğru anda değerlendirme istemek isterim.

### Kabul kriterleri
- [ ] **Senaryo: İzin** · Verilen 3. gönderimin Result'ı · Eğer kapanırsa · O zaman "Yarın hatırlatayım mı?" sorusu bir kez sorulur; evet denirse sistem izni istenir.
- [ ] **Senaryo: Zamanlama** · Verilen izin verildi, saat 10:00 · Eğer gün değişirse · O zaman 10:00'da "Günün Okeyi hazır · #143" bildirimi gelir; bugün zaten gönderildiyse gelmez.
- [ ] **Senaryo: Kapatma** · Verilen Ayarlar'da bildirim kapalı · Eğer gün değişirse · O zaman bildirim gelmez.
- [ ] **Senaryo: Metin** · Verilen TR/EN/DE · Eğer bildirim gelirse · O zaman dil cihaz diline uyar; emoji yok.
- [ ] **Senaryo: Değerlendirme isteği** · Verilen 5. gönderilen gün ve par_percent ≥ %80 · Eğer Result kapanırsa · O zaman mağaza içi değerlendirme isteği bir kez gösterilir; sonraki istek en erken 30 gün sonra.

### Teknik görevler (Unity/C#)
- [ ] Unity Mobile Notifications paketi; Android kanal, iOS izin (Core `LocalNotifications`).
- [ ] `ReminderScheduler`: her gönderimde ertesi gün için planla, açılışta temizle/yeniden planla.
- [ ] Ayarlar: saat seçici, açma/kapama.
- [ ] In-app review (Google Play In-App Review, iOS SKStoreReviewController) tetik kuralı (Core `ReviewPrompt`).

### Analitik
- `notification_opt_in` {enabled: bool, hour: int} · `notification_open` {day_index: int} · `review_prompt` {shown: bool}

### Notlar
- Push (uzak) bildirim yok; yerel yeterli.
- Varsayılan saat 10:00 (45+ kitle sabah rutini; Daily Drop'ta 09:00).

## STORY: Günün Okeyi 20: 101 Okey varyantı
labels: tema:meta, P3, efor:L, SONRA
milestone: SONRA

**Günün Okeyi** epic'inin **20. story'si** (epic #EPIC). Tema: Meta ve varyantlar.

**Öncelik:** P3 · **Efor:** L · **Milestone:** SONRA · **Dokunduğu yer:** `RuleSet` soyutlaması, `Scorer101`, `MeldValidator` 101 dalı, Home ikinci günlük el kartı, ayrı tablo `okey101_daily_{dayIndex}`

### Kullanıcı hikâyesi
Bir 101 Okey oyuncusu olarak günün elini 101 kurallarıyla (açma eşiği 101, çiftle açma, farklı puanlama) oynamak isterim, böylece alıştığım varyant da bulmacaya dönüşsün.

### Kabul kriterleri
- [ ] **Senaryo: Kural seti** · Verilen `RuleSet.Okey101` · Eğer seçilirse · O zaman 21 taş (101 Okey eli), açma eşiği 101 puan, dizisiz taşlar eksi puan (her taşın değeri kadar ceza), çiftler 5 çiftle açma (21. story ile paylaşır); kurallar `docs/okey-rules.md` 101 bölümünde.
- [ ] **Senaryo: Günlük 101 eli** · Verilen Home · Eğer 101 modu açıksa · O zaman ikinci kart "Günün 101'i #N", ayrı seed (`"gununokeyi101:" + tarih`), ayrı gönderim, ayrı seri, ayrı tablo.
- [ ] **Senaryo: Çözücü** · Verilen 21 taş · Eğer çözülürse · O zaman < 300 ms (2^21 DP yerine dizi adayları üzerinde dallanma + budama); doğruluk 20 elle çözülmüş vakayla.
- [ ] **Senaryo: Istaka** · Verilen 21 taş · Eğer Play açılırsa · O zaman 3 sıra × 8 yuva (24 yuva); büyük taş modunda 4 × 6.

### Teknik görevler (Unity/C#)
- [ ] `RuleSet` arayüzü {TileCount, OpeningThreshold, PenaltyForLoose, AllowsPairs, Scorer, Validator}; `Okey` ve `Okey101` uygulamaları; `DailyDeal(ruleSet)`.
- [ ] `Scorer101` (ceza ve açma eşiği), `Solver` 21 taş yolu (dallanma + üst sınır budaması).
- [ ] Home ikinci kart, ayrı `DailyState` anahtarı `daily101/{dayIndex}`, ayrı tablo.
- [ ] Kurallar ekranı 101 bölümü; 20 çözülmüş vaka.

### Analitik
- `variant_select` {variant: "okey"|"okey101"} · `submit` parametrelerine `variant: string` eklenir

### Notlar
- Varyant tartışması issue'su: 101'in açma ve ceza kuralları masalara göre değişir; en yaygın kural seti seçilir, kurallar ekranında yazılır.
- Soft launch verisi "101 nerede?" yorumları getirirse öne alınır.

## STORY: Günün Okeyi 21: Çiftler modu
labels: tema:meta, P3, efor:M, SONRA
milestone: SONRA

**Günün Okeyi** epic'inin **21. story'si** (epic #EPIC). Tema: Meta ve varyantlar.

**Öncelik:** P3 · **Efor:** M · **Milestone:** SONRA · **Dokunduğu yer:** `MeldKind.Pair`, `MeldValidator` çift dalı, `Scorer` çift puanı, `MeldDetector`, haftalık "çiftler günü"

### Kullanıcı hikâyesi
Bir oyuncu olarak haftada bir gün elin yalnızca çiftlerle (aynı renk aynı sayı iki taş) puanlandığı bir varyant oynamak isterim, böylece ritüel tekdüzeleşmesin.

### Kabul kriterleri
- [ ] **Senaryo: Çift doğrulama** · Verilen K7 K7 (iki kopya) · Eğer çiftler modunda doğrulanırsa · O zaman `MeldKind.Pair`, geçerli; K7 S7 geçersiz; OKEY K7 geçerli (okey K7); sahte okey okeyin yüz değeriyle çift olur.
- [ ] **Senaryo: Puan** · Verilen 7 çift (14 taş) · Eğer puanlanırsa · O zaman çiftlerin sayı toplamı + tam el +50; seri ve per bu modda 0.
- [ ] **Senaryo: Gün takvimi** · Verilen Remote Config `pairs_day = Sunday` · Eğer Pazar ise · O zaman günün eli çiftler kuralıyla gelir, Home kartında "Çiftler günü" rozeti, kart ve tablo etiketi; herkes aynı kuralı oynar.
- [ ] **Senaryo: Çözücü** · Verilen çiftler modu · Eğer çözülürse · O zaman eşleştirme problemi olarak < 10 ms (okey ikamesi dahil).

### Teknik görevler (Unity/C#)
- [ ] `RuleSet.Pairs` (20. story soyutlaması); `MeldValidator` çift dalı; `Scorer` çift puanı; `MeldDetector` çift kenar rengi (mor).
- [ ] `Solver` çiftler yolu (açgözlü + okey yerleştirme; en yüksek değerli eşleşmeler).
- [ ] Remote Config takvim şeması `day_rules`; Home rozeti, kart etiketi, tablo kimliği `okey_pairs_daily_{dayIndex}`.
- [ ] Kurallar ekranı çiftler bölümü; Edit Mode testleri.

### Analitik
- `day_rule` {rule: "pairs", day_index: int}

### Notlar
- LiveOps'un en ucuz biçimi: içerik yok, kural var (Daily Drop 27 ile aynı yaklaşım).
- Çiftler kuralı 101 ile paylaşılır (5 çiftle açma); önce bu story, sonra 20.

## STORY: Günün Okeyi 22: Haftalık turnuva
labels: tema:backend, P3, efor:L, SONRA
milestone: SONRA

**Günün Okeyi** epic'inin **22. story'si** (epic #EPIC). Tema: Sosyal.

**Öncelik:** P3 · **Efor:** L · **Milestone:** SONRA · **Dokunduğu yer:** UGS Leaderboards haftalık tablo, Cloud Code toplama, Home "Hafta" sekmesi, hafta sonu özeti

### Kullanıcı hikâyesi
Bir oyuncu olarak haftanın 7 elinin par yüzdesi toplamıyla arkadaşlarım ve dünyayla haftalık sıralamada yarışmak isterim, böylece tek bir kötü gün beni yarıştan düşürmesin ve hafta boyu gelmek için sebebim olsun.

### Kabul kriterleri
- [ ] **Senaryo: Haftalık puan** · Verilen Pazartesi–Pazar 7 gönderim · Eğer hafta biterse · O zaman haftalık puan = par yüzdelerinin toplamı (0–700); eksik günler 0; ilk gönderim sayılır (2. gönderim hariç, adalet).
- [ ] **Senaryo: Tablo** · Verilen Home "Hafta" sekmesi · Eğer açılırsa · O zaman dünya ilk 50, arkadaşlar ve kendi sırası; her gün güncellenir.
- [ ] **Senaryo: Hafta sonu özeti** · Verilen Pazartesi ilk açılış · Eğer Home görünürse · O zaman "Geçen hafta: 512 puan, arkadaşlar arasında 2." kartı ve paylaşım.
- [ ] **Senaryo: Ödül yok** · Verilen turnuva sonu · Eğer sıralama kesinleşirse · O zaman rozet (kozmetik), para ödülü ve IAP avantajı yok.

### Teknik görevler (Unity/C#)
- [ ] Cloud Code `aggregateWeekly`: günlük tablolardan haftalık toplama (zamanlanmış), `okey_weekly_{weekIndex}` tablosu.
- [ ] Home "Hafta" sekmesi, hafta sonu özet kartı, rozetler.
- [ ] Paylaşım kartı haftalık varyantı.

### Analitik
- `weekly_view` {week_index: int, rank: int} · `weekly_summary_view` {score: int}

### Notlar
- Sunucu tarafı doğrulama (tam çözücü doğrulaması) olmadan haftalık turnuva hile riski taşır; 16. story'deki üst sınır kontrolü ile başlanır, gerekirse Cloud Code'a `Game.Rules` portu.

## STORY: Günün Okeyi 23: Tavla bulmacası (ayrı oyun, ortak çekirdek)
labels: tema:meta, P3, efor:L, SONRA
milestone: SONRA

**Günün Okeyi** epic'inin **23. story'si** (epic #EPIC). Tema: Meta ve varyantlar.

**Öncelik:** P3 · **Efor:** L · **Milestone:** SONRA · **Dokunduğu yer:** Yeni repo `MustyGununTavlasi`, Core günlük ritüel sınıfları, `Game.Rules` deseni

### Kullanıcı hikâyesi
Bir tavla oyuncusu olarak her gün herkese aynı pozisyon ve aynı zar verilen, "en iyi hamleyi bul" bulmacasını oynamak isterim; geliştirici olarak Günün Okeyi'nin günlük el, gönderim, sonuç, paylaşım ve arkadaş ligi altyapısını sıfır yeniden yazımla kullanmak isterim.

### Kabul kriterleri
- [ ] **Senaryo: Fizibilite** · Verilen `docs/tavla-feasibility.md` · Eğer yazılırsa · O zaman pozisyon üretimi (seed → gerçekçi orta oyun pozisyonu), hamle değerlendirmesi (equity: basit heuristik mi, GNU Backgammon benzeri sinir ağı mı), "en iyi hamle" tanımı ve puanlama (equity kaybı) kararları, 2–3 haftalık solo MVP planı içerir.
- [ ] **Senaryo: Ortak çekirdek** · Verilen Core · Eğer yeni proje kurulursa · O zaman `DailySeed`, `DailyState`, `SubmitPolicy`, `StreakService`, `ShareCardRenderer`, `LeaderboardService`, `FriendsService` değişiklik olmadan kullanılır; oyuna özgü yalnızca kural motoru ve tahta UI.
- [ ] **Senaryo: Kapsam kararı** · Verilen fizibilite · Eğer equity motoru 2 haftada mümkün değilse · O zaman story kapatılır ve karar epic'e yorum olarak yazılır.

### Teknik görevler (Unity/C#)
- [ ] Fizibilite dokümanı; equity motoru araştırması (lisanslı açık kaynak kullanımı ve GPL uyumu dahil).
- [ ] Core'da eksik kalan oyun bağımsızlıkları listesi (bu epic'in 1. story'sinden sonra kalanlar).

### Analitik
- Yok (fizibilite).

### Notlar
- Bu story bu repoda yalnızca karar ve çekirdek hazırlığı; oyunun kendisi ayrı epic ve repo.
- Günün Okeyi'nin kapı sonucu (TR dışı pay, gönderim oranı) tavla kararını belirler.
