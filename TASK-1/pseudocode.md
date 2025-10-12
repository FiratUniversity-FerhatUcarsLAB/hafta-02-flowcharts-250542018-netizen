BAŞLA

  // --- Değişkenlerin Tanımlanması ---
  TANIMLA dogruSifre = "1234" // Örnek bir şifre
  TANIMLA girilenSifre
  TANIMLA hatalıGirişSayısı = 0
  TANIMLA bakiye = 5000     // Örnek bir bakiye
  TANIMLA gunlukLimit = 10000
  TANIMLA gunlukCekilenTutar = 0
  TANIMLA cekilecekTutar
  TANIMLA devamEt = EVET
  TANIMLA sifreDogrulandi = HAYIR

  // --- Giriş ve Şifre Kontrol Döngüsü ---
  GÖSTER "Lütfen kartınızı takınız."

  SÜRECE (hatalıGirişSayısı < 3 VE sifreDogrulandi == HAYIR)
    İSTE "Şifrenizi Giriniz: " -> girilenSifre

    EĞER girilenSifre == dogruSifre
      sifreDogrulandi = EVET
    DEĞİLSE
      hatalıGirişSayısı = hatalıGirişSayısı + 1
      GÖSTER "Hatalı şifre. Kalan deneme hakkınız: ", (3 - hatalıGirişSayısı)
    SON EĞER
  SON SÜRECE

  // --- Ana İşlem Menüsü ---
  EĞER sifreDogrulandi == EVET
    SÜRECE (devamEt == EVET)
      GÖSTER "Mevcut Bakiyeniz: ", bakiye

      // --- Tutar Giriş ve Kontrol Döngüsü ---
      tutarGecerli = HAYIR
      SÜRECE (tutarGecerli == HAYIR)
        İSTE "Çekmek istediğiniz tutarı giriniz: " -> cekilecekTutar

        EĞER cekilecekTutar > bakiye
          GÖSTER "Bakiye yetersiz. Lütfen tekrar giriniz."
        DEĞİLSE EĞER (cekilecekTutar % 20) != 0
          GÖSTER "Hatalı giriş. Lütfen 20 TL ve katları bir tutar giriniz."
        DEĞİLSE EĞER (gunlukCekilenTutar + cekilecekTutar) > gunlukLimit
          GÖSTER "Günlük para çekme limitini aştınız."
        DEĞİLSE
          tutarGecerli = EVET
        SON EĞER
      SON SÜRECE

      // --- Para Verme ve Güncelleme ---
      bakiye = bakiye - cekilecekTutar
      gunlukCekilenTutar = gunlukCekilenTutar + cekilecekTutar
      GÖSTER "Paranızı alabilirsiniz. Yeni bakiyeniz: ", bakiye

      // --- Başka İşlem Sorgusu ---
      İSTE "Başka bir işlem yapmak ister misiniz? (E/H): " -> cevap
      EĞER cevap == "H" VEYA cevap == "h"
        devamEt = HAYIR
      SON EĞER

    SON SÜRECE
    
    GÖSTER "İyi günler dileriz. Lütfen kartınızı alınız."

  DEĞİLSE // Şifre 3 kez yanlış girildiyse
    GÖSTER "3 kez hatalı giriş yaptınız. Kartınız bloke edilmiştir."
  SON EĞER

BİTİR
