PROGRAM HastaBilgiSistemiAnaMenu

  BAŞLA
    // --- ADIM 1: TEK SEFERLİK KİMLİK DOĞRULAMA ---
    // Kullanıcıyı sisteme almadan önce kim olduğunu doğrula.
    giris_yapan_hasta = KimlikDoğrula()

    // Giriş başarılı olduysa, yani fonksiyon boş bir değer döndürmediyse...
    EĞER giris_yapan_hasta != HİÇBİR_ŞEY İSE
      GÖSTER "Hoş geldiniz, " + giris_yapan_hasta.isim

      // --- ADIM 2: ANA MENÜ DÖNGÜSÜ ---
      // Kullanıcı çıkış yapmadığı sürece menüyü göstermeye devam et.
      SÜRECE DOĞRU: // Sonsuz bir döngü başlat
      
        GÖSTER "------------------------------------"
        GÖSTER "Lütfen yapmak istediğiniz işlemi seçiniz:"
        GÖSTER "1: Hastane Randevu İşlemleri"
        GÖSTER "2: Tahlil Sonuçlarımı Görüntüle"
        GÖSTER "3: Çıkış Yap"
        GÖSTER "------------------------------------"
        OKU secim

        // --- ADIM 3: KULLANICI SEÇİMİNİ YÖNLENDİRME ---
        // Kullanıcının seçimine göre ilgili prosedürü çalıştır.
        SEÇİM DURUMU secim:
          DURUM "1":
            // Randevu sistemi akışını başlat.
            RandevuIslemleriniBaslat(giris_yapan_hasta)
            
          DURUM "2":
            // Tahlil sonucu görüntüleme akışını başlat.
            TahlilSonuclariniGoster(giris_yapan_hasta)
            
          DURUM "3":
            GÖSTER "Oturumunuz sonlandırılıyor. İyi günler dileriz."
            DÖNGÜDEN ÇIK // Sonsuz döngüyü sonlandır.
            
          VARSAYILAN: // 1, 2 veya 3 dışında bir şey girildiyse
            GÖSTER "Hatalı seçim yaptınız. Lütfen menüden geçerli bir numara giriniz."
        BİTİR SEÇİM DURUMU
        
      BİTİR DÖNGÜ

    DEĞİLSE // Kimlik doğrulama başarısız olduysa
      GÖSTER "Hata: Girdiğiniz bilgiler hatalıdır. Sistem kapatılıyor."
    BİTİR EĞER

  BİTİR

// ######################################################################
// # ANA MENÜNÜN ÇAĞIRDIĞI ANA PROSEDÜRLER                             #
// ######################################################################

PROSEDÜR RandevuIslemleriniBaslat(hasta_bilgisi):
  GÖSTER "\n*** Randevu Sistemi Başlatıldı ***"
  secilen_poliklinik = PoliklinikSectir()
  secilen_doktor = DoktorSectir(secilen_poliklinik)
  secilen_saat = SaatSectir(secilen_doktor)

  EĞER secilen_saat != HİÇBİR_ŞEY İSE
    onay_durumu = RandevuOnayla(hasta_bilgisi, secilen_poliklinik, secilen_doktor, secilen_saat)
    EĞER onay_durumu == DOĞRU İSE
      randevu_bilgileri = secilen_poliklinik + ", " + secilen_doktor + ", " + secilen_saat
      SmsGonder(hasta_bilgisi, randevu_bilgileri)
    BİTİR EĞER
  BİTİR EĞER
  GÖSTER "*** Randevu Sistemi Tamamlandı. Ana Menüye Dönülüyor... ***\n"
BİTİR PROSEDÜR


PROSEDÜR TahlilSonuclariniGoster(hasta_bilgisi):
  GÖSTER "\n*** Tahlil Sonuç Sistemi Başlatıldı ***"
  hasta_tahlilleri = VeritabanındanTahlilleriGetir(hasta_bilgisi.id)

  EĞER hasta_tahlilleri LİSTESİ BOŞ İSE
    GÖSTER "Sistemde adınıza kayıtlı herhangi bir tahlil bulunmamaktadır."
  DEĞİLSE
    GÖSTER "Tahlilleriniz:"
    HER tahlil İÇİN hasta_tahlilleri LİSTESİNDE:
      EĞER tahlil.durum == "Hazır" İSE
        GÖSTER tahlil.tarih + " - " + tahlil.adi + ": [Sonucu Görüntüle] [PDF İndir]"
      DEĞİLSE
        GÖSTER tahlil.tarih + " - " + tahlil.adi + ": Henüz Hazır Değil"
      BİTİR EĞER
    BİTİR DÖNGÜ
  BİTİR EĞER
  GÖSTER "*** Tahlil Sonuç Sistemi Tamamlandı. Ana Menüye Dönülüyor... ***\n"
BİTİR PROSEDÜR


// ######################################################################
// # YARDIMCI FONKSİYONLAR VE PROSEDÜRLER (Detayları önceki kodlarda) #
// ######################################################################

// Bu fonksiyonlar, yukarıdaki ana prosedürler tarafından kullanılır.
// Tanımları, daha önceki örneklerde yazdığımız gibidir.

FONKSİYON KimlikDoğrula(): ...
FONKSİYON PoliklinikSectir(): ...
FONKSİYON DoktorSectir(poliklinik): ...
FONKSİYON SaatSectir(doktor): ...
FONKSİYON RandevuOnayla(hasta_bilgisi, poliklinik, doktor, saat): ...
PROSEDÜR SmsGonder(hasta_bilgisi, randevu_detaylari): ...
FONKSİYON VeritabanındanTahlilleriGetir(hasta_id): ...
PROSEDÜR PdfOlusturVeIndir(tahlil_id): ...
