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

Agent-description yazimi ve single-agent vs. multi-agent mimari karari icin
secilen 2 statik kaynak (Databricks tarafindan periyodik cekilip Wiki'ye
yazilacak, agent tasarim kurallari icin referans alinacak). Onceki 3 "prompt
yazma" kaynagi (Anthropic Claude Prompt Engineering, The Prompt Report,
dair-ai Guide) bilerek cikarildi — bu ikisi genel prompt teknikleri degil,
agent'lari nasil boluceginiz/tanimlayacaginiz konusuna dogrudan odaklaniyor:

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

Her ikisi de anthropic.com/engineering altinda, robots.txt tamamen serbest
(`User-Agent: * / Allow: /`).

## Proje hedefi (guncel)

Sifirdan basliyoruz: onceki Nilufer arsa/bina ve TEFAS denemeleri
temizlendi. Su an sadece `Utils.ipynb` icindeki Azure DevOps Wiki REST API
baglanti fonksiyonlari (`get_wiki_page`, `create_wiki_page`,
`update_wiki_page`, `push_wiki_page`) korunuyor; yeni is bunlarin ustune
kurulacak.

Yeni hedef: "prompt yazma / agent optimize etme" konseptli bir calisma —
Databricks'in duzenli olarak calistirip prompt/config bilgilerini
toplayacagi ve Azure DevOps Wiki'ye md/yapisal olarak kaydedecegi bir akis.
Detaylar henuz netlesmedi, bir sonraki oturumda konusulacak veri
kaynaklarina gore ilerlenecek.

## Bekleyen isler / sonraki oturumda yapilacaklar

1. `Prompt Kaynaklari Wiki Sync.ipynb` iki eski (prompt-yazma) kaynaktan
   iki yeni (agent-description/mimari) Anthropic Engineering kaynagina
   gecirildi: extraction mantigi da degisti — `platform.claude.com` JS
   payload regex-hack'i yerine `anthropic.com/engineering/...` duz
   sunucu-render HTML'i icin BeautifulSoup tabanli `<article>` parse'i
   yazildi (`extract_engineering_article_content`). Lokal Python ile
   (Databricks disi) uctan uca test edildi: robots.txt kontrolu ve her
   iki makale icin extraction basarili.
2. Bu statik kaynaklar wiki'ye yazildiktan sonra, agent-yazan/optimize
   eden agent'in system prompt'una nasil gomulecegi (tam metin mi, ozet
   mi) karara baglanacak.
3. Agent-description/agent-optimizer akisinin kendisi (kac alt-agent
   lazim, gap detection, DevOps work item acma vb.) henuz tasarlanmadi —
   bu kaynaklar sadece referans/zemin hazirligi.


## Ortam bilgisi

- Repo hem GitHub (`origin`) hem Azure DevOps (`azure`) remote'larina bagli.
- Databricks Secret Scope: `sql-ozoezer`, key: `devopspac` (Azure DevOps PAT).
- Azure DevOps org: `ozanozeer`, project: `aXet Project`, wiki: `aXet-Project.wiki`.
