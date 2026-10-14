<!-- Kaynak: docs/issues/gunun-okeyi.md (epic bölümü). GitHub'daki epic issue ile aynı metin; karar değişirse önce burası, sonra epic güncellenir. -->

# Günün Okeyi — herkese aynı 14 taş, tek gönderim, en iyi diziliş

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
