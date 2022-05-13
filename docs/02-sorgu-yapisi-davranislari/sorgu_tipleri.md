---
layout: default
title: Sorgu Tipleri
parent: Sorguların Yapısı ve Davranışları
nav_order: 1
---

## Sorgu Tipleri

PostgreSQL’de sorgu tiplerini dört grupta incelemek mümkündür:

1. DDL (Data Definition Language) veri tanımlama görevlerini yerine getiri
2. DML (Data Manipulation Language) veriyi oluşturma, değiştirme ve silme görevlerini yerine getirir
3. DQL (Data Query Language) aranan veriyi sorgulama ve sunma görevlerini yerine getirir.
4. TCL (Transaction Control  Language) transaction kontrolü sağlar.

### DDL Komutları

Bu komutlar veritabanı içindeki tüm nesnelerin oluşturulmasını, tanımlanmasını, düzenlenmesini ve silinmesini yapmakla yükümlüdür. Bu görevleri yerine getirmek için ``CREATE``, ``UPDATE`` ve ``DROP`` blokları kullanılır. Aşağıda veritabanı içinde bir tablo oluşturulması ve silinmesi görülmektedir.

```sql
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric
);
```

Her tablo alanlardan (kolonlardan) oluşur ve her alanın bir veri tipi olur. ``CREATE TABLE`` ifadesi tablonun ve tabloya ait tüm bileşenlerin (kolonların isimleri, veri tipleri ve büyüklükleri, kısıtlar, indeksler, vs...) tanımlanma işlemini gerçekleştirir.  Oluşturulmuş nesnelerin tamamı ``ALTER`` komutlarıyla değiştirilerek yeniden tasarlanabilir. Burada veri türlerinden ya da boyutlarından kaynaklı veri kayıplarına dikkat edilmelidir.

```sql
DROP TABLE my_first_table;
DROP TABLE products;
```

Veritabanı tablolarının tasarımında, kolonlar için varsayılan değerler girilerek veri girişi yapılan her satırda kullanıcı o kolon için bir değer girmese dahi otomatik olarak istenilen  değerin girilmesi sağlanabilir. Bunun için ``DEFAULT`` kısıtı oluşturulmalıdır.

```sql
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric DEFAULT 9.99
);
```

Tanımlar yapılırken önemli konulardan biri de veri süreklilik kontrolüdür. Bunlar kısıt (constraint) adı verilen nesnelerle sağlanır. Kısıtlar belli kontrolleri yaparak girilen verinin istenen bir kurala uygunluğu, benzersizliği, tekrarına izin verilip verilmemesi gibi hususların kontrolünü sağlamaktadır. PostgreSQL’de kullanılan kısıt çeşitleri aşağıdadır:

**1. Check**: Girilen her değer ``CHECK`` kuralıyla test edilir ve ``TRUE`` dönen değerlerin tabloya yazılmasına izin verir. Kısıt tanımı tablo oluşturma sürecinde ``CREATE TABLE`` veya sonrasında ``ALTER TABLE`` aşamalarında yapılabilir. Bu süreçte doğrudan bir (veya birden fazla) kolon üzerinde kısıt oluşturulabilir. Oluşturulan kısıt bir isim verilerek veya verilmeksizin kullanılabilir. Aşağıda CHECK kısıtına örnekler görülmektedir.

```sql
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric CHECK (price > 0)
);

CREATE TABLE products (
    product_no integer,
    name text,
    price numeric CONSTRAINT positive_price CHECK (price > 0)
);

CREATE TABLE products (
    product_no integer,
    name text,
    price numeric CHECK (price > 0),
    discounted_price numeric CHECK (discounted_price > 0),
    CHECK (price > discounted_price)
);
```

**2. Not Null**: Bu kısıt, bir kolona kesinlikle veri girilme zorunluluğunu ifade eder ve herhangi bir veri girişinde ya da güncellenmesinde üzerinde ``NOT NULL`` kısıtı olan kolonun boş bırakılmasının önüne geçer. Bir kolon üzerinde birden fazla kısıt, tanımlama sıralaması önemli olmaksızın sorunsuz çalışabilir.

```sql
CREATE TABLE products (
    product_no integer NOT NULL,
    name text NOT NULL,
    price numeric NOT NULL CHECK (price > 0)
);
```

**3. Unique**: Bu kısıt, tanımlandığı kolonun tüm satırlarındaki verilerin birbirinden farklı olmasını gerektirir. Böylece benzersiz verilerle oluşturulmuş bir veri kolonu elde edilebilir. Bu kısıt, herhangi bir kolon tanımında yazılabileceği gibi tablo tanımlaması içinde parantez içinde kısıtın kontrol edeceği kolonları listeleyerek de yazılabilir.

```sql
CREATE TABLE example (
    a integer,
    b integer,
    c integer,
    UNIQUE (a, c)
);
```

**4. Primary Key**: Bu kısıt, bir veya daha fazla kolonun bir tablodaki satırlar için ``UNIQUE`` ve ``NOT NULL`` kısıtlarına aynı anda uyacağını garanti eder. Bir tabloda ``PRIMARY KEY`` kısıtı oluşturulduğunda otomatik olarak benzersiz B-tree indeks de oluşturulur. Bir tablonun sadece bir PRIMARY KEY kısıtı olabilir, fakat istenirse bu kısıt bir veya birden fazla kolonu aynı anda kontrol edebilir.  Oysa bir tablo üzerinde birden fazla ``UNIQUE`` ya da ``NOT NULL`` key olabilir.

```sql
CREATE TABLE example (
    a integer,
    b integer,
    c integer,
    PRIMARY KEY (a, c)
);
```

**5. Foreign Key**: Bu kısıt, bir tablodaki kolonun satırlarında bulunan verinin ilişkili başka bir tablodaki kolondaki verilerle aynı olmasını ister. Bu sayede iki tablo arasında referans bütünlüğü sağlanmış olur. Bu durumda ``FOREIGN KEY`` tanımladığımız tabloda, referanslandığı tabloda olmayan hiç bir satır girilemeyecek ve bu iki tablo arasında bir bağ kurulmuş olacak.

Örneğin ürünlerin ve siparişlerin bulunduğu iki tablo için ürünler tablosundaki ürünlere **PRIMARY KEY**, siparişler tablosundaki (ürünler tablosuna referanslamak istediğimiz) urun_id kolonuna da **FOREIGN KEY** tanımlarsak verilen siparişlerde satmadığımız bir ürünün girilmemesini garanti etmiş oluruz. Zira siparişler tablosuna, ürünler tablosunda olmayan bir ürün girilmeye çalışıldığı zaman **FOREIGN KEY** kısıtı devreye girecek ve hata üreterek “girilmeye çalışılan ürünün ürünler tablosunda bulunmadığı” uyarısını vererek bu satırın eklenmesini önleyecektir.

**FOREIGN KEY** kısıtı hem referanslandığı tablodaki ilgili kolonu belirterek hem de tanımının yapıldığı kolonun tanım detayında belirtilerek yani iki farklı şekilde oluşturulabilir.

```sql
CREATE TABLE orders (
    order_id integer PRIMARY KEY,
    product_no integer REFERENCES products (product_no),
    quantity integer
);


CREATE TABLE t1 (
  a integer PRIMARY KEY,
  b integer,
  c integer,
  FOREIGN KEY (b, c) REFERENCES other_table (c1, c2)
);
```

Bir tabloda (ilişkide olduğu tablo ölçeğinde) birden fazla **FOREIGN KEY** oluşturulabilir. Aşağıdaki veritabanı yapısında ürünler ve siparişlere bir de sipariş detayları tablosu eklenmiştir. Bu sayede bir sipariş içinde birden fazla ürünün olması sağlanmış ve bu detaylar eklenen yeni tabloda tutulmaya başlanmıştır.

Bu tablo incelendiğinde ürünler ve siparişler tablosuna referanslar görülebilir. Burada ilave olarak ``ON DELETE CASCADE`` ve ``ON DELETE RESTRICT`` ifadeleri görülmektedir. Ayrıca *sipariş_id*ve ürün_no kolonları hem **PRIMARY KEY** hem de **FOREIGN KEY** olarak tanımlanmıştır.

```sql
CREATE TABLE products (
    product_no integer PRIMARY KEY,
    name text,
    price numeric
);

CREATE TABLE orders (
    order_id integer PRIMARY KEY,
    shipping_address text,
    ...
);


CREATE TABLE order_items (
    product_no integer REFERENCES products,
    order_id integer REFERENCES orders,
    quantity integer,
    PRIMARY KEY (product_no, order_id)
);
```

**FOREIGN KEY** sayesinde ürünler tablosunda olmayan bir ürün siparişe eklenemez. Fakat siparişi yapılan bir ürünün satışı durdurulsa ve ürünler tablosundan çıkarılsa ne olurdu? Bunu çözmek için birkaç seçenek bulunmaktadır.

**ON DELETE RESTRICT** seçeneği, silme sonrasında referanslı kolonda da verinin silinmesini engellerken ON DELETE CASCADE seçeneği silme sonrasında referanslı kolondaki ilişkili verilerin de silinmesini sağlar. Bu iki seçenek haricinde ``SET NULL`` ve ``SET DEFAULT`` seçenekleri de bulunmaktadır. **SET NULL** ve **SET DEFAULT** seçenekleri silinmek istenen referanslı tablo kolonlarını sırasıyla NULL ve kolonun DEFAULT değeriyle UPDATE etmesini sağlar.

{% include tip.html content="ON DELETE olayına benzer şekilde ON UPDATE olayı da bulunmaktadır. Bu durumda referanslı tablolardan birinin güncellenmesine bağlı gelişecek süreç tanımlanabilmektedir. ON UPDATE seçeneğinde de yine RESTRICT, CASCADE, SET NULL ve SET DEFAULT seçenekleri kullanılabilir." %}

**6. Exclude**: Bu kısıt, herhangi iki satırın belli kolonlarının veya ifadelerinin belirtilen operatör kullanılarak karşılaştırılması sırasında bu operatör kıyaslamalarından en az birini ``FALSE`` veya ``NULL`` döndürmeye zorlar.

Bazı hallerde oluşturulmuş veritabanı objelerinde değişiklik yapmak gerekebilir. Bu durumda bu tabloları silip yeniden yaratmak yerine gereken yerlerin yeni ihtiyaçlar doğrultusunda  değiştirilmesi çokça görülen bir uygulamadır. Bu değişiklik ihtiyaçları tablolara kolon ekleme, kolon silme, kolonun ismini, veri tipini veya boyutunu değiştirme gibi çok sayıda durumu kapsar. Tablo veya kolonları üzerinde yapılacak tüm değişiklikler için **ALTER COLUMN** komutuyla tablo değiştirme süreci başlatılır. Aşağıda bununla ilgili çeşitli örnekler verilmektedir.

```sql
ALTER TABLE products ADD COLUMN description text CHECK (description <> '');
ALTER TABLE products DROP COLUMN description;
ALTER TABLE products ADD CHECK (name <> '');
ALTER TABLE products ADD CONSTRAINT some_name UNIQUE (product_no);
ALTER TABLE products ADD FOREIGN KEY (product_group_id) REFERENCES product_groups;
ALTER TABLE products ALTER COLUMN product_no SET NOT NULL;
ALTER TABLE products ALTER COLUMN product_no DROP NOT NULL;
ALTER TABLE products RENAME COLUMN product_no TO product_number;
ALTER TABLE products RENAME TO items;
```

### DML Komutları

Bu sorgu komutları arasında ``INSERT``, ``UPDATE`` ve ``DELETE`` bloklarını kullanarak tablolar içindeki veriler oluşturulabilir, güncellenebilir veya silinebilir.

**INSERT** komutu veritabanı tablolarında veri yaratmaya yarar. Tabloya yazılması istenen veriler kolon ismi verilerek atılabildiği gibi, tüm kolonlara birden veri atılacağı durumlarda bu kısım es geçilebilir.

```sql
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric
);

INSERT INTO products VALUES (1, 'Cheese', 9.99);

INSERT INTO products (product_no, name, price) VALUES (1, 'Cheese', 9.99);
INSERT INTO products (name, price, product_no) VALUES ('Cheese', 9.99, 1);
```

Ayrıca tek INSERT cümleciği ile birden fazla satır veri girişi de yapılabilir.

```sql
INSERT INTO products (product_no, name, price) VALUES
    (1, 'Cheese', 9.99),
    (2, 'Bread', 1.99),
    (3, 'Milk', 2.99);
```

INSERT cümlesiyle tabloya yazılacak veriler bir SELECT sorgusuyla da çekilebilir.

```sql
INSERT INTO products (product_no, name, price)
  SELECT product_no, name, price FROM new_products
    WHERE release_date = 'today';

```

Halihazırda tabloda bulunan veriler üzerinde bir değişiklik yapılması işlemi ``UPDATE`` cümleciğiyle sağlanır. **UPDATE** bir tablo üzerinde **SET** ile tanımlanan kolonda güncelleme yapar ve çoğu durumda SET sonrasında **WHERE** ile bir kriter verilerek güncellenecek satır ya da satırlardan oluşan bir alt küme elde edilir. Eğer UPDATE, WHERE ile tanımlanmış bir filtreleme işlemine tabi tutulmadan işletilirse bütün tablo üzerinde güncelleme yapacağı için dikkatli olunmalıdır.

```sql
UPDATE products SET price = 10 WHERE price = 5;
```

UPDATE’e benzer şekilde tabloda girilen bir kriter sonucu elde ettiğimiz alt küme içinde gelen tüm satırları silmek için DELETE cümleciği kullanılır. Eğer bir WHERE kriteri girilmezse DELETE komutu tüm tablodaki satırları silecektir. DELETE ile yalnızca tablodaki satırlar silinebilir. Tablodaki kolonun silinmesi için kullanılan DROP ile karıştırılmamalıdır. Zira DROP, tablo tanımını değiştirmeye yönelik bir işlem iken DELETE veri setini düzenlemeye / değiştirmeye yönelik bir işlemdir.

```sql
DELETE FROM products WHERE price = 10;

DELETE FROM products;
```

### DQL Komutları

Veritabanındaki verileri çekmek ve istenen şekilde görüntülemek için kullanılan komutlar bu grup altında anlatılmaktadır. Bir tabloda bulunan veriler en basit anlamda ``SELECT … FROM`` cümleciği kullanılarak sorgulanır. SELECT sonrasında sorguda incelenmek istenen kolonlar (veya * ile tüm kolonlar) sıralanır ve FROM ibaresini takiben yapılan tablo ya da tablolar verilir.

```sql
SELECT * FROM table1;
```

Kolon isimleri listelenirken tek tek yazılabileceği gibi iki kolonun içeriği otomatik olarak sorgu içinde birleştirilerek de listelenmek istenebilir. Aşağıdaki örnekte a ile b+c kolonundan oluşan iki kolon dönecektir.

```sql
SELECT a, b + c FROM table1;
```

SELECT her zaman veritabanındaki tablolardan sonuç getirmek zorunda değildir. Bir işlem sonucu, bir fonksiyon ya da rastgele üretilmiş bir sayı da dönebilir.

```sql
SELECT 3 * 4;

SELECT random();
```

Sorgular, bir “tablo ifadesi” ile hesaplanan tablolardan çekilir. Bir tablo ifadesi aslında FROM cümleciği ile oluşturulur. FROM cümleciğini gerektiğinde ``WHERE``, ``GROUP BY`` ve ``HAVING`` ibareleri takip eder. Seçim yapılan (veya hesaplama yoluyla elde edilen) tablolara takma isimler verilebilir. Bunun için AS ve sonrasında verilmek istenen takma isim girilir. Ayrıca FROM ibaresinden sonra parantez içinde tanımlanmış bir başka sorgu kullanıldığında, alt sorgudan dönen sonuç tablosu, ana sorgunun içinden seçim yapacağı tablo ifadesi gibi çalışır.

Birden fazla tablo ``JOIN`` ile birleştirilebilir ve sorgularda birçok tablodan gelen ilişkili veriler yer alabilir. Çeşitli JOIN tipleri arasında ``CROSS JOIN``, ``INNER JOIN``, ``LEFT/RIGHT OUTER JOIN`` ve ``FULL OUTER JOIN`` sayılabilir. Birleştirilmiş tablolar, ilgili kurallara göre birleştirilmiş birden fazla tablonun birleşiminden oluşmuş / türemiş tablolardır.

Join tipleri içinde **CROSS JOIN** olarak adlandırılan tür aslında iki tablonun kartezyen çarpımını üretir. CROSS JOIN ile çaprazlanan tablolarda sırayla m ve n satır varsa çaprazlanmış tablonun m*n satırı olur.

İki tablonun join’le birleştirilmesi sırasında ``ON`` ve ``USING`` ifadeleri kullanılarak birleştirme şartı ifade edilir. Aşağıda T1 ve T2 tablolarının birleştirilmesi için üç farklı yazım sunulmuştur. Bu yazım ``INNER`` ve ``LEFT / RIGHT / FULL OUTER JOIN`` türleri için kullanılmaktadır. Yazım sırasında ON ve USING kullanılarak birleştirmeye temel oluşturacak kolon eşleşmesinin yapılacağı kolonlar belirtilir.

```sql
T1 { [INNER] | { LEFT | RIGHT | FULL } [OUTER] } JOIN T2 ON boolean_expression
T1 { [INNER] | { LEFT | RIGHT | FULL } [OUTER] } JOIN T2 USING ( join column list )
T1 NATURAL { [INNER] | { LEFT | RIGHT | FULL } [OUTER] } JOIN T2
```

**Inner Join** yapıldığında, eşleşmenin yapılacağı kolonlar incelenir ve her iki tablonun ilgili kolonunda da olan değerlerin çaprazlandığı bir birleştirme yapılır. Aslında bu birleştirme ``ON`` ifadesinden sonra gelen şartın uygulanması sonrası çalıştırılır .

**Outer Join** işlemleri *Left*, *Right* ve *Full* olarak uygulanabilir. Outer Join işleminde önce Inner Join uygulanır. Yani *ON* ifadesinden sonra gelen birleştirme şartına uyan satırlar JOIN ile oluşacak tabloya yazılır. Sonrası için **LEFT JOIN**’de 'T1' tablosunda 'T1' = 'T2' eşitliğine uymayan tüm satırlar eklenirken, **RIGHT JOIN’**de 'T2' tablosunda 'T1' = 'T2' eşitliğine uymayan tüm satırlar eklenir. **FULL OUTER JOIN** işleminde hem *LEFT* hem de *RIGHT OUTER JOIN* ile eklenen tüm satırlar INNER JOIN’le oluşan tabloya eklenir.

**USING** ifadesinin *ON*’dan farkı, USING ifadesinde birleştirilmek istenen 'T1' ve 'T2' tablolarında eşitlenecek kolonların isimlerinin aynı olması halinden istifade edilerek yazım uzunluğunu kısaltmasıdır. Eğer iki tablo için birleştirilecek kolonların isimleri birebir aynı ise ON yerine USING ifadesi yazılır ve sonrasında isimleri aynı kolonlar virgülle ayrılarak girilir. 'T1' ve 'T2' tablolarında 'a' ve 'b' isminde kolonlar olduğunu, aynı tür veriyi tuttuğunu ve 'a' ile 'b' kolonlarını birleştirme için kullanmak istediğimizi varsayalım. "USING (a,b)" ifadesi yazıldığında bu aslında ON T1.a = T2.a AND T1.b = T2.b ifadesine denktir.

**NATURAL** ise *USING* kullanmanın kısayoludur. NATURAL JOIN ile JOIN yapıldığında arka planda ilgili tabloların aynı isimli kolonlarının listelendiği USING ifadeleri oluşturulur.

Bu JOIN türlerine örnek vermek için sırayla 'T1' ve 'T2' tablolarını aşağıdaki gibi oluşturalım.

```sql
    T1                        T2
num | name                num | value
-----+------             -----+-------
   1 | a                    1 | xxx
   2 | b                    3 | yyy
   3 | c                    5 | zzz

```

Aşağıdaki gibi CROSS JOIN yaptığımızda 3x3=9 satırlık bir tablo üretiriz.

```sql
SELECT * FROM t1 CROSS JOIN t2;
 num | name | num | value
-----+------+-----+-------
   1 | a    |   1 | xxx
   1 | a    |   3 | yyy
   1 | a    |   5 | zzz
   2 | b    |   1 | xxx
   2 | b    |   3 | yyy
   2 | b    |   5 | zzz
   3 | c    |   1 | xxx
   3 | c    |   3 | yyy
   3 | c    |   5 | zzz
(9 rows)
```

ON, USING ve NATURAL kullanarak INNER JOIN yaparsak aşağıdaki sonuçları elde ederiz.

```sql
SELECT * FROM t1 INNER JOIN t2 ON t1.num = t2.num;
 num | name | num | value
-----+------+-----+-------
   1 | a    |   1 | xxx
   3 | c    |   3 | yyy
(2 rows)

SELECT * FROM t1 INNER JOIN t2 USING (num);
 num | name | value
-----+------+-------
   1 | a    | xxx
   3 | c    | yyy
(2 rows)

SELECT * FROM t1 NATURAL INNER JOIN t2;
 num | name | value
-----+------+-------
   1 | a    | xxx
   3 | c    | yyy
(2 rows)
```

Aşağıda ise LEFT, RIGHT ve FULL JOIN örnekleri sunulmaktadır.

```sql
SELECT * FROM t1 LEFT JOIN t2 ON t1.num = t2.num;
 num | name | num | value
-----+------+-----+-------
   1 | a    |   1 | xxx
   2 | b    |     |
   3 | c    |   3 | yyy
(3 rows)

SELECT * FROM t1 LEFT JOIN t2 USING (num);
 num | name | value
-----+------+-------
   1 | a    | xxx
   2 | b    |
   3 | c    | yyy
(3 rows)

SELECT * FROM t1 RIGHT JOIN t2 ON t1.num = t2.num;
 num | name | num | value
-----+------+-----+-------
   1 | a    |   1 | xxx
   3 | c    |   3 | yyy
     |      |   5 | zzz
(3 rows)

SELECT * FROM t1 FULL JOIN t2 ON t1.num = t2.num;
 num | name | num | value
-----+------+-----+-------
   1 | a    |   1 | xxx
   2 | b    |     |
   3 | c    |   3 | yyy
     |      |   5 | zzz
(4 rows)
```

Özellikle Join işlemlerinde Tablo.Kolon şeklinde bir notasyon kullandığımızda ve bu kullanım yüzünden sorgunun yazımı çok uzadığında ya da karmaşanın önüne geçmek istediğimizde kullanılabilecek bir kısayol olarak tablolara takma isim verilebilir. Bunun için tablo isminin sonrasında AS ifadesi ve tablo için kullanılacak takma adına girilmesi yeterlidir. Sorgu içinde bundan sonra geçen ifadelerde tablonun gerçek isminin yanı sıra takma ismi de kullanılabilir. Takma ismin JOIN gibi bir işlemde genel kullanımı aşağıdaki şekildedir.

```sql
SELECT * FROM cok_uzun_tablo_ismi t
JOIN baska_uzun_isimli_bir_tablonun_ismi b
ON t.id = b.num;
```

Bazen yaptığımız sorguyu bir (veya join ile oluşturduğumuz birden fazla) tablodan değil, başka bir sorgunun sonucu olarak dönen tablodan yapmak isteriz. Sorgu sonucu olarak dönen tabloya alt sorgu diyebiliriz ve alt sorguyu ana sorgudan ayırmak için parantezler içinde yazarak sorgumuza yerleştirebiliriz.

```sql
FROM (SELECT * FROM table1) AS alias_name
```

VALUES ifadesi ile bir liste üretildiğinde bu da altsorgu olarak kullanılabilir.

```sql
FROM (VALUES ('anne', 'smith'), ('bob', 'jones'), ('joe', 'blow'))
     AS names(first, last)
```

Alt sorgu ifadesi FROM’dan sonra gelmek zorunda değildir ve tüm sorgu içinde herhangi bir yerde olabilir. Sorgulama kümesini süzmek için kullandığımız WHERE ifadesinden sonra gelen bir altsorgu örneği aşağıdadır.

```sql
SELECT * FROM foo
    WHERE foosubid IN (
                        SELECT foosubid
                        FROM getfoo(foo.fooid) z
                        WHERE z.fooid = foo.fooid
                      );
```

Hatta altsorgu bir başka veritabanına bağlantı cümleciği ile bu veritabanına yapılacak bir sorgunun sonucu da olabilir. Özetle aşağıdaki durumda sorgu yapılan ana tablo ifadesi başka veritabanındaki tablo üzerinde yapılan bir sorgudur.

```sql
SELECT *
    FROM dblink('dbname=mydb', 'SELECT proname, prosrc FROM pg_proc')
      AS t1(proname name, prosrc text)
    WHERE proname LIKE 'bytea%';
```

Burada başka bir veritabanına bağlantı kurmayı sağlayan ``dblink( )`` ifadesi bir fonksiyondur.

Tablo ifadesi içerisinde önemli yeri olan bir diğer cümlecik ``WHERE`` cümleciğidir. WHERE kendisinden sonra gelen filtreleme ifadesine göre seçili tablo kolonları içinde filtreleme yapar. WHERE kullanarak tablodaki satırları belli kriterlere göre süzebiliriz.

Süzme kriteri için örnek vermek gerekirse, bir kolonda 'x' değerinden büyük olan satırlar, bir kolonun bir diğer kolona eşit olduğu satırlar, bir kolonun değerinin bir alt küme içindekilerden herhangi biri olduğu satırlar WHERE sonrasında kullanılabilecek filtre cümlecikleri olabilir.

```sql
SELECT ... FROM fdt WHERE c1 > 5

SELECT ... FROM fdt WHERE c1 IN (1, 2, 3)

SELECT ... FROM fdt WHERE c1 IN (SELECT c1 FROM t2)

SELECT ... FROM fdt WHERE c1 IN (SELECT c3 FROM t2 WHERE c2 = fdt.c1 + 10)

SELECT ... FROM fdt WHERE c1 BETWEEN
           (SELECT c3 FROM t2 WHERE c2 = fdt.c1 + 10) AND 100

SELECT ... FROM fdt WHERE EXISTS (SELECT c1 FROM t2 WHERE c2 > fdt.c1)
```

Bir veritabanı sorgusunda seçili bir kolondaki değerleri gruplara ayırarak toplu istatistikler üretmek de mümkündür. GROUP BY cümleciği kolon bazında satır grupları oluştururken, ancak GROUP BY’dan sonra gelebilen HAVING cümleciği de bu gruplar içinde filtreleme yapar. Ayrıca GROUP BY ile kolon bazında gruplandırdığımız bir veri setinde grup bazında minimum, maksimum, toplam ya da ortalama gibi grup istatistikleri üretebiliriz.

```sql
SELECT product_id, p.name, (sum(s.units) * (p.price - p.cost)) AS profit
    FROM products p LEFT JOIN sales s USING (product_id)
    WHERE s.date > CURRENT_DATE - INTERVAL '4 weeks'
    GROUP BY product_id, p.name, p.price, p.cost
    HAVING sum(p.price * s.units) > 5000;
```

SELECT sonrasında DISTINCT ifadesi eklenirse, dönen veri kümesi içinde birbirini tekrarlayan değerlerin bulunduğu satırlar da silinmiş olur.

```sql
SELECT DISTINCT ON (location) location, time, report
    FROM weather_reports
    ORDER BY location, time DESC;
```

SELECT sonrasında kullanılan ``LIMIT`` ifadesi ise sıralı bir şekilde dönen tabloda LIMIT ile belirlenen adette satırın gösterilmesini sağlar. Örneğin bir sınıfta alınan Matematik sınav notları tablodan sorgulandığında en yüksek puanı alan 3 öğrencinin bulunması için SELECT sonrasında LIMIT 3 ifadesi eklenir. Aynı şekilde ``OFFSET`` ifadesi de sorgu sonucuna bir öteleme verilmesini sağlar. Yine LIMIT’in kullanımına benzer şekilde bir sorguda OFFSET 5 eklenirse, sorgu sonucunda dönen veriler 5 satır ötelenerek gösterileceği için ilk 5 kayıttan sonraki kayıtlar ekrana gelir. Yani PostgreSQL ilk 5 satırı kaydırarak gösterir.

İki sorgunun sonucu UNION, INTERSECTION ve EXCEPT kullanılarak birleştirilebilir. Bunun için aşağıdaki şekilde bir yazım uygulanmalıdır.

```sql
sorgu1 UNION [ALL] sorgu2
sorgu1 INTERSECT [ALL] sorgu2
sorgu1 EXCEPT [ALL] sorgu2
```

UNION, sorgu2’nin sonuçlarını sorgu1’in sonuçlarına ekler. Eğer ALL ibaresi eklenmezse aynı satırları getirmez. ``INTERSECT``, sadece her iki sorguda da dönen satırları gösterir ve ``ALL`` ifadesiyle kullanılmazsa sonuç tablosundaki çoklanarak gelen satırlar elimine edilerek tüm duplikasyonlar teke indirilir. ``EXCEPT`` ise sorguların sorgu1 ile dönen kayıtların sorgu2 ile gelen sonuçlardan  farkını döndürür. Yine benzer şekilde ALL kullanılmazsa çift dönen kayıtlar teke indirgenir.

{% include tip.html content="UNION, INTERSECT ya da EXCEPT kullanarak sorgu sonuçlarını birleştirebilmek için, işleme sokulan iki sorgunun aynı miktarda kolon döndürmesi gerekmektedir ki birleştirme işlemi sorunsuz bir şekilde gerçekleştirilebilsin." %}

Sorgu sonucunda dönen tablonun belli bir düzene göre sıralanması için ise ``ORDER BY`` komutu kullanılır. ORDER BY sonrasında dizilim istenen kolonun veya kolonların ismi, dizilimin yönü (büyükten küçüğe / küçükten büyüğe), NULL dönen satırların dizilimde hangi pozisyonda olacağı ile ilgili etiketler de girilir.

```sql
SELECT select_list
    FROM table_expression
    ORDER BY sort_expression1 [ASC | DESC] [NULLS { FIRST | LAST }]
             [, sort_expression2 [ASC | DESC] [NULLS { FIRST | LAST }] ...]
```

Dizilimin yönü varsayılan olarak artan şekildedir. Bununla birlikte kolon ismini takiben ``DESC`` ifadesi konulursa o kolon için azalan dizilim sağlanır. Ayrıca ORDER BY ifadesinin en sonuna ``NULLS FIRST`` veya ``NULLS LAST`` eklenirse bu dizilimde NULL satırların dizilimin başına ya da sonuna konulacağı konusu PostgreSQL’e bildirilmiş olur.

Veritabanında bir tablo oluşturmaksızın, sadece sorgu içinde kullanılmak amacıyla bir tablo oluşturup kullanabilmek için ``VALUES`` ifadesi ile bir liste yaratılabilir. Bu listelerde birden fazla kolon oluşturabilmek için ifadeler virgülle; birden fazla satır oluşturabilmek için ise parantezler içinde yazılır. Aşağıda örneği bulunmaktadır.

```sql
SELECT * FROM (VALUES (1, 'one'), (2, 'two'), (3, 'three')) AS t (num,letter);
 num | letter
-----+--------
   1 | one
   2 | two
   3 | three
(3 rows)
```

{% include tip.html content="PostgreSQL VALUES ile oluşturulan listelerdeki kolonlar için varsayılan olarak column1, column2, column3 gibi isimler verir." %}

Veritabanında ilave tablolar oluşturmadan geçici tablolar oluşturmak ve karmaşık olabilecek sorguları kısaltmak için kullanılabilecek bir başka yapı daha vardır. ``WITH`` ile oluşturulan geçici tablolar, geçtiği sorgu içinde bellekte oluşturulur ve gerekli satırları döndürdükten sonra yok olurlar.

Aşağıdaki örnekte WITH ile *regional_sales* geçici tablosu oluşturulmuş ve bu geçici tablo kullanılarak yine *top_regions* geçici tablosu oluşturulmuştur. En sonunda ise bu iki geçici tablo, WITH tanımından sonra gelen esas SELECT tablosunda kaynak olarak kullanılmıştır.

```sql
WITH regional_sales AS (
    SELECT region, SUM(amount) AS total_sales
    FROM orders
    GROUP BY region
), top_regions AS (
    SELECT region
    FROM regional_sales
    WHERE total_sales > (SELECT SUM(total_sales)/10 FROM regional_sales)
)
SELECT region,
       product,
       SUM(quantity) AS product_units,
       SUM(amount) AS product_sales
FROM orders
WHERE region IN (SELECT region FROM top_regions)
GROUP BY region, product;
```

``RECURSIVE`` kullanımı ile WITH kendi çıktısına referanslanabilir ve bu sayede kendi içinde tekrarlanabilen sorgular yazılabilir. Aşağıdaki ifade ile 1’den 100’e kadar olan tamsayıların toplamı alınmaktadır.

```sql
WITH RECURSIVE t(n) AS (
    VALUES (1)
   UNION ALL
    SELECT n+1 FROM t WHERE n < 100
)
SELECT sum(n) FROM t;
```

{% include links.html %}
