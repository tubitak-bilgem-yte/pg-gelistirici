---
title: "PostgreSQL Veritabanı Geliştirici Dökümantasyonu"
keywords: Giriş
tags: [postgresql]
sidebar: mydoc_sidebar
permalink: index.html
summary: PostgreSQL Veritabanı Geliştirici Dökümantasyonu
---

![postgres giriş](/images/postgres_giris.png)

## PostgreSQL Nedir?

PostgreSQL, tüm dünyada popüler olan açık kaynak kodlu, platform bağımsız gelişmiş bir nesne ilişkisel (ORDBMS) veritabanı yönetim sistemidir.

Yüksek performanslı, kararlı ve güvenilirdir. Modern kurumsal veritabanı kabiliyet ve özelliklerine sahiptir.

PostgreSQL’in, 1977 yılında başlayan 20 yılı akademik, son 20 yılı endüstride geçen 40 yıllık bir geçmişi olan en eski açık kaynak kodlu yazılımlardan biridir.

PostgreSQL, tüm dünyada kamuda önemli devlet hizmetleri sunan uygulama sistemlerinde (CERN, NASA, Fransa, İngiltere,G.Kore, vb.) finans ve Telekom sektörlerinde iş kritik uygulamalarda, dünyada önde gelen üreticilerin ürünlerinde (Apple, Microsoft, IBM,Amazon, vb.), araştırma merkezleri ve üniversitelerde, küçük ölçekli projelerden çok büyük ölçekli kurumsal altyapılarda güvenilerek kullanılmaktadır.

PostgreSQL, önde gelen ticari veritabanı ürünleri ile rekabet edecek kurumsal veritabanı özelliklerinin yanı sıra günümüzdijital dönüşüm projeleri ve teknolojileri ile uyumlu birçok yeni ve yenilikçi özelliğe sahip-tir (Örneğin; dizi şeklindeki veri tipleri,paralel sorgular, JSON veri tipini desteklemesi ve üzerinde sorgu çalıştırabilmesi).

PostgreSQL, veritabanı ve sistem yöneticileri, yazılım mimarları ve geliştiricileri için çekici gelen yenilikçi birçok özellik sunar.

PostgreSQL’in öğrenmesi, kurulumu, konfigürasyonu, yönetimi, izlemesi ve bakımı kolaydır. Post-greSQL ekosisteminde birçok yönetim, izleme açık kaynaklı ve ticari araç vardır.

PostgreSQL’in çok aktif ve güçlü geliştirici komünitesi vardır. Her yıl bir majör sürümü yayınlanır.

## PostgreSQL Tarihçesi

![postgresql_tarihce](/images/postgresql_tarihce.png)

* **1977-1985 Ingres**: PostgreSQL’in, 1977 yılında başlayan 20 yılı akademik, son 20 yılı endüstride geçen 40 yıllık bir geçmişi vardır. Berkeley Kaliforniya Üniversitesinde Micheal Stonebraker, Ingres adıyla ilişkisel modeli baz alan bir araştırma projesi olarak ilişkisel veritabanı yönetim sistemini geliştirdi.

* **1986-1994 Postgres**: 1985’den itibaren Micheal Stonebraker, kompleks veri yapılarını ve nesne ilişkisel modeli destekleyen postgres veritabanı yönetim sistemini geliştirdi.

* **1994-1995 Postgre95**: 1995’te Andrew Yu ve Jolly Chen tarafından SQL sorgu dili geliştirilerek, önceki POSTQUEL ile değiştirildi ve geliştirilmiş bu versiyona Postgres95 adı verildi.

* **1996 – Günümüz PostgreSQL**: Birçok geliştiricinin Postgres95 üzerinde yoğun çalışması ile birçok yeni özellik eklenmesi ve iyileştirmelerin ardından ilk açık kaynak versiyonu 1997 yılında yayınladı ve endüstride kullanılmaya başlandı. Açık kaynak versiyonunun adı Postgre95 değiştirilerek PostgreSQL oldu.

## PostgreSQL Güçlüdür, Avantajlıdır, Güzeldir

* PostgreSQL’in özellikleri ve kabiliyetleri, mevcut ticari veritabanları ile rekabet edebilecek yenilikçi özelliklere sahiptir.
* Ekonomiktir, lisans ücreti yoktur, size üretici bağımsızlığı sağlar. Kurumsal veritabanı özelliklerine lisans ücreti ödemeden sahip olursunuz. PostgreSQL’i istediğiniz kadar sunucuya kurabilir ve dağıtımını yapabilirsiniz.
* Türkçe’ye yerelleştirilmiştir ve Türkçe desteği vardır.
* Platform bağımsızdır. PostgreSQL’i kullanmak için geliştirme ortamınızı veya sistemlerinizi değiştirmenize gerek yoktur. Tüm modern  işletim sistemleri (Linux, Unix, Windows, Mac OS, vb.) ve işlemciler (x86, x86_64, IA64, vb.) üzerinde [çalışır](http://buildfarm.postgresql.org).
* Yüksek güvenliklidir. Yüksek erişilebilirdir. Genişleyebilir mimariye sahiptir. Her işlem ve veri büyüklüğüne göre ölçeklenebilir, esnektir, genişleyebilir veya daraltılabilir.
* Yüksek performanslıdır. Çok büyük veri operasyonları ve yüksek anlık işlem yükleri karşısında sağlamdır.
* Güvenilirdir, ACID tam uyumludur, sistem sorunlarının etkilerine karşı toleranslıdır.
* SQL Standartlarına (ANSI-SQL 2008/2011) en çok uyum gösteren veri tabanıdır. Ticari veritabanlarından PostgreSQL’e geçiş kolay ve maliyet etkindir. Farklı uygulama sistemleri ve veritabanları ile entegre çalışabilir.
* Öğrenmesi ve kurması kolaydır. Güncel, detaylı, herkese açık ve erişimi kolay yaygın dokümantasyonu vardır. Yönetimi, yedeklemesi, bakımı ve izlemesi kolaydır. Hata mesajları ve log sistemi açıktır, anlaşılırdır. PostgreSQL veritabanı yönetim ve izleme görevleri otomatize edilebilir ve ticari veritabanlarına göre yönetim işleri daha az zaman alır. Planlı bakımlarda düşük kesinti süreleri sağlar. Hata yapmaya engel olan güvenli bir yapısı vardır.
* PostgreSQL’in çok gelişmiş bir sorgu planlayıcısı vardır. Geliştirici dostudur. Farklı yazılım geliştirme platform ve dillerini destekler ve uyumlu çalışır. Geliştiricilerin işini kolaylaştıran çok geniş bir eklenti havuzu vardır. Tüm modern programlama dilleri için sürücülere sahiptir. Coğrafi veri yapılarını ve yeni NoSQL yapısal olmayan veri türlerini (JSON, JSONB, XML, vb.) destekler. Kaynak kodu kullanılarak özelleşmiş açık veya kapalı kodlu çözümler geliştirilebilir.
* Aktif bir topluluk tarafından desteklenmektedir. Tüm dünyadan geliştiricisi bulunan ve çekirdek geliştiricilerin yer aldığı topluluk soru ve sorunlara hızlı geri dönüşlerle çözüm sağlar. Hemen her yıl ticari ürünleri kıskandıran yenilikçi ve güncel özellikler içeren yeni sürümü yayınlanır.

## PostgreSQL Lisansı

*“Permission to use, copy, modify, and distribute this software and its documentation for any purpose, without fee, and without a written agreement is hereby granted, provided that the above copyright notice and this paragraph and the following two paragraphs appear in all copies.”*

***"Bu yazılımı ve belgelerini herhangi bir amaç için, ücretsiz ve yazılı bir anlaşma olmaksızın kullanma, kopyalama, değiştirme ve dağıtma izni verilir.”***

## PostgreSQL’in Kabiliyetleri Nelerdir?

PostgreSQL kurumsal veritabanı yönetimi sistemlerinde olmazsa olmaz başlıca özellikler olan standartlara uyumluluk, yüksek erişilebilirlik, yüksek hacimli veri ve işlemler karşısında sağlamlık ve güvenilirlik, yüksek performans, güvenlik, genişleyebilirlik, kolay yönetim ve bakım, hızlı sorun giderme, planlı bakımlarda düşük kesinti süreleri gibi özelliklere sahiptir. Her yıl çıkan majör sürümleri ile birçok gelişmiş yeni ve yenilikçi özelliği de bünyesine katmaya devam etmektedir.

![postgres_ozellikleri](/images/postgres_ozellikleri.png)

### Yüksek Erişilebilirlik ve Kümeleme

* Yüksek erişilebilir (High Available) mimariye sahiptir. Sunduğu servis kalitesi ile plansız kesinti ve arıza sürelerini minimuma indirir. Yük dengeleme ve yüksek erişilebilirlik için warm standby/hot standby/streaming ve logical (versiyon 10 ile birlikte) replikasyonu destekler
* PostgreSQL, yük dengeleme ve kümeleme yapıları ile veritabanı sunucuları arasındaki iş yüklerinin dengelenmesini sağlar. Sunucuların herhangi birinde aşırı yüklenme önlenirken, kaynak kullanımı optimize edilir, verimlilik en üst düzeye çıkar ve yanıt süreleri en aza iner.
* PostgreSQL’de Multi-Master Replikasyonu ile birden çok sunucu master statüsüne sahip olabilirken, farklı lokasyonlarda dağıtık işyükünü yönetimini, yük dengeleme, kümelemeyi destekler. Multi-Master Replikasyonu dinamik veri toplanması için veri yönetimi tasarımı kolaylığı sağlar. Logical Replikasyon, veritabanı veya tablo başına yapılandırılabilir. Tablo replikasyonu ile farklı yerlerden herhangi bir işlemi sorunsuz bir şekilde bir lokasyonda birleştirmek mümkündür.
* Natif Asenkron Çoğaltma (Native Asynchronous Replication), Tam/Artırımlı Yedekleme (Full/Incremental Backup) ve kurtarma modları, veri yeniden senkronizasyon mekanizmalarının kolaylığı gibi özellikler ile PostgreSQL, Disaster Recovery Center (DRC) hazırlığının tam özellik setini destekleyerek, daha fazla maliyet / araç eklemeden veritabanınızın herhangi bir felaketten kurtarılmasını sağlar.

### Yüksek Performans

* ANSI C programlama dilinde yazılmıştır ve kendisini kanıtlamış yüksek performansı vardır. PostgreSQL, yüksek veri  okuma-yazma  iş  yüklerini istikrarlı ve sağlam şekilde yaparken, veri bütünlüğünü ve güvenliğini korur. Günümüzün ağır ve yüksek hacimli işlem ve sorgu yüklerinde ticari veritabanlarına göre daha düşük kaynakla yüksek performas sunar.
* PostgreSQL’de tablo ve satır seviyelerinde kilitleme (table/row level locking) yapabilirsiniz. Daha fazla kilitlemeyi/engellemeyi önleyen daha granüler kilitler kullanabilir, bu şekilde eşzamanlılık artar ve engelleme süresi azalır. PostgreSQL birçok farklı tip indeksleme yöntemi sunar; B-tree, Hash, GiST (Generalized Search Tree), SP-GiST, GIN (generalized inverted index) ve BRIN. PostgreSQL, kısmi, biricik ve çok sütunlu indeksleri destekler. Ayrıca ifadeler ve operatör sınıfları üzerinde de indekslemeyi destekler.
* PostgreSQL,  performansı  artırmak ve analiz etmek için çeşitli komutlar sağlar. EXPLAIN komutu, bir SQL ifadesinin yürütme planını gösterir. ANALYZE komutu, tablo ve sütunlardaki istatistikleri toplamak için kullanılır. VACUUM komutu, kullanılmayan sabit disk alanını geri kazanmak amacıyla kullanılır. CLUSTER komutu, verileri sabit diskte fiziksel olarak düzenlemek için kullanılır. Tüm bu komutlar veritabanı iş yüküne göre yapılandırılabilir.
* Tablo kalıtımı (table inheritance) ve kısıtlama çıkarma (constraint exclusion) özelliklerine sahiptir. Tablo kalıtımı ile  aynı  yapıya  sahip tablolar kolayca oluşturulur. Bu tablolar, belirli bir ölçüt temelinde veri alt kümelerini depolamak için kullanılabilir ve belirli senaryolarda bilginin çok hızlı bir şekilde alınması sağlanır.

### Ölçeklenebilirlik

* PostgreSQL ölçeklenebilir yapıya sahiptir. Basit tek sunuculu küçük uygulamalardan, yüksek işlem kapasitesi gereken çok sunuculu, çok büyük hacimli veri yoğun iş kritik uygulamaların hepsine çözüm sunma kabiliyeti vardır. Küçük başlayıp, istenilen ölçeğe kolayca  çıkarılabilir. Senkron ve asenkron streaming replikasyon özelliğine sahiptir.
* Dikey ölçeklemeye (scale up) göre çok daha uygun maliyetli olan, yatay ölçekleme (scale out) ile maliyeti azaltılır.
* PostgreSQL, büyük tabloların bölümlenmesini (table partitioning) destekler. Büyük tabloların bölümlenmesi veritabanının performansını artırır. PostgreSQL tablespaces yapısını destekler.
* PostgreSQL’in günümüzün devasa hızda veri büyümesi ve büyüklüğü yönetim ihtiyaçlarına karşı PostgreSQL Natif Bölümleme (Native Partitioning) özelliği vardır.
* Veriler, farklı kısıtlamalara göre bölümlere ayrılabilir ve toplu sonuçlar için sorgulanabilir. İsteğe bağlı bölüm (On-demand  partition), daha önce yapılandırılmış olan tüm kuralları değiştirmeden, tek bir sorgu çalıştırarak kolayca eklenebilir.

![postgres_giris_olceklenebilirlik](/images/postgres_giris_olceklenebilirlik.png)

### Güvenlik

* PostgreSQL’de güvenlik, yalnızca bir özellik değildir, temel yapılarından biridir. Her minör ve majör sürümü yeni güvenlik özellikleri ve güncellemeleri ile gelir.
* Kimlik doğrulama, yetkilendirme, denetim, veri güvenliği, veri şifreleme, satır (row) seviyesinde güvenlik gibi birçok güvenlik yapısı vardır.
* Trust, Password, LDAP, GSSAPI, SSPI, Kerberos, kimlik tabanlı (ident-based), RADIUS, sertifika, PAM, SCRAM (versiyon 11’le birlikte) kimlik doğrulaması gibi çeşitli kimlik doğrulama yöntemlerini destekler.
* PostgreSQL, veritabanı nesne erişimini veritabanı, tablo, görünüm(view), fonksiyon, sıra ve sütun dâhil olmak üzere çeşitli düzeylerde kontrol edebilir ve kullanıcılara yetkilendirebilir. Bu PostgreSQL’in veritabanı nesneleri üzerinde çok farklı seviyelerde yetkilendirme kontrolüne sahip olmasını sağlar.
* Verileri şifrelemek için donanım şifreleme yöntemlerini, pgcrypto uzantısını veyauygulama arayüzü (API) kullanabilir. PostgreSQL ile veri seviyesinde şifreleme için AES, 3DES gibi farklı şifreleme algoritmaları kullanabilirsiniz.
* PostgreSQL istemci sunucu iletişiminde SSL kullanır.
* PostgreSQL’in en katı güvenlik standartlarına (PCI Data Security Standard) tam uyumluluk için altyapısı vardır.

![postgres_giris_guvenlik](/images/postgres_giris_guvenlik.png)

### Gelişmiş Özellikler

* Türkçe dâhil 20’den fazla dile yerelleştirilmiştir.Tam metin aramasını destekler. Diğer veritabanları gibi tetikleyicileri ve fonksiyonları destekler.
* Veritabanı motoru paralel işleme (paralel sorgu, paralel veri tarama, vb.) yapabilir. PostgreSQL belirli türde sorguları birden çok çekirdek ve işlemciye dağıtabilir. Paralel işlemin gelişmiş gücü ile mevcut kaynakların maksimize edilmesi sırasında sorgu yürütme süresi düşer.
* Hot-backup, Point-in-Time Recovery (PITR) özelliklerine sahiptir.
* Çoklu Sürüm Eşzamanlılık Kontrolü (MVCC) kullanarak veri tutarlılığını korur. Bir veritabanı sorgulanırken, her işlem, bir süre önce olduğu gibi, verilerin bir anlık görüntüsünü (bir veritabanı sürümü) görür. İşlemlerin tutarsız verileri görüntülemesini engeller ve eşzamanlı işlemlerde işlem yalıtımı sağlar. Okuyucular yazarları, yazarlar okuyucuları engellemez.
* PostgreSQL, sistem çökmesi veya arıza durumunda kurtarma sağlayan, gerçekleşmeden önce her ekleme/güncelleme/silme işleminin  kaydını yapan, (Oracle REDO kayıtlarına benzer) bir Write Ahead Logging (WAL) mekanizmasına sahiptir.
* Çok zengin SQL yapılarını destekler. İlişkilendirilmiş ve ilgisiz alt sorguları destekler. Ortak tablo ifadesini (CTE), window fonksiyonlarını ve özyinelemeli sorguları destekler. PostgreSQL’e her sürümde yeni SQL özellikleri eklenmektedir.
* PostgreSQL kolayca genişletilebilecek şekilde tasarlanmıştır. Veritabanına yüklenen uzantılar, yerleşik özellikler gibi çalışır. İlk kurulumla gelen eklentileri contrib dizini altında görebilirsiniz.
* İhtiyaç duyabileceğiniz eklentileri, PostgreSQL komünite sayfasındaki [eklenti kataloğunda](https://www.postgresql.org/download/products/6-postgresql-extensions) arayabilirsiniz.
* Veri ambarı olarak konfigüre edebilir ve yüksek hacimli veri yönetimi için kullanabilirsiniz. PostgreSQL’den türetilmiş Greenplum ve Citus Data veritabanları ile gerçek zamanlı analitik uygulamaları, paralel sorgu işlemelerine yönelik veritabanı sistemleri kurabilirsiniz.

### Diğer Veritabanlarından Göç

* PostgreSQL’in mimarisi, hem ilişkisel hem de ilişkisel olmayan modellerden herhangi bir mevcut veritabanı sisteminden geçişi destekleyecek kadar esnektir.
* PostgreSQL’in mimarisi, hem ilişkisel hem de ilişkisel olmayan modellerden herhangi bir mevcut veritabanı sisteminden geçişi destekleyecek kadar esnektir.
* Yabancı Veri Paketleyici eklentileri ile harici veri kaynaklarına bağlanarak üzerinde sorgu çalıştırabilir, veri aktarabilir/alabilir ve yerel veri tabloları üzerinde yapılan sorguların sonuçları ile birleştirebilir, veri entegrasyonu yapabilirsiniz. Genel, spesifik (Oracle, MySQL, PostgreSQL, MS SQL Server, DB2, Teradata), NoSQL (Cassandra, MongoDB, Redis, Neo4j), dosya (XML, CSV, düz metin), coğrafi bilgi sistemleri gibi farklı yapılarda harici veri kaynaklarına ve farklı veritabanlarına [erişebilirsiniz](http://wiki.postgresql.org/wiki/Foreign_data_wrappers).
* SQL standartlarını ile uyumlu olması, özellikle ticari ilişkisel veritabanlarından PostgreSQL’e göçü kolaylaştırmaktadır. Diğer veritabanlarından PostgreSQL’e göç için geliştirilmiş açık kaynak kodlu ve ticari araçlar mevcuttur.

## PostgreSQL Veritabanı Limitleri

.......

{% include links.html %}
