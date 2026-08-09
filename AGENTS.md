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
13. **Bu AGENTS.md dosyasi SADECE aXet.code icindir** (bu depoda calisan
    coding assistant — yani ben). Kullanici "Optimizer'a ekle/degistir",
    "Agent Mimarisi Ureticisi'ne şunu ekle" gibi bir istek yaptiginda, bu
    talep AGENTS.md'ye degil, `Agent Mimarisi Ureticisi - Wiki Yayinla.ipynb`
    notebook'undaki `content` degiskenine (Optimizer'in Wiki'ye yazilacak
    tanimina) uygulanir. AGENTS.md'ye Optimizer'in karar mantigi/cikti
    semasi/davranis kurallari asla yazilmaz (bkz. asagidaki "Onemli mimari
    ayrim" bolumu) — bu dosyaya sadece aXet.code'un bu repo'da nasil
    calisacagina dair kurallar girer.

## Agent-description / agent-mimarisi referans kaynaklari (statik gomulecek)

Agent-description yazimi ve single-agent vs. sequential vs. multi-agent
mimari karari icin secilen 6 statik kaynak (Databricks tarafindan periyodik
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
6. **Vendor**: Microsoft Azure Architecture Center — AI Agent Orchestration
   Patterns
   (https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns)
   — Optimizer'in karar agacindaki en kritik bosluğu dolduran kaynak:
   "start with the right level of complexity" tablosu (direct model call /
   single agent with tools / multiagent orchestration — ne zaman hangisi),
   5 orchestration pattern'i icin ayrintili "when to use / when to avoid"
   listeleri (sequential, concurrent, group chat/maker-checker, handoff,
   magentic), pattern karsilastirma tablosu, maliyet/guvenlik/gozlemlenebilirlik
   notlari, yaygin antipattern listesi.

3-6 numarali kaynaklar bilhassa **paralel (orchestrator-workers) disinda,
sirali (sequential), handoff (dinamik devretme) ve magentic (acik-ucla
dinamik planlama) secenekleri de kaynakli/dogrulanmis sekilde
degerlendirebilmek** icin eklendi. 6 numarali kaynak ozellikle "ne zaman
single-agent yeterli, ne zaman coklu-agent'a (ve hangi patterne) gecilmeli"
sorusuna somut, olculebilir kriterlerle cevap veriyor — Optimizer v0.2'deki
en belirgin bosluk buydu, v0.3'te kapatildi.

anthropic.com, google.github.io, openai.github.io: robots.txt tamamen
serbest (`Allow: /` veya robots.txt yok -> serbest kabul). learn.microsoft.com:
ilgili path'ler Disallow listesinde degil (serbest).

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

1. `Prompt Kaynaklari Wiki Sync.ipynb` 2 kaynaktan 6 kaynaga cikarildi:
   Anthropic'in genel workflow/agent taksonomisine (Building Effective
   Agents, Multi-Agent Research System) ek olarak, **sequential** agent
   zincirleme desenine odaklanan 3 vendor kaynagi (Google ADK Sequential
   Agents, OpenAI Agents SDK Orchestration, Microsoft Semantic Kernel
   Sequential Orchestration) ve "ne zaman single-agent yeterli, ne zaman
   hangi multi-agent pattern'i (sequential/concurrent/group-chat/handoff/
   magentic)" sorusuna dogrudan cevap veren Microsoft Azure Architecture
   Center kaynagi eklendi. Extraction fonksiyonu genellestirildi:
   `render_content_elements` artik `<pre>` kod bloklarini fenced code,
   `<table>` elemanlarini `|`-ayrikli satirlara ceviriyor. MS Learn
   sayfalari icin `extract_ms_learn_content` eklendi (`MS_LEARN_SOURCES`
   listesi altinda 2 kaynak - Semantic Kernel ve Azure Architecture
   Center - ayni fonksiyonu paylasiyor; pivot/dil temizligi sadece
   pivot'u olan sayfalarda etkin, digerinde no-op). Lokal Python ile
   (Databricks disi) tum 6 kaynak icin robots.txt + extraction test
   edildi, hepsi basarili. Kullanici cluster acilinca fiili Databricks
   calistirmasini dogrulayacak.
2. `Agent Mimarisi Ureticisi - Wiki Yayinla.ipynb` guncellendi (v0.3):
   karar sureci Azure Architecture Center'in "start with the right level
   of complexity" tablosuna ve 5 orchestration pattern'ine gore yeniden
   yapilandirildi. 4. karar adimi artik 5 alt-secenek iceriyor:
   `multi-agent-sequential`, `multi-agent-orchestrator-workers`
   (concurrent/paralel), yeni eklenen `multi-agent-handoff` (dinamik
   devretme/routing), `multi-agent-group-chat` (maker-checker/
   collaborative), `multi-agent-magentic` (acik-ucla dinamik planlama -
   en yuksek karmasiklik, varsayilan onerilmez). Ayrica 5. adim olarak
   AAC'nin ortak antipattern listesine karsi kendi-kendini-denetleme
   eklendi (`antipattern_check` cikti alani). Databricks connector
   session gecici olarak kesildigi icin dogrudan yazilamadi, kullanici
   cluster acilinca bu notebook'u da calistiracak. Icerik ilk taslak;
   kullanici degerlendirip onaylayacak/revize edecek.
3. Ornek agent description toplamak (Claude Agent SDK / Claude Code
   subagent formati gibi) suan icin ertelendi — once sozlesme/rubric
   netlesince, sadece gerekirse ve kucuk sayida (3-5) kalibrasyon ornegi
   icin gundeme gelecek.


## Ortam bilgisi

- Repo hem GitHub (`origin`) hem Azure DevOps (`azure`) remote'larina bagli.
- Databricks Secret Scope: `sql-ozoezer`, key: `devopspac` (Azure DevOps PAT).
- Azure DevOps org: `ozanozeer`, project: `aXet Project`, wiki: `aXet-Project.wiki`.
