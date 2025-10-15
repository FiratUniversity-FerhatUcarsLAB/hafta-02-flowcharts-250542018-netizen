BAŞLA

  // --- SİSTEM DEĞİŞKENLERİ VE BAŞLANGIÇ DURUMU ---
  Sistem_Durumu = "AKTİF"     // Sistem devrede mi? (AKTİF/PASİF)
  Ev_Sahibi_Evde_Mi = HAYIR   // Ev sahibinin konum bilgisi (EVET/HAYIR)
  Alarm_Durumu = "CALMIYOR"   // Alarmın mevcut durumu
  Alarm_Seviyesi = 0          // 0: Yok, 1: Düşük, 2: Orta, 3: Yüksek

  // --- ANA GÜVENLİK DÖNGÜSÜ (KESİNTİSİZ ÇALIŞMA) ---
  DÖNGÜ SÜREKLİ OLARAK:

    // --- Koşul 1: Sistem Aktif mi Kontrolü ---
    EĞER (Sistem_Durumu == "AKTİF") ISE:
    
      // --- Adım 1: Sensör Okuma (Sürekli Döngü Parçası) ---
      hareket_durumu = Hareket_Sensoru_Oku()
      kapi_pencere_durumu = Kapi_Pencere_Sensoru_Oku()
      
      // --- Koşul 2 & 3: Hareket veya Kapı/Pencere İhlali Kontrolü ---
      EĞER (hareket_durumu == "HAREKET VAR" VEYA kapi_pencere_durumu == "AÇIK") ISE:
        
        // --- Koşul 4: Yanlış Alarm Kontrolü (Ev Sahibi Evde mi?) ---
        EĞER (Ev_Sahibi_Evde_Mi == HAYIR) ISE:
          // --- GERÇEK TEHDİT DURUMU ---
          
          // --- Adım 2: Kamera Aktivasyonu ---
          EĞER (kameralar_aktif_degil) ISE:
            Kameralari_Aktif_Et_Ve_Kayda_Basla()
          BİTİR EĞER
          
          // --- Adım 3: Alarm Seviyesi Belirleme ---
          EĞER (kapi_pencere_durumu == "AÇIK") ISE:
            Alarm_Seviyesi = 3 // Yüksek Öncelik: Fiziksel ihlal var
          DEGILSE EĞER (hareket_durumu == "HAREKET VAR") ISE:
            Alarm_Seviyesi = 2 // Orta Öncelik: Sadece hareket algılandı
          BİTİR EĞER
          
          // --- Adım 4: Alarmı ve Bildirimleri Tetikleme ---
          EĞER (Alarm_Durumu == "CALMIYOR") ISE:
            Alarm_Durumu = "CALIYOR"
            Alarm_Cal(Alarm_Seviyesi) // Seviyeye göre farklı ses/ışık şiddeti
            
            mesaj = "ALARM SEVİYE " + Alarm_Seviyesi + ": Evde beklenmedik bir durum algılandı!"
            Bildirim_Gonder("SMS", mesaj)
            Bildirim_Gonder("UYGULAMA", mesaj)
            Bildirim_Gonder("EMAIL", mesaj)
          BİTİR EĞER
          
        BİTİR EĞER // Yanlış alarm kontrolü sonu
        
      BİTİR EĞER // Tehdit algılama sonu
      
      // --- Koşul 5: Alarm Sıfırlama veya Devam Ettirme ---
      EĞER (Alarm_Durumu == "CALIYOR") ISE:
        kullanici_komutu = Kullanici_Komutunu_Kontrol_Et()
        EĞER (kullanici_komutu == "ALARMİ SIFIRLA") ISE:
          Alarm_Durumu = "CALMIYOR"
          Alarm_Seviyesi = 0
          Sistemi_Sifirla()
        BİTİR EĞER
      BİTİR EĞER
      
    BİTİR EĞER // Sistem aktif kontrolü sonu
    
    // --- Adım 5: Bekle ve Tekrar Kontrol Et ---
    BEKLE(5 saniye) // Sistemin kaynakları verimli kullanması için bekleme
    
  BİTİR DÖNGÜ

BİTİR
