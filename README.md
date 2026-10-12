# Müşteri Terk Analizi ve Geri Bildirim Analizi — Teknik Vaka

Ana rapor: [`RAPOR.md`](RAPOR.md)

## Klasör yapısı

```
teslim/
├── RAPOR.md                     ← ana rapor (bütün bölümlerin cevapları)
├── README.md                    ← bu dosya
├── requirements.txt             ← gerekli Python paketleri
├── veri/                        ← 4 ham CSV dosyası (vaka ile verilen, değiştirilmedi)
├── kod/
│   ├── bolum1_veri.ipynb        ← Bölüm 1: veri hazırlığı, veri kalitesi, keşifsel analiz, servis merkezi sorusu
│   ├── bolum2_model.ipynb       ← Bölüm 2: terk modeli
│   ├── bolum3_llm.ipynb         ← Bölüm 3: LLM ile etiketleme (Prompt 1), kalite ölçümü, aksiyon önerisi
│   └── bolum3_bonus.ipynb       ← Bonus: Prompt 2 ve önce/sonra karşılaştırması
└── ciktilar/
    ├── llm_etiketleri.csv       ← LLM etiketleri, Prompt 1 (200 geri bildirim)
    ├── llm_etiketleri_v2.csv    ← LLM etiketleri, Prompt 2 (bonus)
    ├── altin_set.csv            ← elle etiketlenmiş 40 kayıt (vakanın istediği altın set)
    ├── altin_etiket.csv         ← elle etiketlenmiş 35 farklı metin (200 kaydın tamamını kapsıyor)
    ├── etiketleme_kilavuzu.md   ← elle etiketlemede kullanılan kurallar
    ├── orneklem_200.csv         ← rastgele seçilen 200 geri bildirim
    ├── analiz_tablosu.csv       ← müşteri seviyesinde analiz tablosu (Bölüm 1 → Bölüm 2)
    ├── risk_skorlari.csv        ← modelin risk skorları (Bölüm 2 → Bölüm 3.3)
    └── *.png                    ← rapordaki grafikler
```

## Kurulum

Python 3.11 ile test edildi.

```bash
pip install -r requirements.txt
```

## Çalıştırma

Notebook'lar `kod/` klasörü içinden, **sırayla** çalıştırılmalı; her biri bir öncekinin ürettiği dosyaları kullanıyor:

| Sıra | Notebook | Okuduğu | Ürettiği |
|---|---|---|---|
| 1 | `bolum1_veri.ipynb` | `veri/*.csv` | `analiz_tablosu.csv`, grafikler |
| 2 | `bolum2_model.ipynb` | `analiz_tablosu.csv` | `risk_skorlari.csv`, grafikler |
| 3 | `bolum3_llm.ipynb` | `veri/`, `altin_etiket.csv`, `risk_skorlari.csv` | `orneklem_200.csv`, `llm_etiketleri.csv`, `altin_set.csv` |
| 4 | `bolum3_bonus.ipynb` | `orneklem_200.csv`, `altin_etiket.csv`, `llm_etiketleri.csv` | `llm_etiketleri_v2.csv` |

Her notebook için: **Restart → Run All.**

## Bölüm 3 hakkında (LLM)

- **Model:** Groq üzerinden `openai/gpt-oss-20b`, `temperature=0`. Etiketlemede (Prompt 1 ve 2) modelin varsayılan akıl yürütme ayarı, 3.3'teki önerilerde `reasoning_effort='low'` kullanıldı. İlk denemelerde `gpt-oss-120b` kullanıldı; günlük token kotası dolduğu için aynı ailenin küçük modeline geçildi.
- **API anahtarı:** Koda yazılmadı. Notebook çalışırken `GROQ_API_KEY` ortam değişkenine bakar; yoksa anahtarı sorar. Ücretsiz anahtar: https://console.groq.com
- **LLM'e yeniden gitmeden çalıştırma:** İki notebook'un başındaki `LLM_YENIDEN = False` ayarıyla `ciktilar/` içindeki kayıtlı etiketler okunur, LLM'e istek gönderilmez. Raporlanan sayılar bu dosyalardan üretildi. `True` yapılırsa etiketleme yeniden yapılır; LLM aynı metne her çalıştırmada küçük farklarla cevap verebildiği için sonuçlar birebir aynı çıkmayabilir.
- **Prompt 1** 200 geri bildirimin her birini ayrı istekle etiketledi (aynı metne verilen cevapların tutarlılığı da ölçülebiliyor).
- **Prompt 2 (bonus)**, ücretsiz katmanın günlük token kotası (model başına 200.000) nedeniyle 35 farklı metnin her biri için bir kez çalıştırıldı ve sonuç metnin 200 yorumdaki tekrarlarına uygulandı (`SADECE_FARKLI_METIN = True`). Doğruluk karşılaştırması bundan etkilenmiyor; Prompt 2'nin tutarlılığı ölçülmedi.
- **3.3'teki öneri hücreleri** (kısıtsız ve kısıtlı prompt) her çalıştırmada LLM'e birkaç istek gönderir; öneri metinleri çalıştırmadan çalıştırmaya küçük farklar gösterebilir.
- **Elle etiketler:** `altin_etiket.csv` elle oluşturuldu (kurallar: `etiketleme_kilavuzu.md`). Şablonu üreten hücre, dosya varsa üzerine yazmaz.

## Tekrarlanabilirlik

- Rastgelelik içeren bütün adımlarda `random_state=42` kullanıldı (örneklem, altın set, çapraz doğrulama).
- Ham veri dosyaları değiştirilmedi; bütün temizlik adımları notebook içinde.