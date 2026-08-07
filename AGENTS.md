# Agent Notları (aXet-Project)

Bu dosya, bu depoda çalışırken izlenmesi gereken proje-özel tercihleri içerir.
README.md'deki "Geliştirme Prensipleri" bölümüyle birebir uyumludur; buradaki
liste aXet.code'un kendi hafızası içindir.

## Katı kurallar

1. Databricks notebook'ları her zaman `.ipynb` olarak yazılır. Düz `.py`
   script önerilmez/yazılmaz (test/prototip aşamasında bile, kullanıcı
   "ipynb'e çevir" isteyebilir — bu durumda hemen çevrilir).
2. Bir fonksiyon 2. bir notebook'ta da kullanılacaksa (veya kullanılma
   ihtimali varsa) direkt `Utils.ipynb`'a yazılır, kopyala-yapıştır yapılmaz.
   Diğer notebook `%run "./Utils"` ile çağırır.
3. `Utils.ipynb` mutlaka markdown başlıklarıyla (`# ...`) bölümlere ayrılır;
   yeni fonksiyon eklerken ilgili başlığın altına konur, yoksa yeni başlık
   açılır. Fonksiyonlar rastgele sıralanmaz.
4. Yeni bir dış API/veri kaynağı kullanılmadan önce mutlaka `robots.txt` ve
   lisans/kullanım şartları kontrol edilir, sonuç kullanıcıya açıkça
   raporlanır. `Disallow` kapsamına giren ya da lisansı belirsiz kaynaklar
   kullanılmaz; ama kod silinmez, "KULLANILMAMALI" notuyla üstte bırakılır ve
   `.gitignore`'a eklenir (repoya girmesin).
5. Veri çekimi "nazik" olmalı: paralel istek sayısı ölçülü tutulur,
   `Crawl-Delay` varsa `time.sleep` ile uyulur, gereksiz tekrar/toplu istek
   atılmaz.
6. Maliyet her zaman önemli bir kriterdir (kullanıcı Databricks compute
   maliyetine hassas): `df.count()` gibi tüm veriyi tarayan pahalı işlemlerden
   kaçınılır, mümkünse `dbutils.fs.ls()` boyutu, `getNumPartitions()` gibi
   ucuz alternatifler kullanılır.
7. Harici kaynaktan indirilen ham dosyalar (xlsx/csv/...) işlenip DataFrame'e
   alındıktan hemen sonra `os.remove()` ile silinir. Localde/repoda kalıcı
   veri dosyası tutulmaz. `.gitignore`'da `*.xlsx`/`*.csv` zaten hariç.
8. Wiki'ye (Azure DevOps) yazılan içerik bir AI agent'ın kolay parse
   edebilecegi sekilde tasarlanir: metadata + schema fenced ```json``` blogu
   icinde, ornek veri de JSON (serbest markdown tablo degil). Tum veri asla
   wiki'ye yazilmaz, sadece `limit(N)` ornegi + ucuz metadata.
9. Her tamamlanan degisiklik hem `origin` (GitHub) hem `azure` (Azure DevOps)
   remote'larina push edilir, ikisi senkron tutulur. Pushtan once ikisinden
   de `pull --no-edit` yapilir.
10. Kullanicidan onay gelmeden (`push edelim mi?` diye sorulduktan sonra
    "evet/pushla" onayi alinmadan) commit/push yapilmaz.
11. Bir veri kaynagi Databricks cluster'indan erisilemiyorsa (coğrafi/IP
    kisitlamasi, connection timeout gibi) ingestion ikiye bolunur: (a) duz
    Python ile calisan, `dbutils`/`spark` icermeyen bir *lokal indirme*
    notebook'u (kullanicinin kendi makinesinde calistirmasi icin), (b) bir
    widget (`dbutils.widgets.text`) ile kaynak path'i alip datalake'e yazan
    bir *Databricks upload* notebook'u. Isim seklinde net ayrilir (orn.
    "... Indir (Local).ipynb" / "... Datalake Upload.ipynb").

## Proje hedefi (guncel)

Bursa Nilufer/Goruklede ev alirken en cok degerlenecek mahalle/sokagi
belirlemeye yardimci, veriye dayali bir analiz/agent gelistirmek. ONEMLI:
kullanici sadece Görükle degil, TUM NILUFER icin veri istiyor (Görükle'ye
ozel filtre/scope uygulanmiyor, o sadece kullanicinin oncelikli ilgi alani).

Arsa birim degerleri (bronze'da, 1986-2026 tum yillar, filtre yok — MIN_YIL
filtresi kaldirildi) ve bina metrekare birim degerleri (bronze'a yazilacak,
1986-2026, mahalle bazli degil tum Nilufer) ilk iki veri kaynagi.

Baska aday CKAN dataset'leri (arastirildi, henuz eklenmedi):
nilufer-ilcesi-mahalle-bazli-nufus-2015-2024, zemin-etut-bilgileri (Görükle
dahil 6 bolge PDF), mahalle-sinirlari (GeoJSON), yesil-alan-ve-parklar,
afad-toplanma-alanlari (GeoJSON), egitim-bilim-teknoloji-mekanlari,
bina-asinma-paylari.

## Bekleyen isler / sonraki oturumda yapilacaklar

1. Kullanici Databricks'te su iki ciftini test edecek:
   - `Nilufer Arsa Verisi Indir (Local).ipynb` -> CSV -> DBFS/Volume'a manuel
     yukleme -> `Nilufer Arsa Datalake Upload.ipynb` (source_path widget'i ile)
   - `Nilufer Bina Verisi Indir (Local).ipynb` -> CSV -> manuel yukleme ->
     `Nilufer Bina Datalake Upload.ipynb`
   Ikisi de lokalde (bu ortamda) test edildi ve calisiyor (arsa: 179704 satir,
   bina: 13254 satir), ama gercek Databricks ortaminda upload adimi henuz
   dogrulanmadi.
2. Test basariliysa/sorun cikarsa bir sonraki adim: bu iki bronze veriyi
   birlestirip bir "gold" analiz katmani (mahalle bazli degerlenme skoru,
   buyume orani/CAGR, trend momentumu) kurmak — henuz baslanmadi.
3. Nufus verisi (nilufer-ilcesi-mahalle-bazli-nufus-2015-2024) muhtemelen
   sirada bir sonraki eklenecek kaynak, ayni Local+Upload deseniyle.
4. Wiki push (`Nilufer Arsa Wiki Push.ipynb`) su an sadece arsa verisi icin
   var; bina verisi icin benzer bir wiki push henuz yazilmadi, istenirse
   `build_agent_friendly_wiki_content` zaten genel amacli, direkt kullanilabilir.

## Ortam bilgisi

- Repo hem GitHub (`origin`) hem Azure DevOps (`azure`) remote'larina bagli.
- Databricks Secret Scope: `sql-ozoezer`, key: `devopspac` (Azure DevOps PAT).
- Azure DevOps org: `ozanozeer`, project: `aXet Project`, wiki: `aXet-Project.wiki`.
- Data lake: `abfss://axetproject@ozandatalake001.dfs.core.windows.net/axet_bronze/...`
