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

- **Prompt Kaynaklari Wiki Sync**: Agent-description / agent-mimarisi
  (single-agent vs. sequential vs. multi-agent) akışı için seçilen 6 statik
  referans kaynağını (Anthropic — Building Effective Agents, Anthropic —
  How We Built Our Multi-Agent Research System, Google ADK — Sequential
  Agents, OpenAI Agents SDK — Agent Orchestration, Microsoft Semantic
  Kernel — Sequential Orchestration, Microsoft Azure Architecture Center —
  AI Agent Orchestration Patterns) çeker, agent-friendly formatta özetler
  ve Azure DevOps Wiki'ye (`/Prompt-Kaynaklari/...`) yazar. 3-5. kaynaklar
  **sıralı (sequential) agent zincirleme** deseninin kaynaklı şekilde
  değerlendirilebilmesi için, 6. kaynak ise "ne zaman single-agent yeterli,
  ne zaman hangi multi-agent pattern'i (sequential/concurrent/group-chat/
  handoff/magentic)" sorusuna somut kriterlerle cevap vermek için eklendi.

- **Agent Mimarisi Ureticisi - Wiki Yayinla**: Yukarıdaki gibi dış
  kaynaktan çekmez — elle yazılmış, statik bir *agent tanım* sayfasını
  (`/Agent-Mimarisi-Ureticisi`) tek seferlik Wiki'ye yayınlar. Bu sayfa,
  **aXet.code'un değil**, **aXet Agentic** tarafından çalıştırılacak ayrı
  bir agent'ın (Optimizer — proje brief'inden agent mimarisi/description
  üreten agent) tanımıdır; görevler bilerek ayrı tutulur.

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
