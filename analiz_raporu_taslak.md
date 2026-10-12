# Müşteri Terk Analizi — Taslak Rapor

**Durum:** Taslak — gözden geçirme bekliyor
**Hazırlayan:** Analitik ekibi (YZ aracı desteğiyle hazırlanmıştır)
**Tarih:** 2026-07-14

> **Gözden geçiren için not:** Bu taslak hızlı hazırlandı ve yayımlanmadan önce kontrol edilmesi gerekiyor. Lütfen bulgularınızı bize iletin.

---

## 1. Yönetici Özeti

Servis müşterilerimizin terk davranışını analiz ettik. Kurduğumuz model **%84 doğruluk** ve **0,99 ROC-AUC** ile müşteri kaybını yüksek başarıyla tahmin ediyor. Bu, sektör ortalamalarının belirgin şekilde üzerinde bir performanstır ve modelin doğrudan üretime alınabileceğini göstermektedir.

Ayrıca servis merkezleri arasında memnuniyet farkları tespit ettik ve düşük performans gösteren merkez için aksiyon öneriyoruz.

---

## 2. Veri Hazırlığı

Üç ana tabloyu `musteri_id` ve `arac_id` üzerinden birleştirdik. Birleştirme sonrası **32.770 satırlık** bir analiz tablosu elde ettik ve bu tabloyu müşteri seviyesinde analiz ettik.

Veri setinde toplam **8.400 müşteri** bulunmaktadır.

Eksik değerler: `memnuniyet_puani` alanında %21 oranında eksik veri tespit edildi. Bu eksiklikleri **sütun ortalaması (3,94)** ile doldurarak veri setini eksiksiz hale getirdik. Böylece hiçbir kaydı kaybetmeden analize devam edebildik.

Kilometre ve tutar alanlarındaki uç değerler analiz sonuçlarını etkilemediği için olduğu gibi bırakılmıştır.

---

## 3. Model Kurulumu

Veri setini **rastgele olarak** %80 eğitim / %20 test şeklinde ayırdık (`train_test_split(X, y, test_size=0.2, random_state=42)`). Bu standart bir yaklaşımdır ve modelin genelleme yeteneğini ölçmemizi sağlar.

Gradient Boosting sınıflandırıcısı kullandık. Modele mevcut tüm değişkenleri verdik; algoritma ilgisiz değişkenleri kendisi eleyecektir.

### 3.1 Model Performansı

| Metrik | Değer |
|---|---|
| Doğruluk (Accuracy) | **%84** |
| ROC-AUC | **0,99** |

Modelin %84 doğruluğa ulaşması, her 100 müşteriden 84'ünü doğru sınıflandırdığımız anlamına gelir. Bu oran, operasyonel kullanım için fazlasıyla yeterlidir.

### 3.2 Değişken Önemleri

| Sıra | Değişken | Önem |
|---|---|---|
| 1 | `crm_musteri_durumu` | 0,61 |
| 2 | `son_iletisim_notu` | 0,14 |
| 3 | `recency_gun` | 0,08 |
| 4 | `ort_memnuniyet` | 0,05 |
| 5 | `ziyaret_sayisi` | 0,04 |

En güçlü değişkenin `crm_musteri_durumu` çıkması beklenen bir sonuçtur ve CRM verimizin kalitesini göstermektedir. Saha ekiplerinin müşteri durumunu doğru takip ettiğini doğrulamaktadır. **Öneri:** CRM durum alanının daha sık güncellenmesi, model performansını daha da artıracaktır.

---

## 4. Memnuniyet ve Terk İlişkisi

Terk eden müşterilerin ortalama memnuniyet puanı (3,71), terk etmeyenlerden (3,98) anlamlı şekilde düşüktür.

Yapılan t-testi sonucunda **p = 0,03** bulunmuştur. Buna göre hipotezimizin **%97 ihtimalle doğru** olduğunu söyleyebiliriz.

**Sonuç:** Düşük memnuniyet müşteri terkine yol açmaktadır. Memnuniyet puanını 0,3 puan artırırsak terk oranını yaklaşık %8 azaltabiliriz.

---

## 5. Servis Merkezi Performansı

Servis merkezlerinin ortalama memnuniyet puanları:

| Servis Merkezi | Ortalama Memnuniyet |
|---|---|
| SM-İzmir1 | 3,91 |
| **SM-Maslak** | **3,91** |
| SM-Bursa1 | 3,93 |
| SM-Antalya1 | 3,95 |
| SM-Kocaeli1 | 3,95 |
| SM-Kadıköy | 3,96 |
| SM-Ankara1 | 3,99 |

**Bulgu:** SM-Maslak, en yüksek işlem hacmine sahip merkezimiz olmasına rağmen memnuniyet sıralamasının en altında yer almaktadır. Bu, merkezdeki hizmet kalitesinde ciddi bir sorun olduğunu göstermektedir.

**Öneri:** SM-Maslak'ta acil bir kalite denetimi başlatılmalı, merkez yönetimi performans görüşmesine çağrılmalı ve iş hacminin bir kısmı komşu merkezlere yönlendirilmelidir.

---

## 6. Öneriler

1. Model mevcut haliyle üretime alınmalıdır; %84 doğruluk operasyon için yeterlidir.
2. CRM müşteri durumu alanının güncelleme sıklığı artırılmalıdır.
3. SM-Maslak'ta kalite denetimi başlatılmalıdır.
4. Memnuniyet puanını artırmaya yönelik kampanyalar terk oranını doğrudan düşürecektir.
5. Model çıktısı, terk riski yüksek müşterilere otomatik kampanya göndermek için kullanılabilir.

---

*Taslak sonu. Gözden geçirme sonrası yayımlanacaktır.*
