# Power Query Temizlik Adımları (v2 — Walmart_Sales.csv + stores.csv)

Önceki plandan farkı: artık **tek fact tablo** var (Dept ve MarkDown yok).
`Date` formatı bu dosyada **DD-MM-YYYY** — önceki setten farklı, dikkat!

---

## 1) Stores (Dim_Stores) — değişmedi

```m
let
    Kaynak = Csv.Document(File.Contents("C:\...\stores.csv"),
        [Delimiter=",", Columns=3, Encoding=65001, QuoteStyle=QuoteStyle.None]),
    Basliklar = Table.PromoteHeaders(Kaynak, [PromoteAllScalars=true]),
    Tipler = Table.TransformColumnTypes(Basliklar,{
        {"Store", Int64.Type}, {"Type", type text}, {"Size", Int64.Type}
    })
in
    Tipler
```

---

## 2) Fact_Sales (Walmart_Sales.csv)
Tek tabloda satış + ekonomik göstergeler + tatil bayrağı birlikte geliyor.
`Date` metnini **gün-ay-yıl** olarak manuel parse ediyoruz (locale hatası
riskini önlemek için `Text.Split` ile) — doğrudan `type date` ile
dönüştürürseniz sisteminizin bölge ayarına göre gün/ay karışabilir.

```m
let
    Kaynak = Csv.Document(File.Contents("C:\...\Walmart_Sales.csv"),
        [Delimiter=",", Columns=8, Encoding=65001, QuoteStyle=QuoteStyle.None]),
    Basliklar = Table.PromoteHeaders(Kaynak, [PromoteAllScalars=true]),
    TarihDonusum = Table.TransformColumns(Basliklar, {
        {"Date", each let
                p = Text.Split(_, "-"),
                g = Number.FromText(p{0}),
                a = Number.FromText(p{1}),
                y = Number.FromText(p{2})
            in #date(y, a, g), type date}
    }),
    Tipler = Table.TransformColumnTypes(TarihDonusum, {
        {"Store", Int64.Type}, {"Weekly_Sales", type number},
        {"Holiday_Flag", type logical}, {"Temperature", type number},
        {"Fuel_Price", type number}, {"CPI", type number},
        {"Unemployment", type number}
    }),
    YenidenAdlandir = Table.RenameColumns(Tipler, {{"Holiday_Flag", "IsHoliday"}})
in
    YenidenAdlandir
```

> Kontrol: Yüklendikten sonra `Date.Min` = **05.02.2010**, `Date.Max` =
> **26.10.2012**, satır sayısı **6.435** olmalı. Farklı çıkarsa tarih
> parse adımı yanlış çalışmış demektir.

---

## 3) Calendar (Dim_Calendar) — tarih aralığı güncellendi
Ekran görüntünüzdeki ay sıralama hatasının çözümü değişmedi, sadece
`BitisTarihi` bu veri setinin son tarihine çekildi (artık forecast/2013
dönemi yok, bu yüzden `IsForecastPeriod` kolonu da kaldırıldı).

```m
let
    BaslangicTarihi = #date(2010,2,5),
    BitisTarihi = #date(2012,10,26),
    TarihListesi = List.Dates(BaslangicTarihi,
        Duration.Days(BitisTarihi - BaslangicTarihi) + 1, #duration(1,0,0,0)),
    Tablo = Table.FromList(TarihListesi, Splitter.SplitByNothing(), {"Date"}),
    Tipler = Table.TransformColumnTypes(Tablo, {{"Date", type date}}),
    Yil = Table.AddColumn(Tipler, "Year", each Date.Year([Date]), Int64.Type),
    Ay = Table.AddColumn(Yil, "MonthNumber", each Date.Month([Date]), Int64.Type),
    AyAdiTR = Table.AddColumn(Ay, "MonthName",
        each {"Ocak","Şubat","Mart","Nisan","Mayıs","Haziran","Temmuz",
              "Ağustos","Eylül","Ekim","Kasım","Aralık"}{[MonthNumber]-1}, type text),
    YilAy = Table.AddColumn(AyAdiTR, "YearMonth",
        each Text.From([Year]) & "-" & Text.PadStart(Text.From([MonthNumber]),2,"0"), type text),
    SiraAnahtari = Table.AddColumn(YilAy, "YearMonthSortKey",
        each [Year]*100 + [MonthNumber], Int64.Type),
    HaftaNo = Table.AddColumn(SiraAnahtari, "ISOWeek",
        each Date.WeekOfYear([Date], Day.Monday), Int64.Type)
in
    HaftaNo
```

Kurulumdan sonra: **Model görünümü → YearMonth → Sort by Column →
YearMonthSortKey**, aynısını **MonthName → Sort by Column → MonthNumber**
için uygulayın.

---

## 4) Holidays (Dim_Holidays) — sadece bu veri setindeki 10 tatil haftası
Veri 2012-10-26'da bittiği için 2012 Thanksgiving/Christmas ve 2013 Super
Bowl bu sette **yok** — eklemeyin, model ile uyuşmaz.

```m
let
    Kaynak = #table(
        {"Date","HolidayName"},
        {
            {#date(2010,2,12), "Super Bowl"},   {#date(2011,2,11), "Super Bowl"},
            {#date(2012,2,10), "Super Bowl"},
            {#date(2010,9,10), "Labor Day"},    {#date(2011,9,9),  "Labor Day"},
            {#date(2012,9,7),  "Labor Day"},
            {#date(2010,11,26),"Thanksgiving"}, {#date(2011,11,25),"Thanksgiving"},
            {#date(2010,12,31),"Christmas"},    {#date(2011,12,30),"Christmas"}
        }
    ),
    Tipler = Table.TransformColumnTypes(Kaynak, {{"Date", type date}, {"HolidayName", type text}})
in
    Tipler
```

---

## Kaldırılan sorgular (bu veri setinde gerek yok)
- ~~Features~~ → Weekly_Sales tablosuna zaten dahil (CPI, Unemployment,
  Temperature, Fuel_Price aynı satırda)
- ~~Sales Train / Test ayrımı~~ → tek tablo, ayrım yok
- MarkDown1-5 → bu veri setinde **yok**, "Kampanyalar" sayfası artık
  `IsHoliday` + `HolidayName` üzerinden kurulacak (bkz. DAX_MEASURES.md)
