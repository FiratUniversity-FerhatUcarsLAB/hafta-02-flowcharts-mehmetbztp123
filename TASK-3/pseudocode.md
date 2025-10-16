Başla

// 1. Kimlik Doğrulama
kimlik = kullanıcıdanKimlikAl()
eğer kimlikDoğruMu(kimlik) değilse
    "Kimlik doğrulama başarısız." yazdır
    işlemiSonlandır()
sonrası

// 2. Kullanıcı işlem seçimi
işlem = kullanıcıdanİşlemSeçimiAl()  // "Randevu" veya "Tahlil Sonucu"

// 3. İşlem bazında akış

eğer işlem == "Randevu" ise
    // Poliklinik Seçimi
    poliklinik = poliklinikSeç()

    // Doktor Listesi Görüntüleme
    doktorlar = doktorListesiniGetir(poliklinik)
    doktor = doktorSeç(doktorlar)

    // Uygun Saatleri Gösterme
    uygunSaatler = uygunSaatleriGetir(doktor)
    eğer uygunSaatler boşsa
        "Uygun saat bulunamadı." yazdır
        işlemiSonlandır()
    sonrası
    seçilenSaat = saatSeç(uygunSaatler)

    // Randevu Onaylama
    onay = randevuOnayla(kimlik, doktor, seçilenSaat)
    eğer onay başarılıysa
        // SMS Gönderme
        smsGönder(kimlik, doktor, seçilenSaat)
        "Randevu başarıyla oluşturuldu ve SMS gönderildi." yazdır
    değilse
        "Randevu oluşturulamadı." yazdır
    sonrası

değilse eğer işlem == "Tahlil Sonucu" ise
    // Tahlil Varlığı Kontrolü
    tahliller = tahlilVarMi(kimlik)
    eğer tahliller boşsa
        "Kayıtlı tahlil bulunamadı." yazdır
        işlemiSonlandır()
    sonrası

    // Sonuç Hazır mı Kontrolü
    sonucHazir = sonucHazirMi(tahliller)
    eğer sonucHazir değilse
        "Sonuç henüz hazırlanmadı, lütfen daha sonra tekrar deneyiniz." yazdır
        işlemiSonlandır()
    sonrası

    // Sonucu Görüntüleme
    sonuc = tahlilSonucunuGetir(tahliller)
    "Sonuçlarınız:" yazdır
    sonucGöster(sonuc)

    // PDF İndirme
    pdfIndir = kullanıcıdanPDFIndirmeTercihiAl()
    eğer pdfIndir ise
        pdfDosyasi = pdfOlustur(sonuc)
        pdfIndir(pdfDosyasi)
    sonrası

değilse
    "Geçersiz işlem seçimi." yazdır
    işlemiSonlandır()
sonrası

Bitir
