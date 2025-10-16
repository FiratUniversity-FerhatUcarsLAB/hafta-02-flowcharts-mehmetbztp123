BAŞLA

KULLANICI_ADI, ŞİFRE ← GİRİŞ YAP
EĞER giriş_bilgileri_doğru_mu(KULLANICI_ADI, ŞİFRE) İSE
    DERS_LISTESİ ← Tüm Açılan Dersleri Listele
    SEÇİLEN_DERSLER ← boş liste
    TOPLAM_KREDİ ← 0

    TEKRAR
        DERS ← Kullanıcıdan ders seçimi al
        EĞER ders HALEN SEÇİLEN_DERSLER içinde ise
            DERSİ çıkarmak ister misiniz?
            EĞER EVET ise
                SEÇİLEN_DERSLER'den DERS'i çıkar
                TOPLAM_KREDİ -= DERS.kredi
            DEĞİLSE
                devam et
        DEĞİLSE
            // 1. Kontenjan kontrolü
            EĞER DERS.kontenjan > 0 İSE

                // 2. Ön koşul kontrolü
                EĞER öğrenci DERS.ön_koşul_dersini geçmiş İSE

                    // 3. Zaman çakışması kontrolü
                    EĞER DERS.zaman DİĞER SEÇİLEN_DERSLER ile çakışmıyor İSE

                        // 4. Kredi limiti kontrolü
                        EĞER (TOPLAM_KREDİ + DERS.kredi) ≤ 35 İSE

                            SEÇİLEN_DERSLER'e DERS ekle
                            TOPLAM_KREDİ += DERS.kredi

                        DEĞİLSE
                            UYARI: Kredi limiti aşıldı! (Max 35)
                        SON

                    DEĞİLSE
                        UYARI: Zaman çakışması var!
                    SON

                DEĞİLSE
                    UYARI: Ön koşul dersi geçilmemiş!
                SON

            DEĞİLSE
                UYARI: Ders kontenjanı dolu!
            SON

    DÖNGÜ SON (Kullanıcı "kayıt işlemini bitir" diyene kadar)

    // 5. Danışman onayı kontrolü
    EĞER öğrenci.GPA < 2.5 İSE
        UYARI: Danışman onayı gereklidir.
        DANIŞMAN ONAYI ALINDI MI?
        EĞER HAYIR ise
            UYARI: Kayıt tamamlanamaz. Danışman onayı bekleniyor.
            DUR
        SON
    SON

    // Kayıt özeti ve onaylama
    SEÇİLEN_DERSLER listesini ve toplam krediyi göster
    ÖĞRENCİYE: "Kaydı onaylıyor musunuz?"
    EĞER EVET ise
        Kayıt tamamlandı
    DEĞİLSE
        Kayıt iptal edildi
    SON

DEĞİLSE
    UYARI: Giriş başarısız. Bilgilerinizi kontrol edin.
SON

BİTİR
