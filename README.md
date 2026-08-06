# aXet-Project
AXet bazlı test projesi

## Notebooklar

- **Data Ingestion**: yfinance üzerinden emtia fiyatlarını (altın, gümüş, Brent petrol) çekip Spark DataFrame'e çevirir ve Azure Data Lake'e (bronze katman) JSON olarak yazar.
- **Utils**: Azure DevOps Wiki sayfalarını okuma/oluşturma/güncelleme için ortak fonksiyonları (`get_wiki_page`, `create_wiki_page`, `update_wiki_page`, `push_wiki_page`) içerir. Diğer notebooklardan `%run "./Utils"` ile çağrılır.
- **Test for GitConnection**: Bronze veriyi okuyup şema ve örnek veriyi Azure DevOps Wiki'ye yazan örnek akış.
