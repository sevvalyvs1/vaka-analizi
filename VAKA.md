# Veri Analitiği & Yapay Zekâ Uzman Yardımcısı — Teknik Vaka

**Süre:** 2 gün
**Beklenen efor:** 8–9 saat. Daha fazlasını harcamanız beklenmiyor — zaman yönetimi de değerlendirme kapsamındadır.
**Teslim:** Tek bir ZIP veya Git repo bağlantısı

---

## Yapay zekâ araçları hakkında

**Claude, ChatGPT, Gemini, Copilot ve benzeri araçları serbestçe kullanabilirsiniz.** Bu bir izin değil, beklentidir. Günlük işimizde bu araçları kullanıyoruz.

Ölçmek istediğimiz şey şu: **bu araçlarla hızlanıp doğru sonuca ulaşabiliyor musunuz?**

İki ayağı var:
1. **Hız** — aracı iyi yönlendirebiliyor musunuz
2. **Doğruluk** — aracın verdiği çıktıyı **kontrol edebiliyor musunuz**

İkincisi daha önemli. Bu veri setinde, yapay zekâ araçlarının varsayılan olarak yanlış cevap üreteceği noktalar bilinçli olarak bulunuyor. **Aracın ürettiğini sorgulamadan kabul eden bir çözüm, hiç araç kullanmayan bir çözümden düşük puan alır.**

Bölüm 5'te kullanım günlüğü var. "Şu noktada araç yanıldı, şöyle fark ettim" demeniz **artı puandır.**

---

## Senaryo

Bir otomotiv perakende şirketinin yetkili servis ağında çalışıyorsunuz. Müşterilerin bir kısmı zamanla servise gelmeyi bırakıyor ve genellikle yetkili servis dışına kayıyor.

İş birimi iki şey istiyor:

1. **Hangi müşterileri kaybetme riskimiz yüksek?**
2. **Elimizdeki müşteri geri bildirimlerinden ne öğrenebiliriz, ne aksiyon alabiliriz?**

Dört veri dosyası `veri/` klasöründe. Veri sentetiktir ama gerçek veride karşılaşacağınız türden sorunlar içerir. Ayrıntılar `veri_sozlugu.md` dosyasında.

### Zaman çerçevesi — dikkatle okuyun

```
|-------- GÖZLEM PENCERESİ --------|------ HEDEF PENCERESİ ------|
2022-01-01                    2025-06-30                   2026-06-30
     (özellikleri buradan üretin)        (hedefi buradan üretin)
```

- **Özellik kesim tarihi: 2025-06-30.** Modelinizin kullandığı her bilgi bu tarihten önce bilinebilir olmalı.
- **Hedef:** Müşteri 2025-07-01 – 2026-06-30 arasında en az bir kez servise geldi mi?
  Geldiyse `churn = 0`, gelmediyse `churn = 1`
- **Popülasyon:** Yalnızca 2025-06-30'dan önce en az bir servis kaydı olan müşteriler

⚠️ Dosyalarda hedef penceresine ait kayıtlar da var. Hedefi üretmek için gerekli — **ama özellik üretirken kullanmayın.**

---

## Bölüm 1 — Veri hazırlığı ve keşif · 25 puan

**1.1 — Analiz tablosu**
Dört dosyayı birleştirip müşteri seviyesinde tek bir tablo oluşturun. En az şu üç metriği hesaplayın: son servisten bu yana geçen gün (recency), toplam ziyaret sayısı (frequency), toplam harcama (monetary).

> 💡 Birleştirme sonrası satır sayınız beklediğinizden fazlaysa, devam etmeden önce nedenini bulun.

**1.2 — Veri kalitesi**
Veride birden fazla sorun var. **En önemli gördüğünüz 3 tanesini** ele alın. Her biri için: *ne buldunuz, ne yaptınız, neden o kararı verdiniz.*

Tespit etmek yeterli değil — kararınızı gerekçelendirin.

**1.3 — Keşifsel analiz**
Terk eden ve etmeyen müşteriler hangi boyutlarda ayrışıyor? **3 bulgu**, her biri bir görsel veya tabloyla.

**1.4 — İş birimi sorusu**

> *"Servis merkezlerimizin müşteri memnuniyeti arasında fark var mı? En kötü performans gösteren merkez için aksiyon almalı mıyız?"*

Bu soruyu yanıtlayın ve **iş birimine ne söyleyeceğinizi** 3-4 cümleyle yazın (teknik terim kullanmadan).

> 💡 Bu soruda en hızlı cevap ile en doğru cevap aynı değil.

---

## Bölüm 2 — Model · 30 puan

**2.1** Hedef değişkeni üretin, sınıf dağılımını raporlayın.

**2.2** Bir sınıflandırma modeli kurun. Model seçimi size ait — **iyi gerekçelendirilmiş basit bir model, gerekçesiz karmaşık bir modelden yüksek puan alır.**

**2.3** Değerlendirin:
- Hangi metrikleri seçtiniz ve **neden?**
- Bir taban çizgisiyle (baseline) karşılaştırın
- Doğrulama stratejinizi gerekçelendirin

**2.4** En önemli 5 değişkeni yorumlayın. Her biri için: neden tahmin gücü taşıyor, iş açısından anlamlı mı?

**2.5** Modelinizin bir sınırını yazın. Üretime alınsa neyin yanlış gitmesinden endişe ederdiniz?

⚠️ **Uyarı:** Modeliniz çok yüksek başarı veriyorsa (ROC-AUC > 0,95), raporlamadan önce **nedenini araştırın.** Bu veri setinde o seviyede bir başarı gerçekçi değildir.

---

## Bölüm 3 — LLM ile metin analizi · 25 puan

`geri_bildirimler.csv` dosyasında Türkçe müşteri geri bildirimleri var.

**3.1 — Etiketleme (7 puan)**

Dosyadan **rastgele 200 geri bildirim** seçin. Bir LLM kullanarak her birinden şunları çıkarın:

| Alan | Değerler |
|---|---|
| `duygu` | Olumlu / Olumsuz / Karışık / Alakasız |
| `konular` | Çoklu etiket: `randevu`, `fiyat`, `süre`, `personel`, `iş_kalitesi`, `yedek_parça`, `temizlik`, `diğer` |

Çıktı yapılandırılmış olmalı (CSV veya JSON).

**3.2 — Kalite ölçümü (13 puan) — bu bölümün en önemli maddesi**

LLM'in ne kadar doğru çalıştığını **ölçün:**

- Bu 200'ün içinden **40 tanesini kendiniz elle etiketleyin** (altın set)
- LLM çıktısını bu setle karşılaştırın, doğruluk oranını raporlayın
- **Sistematik hataları tanımlayın:** LLM en çok hangi tip metinde yanılıyor? En az 2 örnekle gösterin.

> 💡 Tek bir doğruluk sayısı yeterli değil. **Nerede** yanıldığını gösterin.

**3.3 — Aksiyon önerisi (5 puan)**

Terk riski yüksek olup olumsuz geri bildirimi de bulunan müşterilerden **3 tanesini** seçin. Her biri için LLM'e, müşteri temsilcisine yönelik **tek cümlelik bir aksiyon önerisi** ürettirin.

Ardından şu soruyu yanıtlayın: *LLM'in şirketin sunmadığı bir kampanyayı (örneğin "%50 indirim") önermesini nasıl engellersiniz?*

> **API erişimi yoksa sorun değil.** Ücretsiz katman kullanın, örneklemi küçültün veya sohbet arayüzünden toplu işleyip yöntemi belgeleyin. **Yöntemin şeffaflığı, ölçeğinden önemlidir.**

---

## Bölüm 4 — Hazır raporun eleştirisi · 15 puan

`analiz_raporu_taslak.md` dosyasında, aynı veri seti üzerinde hazırlanmış bir analiz raporu taslağı var.

**Bu rapor hatalar içeriyor.** Bazıları teknik, bazıları yorum hatası, en az biri veriyle uyuşmayan bir sayı.

**En az 4 hata bulun.** Her biri için:

| Ne | Nasıl yazılacak |
|---|---|
| **Hata** | Hangi cümle/iddia yanlış |
| **Neden yanlış** | Kısa teknik gerekçe |
| **Doğrusu** | Ne olmalıydı |

> 💡 Bu bölüm, günlük işimizde en sık yaptığımız şeyin provası: bir yapay zekâ aracının veya bir meslektaşın ürettiği analizi gözden geçirmek.

---

## Bölüm 5 — Yapay zekâ kullanım günlüğü · 5 puan

Yarım sayfa yeterli.

1. **Hangi araçları** kullandınız, hangi işler için?
2. **En işe yarayan 2 istem (prompt)** — kopyalayıp yapıştırın
3. ⭐ **Aracın yanıldığı 1 örnek** — ne dedi, neden yanlıştı, **nasıl fark ettiniz?**
4. Hangi işi **bilinçli olarak** yapay zekâya devretmediniz, neden?

> 3. madde bu bölümün asıl sorusudur. "Hiç yanılmadı" cevabı, aracı yeterince zorlamadığınız anlamına gelir.

---

## Teslim formatı

```
teslim/
├── RAPOR.md (veya .pdf)     ← ana rapor, tüm bölümlerin cevapları
├── kod/                      ← notebook veya .py
├── ciktilar/
│   ├── llm_etiketleri.csv
│   └── altin_set.csv         ← elle etiketlediğiniz 40 kayıt
└── README.md                 ← nasıl çalıştırılır
```

- Türkçe veya İngilizce, fark etmez
- Kodu rapora yapıştırmayın, referans verin
- **Kısa ve net olan kazanır.** Rapor uzunluğu puan getirmez.

---

## Değerlendirme ağırlıkları

| Bölüm | Puan |
|---|---|
| 1 · Veri hazırlığı ve keşif | 25 |
| 2 · Model | 30 |
| 3 · LLM ile metin analizi | 25 |
| 4 · Rapor eleştirisi | 15 |
| 5 · YZ kullanım günlüğü | 5 |
| **Toplam** | **100** |

### Bonus (zorunlu değil, +5'e kadar)

Bölüm 3.2'deki bulgulara dayanarak bir iyileştirme yapın (istem değişikliği, örnek ekleme, farklı model) ve aynı altın setle **önce/sonra** karşılaştırın.

---

## Değerlendirmede aradıklarımız

**Öncelikli:**
- Kararlarınızı gerekçelendirmeniz — "şunu yaptım" değil, "şunu şu yüzden yaptım"
- Kendi çalışmanızdaki sınırları görmeniz
- Bulduğunuz sorunları saklamadan raporlamanız
- İş sorusuyla teknik çözüm arasındaki bağı kurmanız

**Öncelikli değil:**
- Model performansının son kırıntısı
- Kütüphane seçimi
- Rapor uzunluğu

**Eksik bırakmak sorun değil.** Bir bölümü yetiştiremezseniz, **ne yapardınız ve neden yetişmediğini yazın.** Dürüstçe yarım bırakılmış bir bölüm, aceleye getirilmiş bir bölümden yüksek puan alır.

---

## Sorular

Vakayla ilgili soru sormaktan çekinmeyin — eksi puan değildir. Cevap gelmezse **kendi varsayımınızı yapıp raporda belirtin.**

Kolay gelsin.
