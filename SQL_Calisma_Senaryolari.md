# 🗄️ SQL Çalışma Senaryoları
**Veri Seti:** İngiltere E-Ticaret Verisi | **Toplam Soru:** 30
---

## 📊 Veri Seti Özeti

### Tablolar

**`customers`** — 1.200 kayıt
| Kolon | Tip | Açıklama |
|---|---|---|
| customer_id | VARCHAR | Müşteri ID (C0001…C1200) |
| gender | VARCHAR | Male / Female |
| age | INT | Yaş |
| city | VARCHAR | 8 şehir (London, Manchester…) |
| signup_date | DATE | Kayıt tarihi |
| loyalty_member | VARCHAR | Yes / No |

**`orders`** — 4.000 kayıt
| Kolon | Tip | Açıklama |
|---|---|---|
| order_id | VARCHAR | Sipariş ID (O00001…) |
| customer_id | VARCHAR | FK → customers |
| product_id | VARCHAR | FK → products |
| order_date | DATE | 2023-01-01 → 2024-05-15 |
| quantity | INT | Sipariş adedi |
| payment_method | VARCHAR | Card / Cash / Online |

**`products`** — 80 kayıt
| Kolon | Tip | Açıklama |
|---|---|---|
| product_id | VARCHAR | Ürün ID (P001…P080) |
| product_name | VARCHAR | Ürün adı |
| category | VARCHAR | Beauty, Home, Electronics, Clothing, Sports |
| price | DECIMAL | 11.07 → 289.84 |

### İlişki Diyagramı
```
customers (customer_id) ──< orders (customer_id)
products  (product_id)  ──< orders (product_id)
```

---

## 🟢 SEVİYE 1 — Başlangıç (Temel SELECT, WHERE, ORDER BY)

---

### Soru 1 — Şehre Göre Müşteri Listesi

**📋 Görev:** London'da yaşayan, 30 yaşından büyük tüm müşterileri yaşa göre küçükten büyüğe listele.

**✅ Çözüm:**
```sql
SELECT
    customer_id,
    gender,
    age,
    city,
    signup_date
FROM customers
WHERE city = 'London'
  AND age > 30
ORDER BY age ASC;
```

---

### Soru 2 — Pahalı Ürünler

**📋 Görev:** Fiyatı 200'ün üzerinde olan ürünleri, en pahalıdan en ucuza sıralayarak getir.

**✅ Çözüm:**
```sql
SELECT
    product_id,
    product_name,
    category,
    price
FROM products
WHERE price > 200
ORDER BY price DESC;
```

---

### Soru 3 — Ödeme Yöntemi Çeşitleri

**📋 Görev:** Siparişlerde kullanılan tüm farklı ödeme yöntemlerini tekrarsız listele.

**✅ Çözüm:**
```sql
SELECT DISTINCT payment_method
FROM orders;
```

---

### Soru 4 — Belirli Tarih Aralığında Siparişler

**📋 Görev:** 2024 yılının ilk çeyreğinde (Ocak–Mart) verilen siparişleri listele.

**✅ Çözüm:**
```sql
SELECT *
FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-03-31'
ORDER BY order_date;
```

---

### Soru 5 — Kategori Bazlı Ürün Sayısı

**📋 Görev:** Her kategoride kaç ürün olduğunu bul.

**✅ Çözüm:**
```sql
SELECT
    category,
    COUNT(*) AS urun_sayisi
FROM products
GROUP BY category
ORDER BY urun_sayisi DESC;
```

---

## 🟡 SEVİYE 2 — Orta (JOIN, GROUP BY + HAVING, Aggregate Fonksiyonlar)

---

### Soru 6 — Müşteri Başına Toplam Harcama

**📋 Görev:** Her müşterinin toplam ne kadar harcama yaptığını hesapla (fiyat × miktar). En çok harcayan 10 müşteriyi göster.

**✅ Çözüm:**
```sql
SELECT
    o.customer_id,
    ROUND(SUM(o.quantity * p.price), 2) AS toplam_harcama
FROM orders o
JOIN products p ON o.product_id = p.product_id
GROUP BY o.customer_id
ORDER BY toplam_harcama DESC
LIMIT 10;
```

---

### Soru 7 — Şehir Bazlı Ortalama Sipariş Tutarı

**📋 Görev:** Her şehirdeki müşterilerin ortalama sipariş tutarını hesapla. Sonuçları yüksekten düşüğe sırala.

**✅ Çözüm:**
```sql
SELECT
    c.city,
    ROUND(AVG(o.quantity * p.price), 2) AS ort_siparis_tutari
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN products p ON o.product_id = p.product_id
GROUP BY c.city
ORDER BY ort_siparis_tutari DESC;
```

---

### Soru 8 — En Az 5 Sipariş Veren Müşteriler

**📋 Görev:** Toplam sipariş sayısı 5 ve üzerinde olan müşterileri bul. Sipariş sayısına göre azalan sırada listele.

**✅ Çözüm:**
```sql
SELECT c.customer_id, COUNT(o.order_id) AS toplam_siparis 
FROM `sql-practise-491318.SQL.customers` AS c
  LEFT JOIN `sql-practise-491318.SQL.orders` AS o
    ON c.customer_id = o.customer_id
GROUP BY c.customer_id
HAVING toplam_siparis >= 5
ORDER BY toplam_siparis DESC;
```

---

### Soru 9 — Kategori Bazlı Toplam Ciro

**📋 Görev:** Her ürün kategorisinin toplam cirosunu hesapla (fiyat × miktar). En kazançlı kategori hangisi?

**✅ Çözüm:**
```sql
SELECT p.category, ROUND(SUM(p.price*o.quantity),2) AS ciro
FROM `sql-practise-491318.SQL.products` AS p
  LEFT JOIN `sql-practise-491318.SQL.orders` AS o
    ON p.product_id = o.product_id
GROUP BY p.category
ORDER BY ciro DESC;
```

---

### Soru 10 — Loyalty Üyesi Olan vs Olmayan: Karşılaştırma

**📋 Görev:** Loyalty üyesi olan ve olmayan müşterilerin ortalama sipariş sayısını karşılaştır.

**✅ Çözüm:**
```sql
SELECT
    c.loyalty_member,
    ROUND(AVG(siparis_sayisi), 2) AS ort_siparis_sayisi
FROM (
    SELECT customer_id, COUNT(*) AS siparis_sayisi
    FROM `sql-practise-491318.SQL.orders` as o
    GROUP BY customer_id
) as siparis_ozet
JOIN `sql-practise-491318.SQL.customers` as c 
  ON siparis_ozet.customer_id = c.customer_id
GROUP BY c.loyalty_member;
```

---

### Soru 11 — Hiç Sipariş Vermemiş Müşteriler

**📋 Görev:** Şimdiye kadar hiç sipariş vermemiş müşterileri listele.

**✅ Çözüm:**
```sql
-- Yöntem 1: LEFT JOIN
SELECT c.customer_id, c.city, c.signup_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;

-- Yöntem 2: NOT EXISTS
SELECT customer_id, city, signup_date
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

---

### Soru 12 — Ödeme Yöntemine Göre Aylık Ciro

**📋 Görev:** Her ödeme yöntemi için aylık toplam ciroyu hesapla. Yıl-ay ve ödeme yöntemine göre grupla.

**✅ Çözüm:**
```sql
SELECT o.payment_method, EXTRACT(YEAR FROM o.order_date) AS yil , EXTRACT(MONTH FROM o.order_date)AS ay, ROUND(SUM(o.quantity*p.price), 2) 
FROM `sql-practise-491318.SQL.orders` AS o
LEFT JOIN `sql-practise-491318.SQL.products` AS p
  ON o.product_id = p.product_id
GROUP BY yil, ay, o.payment_method
ORDER BY yil, ay, o.payment_method;
```

---

## 🟠 SEVİYE 3 — İleri (Subquery, CTE, CASE WHEN)

---

### Soru 13 — Ortalama Üzerinde Harcama Yapan Müşteriler

**📋 Görev:** Kendi toplam harcaması, tüm müşterilerin ortalama harcamasının üzerinde olan müşterileri listele.

**✅ Çözüm:**
```sql
WITH musteri_harcama AS (
    SELECT
        o.customer_id,
        SUM(o.quantity * p.price) AS toplam_harcama
    FROM `sql-practise-491318.SQL.orders` AS o
    JOIN `sql-practise-491318.SQL.products` AS p ON o.product_id = p.product_id
    GROUP BY o.customer_id
)
SELECT
    customer_id,
    ROUND(toplam_harcama, 2) AS toplam_harcama
FROM musteri_harcama
WHERE toplam_harcama > (SELECT AVG(toplam_harcama) FROM musteri_harcama)
ORDER BY toplam_harcama DESC;
```

---

### Soru 14 — Her Kategorinin En Pahalı Ürünü

**📋 Görev:** Her kategorideki en yüksek fiyatlı ürünü bul.

**✅ Çözüm:**
```sql
WITH highest_price AS(
SELECT p.category, p.product_name,p.price, RANK() OVER(
  PARTITION BY p.category ORDER BY p.price DESC
) AS highest_price

FROM `sql-practise-491318.SQL.products` AS p
)

SELECT hp.category, hp.product_name, hp.price FROM highest_price AS hp
WHERE hp.highest_price = 1;
```

---

### Soru 15 — Müşteri Segmentasyonu (CASE WHEN)

**📋 Görev:** Müşterileri toplam sipariş sayısına göre segmentlere ayır:
- `0 sipariş` → "Kayıp Müşteri"
- `1–2 sipariş` → "Pasif"
- `3–5 sipariş` → "Aktif"
- `6+` → "Sadık"

**✅ Çözüm:**
```sql
WITH toplam_sip AS (
SELECT c.customer_id, c.city, count(o.order_id) AS count_order FROM `sql-practise-491318.SQL.customers` AS c
LEFT JOIN `sql-practise-491318.SQL.orders` AS o
  ON c.customer_id = o.customer_id
GROUP BY c.customer_id, city
)

SELECT toplam_sip.customer_id, toplam_sip.city, toplam_sip.count_order,
  CASE
    WHEN count_order = 0 THEN "Kayıp Müşteri"
    WHEN count_order > 0 AND count_order < 3 THEN "Pasif"
    WHEN count_order >= 3 AND count_order <=5 THEN "Aktif"
    WHEN count_order >= 6 THEN "Sadık"
  END AS grup
FROM toplam_sip
ORDER BY count_order DESC;
```

---

### Soru 16 — Aylık Büyüme Oranı (MoM)

**📋 Görev:** Her ayın toplam cirosunu ve bir önceki aya kıyasla yüzdesel büyüme oranını hesapla.

**✅ Çözüm:**
```sql
WITH toplam_ciro AS(
SELECT EXTRACT(YEAR FROM o.order_date) AS yil, EXTRACT(MONTH FROM o.order_date) AS ay,
ROUND(SUM(o.quantity*p.price)) AS ciro FROM `sql-practise-491318.SQL.orders` AS o
LEFT JOIN `sql-practise-491318.SQL.products` AS p
  ON o.product_id = p.product_id
GROUP BY yil, ay
ORDER BY yil, ay
)

SELECT CONCAT(yil, ay) AS donem, ciro,
LAG(ciro)OVER(
  ORDER BY yil, ay
) AS prev_month_ciro,

ROUND((ciro-LAG(ciro)OVER( ORDER BY yil, ay)) / 
  LAG(ciro)OVER(ORDER BY yil, ay) * 100) AS mom
FROM toplam_ciro;
```

---

### Soru 17 — En Çok Satan Ürün (Kategori Bazlı)

**📋 Görev:** Her kategoride en çok satan (toplam adet bazında) ürünü bul.

**✅ Çözüm:**
```sql
WITH top_satis_sayisi AS(
SELECT p.category, p.product_name, ROUND(SUM(o.quantity)) AS satis_sayisi, 
RANK()OVER(
  PARTITION BY p.category ORDER BY SUM(o.quantity) DESC
) AS cok_satan
FROM `sql-practise-491318.SQL.products` AS p
LEFT JOIN `sql-practise-491318.SQL.orders` AS o
  ON p.product_id = o.product_id
GROUP BY p.category, p.product_name
)
SELECT category, product_name, satis_sayisi FROM top_satis_sayisi
WHERE cok_satan = 1;
```

---

### Soru 18 — İlk ve Son Sipariş Tarihi

**📋 Görev:** Her müşterinin ilk ve son sipariş tarihini, iki tarih arasındaki gün farkını ve toplam sipariş sayısını getir.

**✅ Çözüm:**
```sql
WITH date_columns AS(
SELECT c.customer_id, o.order_date,
  MIN(o.order_date)OVER(
    PARTITION BY c.customer_id
  ) AS ilk_tarih,
  MAX(o.order_date)OVER(
    PARTITION BY c.customer_id
  ) AS son_tarih,
  o.quantity
  
FROM `sql-practise-491318.SQL.customers` AS c
LEFT JOIN `sql-practise-491318.SQL.orders` AS o
  ON c.customer_id = o.customer_id
)

SELECT customer_id, ilk_tarih, son_tarih, 
  DATE_DIFF(son_tarih, ilk_tarih, DAY) AS gun_fark,
  COUNT(quantity)AS toplam_siparis
FROM date_columns
GROUP BY customer_id, ilk_tarih, son_tarih, gun_fark;
```

---

### Soru 19 — Tekrar Eden Müşteri Oranı

**📋 Görev:** Birden fazla sipariş veren müşterilerin tüm müşterilere oranını hesapla (Retention Rate).

**✅ Çözüm:**
```sql
WITH musteriler AS(
SELECT c.customer_id, COUNT(o.order_id) AS siparis_sayisi
FROM `sql-practise-491318.SQL.customers` AS c
LEFT JOIN `sql-practise-491318.SQL.orders` AS o
  ON c.customer_id = o.customer_id
GROUP BY c.customer_id
ORDER BY siparis_sayisi DESC
)

SELECT ROUND((SELECT COUNT(customer_id)FROM musteriler WHERE siparis_sayisi > 1)/
  (SELECT COUNT(customer_id)FROM musteriler),2)
FROM musteriler
;

/* customers tablosuyla join yaptım çünkü hiç sipariş vermemiş müşterilerin de
hesaplamaya dahil edilmesi gerektiğini düşündüm
*/
```

---

## 🔴 SEVİYE 4 — Uzman (Window Functions, Gelişmiş CTE, Performance)

---

### Soru 20 — Kümülatif Ciro (Running Total)

**📋 Görev:** Siparişleri tarihe göre sırala, her gün için o güne kadar biriken toplam ciroyu hesapla.

**✅ Çözüm:**
```sql
WITH ciro_top AS(
SELECT o.order_date, 
SUM(o.quantity * p.price)AS ciro 
FROM `sql-practise-491318.SQL.orders` AS o
LEFT JOIN `sql-practise-491318.SQL.products` AS p
  ON o.product_id = p.product_id
GROUP BY o.order_date
)

SELECT order_date, ciro, 
ROUND(SUM(ciro)OVER(
  ORDER BY order_date
),2) AS kumule_ciro
FROM ciro_top
ORDER BY order_date;
```

---

### Soru 21 — Her Müşterinin En Çok Harcadığı Kategori

**📋 Görev:** Her müşteri için en fazla para harcadığı ürün kategorisini bul.

**✅ Çözüm:**
```sql
WITH musteri_harca AS(
SELECT c.customer_id, p.category,
ROUND(SUM(o.quantity * p.price),2) AS toplam_harcama,
RANK()OVER(
  PARTITION BY c.customer_id ORDER BY SUM(o.quantity * p.price) DESC
) AS siralama
FROM `sql-practise-491318.SQL.customers` AS c
LEFT JOIN `sql-practise-491318.SQL.orders` AS o
  ON c.customer_id = o.customer_id
LEFT JOIN `sql-practise-491318.SQL.products` AS p
  ON o.product_id = p.product_id
GROUP BY c.customer_id, p.category
ORDER BY c.customer_id, SUM(o.quantity * p.price) DESC
)

SELECT customer_id, category, toplam_harcama
FROM musteri_harca
WHERE siralama = 1;


```

---

### Soru 22 — 7 Günlük Hareketli Ortalama Ciro

**📋 Görev:** Her gün için, o günü ve önceki 6 günü kapsayan 7 günlük hareketli ortalama ciroyu hesapla.

**✅ Çözüm:**
```sql
SELECT o.order_date, SUM(o.quantity * p.price) AS harcama,
AVG(SUM(o.quantity * p.price))OVER(
  ORDER BY o.order_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
) AS hareketli_ort
FROM `sql-practise-491318.SQL.orders` AS o
LEFT JOIN `sql-practise-491318.SQL.products` AS p
ON o.product_id = p.product_id
GROUP BY o.order_date
ORDER BY o.order_date;
```

---

### Soru 23 — Cohort Analizi: Kayıt Ayına Göre İlk Ay Siparişleri

**📋 Görev:** Müşterileri kayıt oldukları aya (cohort) göre grupla. Her cohort'un kayıt ayında kaç sipariş verdiğini bul.

**✅ Çözüm:**
```sql
WITH cohort AS(
SELECT c.customer_id, c.signup_date, EXTRACT(MONTH FROM c.signup_date)AS ay,
EXTRACT(YEAR FROM c.signup_date) AS yil, 
CASE
  WHEN EXTRACT(MONTH FROM c.signup_date) = 1 THEN "Ocak Grubu"
  WHEN EXTRACT(MONTH FROM c.signup_date) = 2 THEN "Şubat Grubu"
  WHEN EXTRACT(MONTH FROM c.signup_date) = 3 THEN "Mart Grubu"
  WHEN EXTRACT(MONTH FROM c.signup_date) = 4 THEN "Nisan Grubu"
  WHEN EXTRACT(MONTH FROM c.signup_date) = 5 THEN "Mayıs Grubu"
  WHEN EXTRACT(MONTH FROM c.signup_date) = 6 THEN "Haziran Grubu"
  WHEN EXTRACT(MONTH FROM c.signup_date) = 7 THEN "Temmuz Grubu"
  WHEN EXTRACT(MONTH FROM c.signup_date) = 8 THEN "Ağustos Grubu"
  WHEN EXTRACT(MONTH FROM c.signup_date) = 9 THEN "Eylül Grubu"
  WHEN EXTRACT(MONTH FROM c.signup_date) = 10 THEN "Ekim Grubu"
  WHEN EXTRACT(MONTH FROM c.signup_date) = 11 THEN "Kasım Grubu"
  WHEN EXTRACT(MONTH FROM c.signup_date) = 12 THEN "Aralık Grubu"
END AS cohorts
FROM `sql-practise-491318.SQL.customers` AS c
)
SELECT c.yil, c.ay, c.cohorts, COUNT(c.customer_id) AS musteri_sayisi, COUNT(o.order_id) AS siparis_sayisi

FROM cohort AS c
LEFT JOIN `sql-practise-491318.SQL.orders` AS o
  ON c.customer_id = o.customer_id AND
     c.yil = EXTRACT(YEAR FROM o.order_date) AND
     c.ay = EXTRACT(MONTH FROM o.order_date)

GROUP BY c.cohorts, c.yil, c.ay

ORDER BY c.yil, c.ay;
```

---

### Soru 24 — Churn Risk: 90 Günde Sipariş Vermeyen Aktif Müşteriler

**📋 Görev:** En az 1 siparişi olan ama son siparişinden bu yana 90 günden fazla geçmiş müşterileri tespit et. (Referans tarih: 2024-05-15)

**✅ Çözüm:**
```sql
WITH son_siparisler AS(
SELECT c.customer_id, MAX(o.order_date) AS son_tarih, 
  DATE("2024-05-15") AS referans_yil, EXTRACT(DAY FROM DATE("2024-05-15") - MAX(o.order_date)) AS sure_gun
FROM `sql-practise-491318.SQL.customers` AS c
LEFT JOIN `sql-practise-491318.SQL.orders` AS o
  ON o.customer_id = c.customer_id
GROUP BY c.customer_id
)

SELECT * FROM son_siparisler
WHERE sure_gun > 90;

/*
DATE_DIFF versiyonu
*/

WITH son_siparisler AS(
SELECT c.customer_id, MAX(o.order_date) AS son_tarih, 
  DATE_DIFF(DATE("2024-05-15"), MAX(o.order_date), DAY) AS sure_gun
FROM `sql-practise-491318.SQL.customers` AS c
LEFT JOIN `sql-practise-491318.SQL.orders` AS o
  ON o.customer_id = c.customer_id
GROUP BY c.customer_id
)

SELECT * FROM son_siparisler
WHERE sure_gun > 90;

```

---

### Soru 25 — Ürün Bazlı Satış Eğilimi: Çeyrek Karşılaştırması

**📋 Görev:** Her ürün için 2023 Q4 ve 2024 Q1 satışlarını karşılaştır. Satışı artan ürünleri bul.

**✅ Çözüm:**
```sql
WITH siparis_sayi_CTE AS(
SELECT o.order_date, 
CASE
  WHEN o.order_date >= "2023-10-01" AND o.order_date < "2024-01-01" THEN 202304
  WHEN o.order_date >= "2024-01-01" AND o.order_date < "2024-04-01" THEN 202401
END AS ceyrek, 
p.product_name, SUM(o.quantity) AS siparis
FROM `sql-practise-491318.SQL.products` AS p
LEFT JOIN `sql-practise-491318.SQL.orders` AS o
  ON o.product_id = p.product_id
GROUP BY o.order_date, p.product_name
)

SELECT ceyrek, product_name, SUM(siparis)AS toplam_siparis
FROM siparis_sayi_CTE
WHERE ceyrek IS NOT NULL
GROUP BY ceyrek, product_name
;
```

---

### Soru 26 — Gelir Yüzdesine Göre Ürün Katkısı (Pareto Analizi)

**📋 Görev:** Her ürünün toplam ciro içindeki payını ve kümülatif yüzdesini hesapla. (80/20 kuralını test et)

**✅ Çözüm:**
```sql
WITH ciroCTE AS (
    SELECT 
        o.product_id,
        ROUND(SUM(o.quantity * p.price), 1) AS ciro,
    FROM `sql-practise-491318.SQL.orders` AS o
    LEFT JOIN `sql-practise-491318.SQL.products` AS p
        ON p.product_id = o.product_id
    GROUP BY o.product_id
    ORDER BY ciro DESC
),

genel AS(
  SELECT SUM(ciro) AS toplam FROM ciroCTE
)

SELECT
    c.product_id,
    c.ciro,
    ROUND(g.toplam,2),
    ROUND(c.ciro * 100.0 / g.toplam, 2) AS ciro_payi_pct,
    ROUND(SUM(c.ciro) OVER (ORDER BY c.ciro DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
        * 100.0 / g.toplam, 2) AS kumulatif_pct
FROM ciroCTE AS c, genel AS g
ORDER BY c.ciro DESC;
```

---

### Soru 27 — Müşteri Yaş Grubu × Kategori Çapraz Tablo

**📋 Görev:** Müşterileri yaş grubuna göre segmentlere ayır (`<25`, `25–40`, `41–60`, `60+`), her yaş grubunun hangi kategorilere ne kadar harcadığını göster.

**✅ Çözüm:**
```sql
/*
Görev: Müşterileri yaş grubuna göre segmentlere ayır (<25, 25–40, 41–60, 60+), her yaş grubunun hangi kategorilere ne kadar harcadığını göster.
*/
WITH segment AS(
SELECT c.customer_id, c.age, p.category, ROUND(SUM(o.quantity * p.price),2) AS tutar,
CASE
  WHEN c.age < 25 THEN "<25"
  WHEN c.age BETWEEN 25 AND 40 THEN "25–40"
  WHEN c.age BETWEEN 41 AND 60 THEN "41–60"
  WHEN c.age > 60 THEN "60+"
END AS musteri_segment
FROM `sql-practise-491318.SQL.customers` AS c
LEFT JOIN `sql-practise-491318.SQL.orders` AS o
  ON o.customer_id = c.customer_id
LEFT JOIN `sql-practise-491318.SQL.products` AS p
  ON o.product_id = p.product_id

GROUP BY c.customer_id, c.age, p.category

)

SELECT musteri_segment,
ROUND(SUM(CASE WHEN category = "Beauty" THEN tutar END),2) AS Beauty,
ROUND(SUM(CASE WHEN category = "Clothing" THEN tutar END),2) AS Clothing,
ROUND(SUM(CASE WHEN category = "Electronics" THEN tutar END),2) AS Electronics,
ROUND(SUM(CASE WHEN category = "Home" THEN tutar END),2) AS Home,
ROUND(SUM(CASE WHEN category = "Sports" THEN tutar END),2) AS Sports,
FROM segment
GROUP BY musteri_segment;
```

---

### Soru 28 — Consecutive Purchase Gap: Siparişler Arası Ortalama Gün

**📋 Görev:** Birden fazla siparişi olan müşteriler için, ardışık siparişler arasındaki ortalama gün farkını hesapla.

**✅ Çözüm:**
```sql
kod yazılacak
```

---

### Soru 29 — RFM Analizi (Recency, Frequency, Monetary)

**📋 Görev:** Her müşteri için RFM skorunu hesapla:
- **R (Recency):** Son siparişten bu yana kaç gün
- **F (Frequency):** Toplam sipariş sayısı
- **M (Monetary):** Toplam harcama

Her metriği `NTILE(4)` ile 1–4 arası skorla (R'de düşük gün = yüksek skor).

**✅ Çözüm:**
```sql
kod yazılacak
```

---

### Soru 30 — Index Optimizasyonu: Yavaş Sorguyu Hızlandır

**📋 Görev:** Aşağıdaki sorgunun neden yavaş çalışabileceğini açıkla ve hangi index'lerin eklenmesi gerektiğini belirt.

```sql
kod yazılacak

```

**✅ Çözüm:**

**Sorunlar:**


**Eklenecek Index'ler:**
```sql
kod yazılacak

```

**Sorgu Sonrası Kontrol:**
```sql

```

---

## 📌 Özet Tablo

| # | Seviye | Konu | Zorluk |
|---|--------|------|--------|
| 1–5 | 🟢 Başlangıç | SELECT, WHERE, ORDER BY, GROUP BY | ⭐ |
| 6–12 | 🟡 Orta | JOIN, Aggregate, HAVING, LEFT JOIN | ⭐⭐ |
| 13–19 | 🟠 İleri | Subquery, CTE, CASE WHEN, LAG | ⭐⭐⭐ |
| 20–30 | 🔴 Uzman | Window Functions, Cohort, RFM, Index | ⭐⭐⭐⭐ |
