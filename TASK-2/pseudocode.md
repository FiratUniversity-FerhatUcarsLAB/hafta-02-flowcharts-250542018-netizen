// ======================================================
// ADIM 1: KULLANICI GİRİŞİ VE OTURUM YÖNETİMİ
// ======================================================
BASLA KULLANICI_GIRISI

  TANIMLA kullaniciAdi, sifre
  TANIMLA oturumAktifMi = YANLIS

  YAZ "Kullanıcı Adını Giriniz:"
  OKU kullaniciAdi

  YAZ "Şifreyi Giriniz:"
  OKU sifre

  // --- Kontrol Noktası: Veritabanı Kullanıcı Doğrulama ---
  KULLANICI = VeritabanindaKullaniciyiBul(kullaniciAdi)

  EGER KULLANICI VARSA ISE
    EGER KULLANICI.sifre == sifre ISE
      oturumAktifMi = DOGRU
      OturumBaslat(KULLANICI.id)
      YAZ "Giriş Başarılı. Hoş Geldiniz, " + KULLANICI.isim
    DEGILSE
      YAZ "Hatalı şifre."
    SON_EGER
  DEGILSE
    YAZ "Kullanıcı bulunamadı."
  SON_EGER

BITIR KULLANICI_GIRISI


// ======================================================
// ADIM 2: ÜRÜN EKLEME
// ======================================================
BASLA URUN_EKLEME

  TANIMLA secilenUrunID, adet
  TANIMLA kullaniciSepeti = MevcutSepetiGetir()

  YAZ "Eklemek istediğiniz ürünün ID'sini girin:"
  OKU secilenUrunID

  YAZ "Adet girin:"
  OKU adet

  // --- Kontrol Noktası: Stok Kontrolü ---
  STOK_KONTROLU_YAP(secilenUrunID, adet) // Bu fonksiyon aşağıda tanımlanmıştır

  EGER StokYeterliMi == DOGRU ISE
    // --- Kontrol Noktası: Ürün Sepette Var mı? ---
    urunSepetteVarMi = HAYIR
    DONGU kullaniciSepeti icindeki her urun ICIN
      EGER urun.id == secilenUrunID ISE
        urun.adet = urun.adet + adet
        urunSepetteVarMi = EVET
        DUR // Döngüyü sonlandır
      SON_EGER
    SON_DONGU

    EGER urunSepetteVarMi == HAYIR ISE
      YENI_URUN = VeritabanindanUrunGetir(secilenUrunID)
      YENI_URUN.adet = adet
      kullaniciSepeti.Ekle(YENI_URUN)
    SON_EGER

    YAZ secilenUrunID + " kodlu ürün sepete eklendi."
    SepetToplaminiGuncelle()
  DEGILSE
    YAZ "Üzgünüz, bu ürün için yeterli stok bulunmamaktadır."
  SON_EGER

BITIR URUN_EKLEME


// ======================================================
// ADIM 3: STOK KONTROLÜ FONKSİYONU
// ======================================================
BASLA STOK_KONTROLU_YAP(urunID, istenenAdet)

  TANIMLA StokYeterliMi = YANLIS
  URUN_STOK = VeritabanindanStokGetir(urunID)

  // --- Kontrol Noktası: Stok Yeterliliği ---
  EGER URUN_STOK >= istenenAdet ISE
    StokYeterliMi = DOGRU
  DEGILSE
    StokYeterliMi = YANLIS
  SON_EGER

BITIR STOK_KONTROLU_YAP


// ======================================================
// ADIM 4: İNDİRİM KODU UYGULAMA
// ======================================================
BASLA INDIRIM_KODU_UYGULA

  TANIMLA girilenKod, sepetToplami
  sepetToplami = MevcutSepetToplaminiGetir()

  YAZ "İndirim kodunuz var mı? Varsa giriniz:"
  OKU girilenKod

  // --- Kontrol Noktası: Kodun Veritabanında Kontrolü ---
  INDIRIM_KUPONU = VeritabanindaKuponuBul(girilenKod)

  EGER INDIRIM_KUPONU VARSA VE INDIRIM_KUPONU.aktifMi == DOGRU ISE
    // --- Kontrol Noktası: Kupon Koşulları ---
    EGER sepetToplami >= INDIRIM_KUPONU.minimumTutar ISE
      indirimMiktari = (sepetToplami * INDIRIM_KUPONU.indirimOrani) / 100
      yeniToplam = sepetToplami - indirimMiktari
      YAZ "İndirim uygulandı. Yeni Toplam: " + yeniToplam
    DEGILSE
      YAZ "Bu kuponu kullanmak için sepet tutarınız yeterli değil."
    SON_EGER
  DEGILSE
    YAZ "Geçersiz veya süresi dolmuş indirim kodu."
  SON_EGER

BITIR INDIRIM_KODU_UYGULA


// ======================================================
// ADIM 5: KARGO HESAPLAMA
// ======================================================
BASLA KARGO_HESAPLAMA

  TANIMLA teslimatAdresi, sepetAgirligi
  TANIMLA kargoUcreti = 0

  YAZ "Teslimat adresinin şehrini giriniz:"
  OKU teslimatAdresi.sehir

  sepetAgirligi = SepettekiUrunlerinAgirliginiHesapla()

  // --- Kontrol Noktası: Adrese ve Ağırlığa Göre Kargo Ücreti ---
  EGER teslimatAdresi.sehir == "İstanbul" ISE
    kargoUcreti = 20
  DEGILSE
    kargoUcreti = 30
  SON_EGER

  EGER sepetAgirligi > 5 ISE
    kargoUcreti = kargoUcreti + 10 // Ek ağırlık ücreti
  SON_EGER
  
  // --- Kontrol Noktası: Ücretsiz Kargo Kampanyası ---
  sepetToplami = MevcutSepetToplaminiGetir()
  EGER sepetToplami > 500 ISE
    kargoUcreti = 0
    YAZ "500 TL üzeri alışverişlerde kargo ücretsiz!"
  SON_EGER
  
  YAZ "Hesaplanan Kargo Ücreti: " + kargoUcreti
  GENEL_TOPLAM = sepetToplami + kargoUcreti
  YAZ "Ödenecek Tutar: " + GENEL_TOPLAM

BITIR KARGO_HESAPLAMA


// ======================================================
// ADIM 6: ÖDEME AŞAMASI
// ======================================================
BASLA ODEME_YAP

  TANIMLA kartNumarasi, sonKullanmaTarihi, CVC
  TANIMLA odemeBasariliMi = YANLIS

  YAZ "Kredi Kartı Numarasını Giriniz:"
  OKU kartNumarasi

  YAZ "Son Kullanma Tarihini Giriniz (AA/YY):"
  OKU sonKullanmaTarihi

  YAZ "CVC Kodunu Giriniz:"
  OKU CVC

  // --- Kontrol Noktası: Kart Bilgilerinin Format Kontrolü ---
  EGER KartBilgileriGecerliMi(kartNumarasi, sonKullanmaTarihi, CVC) ISE

    // --- Kontrol Noktası: Ödeme Ağ Geçidi (Payment Gateway) İletişimi ---
    odemeOnayi = OdemeAgGecidineGonder(kartNumarasi, odenecekTutar)

    EGER odemeOnayi == "BASARILI" ISE
      odemeBasariliMi = DOGRU
      YAZ "Ödeme başarıyla tamamlandı. Siparişiniz oluşturuldu."

      // --- Kontrol Noktası: Stok Güncelleme ve Sipariş Kaydı ---
      DONGU sepetimdeki her urun ICIN
        VeritabanindaStokGuncelle(urun.id, urun.adet)
      SON_DONGU
      SiparisiVeritabaninaKaydet()
      KullaniciyaOnayEpostasiGonder()
      SepetiTemizle()
    DEGILSE
      YAZ "Ödeme reddedildi. Bankanızla iletişime geçin veya bilgileri kontrol edin."
    SON_EGER

  DEGILSE
    YAZ "Geçersiz kart bilgileri girdiniz."
  SON_EGER

BITIR ODEME_YAP
