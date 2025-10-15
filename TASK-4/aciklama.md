

Hazırlamış olduğum bu pseudocode, üniversite ders kayıt sisteminin mantıksal akışını adım adım modellemektedir. Kod, öğrencinin ders seçimi yaptığı bir ana döngü üzerine kuruludur ve seçilen her dersin; kredi limiti, kontenjan, ön koşul ve zaman çakışması gibi kriterlere uygunluğunu iç içe EĞER-İSE yapılarıyla denetlemektedir. Kontrolleri başarıyla geçen bir ders geçici "ders sepetine" eklenirken, herhangi bir kural ihlalinde sistem kullanıcıya bir hata mesajı iletmektedir. Bu döngü, öğrenci seçimlerini tamamlayıp kaydı danışman onayına gönderene kadar devam edecek şekilde tasarlanmıştır.
