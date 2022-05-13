---
layout: default
title: Temel Eklentiler
parent: Eklentiler ve Veri Aktarımı
nav_order: 1
---

## Temel Eklentiler

### FDW

2003 yılında SQL/MED (SQL harici verinin yönetimi) isminde bir spesifikasyon tanımlanarak SQL standardına eklendi. 2011’de de PostgreSQL 9.1’e read - only olarak eklenen bu destek sayesinde başka veritabanlarındaki veritabanı objeleri PostgreSQL’e alınarak okunabilir hale geldi. 2013 yılında PostgreSQL 9.3 ile birlikte yabancı objeleri yazma desteği de getirildi. Bu işin yapılmasını sağlayan FDW’ler (Foreign Data Wrappers: Yabancı veri paketleyiciler) çok sayıda farklı veritabanından düz dosyalara değişen devasa bir skalada verinin okunup PostgreSQL veritabanına alınabilmesini sağlamaktadır [](https://wiki.postgresql.org/wiki/Foreign_data_wrappers) .

FDW’leri gruplandırmak gerekirse genel SQL veritabanları, gelişmiş SQL veritabanları, NoSQL veritabanları, dosyalar, konumsal (coğrafi) veriler, LDAP, Web, Big Data, işletim sistemi paketleyicileri gibi çok sayıda paketleyici grubu altında onlarca farklı FDW bulmak ve PostgreSQL altında kullanmak mümkündür. İnternette geniş FDW listelerine ulaşmak için sitelere referans olarak yer verilmiştir [](https://pgxn.org/tag/fdw/).

PostgreSQL üzerinde FDW kullanarak veri çekebilmemize olanak tanıyan birkaç FDW aşağıda listelenmiştir. Bunlar eklenti şeklinde olduğu için `CREATE EXTENSION <istenen_fdw_ismi>` şeklinde yüklenebilirler.

- PostgreSQL   ⇒  postgres_fdw
- MSSQL        ⇒  tds_fdw
- MYSQL        ⇒  mysql_fdw
- MongoDB      ⇒  mongo_fdw

FDW, uzak veri kaynağına yapılan bağlantının kurulması, verilerin derlenmesi, çekilmesi ve PostgreSQL bünyesine atılması adımlarının tamamından sorumludur. Eğer çekilen veri güncellenerek PostgreSQL’e atılacaksa, FDW bu görevi de yerine getirebilir.

Bu noktada metin dosyasında saklanmış bir dosyadaki veriyi PostgreSQL’e atmayı inceleyelim. Bir sunucudaki dosya sisteminde bulunan dosyalara erişebilmek için **file_fdw** eklentisinden yararlanabiliriz. Bu eklenti diskteki dosyaları ve verilerini okur, gerekirse sunucudaki programları da işletip çıktılarını da okuyabilir. Tabi başka programın kullanılması halinde çıktıların PostgreSQL’in `COPY FROM` komutuyla okunabilir formatta olması gerekmektedir.

Bu uygulamada sunucudaki bir log dosyası okunup PostgreSQL’de oluşturulacak bir yabancı tabloya aktarılacaktır. Bunun için önce sunucuda eklenti, sonra da yabancı bir sunucu oluşturulur.

```sql
CREATE EXTENSION file_fdw;

CREATE SERVER pglog FOREIGN DATA WRAPPER file_fdw;
```

Sonrasında yabancı tablo oluşturulabilir. Bunun için `CREATE FOREIGN TABLE` komutu kullanılır ve böylece hem verilerin okunduğu CSV dosyasının adı, formatı ile yazılacak tablonun kolonları tanımlanır.

```sql
CREATE FOREIGN TABLE pglog (
  log_time timestamp(3) with time zone,
  user_name text,
  database_name text,
  process_id integer,
  connection_from text,
  session_id text,
  session_line_num bigint,
  command_tag text,
  session_start_time timestamp with time zone,
  virtual_transaction_id text,
  transaction_id bigint,
  error_severity text,
  sql_state_code text,
  message text,
  detail text,
  hint text,
  internal_query text,
  internal_query_pos integer,
  context text,
  query text,
  query_pos integer,
  location text,
  application_name text
) SERVER pglog
OPTIONS ( filename '/home/josh/data/log/pglog.csv', format 'csv' );
```

Bu sayede *pglog* yabancı sunucusuna oluşturulan *pglog* tablosuna tüm veriler atılır. Aynı çalışmayı bu sefer düz dosya yerine bir başka veritabanındaki veriyi çekerek uygulayalım.

```sql
CREATE EXTENSION postgres_fdw;

CREATE SERVER server_name FOREIGN DATA WRAPPER postgres_fdw OPTIONS (host 'host_ip', dbname 'db_name', port 'port_number');
```

Bu şekilde eklentiyi kurup başka bir PostgreSQL veritabanına yapılacak bağlantıyı oluşturduktan sonra kullanıcıları ve şemayı tanımlayalım.

```sql
CREATE USER MAPPING FOR CURRENT_USER
    SERVER server_name
    OPTIONS (user 'user_name', password 'password');

CREATE SCHEMA schema_name;
```

En son olarak tanımladığımız şemaya, uzaktaki şemayı klonlayalım ve sonrasında da verileri içeri çekerek çalışmayı tamamlayalım.

```sql
     IMPORT FOREIGN SCHEMA schema_name_to_import_from_remote_db
     FROM SERVER server_name
     INTO schema_name;

     SELECT * FROM schema_name.table_name;
```

`IMPORT FOREIGN SCHEMA` kullanarak tüm şemayı kopyalamak mümkün olduğu gibi şema altındaki istenen tablolar da kopyalanabilir. Bunun için aşağıda iki örnek verilmiştir.

```sql
IMPORT FOREIGN SCHEMA foreign_films
    FROM SERVER film_server INTO films;

IMPORT FOREIGN SCHEMA foreign_films LIMIT TO (actors, directors)
    FROM SERVER film_server INTO films;
```

### UNACCENT

Bu eklenti, kelimelerin içindeki aksanlı harfleri, aksansız halleri ile değiştiren bir metin arama sözlüğü şeklinde çalışır. Bir tür süzgeç sözlük olan unaccent’in çıktısı, normal sözlüklerin aksine kendi içine döndürülmek yerine her zaman kendinden sonraki sözlüğe aktarılır. Bu sayede aksanlı harflere duyarsız tam metin arama işlemleri çalıştırılabilir.

Bir unaccent sözlüğü aşağıdaki seçenekleri kabul eder:

- Çeviri kuralları listesini içeren dosyanın ismi **RULES** olmalıdır. Bu dosya, PostgreSQL kurulumunun paylaşılmış veri klasörünün (**$SHAREDIR**) altındaki **$SHAREDIR/tsearch_data/** klasöründe saklanır. Dosyanın ismi **.rules** ile bitmelidir.

Kurallar aşağıdaki formata uygun olmalıdır.

- Her satırda bir çeviri kuralı bulunmalıdır. Bu kurallar aksanlı harfin dönüşeceği aksansız harf şeklinde olmalıdır. Bir sonraki satır aynı harf için diğer bir aksanlı harfe ayrılmalıdır. Şu örnekteki gibi,

```text
À        A
Á        A
Ã        A
Ä        A
Å        A
Æ        AE
```

- Alternatif olarak eğer bir satırda sade tek bir karakter verilirse, bu karakterin olduğu her yer silinir. Bu özellikle aksanların ayrı karakterlerle ifade edildiği dillerde kullanışlıdır.

- Her **karakter**, boşluk karakterinden oluşmayan herhangi bir metin olabilir. Bu yüzden unaccent sözlükleri, sadece aksanlı harf sözlüğü değil, bir tür metin (arama ve) değiştirme sözlüğü olarak da kullanılabilir.

- Diğer PostgreSQL metin arama konfigürasyon dosyalarıyla olduğu gibi kurallar da UTF-8 kodlamasıyla saklanmalıdır. Bu veri otomatik olarak varsayılan veritabanı kodlamasına çevrilecektir. Çevrilemez karakterlerin olduğu herhangi bir satır (hata üretmeksizin) göz ardı edilir. Bu yüzden kural dosyaları, varsayılan kodlamada olmayan karakterlere yönelik kuralları da içerebilir.

Kapsamlı bir örnek, unaccent eklentisi kurulduğunda **$SHAREDIR/tsearch_data/** klasöründe **unaccent.rules** dosyasında da incelenebilir. Bu dosyada çoğu Avrupa dilleri için kapsamlı bir örnek sunulmaktadır. Bu kural dosyası aksanlı karakterleri aynı harflerin aksansız formlarına dönüştürmekle kalmaz, bağlama formundaki kaynaştırılmış harfleri de (Æ karakterini AE’ye çevirmek gibi) basit harf ikililerine çevirir.

PostgreSQL’e unaccent eklentisi kurulduğunda unaccent isminde bir metin arama şablonu ve bir sözlük de eklenir. Unaccent sözlüğünde varsayılan `RULES='unaccent'` parametresi ayarlı durumdadır. Bu sayede unaccent, kurulur kurulmaz unaccent.rules dosyasıyla birlikte kullanıma hazırdır. Tabi bu kural dosyası da aşağıdaki komutla istendiği an değiştirilebilir ya da şablon esas alınarak yeni bir sözlük dosyası yaratılabilir.

```sql
ALTER TEXT SEARCH DICTIONARY unaccent (RULES='my_rules');
```

Sözlüğü test etmek için aşağıdaki sorgu kullanılabilir:

```sql
select ts_lexize('unaccent','Hôtel');

ts_lexize
-----------
 {Hotel}
(1 row)
```

Aşağıda ise bir metin arama konfigürasyonuna nasıl unaccent sözlüğü eklenebileceğinin örneği verilmiştir.

```sql

CREATE TEXT SEARCH CONFIGURATION fr ( COPY = french );
ALTER TEXT SEARCH CONFIGURATION fr
        ALTER MAPPING FOR hword, hword_part, word
        WITH unaccent, french_stem;

select to_tsvector('fr','Hôtels de la Mer');

    to_tsvector
-------------------
 'hotel':1 'mer':4
(1 row)

select to_tsvector('fr','Hôtel de la Mer') @@ to_tsquery(‘fr’, ‘Hotels’);

 ?column?
----------
 t
(1 row)

select ts_headline('fr','Hôtel de la Mer’, to_tsquery(‘fr’, ‘Hotels’));

      ts_headline
------------------------
 <b>Hôtel</b> de la Mer
(1 row)
```

Bir metindeki aksanlı harfleri almak için `unaccent( )` fonksiyonu kullanılır. Bu fonksiyon unaccent-type sözlükleri kullanan bir wrapper fonksiyon olup normal metin arama ihtiyaçları kapsamında da kullanılabilir.

```sql
unaccent([dictionary regdictionary, ] string text) returns text
```

Eğer dictionary argümanı kullanılmazsa, metin arama sözlüğü olarak unaccent varsayılanı kullanılır. Aşağıdaki iki ifade birbirine denktir.

```sql
SELECT unaccent('unaccent', 'Hôtel');
SELECT unaccent('Hôtel');
```

### CITEXT

Bu modül (citext), büyük - küçük harfe duyarsız (case-insensitive:ci) bir metin veri tipi sağlar. Temelde yaptığı iş, metinsel ifadeleri karşılaştırırken metinleri `lower( )` fonksiyonu içinde çağırmaktır. Bununla birlikte tam olarak text veri tipi gibi davranır.

PostgreSQL’de metinleri küçük harflerle karşılaştırmak için standart yaklaşım `lower( )` fonksiyonunu kullanmaktır:

```sql
SELECT * FROM tab WHERE lower(col) = LOWER(?);
```

Bu yeteri kadar başarılı bir yöntem olmasına rağmen bazı dezavantajları vardır:

- Bu kullanım SQL yazımını gereğinden fazla ayrıntıya boğarken, kullanıcıyı her zaman lower kullanarak sonuç çektiğini hatırlamak durumunda bırakır.

- Eğer `lower( )` kullanarak fonksiyonel bir başka indeks yaratılmadıysa, indeks kullanmak mümkün olmayacaktır.

- Kolon üzerinde UNIQUE ya da PRIMARY KEY tanımlandıysa, bunun üzerine oluşturulacak indeks de büyük - küçük harfe duyarlı olarak oluşacaktır. Bu yüzden büyük - küçük harfe duyarsız aramalar yapmak kullanışsız olacak, tekilliğe zorlayan aramalar yapılamayacaktır.

citext veri tipi SQL sorgularından `lower( )` fonksiyonunu elemeye ve birincil anahtarda dahi büyük - küçük harfe duyarsız aramalar yapmaya izin verecektir. Aynı veritabanının **LC_CTYPE** ayar kurallarına bağlı kalarak büyük ve küçük harflerin uyumla örtüşmesini denetleyen text veri tipi gibi, citext yerel dil ayarlarına uyum sağlar. Yine bu davranış da sorgularda `lower( )` fonksiyonunun kullanımında olduğu gibidir. Fakat bu işlev veri tipi tarafından şeffaf bir şekilde yerine getirildiğinden sorgularda özel olarak bu hususun hatırlanmasına da gerek kalmaz.

Bu örnekte SELECT cümlesi, nick kolonu *larry* değerini içermesine ve sorguda *Larry* istenmesine karşın bir satırlık sonucu döndürür.  

```sql
CREATE TABLE users (
    nick CITEXT PRIMARY KEY,
    pass TEXT   NOT NULL
);


INSERT INTO users VALUES ( 'larry', sha256(random()::text::bytea) );
INSERT INTO users VALUES ( 'Tom', sha256(random()::text::bytea) );
INSERT INTO users VALUES ( 'Damian',sha256(random()::text::bytea) );
INSERT INTO users VALUES ( 'NEAL', sha256(random()::text::bytea) );
INSERT INTO users VALUES ( 'Bjørn', sha256(random()::text::bytea) );

SELECT * FROM users WHERE nick = 'Larry';
```

citext modülü her metinsel ifadeyi, sanki `lower( )` ile çağrılmış gibi, küçük karaktere dönüştürerek karşılaştırma işlemlerini yapar. Dolayısıyla, küçük harfli halleri birbirinin aynısı olan ifadeler eşit olarak algılanır.

citext eklentisine özgü geliştirilmiş metin işleme operatörleri ve fonksiyonları bulunmaktadır. Bunların kullanımı sayesinde büyük - küçük harfe duyarsız bir sıralama (COLLATION) elde edilebilir. Örneğin `~` ve `~*` düzenli ifade (regex / regular expressions) operatörleri citext’te uygulandığında aynı davranışı sergilerler. Bu operatörlerin her ikisi de büyük - küçük harfe duyarsız bir şekilde eşleştirme yaparlar. Aynı şart, benzer şekilde, `!~` ve `!~*` operatörleri arasında, `LIKE` operatörleri `~~` ve `~~*` arasında ve `!~~` ile `!~~*` arasında da geçerlidir. Eğer büyük küçük harfe duyarlı şekilde bir karşılaştırma yapılmak istenirse o zaman operatör argümanları text tipine dönüştürülebilir.

Benzer şekilde aşağıdaki fonksiyonların tamamı, fonksiyon argümanları citext olarak verilmiş ise büyük - küçük harfe duyarsız bir şekilde karşılaştırma yapar:

- `regexp_match()`
- `regexp_matches()`
- `regexp_replace()`
- `regexp_split_to_array()`
- `regexp_split_to_table()`
- `replace()`
- `split_part()`
- `strpos()`
- `translate()`

{% include note.html content="Düzenli ifade fonksiyonları kullanılırken, büyük - küçük harfe duyarlı karşılaştırma yapılmak isteniyorsa `c` opsiyonu belirtilerek büyük - küçük harf duyarlılığı ile eşleştirme yapılması sağlanmalıdır. Yoksa, büyük küçük harf duyarlılığı, ancak bu fonksiyonlardan herhangi biri kullanılmadan önce text tipine dönüşüm yapılarak elde edilebilir." %}

### BTREE_GiST

btree_gist, çeşitli veri tipleri (int2, int4, int8, float4, float8, numeric, timestamp with time zone, timestamp without time zone, time with time zone, time without time zone, date, interval, oid, money, char, varchar, text, bytea, bit, varbit, macaddr, macaddr8, inet, cidr, uuid, ve tüm enum tipleri) için, B-tree indeksine denk davranışa sahip GiST indeks operatör sınıfları sunar. Genel olarak bu operatör sınıfları B-tree indeks metotlarından daha iyi bir performans sunmayacak hatta standart B-tree kodunun önemli bir özelliğini, tekilliğe zorlama kabiliyetini de sağlamayacaktır. Fakat yine de B-tree indeksinde olmayan, aşağıda anlatılmış bazı yetenekleri de beraberlerinde getirecektir. Ayrıca bu operatör sınıfları, çok kolonlu GiST indeksine ihtiyaç duyulan yerlerde de kullanışlı bir durum kazanacaktır.

Tipik B-tree arama operatörlerine ilave olarak btree_gist eklentisi **eşit değildir** (`<>`) operatörü için de indeks desteği sunar. Bu, aşağıda da anlatıldığı gibi, hariç tutma kısıtında (exclusion constraint) kullanışlı olacaktır.

Ayrıca yapısı gereği mesafe ölçümü içeren veri türleri için btree_gist indeksi bir mesafe operatörü (btree_gist `<->` operatörünü kullanır) tanımlar ve böylece bu operatörü kullanarak **en yakın komşu** aramaları için GiST indeksiyle arama desteği getirir. Mesafe operatörü int2, int4, int8, float4, float8, timestamp with time zone, timestamp without time zone, time without time zone, date, interval, oid ve money türü verilerde uygulatılabilir.

Btree yerine btree_gist kullanarak yapılan örnek aşağıda sunulmuştur:

```sql
CREATE TABLE test (a int4);
-- create index
CREATE INDEX testidx ON test USING GIST (a);
-- query
SELECT * FROM test WHERE a < 10;
-- nearest-neighbor search: find the ten entries closest to "42"
SELECT *, a <-> 42 AS dist FROM test ORDER BY a <-> 42 LIMIT 10;
```

Bir hayvanat bahçesi kafesinde sadece tek türde hayvan tutulmasına yönelik bir kural koyabilmek için aşağıda bir hariç tutma kısıtı (exclusion constraint) örneği gösterilmiştir.

```sql
CREATE TABLE zoo (
  cage   INTEGER,
  animal TEXT,
  EXCLUDE USING GIST (cage WITH =, animal WITH <>)
);

INSERT INTO zoo VALUES(123, 'zebra');
INSERT 0 1

INSERT INTO zoo VALUES(123, 'zebra');
INSERT 0 1

INSERT INTO zoo VALUES(123, 'lion');
ERROR:  conflicting key value violates exclusion constraint "zoo_cage_animal_excl"
DETAIL:  Key (cage, animal)=(123, lion) conflicts with existing key (cage, animal)=(123, zebra).

INSERT INTO zoo VALUES(124, 'lion');
INSERT 0 1
```

{% include links.html %}
