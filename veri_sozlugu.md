# Veri Sözlüğü

Dört dosya, `veri/` klasöründe. Tümü UTF-8 kodlu CSV.

---

## `musteriler.csv`

Müşteri ana kartı. CRM sisteminden alınmıştır.

| Kolon | Tip | Açıklama |
|---|---|---|
| `musteri_id` | tam sayı | Müşteri numarası |
| `sehir` | metin | Müşterinin kayıtlı olduğu şehir |
| `musteri_tipi` | metin | `Bireysel` veya `Kurumsal` (filo müşterileri kurumsaldır) |
| `dogum_yili` | ondalık | Doğum yılı (kurumsal müşterilerde anlamsız olabilir) |
| `ilk_temas_tarihi` | tarih | Müşterinin sistemde ilk göründüğü tarih |
| `kayit_kanali` | metin | `Bayi`, `Web`, `Çağrı Merkezi`, `Mobil Uygulama` |
| `crm_musteri_durumu` | metin | **CRM'de bugün itibarıyla görünen müşteri durumu.** `Aktif`, `Pasif`, `Kaybedildi`. Saha ekipleri tarafından güncellenir. |
| `son_iletisim_notu` | metin | **Müşteriyle yapılan son görüşmeye ait serbest not.** Çağrı merkezi tarafından girilir, tarihsizdir. |

---

## `araclar.csv`

Araç kayıtları. Bir müşterinin birden fazla aracı olabilir.

| Kolon | Tip | Açıklama |
|---|---|---|
| `arac_id` | tam sayı | Araç numarası |
| `musteri_id` | tam sayı | Aracın sahibi |
| `marka` | metin | `Marka-A`, `Marka-B`, `Marka-C` (anonimleştirilmiş) |
| `model` | metin | Model adı |
| `model_yili` | tam sayı | Aracın model yılı |
| `satin_alma_tarihi` | tarih | Satın alma tarihi |
| `garanti_bitis_tarihi` | tarih | Fabrika garantisinin bittiği tarih |
| `ilk_km` | tam sayı | Satın alma anındaki kilometre (×1000) |

---

## `servis_kayitlari.csv`

Servis işlem kayıtları. Ana işlem tablosudur.

| Kolon | Tip | Açıklama |
|---|---|---|
| `kayit_id` | tam sayı | İşlem numarası |
| `arac_id` | tam sayı | İşlemin yapıldığı araç |
| `servis_tarihi` | metin | İşlem tarihi. **Kaynak sistemler farklı olduğu için biçim tek tip değildir.** |
| `servis_tipi` | metin | `Periyodik Bakım`, `Arıza Onarım`, `Kaporta/Boya`, `Lastik/Balans` |
| `servis_merkezi` | metin | İşlemin yapıldığı servis merkezi |
| `km` | tam sayı | İşlem anındaki kilometre (×1000) |
| `tutar` | ondalık | Fatura tutarı (TL) |
| `memnuniyet_puani` | ondalık | İşlem sonrası anket puanı, 1–5. **Anket her işlemden sonra gönderilir, cevaplama zorunlu değildir.** |

---

## `geri_bildirimler.csv`

Serbest metin müşteri geri bildirimleri. Farklı kanallardan toplanmıştır.

| Kolon | Tip | Açıklama |
|---|---|---|
| `geri_bildirim_id` | tam sayı | Geri bildirim numarası |
| `arac_id` | tam sayı | İlgili araç |
| `tarih` | tarih | Geri bildirim tarihi |
| `kanal` | metin | `Anket`, `Çağrı Merkezi`, `E-posta`, `Google Yorum` |
| `metin` | metin | Müşterinin yazdığı serbest metin (Türkçe) |

---

## Tablolar arası ilişki

```
musteriler (1) ──< araclar (N) ──< servis_kayitlari (N)
                        │
                        └──< geri_bildirimler (N)
```

---

## Notlar

- Veri sentetiktir. Gerçek müşteri verisi değildir.
- Tutarlar ve kilometreler gerçekçi ölçeklere yaklaştırılmıştır ancak gerçek fiyat verisi değildir.
- Dosyalar üretim sistemlerinden **ham** olarak alınmıştır; herhangi bir temizlik uygulanmamıştır.
