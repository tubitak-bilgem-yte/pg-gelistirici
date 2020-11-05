---
title: "PostgreSQL Sorgular ve Yazım Kuralları"
tags: [PostgreSQL]
keywords: PostgreSQL
last_updated: November 5, 2020
summary: "PostgreSQL Soruguları ve Yazım Kuralları"
sidebar: mydoc_sidebar
permalink: mydoc_sorgu_yazım_kuralları.html
folder: mydoc
---

## Sorgular ve Yazım Kuralları

SQL cümlecikleri aslında bir komutlar dizisi olarak oluşturulur. Her cümlecik, bir sembol dizisinden ibaret olup noktalı virgülle sonlanır. Cümlecikleri oluşturan sembol dizileri anahtar kelimeler, tanımlayıcılar, tırnak içinde ifadeler, sabit veya tanımlanmış değerler ya da özel bir karakter sembolü olabilirler. Her sembol birbirinden boşluk karakteri ile ayrışır.

SQL’de anahtar kelimeler bir işlem süreci başlatırlar. Bu anahtar kelimelerden bazıları yazım olarak başka anahtar kelimeleri ya da seçeneklerin girilmesini gerektirir. Örneğin ``UPDATE`` kullanıldığında ``SET``’in kullanılma gerekliliği gibi.

SQL’de anahtar kelimeler ve tırnak içine alınmamış nesne isimleri büyük-küçük harf duyarsızdır. İfadesel olarak şu ikisi arasında fark yoktur.

```sql
UPDATE MY_TABLE SET A = 5;           uPDaTE my_TabLE SeT a = 5;
```

SQL’de yorum eklemek için çeşitli yollar bulunmaktadır. Tek satırlık yorum eklemek için çift tire ``(--)`` ve C stilinde çok satırlı yorumlar eklemek için ``/*`` kullanabilir. Örnekleri  şu şekildedir:

```sql
-- This is a standard SQL comment

/* multiline comment
 * with nesting: /* nested block comment */
 */
```

Girilen sorgu, yorumlayıcıda belli bir sırayla işlenmektedir. Aşağıdaki tabloda en yüksek öncelikli operatörden en düşük öncelikli olana doğru bir sıralama sunulmuştur.

| Operatör/Element | İlişki kurduğu taraf | Tanım |
|-------|--------|--------|
| ``.`` | sol | tablo / kolon adı ayracı |
| ``::`` | sol | PostgreSQL-stili tip dönüştürücü |
| ``[ ]`` | sol | dizi elemanı seçme operatörü |
| ``+, -`` | sağ | pozitif ve negatif operatörü |
| ``^`` | sol | üs alma |
| ``*, / ,  %`` | sol | çarpma, bölme, mod alma |
| ``+, -`` | sol | toplama, çıkarma |
| ``(any other operator)`` | sol | tüm diğer dahili ve kullanıcı tanımlı operatörler |
| ``BETWEEN, IN, LIKE, ILIKE, SIMILAR`` | | değer aralığına aidiyet sorgulama, küme üyeliği sorgulama, metinsel eşleştirme |
| ``<, >, =, <=, >=, <>`` | | karşılaştırma operatörleri |
| ``IS, ISNULL, NOTNULL`` | | IS TRUE, IS FALSE, IS NULL, IS DISTINCT FROM, vs |
| ``NOT`` | sağ | mantıksal olumsuzluk ifadesi |
| ``AND`` | sol | mantıksal bağlaç |
| ``OR`` | sol | mantıksal ayraç |

SQL yazımında çokça rastlanan değer ifadeleri vardır. Bu ifadeler yazım içinde çoğu yerde çok farklı amaçlarla kullanılmaktadır. Bunların kullanım yerleri ve örnekleri aşağıdaki tabloda sunulmuştur.

| Değer İfadesi | Örnek |
|-------|--------|
| Kolon referansı | ``correlation.columnname``|
| SQL cümleciğine haricen iletilen parametreler | **$number**   ``CREATE FUNCTION dept(text) RETURNS dept AS $$ SELECT * FROM dept WHERE name = $1 $$ LANGUAGE SQL;`` |
| Altsorgular |``mytable.arraycolumn[4] mytable.two_d_column[17][34] $1[10:42] (arrayfunction(a,b))[42]``|
| Alan/kolon seçimi | **expression.fieldname** ``mytable.mycolumn $1.somecolumn (rowfunction(a,b)).col3`` |
| Fonksiyon çağırma | **function_name** ([**expression** [, **expression** ... ]] ) |
| Tip dönüşümleri | CAST ( **expression** AS **type** ) **expression**::**type** |
| Skalar alt sorgular |``SELECT name, (SELECT max(pop) FROM cities WHERE cities.state = states.name) FROM states;`` |

{% include links.html %}
