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

## Prompt-yazma referans kaynaklari (statik gomulecek)

Prompt-writer/agent-optimizer akisi icin secilen 3 statik kaynak (Databricks
tarafindan periyodik cekilip Wiki'ye yazilacak, prompt yazma kurallari icin
referans alinacak):

1. **Vendor**: Anthropic — Claude Prompt Engineering guide
   (https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)
   — XML tag yapilandirma, few-shot format, uzun-context yerlesimi, agentic
   sistem kurallari.
2. **Akademik**: The Prompt Report — Schulhoff et al., 2024
   (https://arxiv.org/abs/2406.06608) — 58 teknik + standart terminolojiyle
   kapsamli taksonomi/sozluk.
3. **Topluluk**: Prompt Engineering Guide — dair-ai
   (https://www.promptingguide.ai/, repo: https://github.com/dair-ai/Prompt-Engineering-Guide)
   — teknik-basina kisa/ornekli katalog; repo duz markdown oldugu icin
   Databricks'in git clone/raw-fetch ile cekmesi kolay.

Not: APE (arxiv.org/abs/2211.01910) ve DSPy (arxiv.org/abs/2310.03714) bilerek
disarida tutuldu — bunlar statik "kural kaynagi" degil, otomatik prompt
uretme/optimize etme *algoritmalari*; ileride optimizer-agent'in kendi
mantigini kurarken referans olarak kullanilabilir, simdilik gomulmuyor.

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

1. `Prompt Kaynaklari Wiki Sync.ipynb` lokal Python ile (Databricks
   disi, `dbutils`/`spark` icermeyen fonksiyonlar) uctan uca test edildi:
   robots.txt kontrolu, Anthropic extraction (prose-filtre iyilestirildi),
   arXiv abstract parse, dair-ai 9 teknik dosyasi (duz `.mdx` path,
   klasorsuz — ilk denemede yanlis path 404 verdi, duzeltildi) hepsi
   basarili. Databricks'te fiili `push_wiki_page` calistirmasi (PAT,
   secret scope) henuz dogrulanmadi — bir sonraki oturumda kullanici
   Databricks'te calistirip sonucu paylasmali.
2. Bu statik kaynaklar wiki'ye yazildiktan sonra, prompt-writer agent'inin
   system prompt'una nasil gomulecegi (tam metin mi, ozet mi) karara
   baglanacak.
3. Prompt-writer/agent-optimizer akisinin kendisi (kac alt-agent lazim,
   gap detection, DevOps work item acma vb.) henuz tasarlanmadi — bu
   kaynaklar sadece referans/zemin hazirligi.


## Ortam bilgisi

- Repo hem GitHub (`origin`) hem Azure DevOps (`azure`) remote'larina bagli.
- Databricks Secret Scope: `sql-ozoezer`, key: `devopspac` (Azure DevOps PAT).
- Azure DevOps org: `ozanozeer`, project: `aXet Project`, wiki: `aXet-Project.wiki`.
