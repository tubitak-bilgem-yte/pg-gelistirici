---
title: "pgLoader"
tags: [PostgreSQL]
keywords: PostgreSQL
last_updated: November 9, 2020
summary: "pgLoader"
sidebar: mydoc_sidebar
permalink: mydoc_pgloader.html
folder: mydoc
---

## pgLoader

pgLoader PostgreSQL için geliştirilmiş bir veri kopyalama aracı olduğu kadar, MS SQL Server, MySQL veya SQLite gibi başka veritabanlarından PostgreSQL’e göç işlemleri için de kullanabileceğimiz bir araçtır. pgLoader, birinci çalışma modu olarak dosyadan okur ve hedef veritabanına okuduğu verileri yükleyebilir. pgLoader’ın ikinci çalışma modu sayesinde ise kaynak veritabanına ve hedef veritabanına aynı anda erişebilir ve böylece **kesintisiz göç** (continuous migration) uygulamasında kullanılabilir. pgLoader göç işlemi için kullanıldığında tanımlanan bağlantı üzerinden veritabanının katalog tablolarına erişir ve mevcut tablolara ait şemayı bularak, aynı yapıyı PostgreSQL’de de oluşturur. Sonrasında ise veritabanlarındaki verileri ve diğer objeleri senkronize eder. Bunu yaparken verileri bağlantı anında okuyup transform edebilir, ham veri satırlarını hedef tabloya yükleme öncesinde ya da sonrasında gönderebilir. pgLoader, her iki çalışma modunda da verileri hedef sunucuya atarken PostgreSQL’in `COPY` komutunu kullanır ve hataları da **reject.dat** ve **reject.log** isminde iki tabloya yazar.

Standart bir PostgreSQL kurulumunda olmayan pgLoader için github sayfası veya PostgreSQL reposu kullanılabilir. Bunun için [buradan](https://github.com/dimitri/pgloader) veya işletim sürümüne göre aşağıdaki komut kullanılarak kurulum yapılabilir. Kurulum sonrasında ise komut satırı kullanılarak tek komutla tüm veri yükleme ya da göç işlemi gerçekleştirilebilir.

```sh
sudo apt-get install pgloader  
veya
sudo yum install pgloader
```

pgLoader ile veri yükleme yapabilmek için CSV, DBF gibi yedek dosyalarından ya da arşivden veri yüklemesi yapılabilir. Bunun için önce bir dosyada yapılacak veri yüklemesinin detaylarına ilişkin bilgiler yazılır ve kaydedilir. Son adımda ise komut satırında pgloader komutu ile birlikte oluşturulan bu komut dosyası çağırılır. Aşağıda CSV dosyaları kullanılarak yapılan veri yükleme işlemini tanımlayan bu tür bir dosya örnek olarak oluşturulmuştur.  Sonrasında (örneğin isim olarak 'pgload_test.load' ismi verilmiş olsun) ise komut satırında pgloader `pgload_test.load` komutu ile bu komut dosyası çağrılarak aşağıda tanımlanan detayların hedef veritabanında uygulanması sağlanır.

```sql
LOAD CSV
   FROM 'GeoLiteCity-Blocks.csv' WITH ENCODING iso-646-us
        HAVING FIELDS
        (
           startIpNum, endIpNum, locId
        )
   INTO postgresql://user@localhost:54393/dbname
        TARGET TABLE geolite.blocks
        TARGET COLUMNS
        (
           iprange ip4r using (ip-range startIpNum endIpNum),
           locId
        )
   WITH truncate,
        skip header = 2,
        fields optionally enclosed by '"',
        fields escaped by backslash-quote,
        fields terminated by '\t'

   SET work_mem to '32 MB', maintenance_work_mem to '64 MB';
```

Burada komut dosyası incelendiğinde komutların 3 gruba ayrıldığı görülür. Önce ne tür bir dosyayı okuyarak hedef veritabanına yazılacağı bilgisi (LOAD CSV) girildikten sonra, **FROM** kısmında kaynak veriye erişim linki (bu veritabanı bağlantısı ya da CSV gibi metinsel bir dosya olabilir); **INTO** kısmında ise hedef veritabanına erişim linki URI olarak verilir. Son kısımda ise **WITH** ile veri aktarımı sırasında uygulanması istenen pgLoader davranışları belirtilir. Örneğin yazılacak tabloda önce truncate uygulamak, trigger ya da indeksleri devre dışı bırakmak, csv’yi okurken hesaba katılacak delimiter karakterini ya da kaçış karakterini belirtmek gibi.

Örneği genişletmek gerekirse, öncelikle bir csv dosyası üretelim ve bunu *load.csv* olarak diskimize kaydedelim.

```sql
"2.6.190.56","2.6.190.63","33996344","33996351","GB","United Kingdom"
"3.0.0.0","4.17.135.31","50331648","68257567","US","United States"
"4.17.135.32","4.17.135.63","68257568","68257599","CA","Canada"
"4.17.135.64","4.17.142.255","68257600","68259583","US","United States"
"4.17.143.0","4.17.143.15","68259584","68259599","CA","Canada"
"4.17.143.16","4.18.32.71","68259600","68296775","US","United States"
```

Sonra da terminalde aşağıdaki pgloader’ı aşağıdaki parametrelerle çağıralım. Tahmin edileceği üzere, daha önce hedef veritabanımızda oluşturduğumuz uygun tabloya yukarıdaki csv dosyasındaki tüm veriler kopyalanacaktır.

```sh
pgloader load.csv pgsql://bilgempg:password@localhost/target_db
```

{% include tip.html content="pgLoader veri göçü ve veri kopyalama anlamında detaylı yetenekler sergiler. Bunun için sayfasındaki [dökümanlar](https://pgloader.io/) incelenerek veri kopyalama süreci özelleştirilebilir [](https://pgloader.readthedocs.io/en/latest/)." %}

{% include links.html %}
