# Veri Modeli (v2 — Walmart_Sales.csv + stores.csv)

Önceki 4 dosyalı yapıdan çok daha basit: artık klasik **2 boyut + 1 fact**
yıldız şeması.

```
                 ┌───────────────┐
                 │ Dim_Calendar  │
                 │ Date (PK)     │
                 └──────┬────────┘
                        │ 1
                        │ *
                 ┌──────▼────────────┐
                 │    Fact_Sales      │
                 │ Store, Date        │
                 │ Weekly_Sales       │
                 │ IsHoliday          │
                 │ Temperature        │
                 │ Fuel_Price         │
                 │ CPI, Unemployment  │
                 └──────┬─────────────┘
                        │ * (Store)
                        │ 1
                 ┌──────▼──────┐
                 │ Dim_Stores  │
                 │ Store (PK)  │
                 │ Type, Size  │
                 └─────────────┘

Dim_Calendar[Date] ── 1:1 ──> Dim_Holidays[Date]
```

## İlişkiler (Model görünümü)
| Tablo A | Kolon | Tablo B | Kolon | Kardinalite | Yön |
|---|---|---|---|---|---|
| Fact_Sales | Store | Dim_Stores | Store | Çok:1 | Tek yön |
| Fact_Sales | Date | Dim_Calendar | Date | Çok:1 | Tek yön |
| Dim_Holidays | Date | Dim_Calendar | Date | 1:1 | Tek yön |

## Önceki modele göre ne değişti?
| Konu | v1 (train/test/features/stores) | v2 (Walmart_Sales + stores) |
|---|---|---|
| Fact tablo sayısı | 2 (Sales + Features) | 1 (birleşik) |
| Departman kırılımı | Var (Dept) | **Yok** — satış zaten mağaza+hafta bazında toplanmış |
| Promosyon verisi | MarkDown1-5 | **Yok** — sadece IsHoliday var |
| Mağaza tipi/büyüklüğü | Var | Var (stores.csv ile korunuyor) |
| Forecast ufku (test.csv) | Var (2012-11→2013-07) | **Yok** — veri 2012-10-26'da bitiyor |
| Satır sayısı | 421.571 (train) | 6.435 |
| Veri kalitesi | NA/negatif değer temizliği gerekli | Temiz, boş/negatif değer yok |

## Etkilenen dashboard sayfaları
- **Genel Bakış** — aynen kurulabilir (KPI kartları, haftalık trend)
- **Mağaza Analizi** — Type/Size ile aynen kurulabilir; sadece **Dept bazlı**
  kırılımlar (örn. "en çok satan departman") artık **yapılamaz**
- **Ekonomik Etkiler** — aynen kurulabilir, hatta daha temiz (CPI/Unemployment
  boşluğu yok)
- **Kampanyalar** — MarkDown verisi olmadığı için içerik değişti: artık
  "promosyon etkisi" yerine **"tatil haftası satış etkisi"** (Super Bowl,
  Labor Day, Thanksgiving, Christmas karşılaştırması) üzerine kuruluyor —
  bkz. `DAX_MEASURES.md`
- **Forecast sayfası** — bu veri setiyle **kurulamaz** (test/gelecek ufku
  yok). İleride forecast eklemek isterseniz `CALENDAR()` ile gelecek
  tarihler üretip Power BI'ın Analytics → Forecast özelliğini
  kullanabilirsiniz, ama gerçek karşılaştırma verisi olmaz.
