# DAX Ölçüleri (v2 — Walmart_Sales.csv + stores.csv)

## Ortak temel ölçüler
```dax
Toplam Satış = SUM(Fact_Sales[Weekly_Sales])

Ortalama Haftalık Satış =
AVERAGEX(VALUES(Dim_Calendar[Date]), [Toplam Satış])

Mağaza Sayısı = DISTINCTCOUNT(Fact_Sales[Store])

Tatil Haftası Satışı =
CALCULATE([Toplam Satış], Fact_Sales[IsHoliday] = TRUE)

Normal Hafta Satışı =
CALCULATE([Toplam Satış], Fact_Sales[IsHoliday] = FALSE)
```
> Not: Bu veri setinde negatif satış/iade kaydı yok, bu yüzden önceki
> planımdaki "İade Tutarı" ölçüsüne artık gerek yok.

## Genel Bakış sayfası
```dax
YBB Satış (Yıllık Büyüme) =
VAR GecenYil = CALCULATE([Toplam Satış], SAMEPERIODLASTYEAR(Dim_Calendar[Date]))
RETURN DIVIDE([Toplam Satış] - GecenYil, GecenYil)

Kümülatif Satış (YTD) =
TOTALYTD([Toplam Satış], Dim_Calendar[Date])
```

## Mağaza Analizi sayfası
```dax
Mağaza Başına Ortalama Satış =
AVERAGEX(VALUES(Dim_Stores[Store]), [Toplam Satış])

m² Başına Satış =
DIVIDE([Toplam Satış], SUM(Dim_Stores[Size]))

Tip Bazında Ciro Payı (%) =
DIVIDE([Toplam Satış], CALCULATE([Toplam Satış], ALL(Dim_Stores[Type])))
```
> Bu ölçü tam olarak ekran görüntünüzdeki "Mağaza Tiplerinin Toplam Ciro
> İçindeki Payı" donut grafiğini üretir (A/B/C kırılımında).

## Ekonomik Etkiler sayfası
```dax
Ortalama CPI = AVERAGE(Fact_Sales[CPI])
Ortalama İşsizlik = AVERAGE(Fact_Sales[Unemployment])
Ortalama Yakıt Fiyatı = AVERAGE(Fact_Sales[Fuel_Price])
Ortalama Sıcaklık = AVERAGE(Fact_Sales[Temperature])
```
> Bu veri setinde CPI/Unemployment'ta boşluk olmadığı için filtreleme
> gerekmiyor — v1'deki `IsForecastPeriod` uyarısına burada gerek yok.

## Kampanyalar sayfası — YENİDEN KURGULANDI
MarkDown verisi olmadığı için bu sayfa artık **tatil haftalarının satışa
etkisi** üzerine kuruluyor. `Dim_Holidays[HolidayName]` ile kırılım alın.

```dax
Tatil Etkisi (%) =
VAR TatilOrt = CALCULATE([Ortalama Haftalık Satış], Fact_Sales[IsHoliday] = TRUE)
VAR NormalOrt = CALCULATE([Ortalama Haftalık Satış], Fact_Sales[IsHoliday] = FALSE)
RETURN DIVIDE(TatilOrt - NormalOrt, NormalOrt)

Tatile Göre Ortalama Satış =
CALCULATE(
    [Ortalama Haftalık Satış],
    TREATAS(VALUES(Dim_Holidays[HolidayName]), Dim_Holidays[HolidayName])
)
```
Görsel önerisi: X ekseninde `HolidayName` (Super Bowl / Labor Day /
Thanksgiving / Christmas), Y ekseninde `Tatile Göre Ortalama Satış` —
hangi tatilin satışı en çok artırdığını gösteren bar chart.

## Kaldırılan ölçüler (bu veri setinde uygulanamaz)
- `Toplam Markdown`, `Markdown'lı Haftalarda Satış`, `Promosyon Etkisi (%)`
  → MarkDown kolonları yok
- `En Yüksek Cirolu Departman` → Dept kolonu yok
- `Net Satış (İadesiz)`, `İade Tutarı` → negatif satış kaydı yok
