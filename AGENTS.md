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
12. aXet.code bu projede kullanicinin her istegini sorgusuz uygulamaz.
    Mantiksiz, mevcut mimariyi/akisi bozan, gereksiz teknik borc yaratan veya
    daha once alinmis bir karara (bu dosyadaki kurallar, secilen kaynaklar,
    mevcut notebook yapisi vb.) acikca ters dusen bir talep gelirse:
    (a) once neden sorunlu oldugunu kisa ve durust sekilde aciklar,
    (b) varsa daha mantikli bir alternatif onerir,
    (c) kullanici ustte durup acikca onay verirse ("evet boyle istiyorum"
    gibi) yine de uygular — karar nihayetinde kullanicinin, ama sessizce
    "evet efendim" deyip kotu bir yon degistirmez.

## Agent-description / agent-mimarisi referans kaynaklari (statik gomulecek)

Agent-description yazimi ve single-agent vs. sequential vs. multi-agent
mimari karari icin secilen 5 statik kaynak (Databricks tarafindan periyodik
cekilip Wiki'ye yazilacak, agent tasarim kurallari icin referans alinacak).
Onceki 3 "prompt yazma" kaynagi (Anthropic Claude Prompt Engineering, The
Prompt Report, dair-ai Guide) bilerek cikarildi — bunlar genel prompt
teknikleri, agent'lari nasil boluceginiz/tanimlayacaginiz konusuna dogrudan
odaklanmiyor:

1. **Vendor**: Anthropic — Building Effective Agents
   (https://www.anthropic.com/engineering/building-effective-agents)
   — workflow vs. agent ayrimi, ne zaman agent kullanilmali/kullanilmamali,
   temel agent desenleri (prompt chaining, routing, parallelization,
   orchestrator-workers, evaluator-optimizer), agent-computer interface
   (tool/description yazimi) prensipleri.
2. **Vendor**: Anthropic — How We Built Our Multi-Agent Research System
   (https://www.anthropic.com/engineering/multi-agent-research-system)
   — ne zaman multi-agent'a gecilmeli (maliyet/token tradeoff'u dahil),
   orchestrator-worker mimarisi, lead agent'in subagent'lara verdigi task
   description'in nasil yazilmasi gerektigi, effort/scale kurallari,
   subagent sayisi belirleme heuristikleri.
3. **Vendor**: Google ADK — Sequential Agents
   (https://google.github.io/adk-docs/agents/workflow-agents/sequential-agents/)
   — sadece **sequential (sirali)** agent zincirleme deseni: `SequentialAgent`
   sinifi, sub-agent'larin sabit sirada calistirilmasi, `output_key` ile
   state uzerinden veri aktarimi (CodeWriter -> CodeReviewer -> CodeRefactorer
   ornegi).
4. **Vendor**: OpenAI Agents SDK — Agent Orchestration
   (https://openai.github.io/openai-agents-python/multi_agent/)
   — "orchestrating via code" bolumunde sequential chaining (bir agent'in
   ciktisini digerinin girdisine donusturme) ile paralel calistirmanin
   (`asyncio.gather`) acik karsilastirmasi; "agents as tools" vs "handoffs"
   tablosu.
5. **Vendor**: Microsoft Semantic Kernel — Sequential Orchestration
   (https://learn.microsoft.com/en-us/semantic-kernel/frameworks/agent/agent-orchestration/sequential)
   — `SequentialOrchestration` sinifi, agent pipeline ornegi (analyst ->
   copywriter -> editor), ne zaman sequential pattern kullanilmali/
   kullanilmamali (Azure Architecture Center'a referansla).

3-5 numarali kaynaklar bilhassa **paralel (orchestrator-workers) disinda,
sirali (sequential) agent zincirleme secenegini de kaynakli/dogrulanmis
sekilde degerlendirebilmek** icin eklendi — Optimizer'in "sadece paralel
degil, sequential agent mimarisi de kurabiliyoruz" karar noktasi bu 3
kaynaga dayanacak.

anthropic.com, google.github.io, openai.github.io: robots.txt tamamen
serbest (`Allow: /` veya robots.txt yok -> serbest kabul). learn.microsoft.com:
ilgili path Disallow listesinde degil (serbest).

## Proje hedefi (guncel)

Sifirdan basliyoruz: onceki Nilufer arsa/bina ve TEFAS denemeleri
temizlendi. Su an sadece `Utils.ipynb` icindeki Azure DevOps Wiki REST API
baglanti fonksiyonlari (`get_wiki_page`, `create_wiki_page`,
`update_wiki_page`, `push_wiki_page`) korunuyor; yeni is bunlarin ustune
kurulacak.

**Onemli mimari ayrim:** Bu repo (ve AGENTS.md) sadece **aXet.code**'un
(bu depoda calisan coding assistant) hafizasidir — burada tanimlanan hicbir
kural/sozlesme baska bir agent tarafindan okunmaz. Asil is olan
**"Agent Mimarisi Ureticisi" (Optimizer)** — kullanicidan bir proje/gorev
brief'i alip buna en uygun agent mimarisini (single-agent / workflow deseni
/ multi-agent) ve her agent'in description'ini ureten agent — **aXet
Agentic** tarafindan calistirilacak, tanimi da tamamen **Azure DevOps
Wiki**'de tutulacak (`/Agent-Mimarisi-Ureticisi` sayfasi), bu repo'nun
dosyalarinda degil. Bu repo'nun (Databricks notebook'lari) buradaki tek
gorevi: Optimizer'in referans alacagi statik kaynaklari (yukaridaki bolum)
periyodik cekip Wiki'ye yazmak. Optimizer'in karar mantigi/cikti semasi
gibi *is mantigi* icerikleri asla bu repo'ya (AGENTS.md, notebook, vb.)
yazilmaz — sadece Wiki'ye.

## Bekleyen isler / sonraki oturumda yapilacaklar

1. `Prompt Kaynaklari Wiki Sync.ipynb` 2 kaynaktan 5 kaynaga cikarildi:
   Anthropic'in genel workflow/agent taksonomisine (Building Effective
   Agents, Multi-Agent Research System) ek olarak, sadece **sequential**
   (sirali) agent zincirleme desenine odaklanan 3 yeni vendor kaynagi
   (Google ADK Sequential Agents, OpenAI Agents SDK Orchestration,
   Microsoft Semantic Kernel Sequential Orchestration) eklendi. Extraction
   fonksiyonu genellestirildi: `render_content_elements` artik `<pre>` kod
   bloklarini fenced code, `<table>` elemanlarini `|`-ayrikli satirlara
   ceviriyor (OpenAI kaynagindaki karsilastirma tablosu icin). MS Learn
   sayfasi icin ayrica `extract_ms_learn_content` eklendi (C#/Java pivot'lari
   `decompose()` ile silinip sadece Python pivot'u tutuluyor, `<article>`
   yerine `div.content` yapisi kullaniliyor). Lokal Python ile (Databricks
   disi) tum 5 kaynak icin robots.txt + extraction test edildi, hepsi
   basarili (OpenAI kaynaginda tablo dogru cikarildi, MS Learn'de dil
   sizintisi yok). Kullanici cluster acilinca fiili Databricks
   calistirmasini dogrulayacak.
2. `Agent Mimarisi Ureticisi - Wiki Yayinla.ipynb` guncellendi (v0.2):
   karar surecine 4. adim olarak "sequential mi, parallel mi" ayrimi
   eklendi — `multi-agent-sequential` (Google ADK/MS Semantic
   Kernel/OpenAI kaynaklarina dayanan, sabit sirali agent pipeline'i) artik
   `multi-agent-orchestrator-workers` (paralel) ile esit bir secenek olarak
   cikti semasinda yer aliyor (`architecture` enum'una eklendi,
   `execution_order` alani eklendi). Databricks connector session gecici
   olarak kesildigi icin dogrudan yazilamadi, kullanici cluster acilinca bu
   notebook'u da calistiracak. Icerik ilk taslak; kullanici degerlendirip
   onaylayacak/revize edecek.
3. Ornek agent description toplamak (Claude Agent SDK / Claude Code
   subagent formati gibi) suan icin ertelendi — once sozlesme/rubric
   netlesince, sadece gerekirse ve kucuk sayida (3-5) kalibrasyon ornegi
   icin gundeme gelecek.


## Ortam bilgisi

- Repo hem GitHub (`origin`) hem Azure DevOps (`azure`) remote'larina bagli.
- Databricks Secret Scope: `sql-ozoezer`, key: `devopspac` (Azure DevOps PAT).
- Azure DevOps org: `ozanozeer`, project: `aXet Project`, wiki: `aXet-Project.wiki`.
