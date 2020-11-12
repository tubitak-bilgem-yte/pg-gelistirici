---
title: "PL/pgSQL - Prosedürel SQL Dili"
tags: [PostgreSQL]
keywords: PostgreSQL
last_updated: November 9, 2020
summary: "PL/pgSQL - Prosedürel SQL Dili"
sidebar: mydoc_sidebar
permalink: mydoc_pl_pgsql.html
folder: mydoc
---

## PL/pgSQL - Prosedürel SQL Dili

### PL/pgSQL'in Yapısı

PL/pgSQL, PostgreSQL veritabanı sistemi için yüklenebilir prosedürel bir dildir. PL/pgSQL’in tasarım amaçları şu özelliklere sahip yüklenebilir bir prosedürel dil yaratmaktır:

- Fonksiyon ve tetikleyici yazmakta kullanılabilmesi
- SQL diline kontrol yapıları ekleyebilmesi
- Karmaşık hesaplamalar yapabilmesi
- Tüm kullanıcı tanımlı tipleri, fonksiyonları ve operatörleri miras alabilmesi
- Sunucu tarafından güvenilir olarak tanımlanabilmesi
- Kullanımının kolay olması

PL/pgSQL’de yaratılmış fonksiyonlar, gömülü PostgreSQL fonksiyonlarının kullanılabildiği her yerde kullanılabilmektedir. Örneğin, PL/pgSQL kullanarak karmaşık hesaplama fonksiyonları yazabilmek ve bunları sonra operatör tanımlamada ya da indeks ifadelerinde kullanabilmek mümkündür.

PostgreSQL 9.0 ve sonrasında PL/pgSQL varsayılan olarak kurulmaktadır.

PostgreSQL ve birçok diğer ilişkisel veritabanı, sorgulama dili olarak SQL kullanır. SQL kodu, aktarım ve öğrenim kolaylığı sunar. Bununla birlikte her SQL cümleciği veritabanı sunucusu tarafından ayrı ayrı işletilmelidir. Bu, istemci uygulamasının her sorguyu veritabanı sunucusuna göndermesi, işlenmesi ve sonuçların sunulması için beklemesi anlamına gelmektedir. Tüm bu adımlar, süreçler arası iletişim, ve istemci ile sunucunun ayrı makinalarda yer alması halinde de network iletişimi sayesinde gerçekleşir.

PL/pgSQL sayesinde bir hesaplama bloğu ve veritabanı sunucusu içindeki sorgu serileri gruplanabilir. Böylece prosedürel bir dilin gücü ile SQL kullanmanın kolaylığı bir arada elde edilirken istemci / sunucu iletişiminden kaynaklı yüklenmeden / gecikmeden tasarruf edilebilir. Böylece, saklı fonksiyon / yordamları kullanmayan bir uygulamaya kıyaslandığında, PL/pgSQL kullanımı sayesinde ölçülebilir derecede yüksek bir performans artışı kazanılır. Hepsinin yanında PL/pgSQL, SQL’in tüm veri tiplerini, operatör ve fonksiyonlarını kullanabilir.

PL/pgSQL’de yazılmış fonksiyonlar, sunucu tarafında kabul edilen **skalar** veya **dizi** tipinde argümanları kabul edebilir, fonksiyon sonucu olarak bu türlerde çıktılar döndürebilir. Ayrıca isimlendirilerek tanımlanmış herhangi türde bir **kompozit** veri (satır) tipini de kabul edebilir ve döndürebilir.

PL/pgSQL fonksiyonları, ``VARIADIC`` işaretleyicisini kullanmak suretiyle değişken sayıda argümanı kabul edebilecek şekilde de tanımlanabilirler. Bu, SQL fonksiyonları için de aynı şekilde çalışır.  PL/pgSQL fonksiyonları benzer şekilde ``anyelement``, ``anyarray``, ``anynonarray``, ``anyenum`` veya ``anyrange`` gibi çok-biçimli (polimorfik) tipleri de kabul edecek / döndürecek şekilde tanımlanabilir. PL/pgSQL fonksiyonlar, bir küme (set) ya da tablo döndürecek şekilde de tanımlanabilirler. Bu türde bir fonksiyon çıktısını, sonuç setinde istenen her bir eleman için ``RETURN NEXT`` komutunu işleterek veya çalıştırılan bir sorgunun sonuç çıktısını üretmek için ``RETURN QUERY`` kullanarak kendi çıktısını üretir.

Son olarak bir PL/pgSQL fonksiyonu eğer bir değer döndürmeyecekse ``RETURN VOID`` ifadesini içerebilir. Tabi bu tür bir durumda alternatif olarak fonksiyon yerine prosedür olarak da yazılabilir.

PL/pgSQL’de yazılan fonksiyonlar, sunucuda ``CREATE FUNCTION`` komutu işletilerek tanımlanır. Bu türde bir komutun yapısı şu şekilde olmalıdır:

```sql
CREATE FUNCTION somefunc(integer, text) RETURNS integer
AS 'fonksiyonun gövdesini oluşturan ifadeler'
LANGUAGE plpgsql;
```

PL/pgSQL blok yapılı bir dildir. Fonksiyonun gövdesini oluşturan ifadelere **blok** denilir. Bir PL/pgSQL bloğu şu şekilde tanımlanmalıdır:

```sql
[ <<etiket>> ]
[ DECLARE
    tanımlamalar]
BEGIN
    ifadeler
END [ etiket ];
```

Blok içindeki tüm tanımlamalar ve tüm ifadeler ayrı ayrı noktalı virgül kullanarak sonlandırılır. Bir bloğun içindeki bir başka blok da ``END`` ifadesinden sonra noktalı virgülle sonlandırılmalıdır. Fakat fonksiyonun gövdesini sonlandıran en sondaki nihai END ifadesinin noktalı virgülle sonlandırılma zorunluluğu yoktur. ``BEGIN``’den sonra noktalı virgül konulmaz, konulursa hata üretir.

Tek satırlık bir yorum ifadesi yazmak için çift tire (``--``) kullanılırken birden fazla satırı kapsayan yorumlar ``/*`` ve ``*/`` bloğu arasında yazılır.

{% include note.html content="PL/pgSQL’de kullanılan BEGIN / END anahtar sözcüklerini transaction kontrolünde kullanılan ifadelerle karıştırmamak gereklidir. Burada sadece gruplama amacıyla kullanılan BEGIN / END transaction’ı başlatıp durdurmaz, sadece yazılan kod bloğu üzerinde bir gruplama yapar, iç içe girebilir." %}

### Deklerasyonlar

Bir bloktaki tüm değişkenler bloğun deklerasyonlar kısmında tanımlanmalıdır. Bunun tek istisnası bir FOR döngüsünün içinde tanımlanan döngü değişkenleridir. Değişkenler, herhangi bir SQL veri tipinde tanımlanabilir. Aşağıda değişken tanımlama söz dizimi ve tanımlara ait bazı örnekler bulunmaktadır:

```sql
name [ CONSTANT ] type [ COLLATE collation_name ] [ NOT NULL ] [ { DEFAULT | := | = } expression ];

user_id integer;
quantity numeric(5);
url varchar;
myrow tablename%ROWTYPE;
myfield tablename.columnname%TYPE;
arow RECORD;
```

Değişkenlere ``DEFAULT`` ile varsayılan bir değer verilebilir, ya da verilmeyip ``NULL`` olarak yaratılabilir.

Fonksiyon parametreleri ise ``$1``, ``$2`` gibi tanımlayıcılarla (identifier) tanımlanır. Okunurluğu arttırmak için $n notasyonu yerine fonksiyon parametrelerine de takma isim (alias) verilebilir. Takma isimler fonksiyon tanımı içinde doğrudan verilebileceği gibi $n değişkenine de açıkça atanabilir. Örnek:

```sql
CREATE FUNCTION sales_tax(subtotal real) RETURNS real AS $$

-- veya

name ALIAS FOR $n;

```

PL/pgSQL ifadelerinin içindeki tüm alt ifadeler sunucunun ana SQL işlecinde işletilir. Örneğin bir PL/pgSQL ifadesinde,

```sql
IF expression THEN ...
```

şeklinde bir ifade kurulduğunda PL/pgSQL ifadeyi

```sql
SELECT expression
```

tarzında bir sorguyu, ana SQL motoruna besleyerek işletecektir. Burada SELECT komutunu oluştururken, PL/pgSQL’de oluşturulmuş tüm değişken isimleri, arka planda kendisine karşılık gelen parametrelerle değiştirilir. Bu uygulama, SELECT sorgu planının sadece bir kere hazırlanmasını ve sonra ardısıra gelen değerlendirmelerin değişkenlerin farklı değerleriyle de tekrar tekrar kullanılabilmesini sağlar. Dolayısıyla, bir ifadenin ilk sefer kullanılışı sırasında aslında gerçekten olan şey, ``PREPARE`` komutunun kullanılışıdır. Örneğin, x ve y şeklinde iki değişken tanımlamış olsak ve şunu yazsak,

```sql
IF x < y THEN ...
```

arka planda buna denk şekilde olan durum şu komuttur.

```sql
PREPARE statement_name(integer, integer) AS SELECT $1 < $2;
```

Hazırlanmış olan bu sorgu her ``IF`` ifadesi ile yeniden işletilir (EXECUTE edilir). Bu işletme sırasında PL/pgSQL değişkenlerinin değerleri parametre değerleri olarak iletilir. Normal olarak bu detaylar, PL/pgSQL kullanıcı için önemsiz olmakla birlikte karşılaşılan bir sorunu çözmede yardımcı olacak detaylardır.

### PL/pgSQL’de Kontrol Yapıları

PL/pgSQL’in en önemli kısımlarında biri muhtemelen kontrol yapılarıdır. Bu yapıların kullanımı sayesinde PostgreSQL verileri çok daha güçlü ve esnek bir şekilde manipüle edilebilir. Bu bölümde veriyi kontrol etmek için kullanılan anahtar sözcüklerden bahsedilecektir.

Bir fonksiyondan değer döndürmek için kullanılan iki anahtar kelime mevcuttur: ``RETURN`` ve ``RETURN NEXT``.

Bir ifade (expression) içeren RETURN anahtar sözcüğü fonksiyonu sonlandırarak ifadenin değerini çağrıcıya döndürür. PL/pgSQL fonksiyonlarında bu tip bir dönüşte bir veri kümesi dönmez.

```sql
RETURN expression;
```

Eğer yazılan fonksiyonda **RETURN** sonrasında bir ifade yoksa (yani sadece RETURN ibaresi ile fonksiyon tanımlanmışsa) çıktı parametre değişkenlerinin en güncel değerleri döndürülür.

Eğer fonksiyon RETURN ifadesinin void olarak döneceği tanımlanmışsa, fonksiyondan erken çıkmak için bir RETURN ifadesi kullanılabilir, fakat bu durumda RETURN’ü takiben bir ifade yazılmamalıdır.

Bir fonksiyonun RETURN değeri kesinlikle tanımsız bırakılamaz. Eğer kontrol, en üst kod düzeyinin sonuna ulaştığında bir RETURN ifadesine halen denk gelmemiş durumdaysa, bir çalışma zamanı hatası (run-time error) üretecektir. Bu kısıtlama, çıktı parametresi olan fonksiyonlara ve void döndüren fonksiyonlara uygulanmaz. Bu durumlarda en üst kod düzeyi tamamlandığında RETURN otomatik olarak işletilir.

Bir PL/pgSQL fonksiyonu ``SETOF <herhangi_bir_türde_veri>`` şeklinde tanımlandıysa, prosedür bir miktar farklı olacaktır. Bu durumda, dönüş ifadesinin her bir bileşeni bir ``RETURN NEXT`` veya ``RETURN QUERY`` komut dizisiyle tanımlanacak ve sonrasında hiçbir argüman almamış nihai bir RETURN komutu kullanılarak fonksiyon çalışmayı tamamlayacaktır. Hem skaler hem de vektörel değerleri döndürmek için RETURN NEXT, bir sorgu işleterek bunun sonuçlarını döndürmek için ise RETURN QUERY kullanılır. Bu ikisi aynı komut kümesinin içinde sınırsızca kullanılabilir ve kümenin çalışması halinde sonuçlar birbirine katıştırılarak gösterilir.

```sql
RETURN NEXT expression;
RETURN QUERY query;
RETURN QUERY EXECUTE command-string [ USING expression [, ... ] ];
```

``RETURN QUERY EXECUTE`` ise sorgunun dinamik olarak işletilmesi durumunda kullanılır. Burada ``USING`` ifadesi kullanılırsa parametreler de sorguya dahil edilebilir.

RETURN NEXT ve RETURN QUERY için iki örnek aşağıda sırayla sunulmuştur.

```sql
CREATE TABLE foo (fooid INT, foosubid INT, fooname TEXT);
INSERT INTO foo VALUES (1, 2, 'three');
INSERT INTO foo VALUES (4, 5, 'six');

CREATE OR REPLACE FUNCTION get_all_foo() RETURNS SETOF foo AS
$BODY$
DECLARE
    r foo%rowtype;
BEGIN
    FOR r IN
        SELECT * FROM foo WHERE fooid > 0
    LOOP
        -- can do some processing here
        RETURN NEXT r; -- return current row of SELECT
    END LOOP;
    RETURN;
END
$BODY$
LANGUAGE plpgsql;

SELECT * FROM get_all_foo();
```

```sql
CREATE FUNCTION get_available_flightid(date) RETURNS SETOF integer AS
$BODY$
BEGIN
    RETURN QUERY SELECT flightid
                   FROM flight
                  WHERE flightdate >= $1
                    AND flightdate < ($1 + 1);

    -- Since execution is not finished, we can check whether rows were returned
    -- and raise exception if not.
    IF NOT FOUND THEN
        RAISE EXCEPTION 'No flight at %.', $1;
    END IF;

    RETURN;
 END
$BODY$
LANGUAGE plpgsql;

-- Returns available flights or raises exception if there are no
-- available flights.
SELECT * FROM get_available_flightid(CURRENT_DATE);
```

Bir prosedürde, fonksiyonun aksine, dönüş değeri olmaz. Dolayısıyla RETURN türü anahtar kelimeler içermezler. Bununla birlikte kod tamamlanmadan prosedürden çıkılması gereken durumlar varsa, bunun için argüman eklemeden RETURN ifadesi kullanılabilir.

```sql
CREATE PROCEDURE triple(INOUT x int)
LANGUAGE plpgsql
AS $$
BEGIN
    x := x * 3;
END;
$$;

DO $$
DECLARE myvar int := 5;
BEGIN
  CALL triple(myvar);
  RAISE NOTICE 'myvar = %', myvar;  -- prints 15
END
$$;
```

### Şart İfadeleri

PL/pgSQL’de ``IF`` veya ``CASE`` gibi yapılar, bazı şartları göz önüne alarak belli komutları işletebilmeye olanak tanır. IF için üç yazım formu bulunmaktadır.

- **IF** ... **THEN** ... **END IF**
- **IF** ... **THEN** ... **ELSE** ... **END IF**
- **IF** ... **THEN** ... **ELSIF** ... **THEN** ... **ELSE** ... **END IF**

``CASE`` için ise iki form bulunmaktadır.

- **CASE** ... **WHEN** ... **THEN** ... **ELSE** ... **END CASE**
- **CASE WHEN** ... **THEN** ... **ELSE** ... **END CASE**

IF ile kurulan şart ifadelerinde IF ve THEN arasında her zaman şart ifadesi olur. Bu ifadenin boolean bir sonuç üretebiliyor olması gerekir. Durumların sayısına göre THEN’den sonra şart ifadesi END IF ile sonlanabileceği gibi diğer şartların da girilebilmesi için ``ELSE`` ya da ``ELSIF`` de girilebilir. ELSIF yerine ``ELSEIF`` de yazılabilir. Aşağıda bir örnek sunulmuştur:

```sql
IF number = 0 THEN
    result := 'zero';
ELSIF number > 0 THEN
    result := 'positive';
ELSIF number < 0 THEN
    result := 'negative';
ELSE
    result := 'NULL';
END IF;
```

``CASE`` ise eşitlik durumlarının sorgulanması için kullanılır. Arama ifadesi her şart ile birer birer kıyaslanır ve TRUE döndürdüğü şart için yazılmış ifadeler uygulanır. Aşağıda bir örnek sunulmuştur. Örnekte *x*’in 1 ya da 2 olması durumu ile bunların dışındaki değerlerden birine eşit olması durumları için iki şart ifadesi vardır. Şart durumlarının sayısı arttıkça her yeni ifade yeni bir WHEN satırıyla araya eklenir ve bunların dışında kalan tüm eşitlik halleri için buradaki gibi ELSE girilir.

```sql
CASE x
    WHEN 1, 2 THEN
        msg := 'one or two';
    ELSE
        msg := 'other value than one or two';
END CASE;
```

### Döngüler

PL/pgSQL dilinde ``LOOP``, ``EXIT``, ``CONTINUE``, ``WHILE``, ``FOR`` ve ``FOREACH`` ifadeleri ile tekrar eden komut serileri işletilebilir.

``LOOP`` şarta bağlı olmayan ve EXIT ya da RETURN ile sonlandırılana kadar tekrarlanarak işletilen döngülerdir. Opsiyonel olarak label ile etiketlenerek, ``EXIT`` ya da ``CONTINUE`` anahtar sözcükleri tarafından (özellikle iç içe ifadelerden) çağrılabilir.  

```sql
[ <<label>> ]
LOOP
    statements
END LOOP [ label ];
```

``EXIT`` her tür döngüden çıkmak için kullanılabilir.

```sql
EXIT [ label ] [ WHEN boolean-expression ];
```

Eğer EXIT sözcüğü BEGIN ile başlayan bir bloğun içinde kalıyorsa, bu blok sonlanır ve fonksiyon ifadesi BEGIN bloğunun bittiği yerden işlemeye devam eder.

```sql
CONTINUE [ label ] [ WHEN boolean-expression ];
```

``CONTINUE`` kullanımı da EXIT’e benzer, her tür döngü ile kullanılabilen CONTINUE bloğu içinde WHEN tanımlandığında sonrasında gelen boolean ifadenin TRUE olması halinde ifade çalışırken, FALSE ise bloğun bittiği yere sıçrar ve oradan devam eder.

``WHILE`` ifadesi döngü başlamadan önceki boolean ifade TRUE döndüğü sürece çalışır. FALSE olur olmaz döngüden çıkar. Dolayısıyla döngünün her turunda boolean ifadenin doğruluğu da tekrardan kontrol edilir.

```sql
[ <<label>> ]
WHILE boolean-expression LOOP
    statements
END LOOP [ label ];
```

``FOR``, belli bir tam sayı aralığı boyunca hareket ederek blok içinde kalan işlevi yerine getirir. Aralık tamamlandığında döngüden çıkar. Birkaç farklı FOR kullanımı mevcuttur.  FOR döngüsünde tanımlanan name değişkeni otomatik olarak tamsayı (integer) cinsinden tanımlanacak ve sadece FOR döngüsünün içinde var olarak döngü bitişinde yok edilecektir. ``BY`` ile ifade edilen değer döngüdeki değer aralığıdır ve açıkça tanımlanmadığı sürece her iterasyondaki artış miktarı 1 olarak ön tanımlıdır. Eğer ``REVERSE`` ifadesi konulursa değer artmaz, azalır.

```sql
[ <<label>> ]
FOR name IN [ REVERSE ] expression .. expression [ BY expression ] LOOP
    statements
END LOOP [ label ];
```

FOR döngüsünün farklı yazım formları için çeşitli örnekler aşağıda verilmiştir.

```sql
FOR i IN 1..10 LOOP
    -- i will take on the values 1,2,3,4,5,6,7,8,9,10 within the loop
END LOOP;

FOR i IN REVERSE 10..1 LOOP
    -- i will take on the values 10,9,8,7,6,5,4,3,2,1 within the loop
END LOOP;

FOR i IN REVERSE 10..1 BY 2 LOOP
    -- i will take on the values 10,8,6,4,2 within the loop
END LOOP;
```

FOR döngüsünün bir diğer kullanımı ise tamsayılar kullanarak döngüde gezinmek yerine bir sorgu sonucunda gezinmek ve gerekiyorsa data manipülasyonu yapmak şeklindedir. Bu tür bir ihtiyaç için aşağıdaki söz dizimi kullanılır.

```sql
[ <<label>> ]
FOR target IN query LOOP
    statements
END LOOP [ label ];
```

Bu söz diziminde target, ya bir kayıt değişkeni, ya satır değişkeni ya da skaler değişkenlerden oluşan virgülle ayrılmış bir liste şeklindedir. Query’den dönen her satıra target atanır ve döngü (loop) gövdesi bu her satır için işletilir. Aşağıda bunun bir örneği verilmiştir.

```sql
CREATE FUNCTION cs_refresh_mviews() RETURNS integer AS $$
DECLARE
    mviews RECORD;
BEGIN
    RAISE NOTICE 'Refreshing materialized views...';

    FOR mviews IN SELECT * FROM cs_materialized_views ORDER BY sort_key LOOP

-- Burada "mviews", cs_materialized_views’dan
-- bir kayda sahiptir.

        RAISE NOTICE 'Refreshing materialized view %s ...', quote_ident(mviews.mv_name);
        EXECUTE format('TRUNCATE TABLE %I', mviews.mv_name);
        EXECUTE format('INSERT INTO %I %s', mviews.mv_name, mviews.mv_query);
    END LOOP;

    RAISE NOTICE 'Materialized view yenilendi.';
    RETURN 1;
END;
$$ LANGUAGE plpgsql;
```

### Hata Yakalama

Varsayılan olarak PL/pgSQL fonksiyonlarında ortaya çıkan bir hata sonucu fonksiyonun ve onu çevreleyen fonksiyon kod bloklarının tamamının işletilmesi durdurulur. Fonksiyon içinde çeşitli noktalarda oluşacak hatalar BEGIN bloğuna ``EXCEPTION`` eklenerek yakalanabilir.

```sql
[ <<label>> ]
[ DECLARE
    declarations ]
BEGIN
    statements
EXCEPTION
    WHEN condition [ OR condition ... ] THEN
        handler_statements
    [ WHEN condition [ OR condition ... ] THEN
          handler_statements
      ... ]
END;
```

Eğer bir hata ortaya çıkmazsa zaten bu söz dizimi kodu olduğu gibi işletecektir. Hata olması durumunda ise ifadelerin işletilmesi dururken bloğun işletilme kontrolü EXCEPTION kısmına geçer. Bu durumda WHEN altındaki şartlar listesi gözden geçirilerek alınan hata işe eşleştirilir ve sonrasında ise bulunan hata için yazılmış şartlar uygulanmaya başlar. Bu uygulamanın bitişiyle beraber fonksiyonun END kısmına geçilir ve fonksiyondan çıkılır.

WHEN bloğu içindeki condition ifadesinde *postgresql.org* dökümantasyonunda hata kodları tablosunda yer alan kod tanımlarına başvurulur. Burada condition ile ifade edilen hata şartı için hata kodu ya da hata tanımı değişken adı yerine kullanılabilir. Aşağıda bir örnek verilmiştir.

```sql
INSERT INTO mytab(firstname, lastname) VALUES('Tom', 'Jones');
BEGIN
    UPDATE mytab SET firstname = 'Joe' WHERE lastname = 'Jones';
    x := x + 1;
    y := x / 0;
EXCEPTION
    WHEN division_by_zero THEN
        RAISE NOTICE 'caught division_by_zero';
        RETURN x;
END;
```

### Cursor

Bütün bir sorguyu bir kerede işletmek yerine, bir **cursor** ayarlayıp sorguyu paketlemek (encapsulation) ve her adımda birkaç satır için işletmek de mümkündür. Bunu yapmanın en önemli sebebi ise, kod çok fazla sayıda satır üzerinde çalışacaksa, belleğin tüm veriyi bir kerede okuyarak şişmesinin önüne geçmektir. FOR döngüleri otomatik olarak cursor oluşturur ve kullanırlar. Cursor’un daha da ilginç bir başka kullanımı ise, oluşturulan fonksiyonda bir cursor’e referans döndürmek ve böylece kullanıcının sorgudan dönen satırları okumasını sağlamaktır. Özellikle çok fazla satırın okunması sırasında bu yöntem uygulanmaktadır.

PL/pgSQL’de cursor’lere erişen bütün yollar cursor değişkenlerinden geçer. Bu değişkenler özel bir veri tipinde olmalıdır, refcursor tipinde. Bir cursor değişkeni oluşturmanın yollarından birisi, refcursor tipinde bir değişken deklare etmek iken diğeri, cursor deklerasyon söz dizimini kullanmaktır. Bu söz dizimi şu şekildedir,

```sql
name [ [ NO ] SCROLL ] CURSOR [ ( arguments ) ] FOR query;
```

Oracle uyumluluğu için FOR yerine `IS` de kullanılabilir. Eğer ``SCROLL`` tanımlanırsa geriye doğru cursor hareketi de mümkün hale gelir. Bunun olmaması için ``NO SCROLL`` kullanılmalıdır. Eğer bu iki ibareden hiçbiri yer almazsa, sorguya bağlı olarak ileri / geri satır derleme işletilmesine izin verilebilir. Tanımlanması halinde arguments, virgülle ayrılmış bir  name - veritipi çifti tanımına izin verir. Bu sayede verilen sorgudaki parametre değerleri bu isimler (name) yerine geçirilir. Bu deklarasyona bazı örnekler aşağıda sunulmuştur:

```sql
DECLARE
    curs1 refcursor;
    curs2 CURSOR FOR SELECT * FROM tenk1;
    curs3 CURSOR (key integer) FOR SELECT * FROM tenk1 WHERE unique1 = key;
```

Bu üç değişken tanımlamasında da refcursor veri tipi kullanılmıştır. Fakat ilki her sorguda kullanılabilirken, ikincisi kendisine bağlı tamamen özelleşmiş bir sorguda kullanılabilir. Sonuncu tanımlama ise parametrik bir şekilde bağlanmıştır ve burada tanımlanan key ifadesi, cursor açıldığında tam sayı şeklindeki bir parametre değeriyle değiştirilerek kullanılacaktır. Bu durumda, ilk değişken curs1’in hiç bir sorguya bağımlılığı yok denilebilir.

Bir cursor, satırları almaya başlamadan önce açılmalıdır. Cursor’lar üç şekilde açılabilir ve satır okumaya başlatılabilirler. Bu metotlardan birisi bağlı cursor değişkenlerini açmakta kullanılırken, ikisi bağımsız cursor değişkenlerinde kullanılırlar.

Bu metotlar sırayla:

1. **OPEN FOR** query
2. **OPEN FOR EXECUTE**
3. **OPEN** bound cursor
metotları olarak sıralanabilir. Son metot bahsi geçen bağlı cursor’ı açmakta kullanılır.

İlk metotta cursor değişkeni açılır ve istenen sorgu işletilir. Burada cursor halihazırda açılmış olamaz ve öncesinde bağımsız bir cursor değişkeni (refcursor tipinde) olarak da deklare edilmiş olmalıdır. Sorgu olarak SELECT ya da bunun (örneğin EXPLAIN) gibi satır döndüren bir sorguya izin verilmektedir. Eğer cursor sorgusunun içinde bir başka PL/pgSQL değişkeni yazılmışsa, değişken değeri, cursor OPEN ile açıldığında sahip olduğu değerle değiştirilir. Bunun ardından bahsi geçen değişkende olan değişiklikler cursor’ün davranışını değiştirmeyecektir. Bu tanımlamanın söz dizimi ve bir örnek aşağıda verilmiştir.

```sql
OPEN unbound_cursorvar [ [ NO ] SCROLL ] FOR query;

OPEN curs1 FOR SELECT * FROM foo WHERE key = mykey;
```

İlk metotta cursor değişkeni açılır ve istenen sorgu işletilir. Burada cursor halihazırda açılmış olamaz ve öncesinde bağımsız bir cursor değişkeni (refcursor tipinde) olarak da deklare edilmiş olmalıdır. Belirtilecek sorgu aynı EXECUTE komutunda olduğu gibi bir string ifade olmalıdır. Bu durumda sorgu planı her çalıştırmada değişiklik gösterecek ve ayrıca komut ifadesi içinde değişken değişimi yapılamayacaktır. EXECUTE ile olduğu üzere, parametre değerleri dinamik komutun içine ``format( )`` ve ``USING`` kullanılarak yerleştirilebilir. Yazım biçimi ve örnek yine aşağıda verilmiştir.

```sql
OPEN unbound_cursorvar [ [ NO ] SCROLL ] FOR EXECUTE query_string

OPEN curs1 FOR EXECUTE format('SELECT * FROM %I WHERE col1 = $1',tabname) USING keyvalue;
```

Yukarıdaki örnekte tablo ismi ``format ( )`` anahtar sözcüğü kullanılarakk sorguya yerleştirilmiştir. *col1* için karşılaştırma değeri ise USING ile parametre değeri kullanılarak oluşturulmuştur. Bu yüzden de tırnak kullanılmamıştır.

Son ``OPEN`` formu, sorgusu, deklare edildiği anda bağlı olan bir cursor değişkenini açmak için kullanılır. Cursor zaten açık olamaz. O anki argüman değer ifadelerinin bir listesi sadece ve sadece cursor, argümanları alacak şekilde deklare edildiyse görünmelidir.

Bağlı bir cursor’ün sorgu planı daira önbelleklenebilir olarak varsayılır. Bu durumda EXECUTE ifadesine denk bir ifade yoktur. SCROLL veya NO SCROLL ifadesi de OPEN içinde tanımlanamaz, zira bu noktada cursor’ün scroll davranışı zaten belirlenmiştir.

Argüman değerleri ya pozisyonel ya da tanımlı notasyonlar iletilebilir. Pozisyonel notasyon, tüm argümanların sıralı bir şekilde tanımlandığı yazım şeklidir. Tanımlı notasyonda ise her argümanın ismi := kullanılarak argüman ifadesinden ayrıştırılır. Çağrı fonksiyonlarında da olduğu gibi, pozisyonel ve tanımlı notasyonlar birbiriyle karışık bir şekilde de kullanılabilir.  Aşağıda bu notasyonlar örneklenmiştir:

```sql
OPEN curs2;
OPEN curs3(42);
OPEN curs3(key := 42);
```

Değişkenlerin değişimi işi, bağlı bir cursor’ün sorgusu üzerinde yapıldığı için, gerçekten de değerleri cursor’e iletmek için iki yol bulunur: Bunlardan biri açıkça OPEN argümanı kullanmak veya dolaylı olarak sorgudaki bir PL/pgSQL değişkenine referanslamaktır. Fakat sadece bağlı cursor’den önce deklare edilmiş değişkenler değiştirilecektir. Her iki durumda da iletilecek değer, OPEN anında tanımlanacaktır. Örneğin yukarıdaki örnekteki *curs3* ile aynı etkiyi almak için şöyle bir uygulama yapılabilir.

```sql
DECLARE
    key integer;
    curs4 CURSOR FOR SELECT * FROM tenk1 WHERE unique1 = key;
BEGIN
    key := 42;
    OPEN curs4;
```

### Cursor Kullanımı

Cursor yukarıda anlatılan yollardan birisiyle açıldıktan sonra bu bölümde tanımlanacak ifadelerle yönlendirilebilir. Bu yönlendirme ifadeleri, cursor’ı açan fonksiyonla aynı yapının içinde bulunmak zorunda değildir. Bir fonksiyondan refcursor türünde bir değer döndürülerek, çağrıcının cursor’ü işletmesine izin verilir.

``FETCH``, cursor’den hedefe giden bir sonraki satırı getirir. Sonraki satır satır değişkeni, kayıt değişkeni ya da SELECT INTO gibi basit değişkenlerin virgülle ayrılmış bir listesi olabilir. Eğer bir andaki işlemden sonraya kayıt kalmadıysa hedef NULL’a ayarlanır. SELECT INTO’daki gibi, ``FOUND`` isimli özel değişken, bir satır alınıp alınmadığını görmek için kontrol edilebilir. FETCH ile ilgili örnekler ve sözdizimi aşağıda sunulmuştur.

```sql
FETCH [ direction { FROM | IN } ] cursor INTO target;
FETCH curs1 INTO rowvar;
FETCH curs2 INTO foo, bar, baz;
FETCH LAST FROM curs3 INTO x, y;
FETCH RELATIVE -2 FROM curs4 INTO x;
```

``MOVE``, bir cursor’ü, herhangi bir veri almaksızın hareket ettirir. MOVE tam olarak FETCH gibi çalışır, fakat MOVE, cursor’ü sadece yeniden konumlandırır, ayrıldığı satıra geri döndürmez. Burada da FOUND kullanılarak, devam edilecek *bir sonraki satır* olup olmadığı kontrol edilebilir. Söz dizimi ve örnekler aşağıda verilmiştir.

```sql
MOVE [ direction { FROM | IN } ] cursor;

MOVE curs1;
MOVE LAST FROM curs3;
MOVE RELATIVE -2 FROM curs4;
MOVE FORWARD 2 FROM curs4;
```

Bir cursor bir tablo satırında konum aldığı zaman, bu satır cursor tarafından güncellenebilir ya da silinebilir. Yazım için söz dizini aşağıdadır.

```sql
UPDATE table SET ... WHERE CURRENT OF cursor;
DELETE FROM table WHERE CURRENT OF cursor;
```

Cursor’deki sorgunun ne olabileceği konusunda kısıtlamalar olabilir ve cursor içinde ``FOR UPDATE`` olarak kullanmak en doğrusudur. Örneğin,

```sql
UPDATE foo SET dataval = myval WHERE CURRENT OF curs1;
```

Bir cursor’ün dayandığı *portal*, ``CLOSE`` ile kapatılır. Bu komut, sistem kaynaklarını, transaction’ın sonu gelmeden önce serbest bırakır ya da cursor değişkenini boşaltarak yeniden kullanılabilir / açılabilir hale getirir.

```sql
CLOSE cursor;
```

{% include links.html %}
