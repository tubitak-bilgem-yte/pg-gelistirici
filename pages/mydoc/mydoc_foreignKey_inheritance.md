---
title: "PostgreSQL Yabancı Anahtar ve Miras"
tags: [PostgreSQL]
keywords: PostgreSQL
last_updated: November 6, 2020
summary: "PostgreSQL Yabancı Anahtar ve Miras"
sidebar: mydoc_sidebar
permalink: mydoc_foreignKey_inheritance.html
folder: mydoc
---

## Yabancı Anahtar ( FOREIGN KEY )

**Yabancı anahtar**, birbiriyle ilişkili tablolar arasındaki referanssal bütünlüğü sağlar. Açık bir ifadeyle bir tablonun bir kolonundaki değerlerin, ilişkili diğer tablodaki satırlarla uyuşmasını zorunlu kılar. Örneğin birbiriyle ilişkili bir ortak kolonu olan iki tablo bulunsun. Şehirlerin ve şehir bazlı hava sıcaklık ölçümlerinin olduğu iki tablo üzerinden örnek verelim.

```sql
CREATE TABLE cities (
        city     varchar(80) primary key,
        location point
);

CREATE TABLE weather (
        city      varchar(80) references cities(city),
        temp_lo   int,
        temp_hi   int,
        prcp      real,
        date      date
);
```

Hava durumu tablosundaki şehir kolonu şehirler tablosundaki şehir kolonuna referanslanmış durumda. Bu tasarımda hava durumu tablosuna bir ölçüm eklemek için şehir bilgisinin şehirler tablosundan kontrol edilmesi gerekmektedir. Eğer INSERT satırımızda bulunan şehir, şehirler tablosunda bulunmuyorsa bu durumda satırın *weather* tablosuna yazılması mümkün olmayacaktır. Bu hava durumu tablosunda bizim tanıdığımız şehirlerden başka şehirlerin girilmesini önleyecek ve bu iki tablo arasında bir veri bütünlüğü kurulmuş olacaktır. Bunu sağlayabilmek için *city* kolonunu *cities* tablosunda *primary key*, *weather* tablosunda ise (cities(city) ye referanslayarak) *foreign key* olarak tanımlamalıyız.

{% include note.html content="Bir tabloda birden fazla kolon, birden fazla başka tabloya referanslanabilir ya da diğer bir deyişle bir tabloda, birincil anahtarın aksine, birden fazla yabancı anahtar olabilir." %}

```sql
CREATE TABLE t1 (
  a integer PRIMARY KEY,
  b integer,
  c integer,
  FOREIGN KEY (b, c) REFERENCES other_table (c1, c2)
);
```

**Foreign Key** yapısının getirdiği ilave kontrollerden birisi ilişkili tablolara ait satırlardan birisinin silinmesi durumunda diğerini de silmesi ya da birbirlerinin silinmesinin engellenmesini sağlayan ``ON DELETE RESTRICT / CASCADE`` ve ``NO ACTION`` anahtar kelimeleridir. Normalde bu spesifikasyonlar yapılmadığı sürece FOREIGN KEY varsayılan olarak **NO ACTION** modunda çalışır ve silinme olayını bildiren bir hata verir.
Tabloda nasıl tanımlandığına ilişkin örnek:

```sql
CREATE TABLE order_items (
    product_no integer REFERENCES products ON DELETE RESTRICT,
    order_id integer REFERENCES orders ON DELETE CASCADE,
    quantity integer,
    PRIMARY KEY (product_no, order_id)
);
```

FOREIGN KEY, tanımında ``ON DELETE CASCADE`` ifadesini içeriyorsa bir tabloda silme işlemi yapıldığında diğer ilişkili tablodaki ilişkili tüm satırlar da silinir. ``RESTRICT`` ise silinme işlemini durdurur. Devamında kullanılabilecek ``SET DEFAULT`` ve ``SET NULL`` ifadeleri ise silme işlemine müteakip ilişkili kolonların değerlerini o kolonun DEFAULT’una ya da NULL’a UPDATE eder.

Normalde referanslanan bir satır, kolonlarından bazıları NULL ise, yabancı anahtar kısıtını tamamen karşılamak zorunda değildir. NULL gelen kolonlar, NULL olarak hatasız yazılır. Eğer tamamen karşılaması isteniyorsa ``MATCH FULL`` anahtar kelimesi eklenir ve NULL kolonlar olması halinde hata alınması sağlanır. Bununla birlikte aynı şartlar altında, referanslanan tüm kolonların NULL olması durumunda yine hata üretilmez ve bu satır kontrolden kaçabilir. Bunun önüne geçebilmek için alınabilecek tek önlem kolonlara ``NOT NULL`` kısıtının eklenmesidir.

## Miras ( INHERITANCE )

Miras, nesne tabanlı veritabanlarında kullanılan bir kavramdır. Miras, veritabanı tasarımına yeni seçenekler ekler. Örneğin aşağıdaki gibi iki tane tablomuz olsun.

```sql
CREATE TABLE capitals (
  name       text,
  population real,
  altitude   int,  
  state      char(2)
);
CREATE TABLE non_capitals (
  name       text,
  population real,
  altitude   int
);

CREATE VIEW cities AS
  SELECT name, population, altitude FROM capitals
    UNION
  SELECT name, population, altitude FROM non_capitals;
```

Tablo olarak *Capitals* (Başkentler) ve *non_Capitals* (Başkent olmayan şehirler) ile view olarak *cities* (Şehirler) oluşturuldu. Normalde başkentler doğal olarak şehirdirler. Bu yüzden tüm şehirleri görmek istediğimizde bir şekilde başkentlerin de bu listede bulunması gerekir. Şehirler için oluşturduğumuz view bunun akılcıl bir çözümünü içerir. Bunun için, hem capitals hem de non_capitals tablolarından name, population ve altitude kolonları çekilip  birleştirilir

Burada cities view olarak oluşturulmuştur ve bakıldığında bu üç tablo içinde sadece *capitals*’ta *state* kolonu ilave bilgi olarak bulunmaktadır. Bunu yapmak yerine aşağıdaki gibi bir çözüm, bu durum özelinde daha akılcı olacaktır.

```sql
CREATE TABLE cities (
  name       text,
  population real,
  altitude   int
);

CREATE TABLE capitals (
  state      char(2)
) INHERITS (cities);
```

``INHERITS`` anahtar sözcüğü kullanma durumunda, önce *cities* tablosu yaratılır ve *capitals* tablosu için de bunun tüm özelliklerini miras alarak *state* kolonunu da ekleten bir tasarımına izin verilir. PostgreSQL’de bir tablo başka bir tablodan sıfır veya daha çok kolonu miras alabilir.

Miras tablolardaki parent - child ilişkisi için sorgu davranışında görülen farklı bir durum vardır. Parent tablodan sorgu yapıldığında sorgulama tablosu, FROM yerine ``ONLY FROM`` kullanılarak sınırlandırılmazsa hem parent hem de (kendisinden miras alarak oluşturduğumuz) child tabloda kriterleri karşılayan satırlar gelir. ONLY FROM *parent_table* ile yapılan sorgulama sadece parent tablonun değerlerini görmemizi sağlar. SELECT, UPDATE ve DELETE komutları bu ONLY spesifikasyonunu destekler.

{% include links.html %}
