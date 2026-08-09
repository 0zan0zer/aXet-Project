# aXet-Project

aXet tabanlı proje deposu. Databricks notebook'ları ile veri/agent işleri
yapılır, Azure DevOps Wiki'ye raporlanır.

## Notebooklar

- **Utils**: Ortak yardımcı fonksiyonları içerir, başlıklara ayrılmıştır:
  - *Azure DevOps Wiki Helpers*: `get_wiki_page`, `create_wiki_page`,
    `update_wiki_page`, `push_wiki_page` — Azure DevOps Wiki REST API'sine
    (PAT, Databricks Secret Scope üzerinden) bağlanan temel fonksiyonlar.
  - *Web Fetch Helpers*: `check_robots_allowed`, `fetch_url_text` — dış
    kaynaklardan veri çekerken robots.txt kontrolü ve genel amaçlı HTTP GET.

  Diğer notebook'lardan `%run "./Utils"` ile çağrılır.

- **Prompt Kaynaklari Wiki Sync**: Prompt-writer/agent-optimizer akışı için
  seçilen 3 statik referans kaynağını (Anthropic Claude Prompt Engineering,
  The Prompt Report — arXiv, dair-ai Prompt Engineering Guide) çeker,
  agent-friendly formatta özetler ve Azure DevOps Wiki'ye
  (`/Prompt-Kaynaklari/...`) yazar.

## Repo / Remote

Proje hem GitHub (`origin`) hem Azure DevOps (`azure`) remote'larına push
edilir, ikisi de senkron tutulur.

## Geliştirme Prensipleri

1. **Databricks notebook'ları `.ipynb` formatında tutulur** — düz `.py`
   script yazılmaz.
2. **Tekrarlanan kod `Utils.ipynb`'a taşınır** — herhangi bir fonksiyon
   birden fazla notebook'ta kullanılacaksa (veya kullanılma potansiyeli
   varsa) `Utils.ipynb`'a eklenir, diğer notebook'lar `%run "./Utils"` ile
   çağırır. Kod tekrarı bırakılmaz.
3. **Utils.ipynb başlıklara (markdown hücreleriyle) bölünür** — fonksiyonlar
   rastgele sıralanmaz, ilgili oldukları konuya göre markdown başlıklarıyla
   gruplanır.
4. **Dış kaynaklardan veri çekmeden önce `robots.txt` ve kullanım
   şartları/lisans kontrol edilir.**
5. **Nazik ve performanslı veri çekimi tercih edilir** — paralel istek
   sayısı, `Crawl-Delay` gibi kurallara saygı gösterilir; maliyet
   (Databricks compute, depolama) her adımda göz önünde bulundurulur.
6. **İndirilen geçici dosyalar hemen silinir** — ham dosyalar (xlsx, csv
   vb.) işlenip DataFrame'e çevrildikten sonra silinir, localde/repoda
   kalıcı veri tutulmaz.
7. **Wiki'ye yazılan içerik, bir AI agent'ın kolayca parse edebileceği
   şekilde tasarlanır** — metadata/schema fenced JSON kod bloğu içinde
   verilir.
8. **Her değişiklik hem GitHub hem Azure DevOps remote'larına push edilir.**
