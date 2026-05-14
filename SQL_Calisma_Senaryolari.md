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

**💡 İpucu:** `WHERE` içinde birden fazla koşul birleştirmek için `AND` kullanabilirsin. Sıralama için `ORDER BY` yeterli.

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

**💡 İpucu:** `WHERE` ile fiyat filtresi, `ORDER BY price DESC` ile sıralama yapabilirsin.

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

**💡 İpucu:** Tekrar eden satırları kaldırmak için `SELECT DISTINCT` kullan.

**✅ Çözüm:**
```sql
SELECT DISTINCT payment_method
FROM orders;
```

---

### Soru 4 — Belirli Tarih Aralığında Siparişler

**📋 Görev:** 2024 yılının ilk çeyreğinde (Ocak–Mart) verilen siparişleri listele.

**💡 İpucu:** Tarih filtrelemesi için `BETWEEN` ya da `>=` / `<=` operatörlerini kullanabilirsin.

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

**💡 İpucu:** `GROUP BY` ile kategorileri grupla, `COUNT(*)` ile say.

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

**💡 İpucu:** `orders` ve `products` tablolarını `JOIN` ile birleştir. `quantity * price` ile toplam tutarı hesapla, `GROUP BY customer_id` ile grupla.

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

**💡 İpucu:** Üç tabloyu birleştirmen gerekecek: `customers`, `orders`, `products`. `AVG(quantity * price)` hesapla.

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

**💡 İpucu:** `GROUP BY` sonrası filtreleme için `HAVING` kullanmalısın, `WHERE` değil.

**✅ Çözüm:**
```sql
SELECT
    customer_id,
    COUNT(*) AS siparis_sayisi
FROM orders
GROUP BY customer_id
HAVING COUNT(*) >= 5
ORDER BY siparis_sayisi DESC;
```

---

### Soru 9 — Kategori Bazlı Toplam Ciro

**📋 Görev:** Her ürün kategorisinin toplam cirosunu hesapla (fiyat × miktar). En kazançlı kategori hangisi?

**💡 İpucu:** `orders` ve `products` JOIN'lendikten sonra kategori bazında `SUM(quantity * price)` hesapla.

**✅ Çözüm:**
```sql
SELECT
    p.category,
    ROUND(SUM(o.quantity * p.price), 2) AS toplam_ciro
FROM orders o
JOIN products p ON o.product_id = p.product_id
GROUP BY p.category
ORDER BY toplam_ciro DESC;
```

---

### Soru 10 — Loyalty Üyesi Olan vs Olmayan: Karşılaştırma

**📋 Görev:** Loyalty üyesi olan ve olmayan müşterilerin ortalama sipariş sayısını karşılaştır.

**💡 İpucu:** Önce her müşterinin sipariş sayısını bul, sonra bunu `customers` tablosuyla `JOIN`'le ve `loyalty_member` bazında grupla.

**✅ Çözüm:**
```sql
SELECT
    c.loyalty_member,
    ROUND(AVG(siparis_sayisi), 2) AS ort_siparis_sayisi
FROM (
    SELECT customer_id, COUNT(*) AS siparis_sayisi
    FROM orders
    GROUP BY customer_id
) siparis_ozet
JOIN customers c ON siparis_ozet.customer_id = c.customer_id
GROUP BY c.loyalty_member;
```

---

### Soru 11 — Hiç Sipariş Vermemiş Müşteriler

**📋 Görev:** Şimdiye kadar hiç sipariş vermemiş müşterileri listele.

**💡 İpucu:** `LEFT JOIN` + `WHERE orders.customer_id IS NULL` ya da `NOT IN` / `NOT EXISTS` yaklaşımını kullanabilirsin.

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

**💡 İpucu:** `DATE_FORMAT(order_date, '%Y-%m')` (MySQL) veya `TO_CHAR(order_date, 'YYYY-MM')` (PostgreSQL) ile ay bilgisini çıkar.

**✅ Çözüm:**
```sql
-- PostgreSQL
SELECT
    TO_CHAR(o.order_date::date, 'YYYY-MM') AS ay,
    o.payment_method,
    ROUND(SUM(o.quantity * p.price), 2) AS toplam_ciro
FROM orders o
JOIN products p ON o.product_id = p.product_id
GROUP BY ay, o.payment_method
ORDER BY ay, o.payment_method;
```

---

## 🟠 SEVİYE 3 — İleri (Subquery, CTE, CASE WHEN)

---

### Soru 13 — Ortalama Üzerinde Harcama Yapan Müşteriler

**📋 Görev:** Kendi toplam harcaması, tüm müşterilerin ortalama harcamasının üzerinde olan müşterileri listele.

**💡 İpucu:** Önce her müşterinin toplam harcamasını hesapla, ardından `HAVING` içinde veya subquery olarak genel ortalamayla karşılaştır.

**✅ Çözüm:**
```sql
WITH musteri_harcama AS (
    SELECT
        o.customer_id,
        SUM(o.quantity * p.price) AS toplam_harcama
    FROM orders o
    JOIN products p ON o.product_id = p.product_id
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

**💡 İpucu:** Correlated subquery ya da `RANK()` window function ile çözebilirsin. Kategoriye göre grupla ve `MAX(price)` ile eşleştir.

**✅ Çözüm:**
```sql
-- Yöntem 1: Subquery
SELECT product_id, product_name, category, price
FROM products p
WHERE price = (
    SELECT MAX(price)
    FROM products
    WHERE category = p.category
);

-- Yöntem 2: CTE + RANK
WITH siralama AS (
    SELECT *,
           RANK() OVER (PARTITION BY category ORDER BY price DESC) AS rn
    FROM products
)
SELECT product_id, product_name, category, price
FROM siralama
WHERE rn = 1;
```

---

### Soru 15 — Müşteri Segmentasyonu (CASE WHEN)

**📋 Görev:** Müşterileri toplam sipariş sayısına göre segmentlere ayır:
- `0 sipariş` → "Kayıp Müşteri"
- `1–2 sipariş` → "Pasif"
- `3–5 sipariş` → "Aktif"
- `6+` → "Sadık"

**💡 İpucu:** `LEFT JOIN` ile sipariş vermeyenleri de dahil et. `COALESCE` ile NULL değerleri 0'a çevir. `CASE WHEN` ile segmente at.

**✅ Çözüm:**
```sql
SELECT
    c.customer_id,
    c.city,
    COALESCE(siparis.siparis_sayisi, 0) AS siparis_sayisi,
    CASE
        WHEN COALESCE(siparis.siparis_sayisi, 0) = 0 THEN 'Kayıp Müşteri'
        WHEN siparis.siparis_sayisi <= 2 THEN 'Pasif'
        WHEN siparis.siparis_sayisi <= 5 THEN 'Aktif'
        ELSE 'Sadık'
    END AS segment
FROM customers c
LEFT JOIN (
    SELECT customer_id, COUNT(*) AS siparis_sayisi
    FROM orders
    GROUP BY customer_id
) siparis ON c.customer_id = siparis.customer_id
ORDER BY siparis_sayisi DESC;
```

---

### Soru 16 — Aylık Büyüme Oranı (MoM)

**📋 Görev:** Her ayın toplam cirosunu ve bir önceki aya kıyasla yüzdesel büyüme oranını hesapla.

**💡 İpucu:** `LAG()` window function ile önceki ayın değerine ulaşabilirsin. `(bu_ay - onceki_ay) / onceki_ay * 100` formülünü kullan.

**✅ Çözüm:**
```sql
WITH aylik_ciro AS (
    SELECT
        TO_CHAR(o.order_date::date, 'YYYY-MM') AS ay,
        ROUND(SUM(o.quantity * p.price), 2) AS ciro
    FROM orders o
    JOIN products p ON o.product_id = p.product_id
    GROUP BY ay
),
buyume AS (
    SELECT
        ay,
        ciro,
        LAG(ciro) OVER (ORDER BY ay) AS onceki_ay_ciro
    FROM aylik_ciro
)
SELECT
    ay,
    ciro,
    onceki_ay_ciro,
    ROUND(
        (ciro - onceki_ay_ciro) / onceki_ay_ciro * 100, 2
    ) AS buyume_orani_pct
FROM buyume
ORDER BY ay;
```

---

### Soru 17 — En Çok Satan Ürün (Kategori Bazlı)

**📋 Görev:** Her kategoride en çok satan (toplam adet bazında) ürünü bul.

**💡 İpucu:** `SUM(quantity)` ile ürün bazında toplam satışı hesapla, `RANK() OVER (PARTITION BY category ...)` ile kategori içinde sırala.

**✅ Çözüm:**
```sql
WITH urun_satis AS (
    SELECT
        p.category,
        p.product_name,
        SUM(o.quantity) AS toplam_adet,
        RANK() OVER (PARTITION BY p.category ORDER BY SUM(o.quantity) DESC) AS rn
    FROM orders o
    JOIN products p ON o.product_id = p.product_id
    GROUP BY p.category, p.product_name
)
SELECT category, product_name, toplam_adet
FROM urun_satis
WHERE rn = 1
ORDER BY category;
```

---

### Soru 18 — İlk ve Son Sipariş Tarihi

**📋 Görev:** Her müşterinin ilk ve son sipariş tarihini, iki tarih arasındaki gün farkını ve toplam sipariş sayısını getir.

**💡 İpucu:** `MIN()` ve `MAX()` ile tarihleri bul. `MAX(date) - MIN(date)` veya `DATEDIFF()` ile gün farkını hesapla.

**✅ Çözüm:**
```sql
SELECT
    customer_id,
    MIN(order_date::date) AS ilk_siparis,
    MAX(order_date::date) AS son_siparis,
    MAX(order_date::date) - MIN(order_date::date) AS aktiflik_gun,
    COUNT(*) AS toplam_siparis
FROM orders
GROUP BY customer_id
ORDER BY aktiflik_gun DESC;
```

---

### Soru 19 — Tekrar Eden Müşteri Oranı

**📋 Görev:** Birden fazla sipariş veren müşterilerin tüm müşterilere oranını hesapla (Retention Rate).

**💡 İpucu:** Subquery ile sipariş sayısı > 1 olan müşterileri say, toplam müşteri sayısına böl.

**✅ Çözüm:**
```sql
SELECT
    COUNT(DISTINCT CASE WHEN siparis_sayisi > 1 THEN customer_id END) AS tekrar_eden,
    COUNT(DISTINCT customer_id) AS toplam,
    ROUND(
        COUNT(DISTINCT CASE WHEN siparis_sayisi > 1 THEN customer_id END) * 100.0
        / COUNT(DISTINCT customer_id), 2
    ) AS retention_orani_pct
FROM (
    SELECT customer_id, COUNT(*) AS siparis_sayisi
    FROM orders
    GROUP BY customer_id
) t;
```

---

## 🔴 SEVİYE 4 — Uzman (Window Functions, Gelişmiş CTE, Performance)

---

### Soru 20 — Kümülatif Ciro (Running Total)

**📋 Görev:** Siparişleri tarihe göre sırala, her gün için o güne kadar biriken toplam ciroyu hesapla.

**💡 İpucu:** `SUM() OVER (ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)` ile kümülatif toplam alabilirsin.

**✅ Çözüm:**
```sql
WITH gunluk_ciro AS (
    SELECT
        o.order_date::date AS gun,
        ROUND(SUM(o.quantity * p.price), 2) AS gunluk_ciro
    FROM orders o
    JOIN products p ON o.product_id = p.product_id
    GROUP BY gun
)
SELECT
    gun,
    gunluk_ciro,
    ROUND(SUM(gunluk_ciro) OVER (ORDER BY gun
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW), 2) AS kumulatif_ciro
FROM gunluk_ciro
ORDER BY gun;
```

---

### Soru 21 — Her Müşterinin En Çok Harcadığı Kategori

**📋 Görev:** Her müşteri için en fazla para harcadığı ürün kategorisini bul.

**💡 İpucu:** Müşteri + kategori bazında harcama hesapla, ardından `RANK() OVER (PARTITION BY customer_id ORDER BY harcama DESC)` ile her müşteri için ilk sırayı al.

**✅ Çözüm:**
```sql
WITH musteri_kategori AS (
    SELECT
        o.customer_id,
        p.category,
        ROUND(SUM(o.quantity * p.price), 2) AS harcama,
        RANK() OVER (
            PARTITION BY o.customer_id
            ORDER BY SUM(o.quantity * p.price) DESC
        ) AS rn
    FROM orders o
    JOIN products p ON o.product_id = p.product_id
    GROUP BY o.customer_id, p.category
)
SELECT customer_id, category AS favori_kategori, harcama
FROM musteri_kategori
WHERE rn = 1
ORDER BY harcama DESC;
```

---

### Soru 22 — 7 Günlük Hareketli Ortalama Ciro

**📋 Görev:** Her gün için, o günü ve önceki 6 günü kapsayan 7 günlük hareketli ortalama ciroyu hesapla.

**💡 İpucu:** `AVG() OVER (ORDER BY gun ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)` kullanımına bak.

**✅ Çözüm:**
```sql
WITH gunluk AS (
    SELECT
        o.order_date::date AS gun,
        ROUND(SUM(o.quantity * p.price), 2) AS ciro
    FROM orders o
    JOIN products p ON o.product_id = p.product_id
    GROUP BY gun
)
SELECT
    gun,
    ciro,
    ROUND(AVG(ciro) OVER (
        ORDER BY gun
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ), 2) AS hareketli_ort_7gun
FROM gunluk
ORDER BY gun;
```

---

### Soru 23 — Cohort Analizi: Kayıt Ayına Göre İlk Ay Siparişleri

**📋 Görev:** Müşterileri kayıt oldukları aya (cohort) göre grupla. Her cohort'un kayıt ayında kaç sipariş verdiğini bul.

**💡 İpucu:** `TO_CHAR(signup_date, 'YYYY-MM')` ile cohort'u belirle. `order_date` ile `signup_date` arasındaki ay farkını hesapla. `DATE_PART('month', age(order_date, signup_date))` kullanabilirsin.

**✅ Çözüm:**
```sql
WITH cohort AS (
    SELECT
        c.customer_id,
        TO_CHAR(c.signup_date::date, 'YYYY-MM') AS cohort_ay,
        o.order_id,
        DATE_PART('month', AGE(o.order_date::date, c.signup_date::date)) AS ay_farki
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
)
SELECT
    cohort_ay,
    COUNT(DISTINCT customer_id) AS musteri_sayisi,
    COUNT(CASE WHEN ay_farki = 0 THEN order_id END) AS ilk_ay_siparis
FROM cohort
GROUP BY cohort_ay
ORDER BY cohort_ay;
```

---

### Soru 24 — Churn Risk: 90 Günde Sipariş Vermeyen Aktif Müşteriler

**📋 Görev:** En az 1 siparişi olan ama son siparişinden bu yana 90 günden fazla geçmiş müşterileri tespit et. (Referans tarih: 2024-05-15)

**💡 İpucu:** `MAX(order_date)` ile son sipariş tarihini bul. Referans tarihinden çıkar, 90 günden büyük olanları filtrele.

**✅ Çözüm:**
```sql
SELECT
    customer_id,
    MAX(order_date::date) AS son_siparis,
    DATE '2024-05-15' - MAX(order_date::date) AS gecen_gun
FROM orders
GROUP BY customer_id
HAVING DATE '2024-05-15' - MAX(order_date::date) > 90
ORDER BY gecen_gun DESC;
```

---

### Soru 25 — Ürün Bazlı Satış Eğilimi: Çeyrek Karşılaştırması

**📋 Görev:** Her ürün için 2023 Q4 ve 2024 Q1 satışlarını karşılaştır. Satışı artan ürünleri bul.

**💡 İpucu:** `CASE WHEN` veya `FILTER` ile quarter bazlı SUM alabilirsin. `PIVOT` benzeri bir yapı kuracaksın.

**✅ Çözüm:**
```sql
SELECT
    p.product_name,
    p.category,
    SUM(CASE WHEN o.order_date BETWEEN '2023-10-01' AND '2023-12-31'
             THEN o.quantity ELSE 0 END) AS q4_2023,
    SUM(CASE WHEN o.order_date BETWEEN '2024-01-01' AND '2024-03-31'
             THEN o.quantity ELSE 0 END) AS q1_2024,
    SUM(CASE WHEN o.order_date BETWEEN '2024-01-01' AND '2024-03-31'
             THEN o.quantity ELSE 0 END)
    - SUM(CASE WHEN o.order_date BETWEEN '2023-10-01' AND '2023-12-31'
               THEN o.quantity ELSE 0 END) AS degisim
FROM orders o
JOIN products p ON o.product_id = p.product_id
GROUP BY p.product_name, p.category
HAVING SUM(CASE WHEN o.order_date BETWEEN '2024-01-01' AND '2024-03-31'
                THEN o.quantity ELSE 0 END)
     > SUM(CASE WHEN o.order_date BETWEEN '2023-10-01' AND '2023-12-31'
                THEN o.quantity ELSE 0 END)
ORDER BY degisim DESC;
```

---

### Soru 26 — Gelir Yüzdesine Göre Ürün Katkısı (Pareto Analizi)

**📋 Görev:** Her ürünün toplam ciro içindeki payını ve kümülatif yüzdesini hesapla. (80/20 kuralını test et)

**💡 İpucu:** `SUM() OVER (ORDER BY ciro DESC)` ile kümülatif toplamı al, genel toplamla böl.

**✅ Çözüm:**
```sql
WITH urun_ciro AS (
    SELECT
        p.product_name,
        p.category,
        ROUND(SUM(o.quantity * p.price), 2) AS ciro
    FROM orders o
    JOIN products p ON o.product_id = p.product_id
    GROUP BY p.product_name, p.category
),
genel AS (
    SELECT SUM(ciro) AS toplam FROM urun_ciro
)
SELECT
    u.product_name,
    u.category,
    u.ciro,
    ROUND(u.ciro * 100.0 / g.toplam, 2) AS ciro_payi_pct,
    ROUND(SUM(u.ciro) OVER (ORDER BY u.ciro DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
        * 100.0 / g.toplam, 2) AS kumulatif_pct
FROM urun_ciro u, genel g
ORDER BY u.ciro DESC;
```

---

### Soru 27 — Müşteri Yaş Grubu × Kategori Çapraz Tablo

**📋 Görev:** Müşterileri yaş grubuna göre segmentlere ayır (`<25`, `25–40`, `41–60`, `60+`), her yaş grubunun hangi kategorilere ne kadar harcadığını göster.

**💡 İpucu:** `CASE WHEN` ile yaş grubu oluştur, sonra kategori ile `GROUP BY` yap. Pivot benzeri sonuç için ayrı `SUM(CASE WHEN category=...)` kullan.

**✅ Çözüm:**
```sql
SELECT
    CASE
        WHEN c.age < 25 THEN '<25'
        WHEN c.age BETWEEN 25 AND 40 THEN '25-40'
        WHEN c.age BETWEEN 41 AND 60 THEN '41-60'
        ELSE '60+'
    END AS yas_grubu,
    ROUND(SUM(CASE WHEN p.category='Beauty'      THEN o.quantity*p.price ELSE 0 END),2) AS Beauty,
    ROUND(SUM(CASE WHEN p.category='Electronics' THEN o.quantity*p.price ELSE 0 END),2) AS Electronics,
    ROUND(SUM(CASE WHEN p.category='Clothing'    THEN o.quantity*p.price ELSE 0 END),2) AS Clothing,
    ROUND(SUM(CASE WHEN p.category='Home'        THEN o.quantity*p.price ELSE 0 END),2) AS Home,
    ROUND(SUM(CASE WHEN p.category='Sports'      THEN o.quantity*p.price ELSE 0 END),2) AS Sports
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN products p ON o.product_id = p.product_id
GROUP BY yas_grubu
ORDER BY yas_grubu;
```

---

### Soru 28 — Consecutive Purchase Gap: Siparişler Arası Ortalama Gün

**📋 Görev:** Birden fazla siparişi olan müşteriler için, ardışık siparişler arasındaki ortalama gün farkını hesapla.

**💡 İpucu:** `LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date)` ile önceki siparişi bul, farkı hesapla, müşteri bazında ortala.

**✅ Çözüm:**
```sql
WITH siparis_araligi AS (
    SELECT
        customer_id,
        order_date::date AS siparis_tarihi,
        LAG(order_date::date) OVER (
            PARTITION BY customer_id ORDER BY order_date
        ) AS onceki_siparis
    FROM orders
),
farklar AS (
    SELECT
        customer_id,
        siparis_tarihi - onceki_siparis AS gun_farki
    FROM siparis_araligi
    WHERE onceki_siparis IS NOT NULL
)
SELECT
    customer_id,
    COUNT(*) AS aralik_sayisi,
    ROUND(AVG(gun_farki), 1) AS ort_gun_farki
FROM farklar
GROUP BY customer_id
HAVING COUNT(*) >= 2
ORDER BY ort_gun_farki ASC;
```

---

### Soru 29 — RFM Analizi (Recency, Frequency, Monetary)

**📋 Görev:** Her müşteri için RFM skorunu hesapla:
- **R (Recency):** Son siparişten bu yana kaç gün
- **F (Frequency):** Toplam sipariş sayısı
- **M (Monetary):** Toplam harcama

Her metriği `NTILE(4)` ile 1–4 arası skorla (R'de düşük gün = yüksek skor).

**💡 İpucu:** CTE zinciri kullan: önce ham metrikleri hesapla, sonra `NTILE(4)` ile skorla, son CTE'de birleştir.

**✅ Çözüm:**
```sql
WITH rfm_ham AS (
    SELECT
        o.customer_id,
        DATE '2024-05-15' - MAX(o.order_date::date) AS recency_gun,
        COUNT(*) AS frequency,
        ROUND(SUM(o.quantity * p.price), 2) AS monetary
    FROM orders o
    JOIN products p ON o.product_id = p.product_id
    GROUP BY o.customer_id
),
rfm_skor AS (
    SELECT *,
        NTILE(4) OVER (ORDER BY recency_gun DESC) AS r_skor,
        NTILE(4) OVER (ORDER BY frequency ASC)    AS f_skor,
        NTILE(4) OVER (ORDER BY monetary ASC)     AS m_skor
    FROM rfm_ham
)
SELECT
    customer_id,
    recency_gun,
    frequency,
    monetary,
    r_skor, f_skor, m_skor,
    (r_skor + f_skor + m_skor) AS rfm_toplam,
    CASE
        WHEN (r_skor + f_skor + m_skor) >= 10 THEN '🏆 Şampiyonlar'
        WHEN (r_skor + f_skor + m_skor) >= 7  THEN '⭐ Sadık Müşteri'
        WHEN (r_skor + f_skor + m_skor) >= 5  THEN '🔄 Potansiyel'
        ELSE '⚠️ Risk Altında'
    END AS rfm_segment
FROM rfm_skor
ORDER BY rfm_toplam DESC;
```

---

### Soru 30 — Index Optimizasyonu: Yavaş Sorguyu Hızlandır

**📋 Görev:** Aşağıdaki sorgunun neden yavaş çalışabileceğini açıkla ve hangi index'lerin eklenmesi gerektiğini belirt.

```sql
SELECT c.city, p.category, SUM(o.quantity * p.price) AS ciro
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
WHERE c.city = 'London'
  AND o.order_date >= '2024-01-01'
GROUP BY c.city, p.category;
```

**💡 İpucu:** `EXPLAIN ANALYZE` çıktısını yorumlamayı düşün. JOIN yapılan kolonlarda ve `WHERE` filtrelerinde index olmaması `Seq Scan` tetikler.

**✅ Çözüm:**

**Sorunlar:**
1. `orders.customer_id` — JOIN kolonu, index yok → Seq Scan
2. `orders.product_id` — JOIN kolonu, index yok → Seq Scan
3. `orders.order_date` — WHERE filtresi, index yok → tüm tablo taranır
4. `customers.city` — WHERE filtresi, index yok → Seq Scan

**Eklenecek Index'ler:**
```sql
-- Join kolonları
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_orders_product_id  ON orders(product_id);

-- WHERE filtreleri
CREATE INDEX idx_orders_order_date  ON orders(order_date);
CREATE INDEX idx_customers_city     ON customers(city);

-- Composite index (city + order_date birlikte kullanılıyorsa)
CREATE INDEX idx_orders_date_customer ON orders(order_date, customer_id);
```

**Sorgu Sonrası Kontrol:**
```sql
EXPLAIN ANALYZE
SELECT c.city, p.category, SUM(o.quantity * p.price) AS ciro
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
WHERE c.city = 'London'
  AND o.order_date >= '2024-01-01'
GROUP BY c.city, p.category;
```

---

## 📌 Özet Tablo

| # | Seviye | Konu | Zorluk |
|---|--------|------|--------|
| 1–5 | 🟢 Başlangıç | SELECT, WHERE, ORDER BY, GROUP BY | ⭐ |
| 6–12 | 🟡 Orta | JOIN, Aggregate, HAVING, LEFT JOIN | ⭐⭐ |
| 13–19 | 🟠 İleri | Subquery, CTE, CASE WHEN, LAG | ⭐⭐⭐ |
| 20–30 | 🔴 Uzman | Window Functions, Cohort, RFM, Index | ⭐⭐⭐⭐ |
