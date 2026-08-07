# aXet-Project

AXet bazlı test projesi. Databricks notebook'ları ile açık veri kaynaklarından
(yfinance, TEFAS, Nilüfer Belediyesi Açık Veri Portalı vb.) veri çekip Azure
Data Lake'e (bronze katman) yazan ve sonuçları Azure DevOps Wiki'ye raporlayan
bir veri mühendisliği örnek projesi.

## Notebooklar

- **Utils**: Tüm ortak yardımcı fonksiyonları içerir, başlıklara ayrılmıştır:
  - *Azure DevOps Wiki Helpers*: `get_wiki_page`, `create_wiki_page`, `update_wiki_page`, `push_wiki_page`
  - *Data Lake I/O Helpers*: `write_to_datalake`, `read_from_datalake`, `get_datalake_size_bytes`
  - *Open Data Portal Helpers (CKAN)*: `get_ckan_resource`, `download_file`
  - *Agent-Friendly Wiki Content Builders*: `build_agent_friendly_wiki_content`

  Diğer notebooklardan `%run "./Utils"` ile çağrılır.

- **Data Ingestion**: yfinance üzerinden emtia fiyatlarını (altın, gümüş, Brent petrol) çekip Spark DataFrame'e çevirir ve Azure Data Lake'e (bronze katman) JSON olarak yazar.

- **Nilufer Arsa Birim Degerleri**: Nilüfer Belediyesi Açık Veri Portalı'ndan (CKAN, CC BY 4.0) arsa birim değerleri verisini indirir, 2010 ve sonrası ile filtreler, bronze katmana yazar. İndirilen dosya işlendikten sonra hemen silinir, localde kalıcı veri tutulmaz.

- **Nilufer Arsa Wiki Push**: Bronze katmandaki arsa birim değerleri verisinin metadata + schema + örnek verisini, bir AI agent'ın kolayca parse edebileceği yapılandırılmış (JSON + markdown tablo) formatta Azure DevOps Wiki'ye yazar.

- **Test for GitConnection**: Bronze veriyi okuyup şema ve örnek veriyi Azure DevOps Wiki'ye yazan örnek akış.

- **TEFAS Fetch** *(kullanılmamalı, `.gitignore`'da)*: TEFAS API'sinin `robots.txt` üzerinden tüm otomatik erişimlere kapattığı (`Disallow: /api/`) bir endpoint'i kullanır. Sadece geçmiş bir denemenin kaydı olarak tutulur, repoya girmez.

## Repo / Remote

Proje hem GitHub (`origin`) hem Azure DevOps (`azure`) remote'larına push edilir, ikisi de senkron tutulur.

## Geliştirme Prensipleri

Bu projede AI destekli geliştirme yapılırken şu yaklaşımlar izlenir:

1. **Databricks notebook'ları `.ipynb` formatında tutulur** — düz `.py` script yazılmaz, her notebook Databricks'te doğrudan çalıştırılabilir olmalı (`%pip install`, `%run`, `dbutils`, `spark` gibi Databricks-native çağrılar kullanılır).
2. **Tekrarlanan kod `Utils.ipynb`'a taşınır** — herhangi bir fonksiyon birden fazla notebook'ta kullanılacaksa (veya kullanılma potansiyeli varsa) `Utils.ipynb`'a eklenir, diğer notebook'lar `%run "./Utils"` ile çağırır. Kod tekrarı bırakılmaz.
3. **Utils.ipynb başlıklara (markdown hücreleriyle) bölünür** — fonksiyonlar rastgele sıralanmaz, ilgili oldukları konuya göre (`# Azure DevOps Wiki Helpers`, `# Data Lake I/O Helpers` vb.) markdown başlıklarıyla gruplanır, okunabilirlik korunur.
4. **Dış kaynaklardan veri çekmeden önce `robots.txt` ve kullanım şartları/lisans kontrol edilir** — bir endpoint'in teknik olarak çalışması yeterli değildir; `Disallow` kuralları, resmi API dokümantasyonu var mı, lisans (CC BY vb.) neyi gerektiriyor kontrol edilip kullanıcıya raporlanır. Şüpheli/riskli bulunan kod kullanılmaz, ama silinmeden not düşülüp `.gitignore`'a eklenir.
5. **Nazik ve performanslı veri çekimi tercih edilir** — paralel istek sayısı, `Crawl-Delay` gibi kurallara saygı, gereksiz tekrar istek atılmaz; maliyet (Databricks compute, depolama) her adımda göz önünde bulundurulur.
6. **İndirilen geçici dosyalar hemen silinir** — CKAN/harici API'lerden indirilen ham dosyalar (xlsx, csv vb.) işlenip DataFrame'e çevrildikten sonra `os.remove()` ile silinir, localde/repoda kalıcı veri tutulmaz (`.gitignore`'da `*.xlsx`, `*.csv` zaten hariç tutulur).
7. **Wiki'ye yazılan içerik, insan yerine bir AI agent'ın kolayca parse edebileceği şekilde tasarlanır** — metadata ve schema bilgisi fenced JSON kod bloğu içinde verilir (serbest metin/markdown tablo değil), örnek veri de JSON formatında sunulur. Tüm veri asla wiki'ye basılmaz, sadece küçük bir örnek (`limit(N)`) ve özet metadata (satır sayısı yerine boyut/partition gibi ucuz metrikler) yer alır.
8. **Her değişiklik hem GitHub hem Azure DevOps remote'larına push edilir** — iki remote da (`origin`, `azure`) senkron tutulur.
