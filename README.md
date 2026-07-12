# Walmart Sales Dashboard (Power BI)

Walmart mağaza satışları üzerine tanımlayıcı analiz dashboard'u.
**v2**: tekli `Walmart_Sales.csv` + `stores.csv` ile sadeleştirilmiş veri
modeli (Dept/MarkDown/forecast ufku bu sürümde yok).

## 📁 Repo yapısı

```
walmart-sales-dashboard/
├── README.md
├── .gitignore
├── data/
│   └── raw/
│       ├── Walmart_Sales.csv   # 6.435 satır, 45 mağaza × 143 hafta
│       └── stores.csv          # 45 mağaza, Type (A/B/C), Size
├── power-query/
│   └── README_ADIMLAR.md       # her sorgu için hazır M kodu
├── docs/
│   ├── DATA_MODEL.md           # yıldız şema + ilişkiler
│   ├── DAX_MEASURES.md         # sayfa bazlı DAX ölçüleri
│   └── screenshots/            # sayfa görselleri (PNG)
└── WalmartSalesDashboard.pbix
```

## 🗂️ Veri sözlüğü

| Tablo | Satır | Anahtar | İçerik |
|---|---|---|---|
| stores | 45 | Store | Dim tablo — Type, Size |
| Walmart_Sales | 6.435 | Store+Date | Fact — Weekly_Sales, IsHoliday, Temperature, Fuel_Price, CPI, Unemployment |

Veri aralığı: **2010-02-05 → 2012-10-26**, boş/negatif değer yok.

## ⚠️ Bilinen kısıtlar (v1'den farkı)
- **Departman (Dept) kırılımı yok** — satış zaten mağaza+hafta bazında.
- **Promosyon/MarkDown verisi yok** — "Kampanyalar" sayfası bu yüzden
  **tatil haftası etkisi** (Super Bowl, Labor Day, Thanksgiving, Christmas)
  üzerine kuruldu, indirim kampanyası analizi değil.
- **Forecast ufku yok** — veri 2012-10-26'da bitiyor, gelecek tahmini için
  ayrı bir yöntem (Power BI Analytics → Forecast) gerekecek, karşılaştırma
  yapacak gerçek veri olmayacak.
- Mağaza şehir/eyalet bilgisi hâlâ yok (harita görseli kapsam dışı).

## 🚀 Kurulum adımları
1. `data/raw/` klasörüne 2 CSV'yi koyun.
2. Power BI Desktop → Get Data → CSV, her iki dosyayı içe aktarın.
3. `power-query/README_ADIMLAR.md` içindeki M kodlarını sırayla
   **Advanced Editor**'e uygulayın (Stores → Fact_Sales → Calendar →
   Holidays).
4. `docs/DATA_MODEL.md`'deki ilişkileri kurun.
5. `docs/DAX_MEASURES.md`'deki ölçüleri ekleyin.
6. Sayfaları kurun: Genel Bakış, Mağaza Analizi, Ekonomik Etkiler,
   Kampanyalar (tatil etkisi versiyonu).

## .gitignore önerisi
```
*.pbix.bak
~$*.pbix
data/processed/
.DS_Store
```
