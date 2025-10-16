BASLA Kullanıcı_Girisi
  EGER KullaniciGirisYapti ISE
    KullaniciBilgileriniGetir()
  DEĞILSE
    MisafirAlisverisiIzinVer()
BİTİR

BASLA Sepet_Yonetimi
  DONG (KullaniciUrunEklemekIstiyor ISE)
    Ürün = ÜrünSec()
    EGER StoktaVar(Ürün) ISE
      SepeteEkle(Ürün)
    DEĞILSE
      UyariMesaji("Stokta yeterli ürün yok")
    BitirEger
  BitirDong
BİTİR

BASLA Indirim_Kodu_Kontrolu
  EGER IndirimKoduGirildi ISE
    EGER IndirimKoduGecerli ISE
      SepeteIndirimUygula(IndirimKodu)
    DEĞILSE
      UyariMesaji("Indirim kodu gecersiz")
    BitirEger
  BitirEger
BİTİR

BASLA Kargo_Hesaplama
  TeslimatAdresi = AdresAl()
  KargoUcreti = KargoUcretiniHesapla(TeslimatAdresi)
  SepeteKargoEkle(KargoUcreti)
BİTİR

BASLA Odeme_Islemi
  EGER SepetBosDegil ISE
    OdemeYontemi = OdemeYontemiSec()
    OdemeBilgileri = BilgiGir(OdemeYontemi)
    EGER OdemeGecerli(OdemeBilgileri) ISE
      OdemeYap(OdemeBilgileri)
      EGER OdemeBasarili ISE
        SiparisiKaydet()
        StokGuncelle()
        SiparisOnayiGoster()
      DEĞILSE
        UyariMesaji("Ödeme başarısız")
      BitirEger
    DEĞILSE
      UyariMesaji("Geçersiz ödeme bilgileri")
    BitirEger
  DEĞILSE
    UyariMesaji("Sepet boş")
  BitirEger
BİTİR
