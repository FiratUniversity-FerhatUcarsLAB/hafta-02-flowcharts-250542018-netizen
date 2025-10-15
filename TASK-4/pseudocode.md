BAŞLA:

//--- Değişkenleri ve öğrenci bilgilerini hazırla ---
ogrenci = Mevcut_Ogrenci_Bilgilerini_Getir()
ogrenci_transkripti = ogrenci.Transkriptini_Getir()
donem_dersleri = Acilan_Tum_Dersleri_Listele()
ders_sepeti = BOS_LISTE
mevcut_kredi = 0
kredi_limiti = 30 // Üniversitenin belirlediği limit

//--- Öğrenci ders seçimi yapmaya başlar ---
DÖNGÜ (öğrenci "Ders Seçimini Bitir" butonuna basana kadar):

    secilecek_ders = ogrenci.Sectigi_Dersi_Getir() // Öğrenci listeden bir derse tıklar

    //--- KONTROL 1: KREDİ LİMİTİ ---
    EĞER (mevcut_kredi + secilecek_ders.kredi > kredi_limiti) ISE:
        Hata_Goster("Maksimum kredi limitini aşıyorsun!")
        DÖNGÜYE_DEVAM_ET // Bu dersi ekleme, sonraki seçimi bekle
    DEGILSE:
        // Kredi limiti sorunu yok, diğer kontrollere geç

        //--- KONTROL 2: KONTENJAN ---
        EĞER (secilecek_ders.dolu_kontenjan >= secilecek_ders.toplam_kontenjan) ISE:
            Hata_Goster("Bu dersin kontenjanı dolu!")
            DÖNGÜYE_DEVAM_ET
        DEGILSE:
            // Kontenjan sorunu yok, devam et

            //--- KONTROL 3: ÖN KOŞUL ---
            on_kosul_sorunu_var_mi = HAYIR
            EĞER (secilecek_ders.on_kosulu_var_mi) ISE:
                on_kosul_dersi = secilecek_ders.On_Kosulunu_Getir()
                EĞER (ogrenci_transkripti.Bu_Dersi_Basariyla_Gecti_Mi(on_kosul_dersi) == HAYIR) ISE:
                    Hata_Goster("Bu dersi alabilmek için önce '" + on_kosul_dersi.adi + "' dersini başarmalısın!")
                    on_kosul_sorunu_var_mi = EVET
                BITIR_EĞER
            BITIR_EĞER

            EĞER (on_kosul_sorunu_var_mi) ISE:
                DÖNGÜYE_DEVAM_ET
            DEGILSE:
                // Ön koşul sorunu yok, devam et

                //--- KONTROL 4: ZAMAN ÇAKIŞMASI ---
                cakisma_var_mi = HAYIR
                // Sepetteki mevcut her dersle yeni dersi karşılaştır
                DÖNGÜ (sepetteki_her_ders IÇIN ders_sepeti):
                    EĞER (secilecek_ders.zamani == sepetteki_her_ders.zamani) ISE:
                        Hata_Goster("'" + secilecek_ders.adi + "' dersi, '" + sepetteki_her_ders.adi + "' dersiyle çakışıyor!")
                        cakisma_var_mi = EVET
                        DÖNGÜYÜ_KIR // Başka dersle karşılaştırmaya gerek yok
                    BITIR_EĞER
                BITIR_DÖNGÜ

                EĞER (cakisma_var_mi) ISE:
                    DÖNGÜYE_DEVAM_ET
                DEGILSE:
                    // --- TÜM KONTROLLER BAŞARILI ---
                    // Tebrikler! Hiçbir sorun yok, dersi sepete ekle.
                    ders_sepeti.Ekle(secilecek_ders)
                    mevcut_kredi = mevcut_kredi + secilecek_ders.kredi
                    Mesaj_Goster("'" + secilecek_ders.adi + "' sepete eklendi. Toplam Kredi: " + mevcut_kredi)
                BITIR_EĞER // Çakışma kontrolü sonu
            BITIR_EĞER // Ön koşul kontrolü sonu
        BITIR_EĞER // Kontenjan kontrolü sonu
    BITIR_EĞER // Kredi limiti kontrolü sonu

BITIR_DÖNGÜ // Öğrenci ders seçimini bitirdi

//--- SON ADIM: ONAYLAMA ---
EĞER (öğrenci "Kaydı Tamamla" butonuna basarsa):
    Danisman_Onayina_Gonder(ders_sepeti)
    Bilgi_Ver("Ders kaydınız danışman onayına gönderilmiştir.")
DEGILSE:
    Bilgi_Ver("Ders kaydı iptal edildi.")
BITIR_EĞER

BİTİR.
