BASLA

    TANIMLA dogruPIN ← "1234"
    TANIMLA bakiye ← 1500
    TANIMLA gunlukLimit ← 1000
    TANIMLA gunlukCekilen ← 0
    TANIMLA pinHak ← 3

    DONGU  # Ana işlem döngüsü

        # PIN DOĞRULAMA
        DONGU
            EGER pinHak = 0 ISE
                YAZ "PIN hakkiniz bitti. Kart bloke edildi."
                CIK
            BITIR_EGER

            YAZ "Lütfen 4 haneli PIN'inizi giriniz:"
            OKU girilenPIN

            EGER girilenPIN = dogruPIN ISE
                YAZ "PIN doğru. Giriş başarılı."
                CIK
            DEGILSE
                pinHak ← pinHak - 1
                YAZ "Yanlış PIN. Kalan hak: " + pinHak
            BITIR_EGER
        BITIR_DONGU

        # PARA ÇEKME İŞLEMİ
        DONGU
            YAZ "Çekmek istediğiniz tutarı giriniz (20 TL ve katı):"
            OKU cekilecekTutar

            EGER cekilecekTutar <= 0 ISE
                YAZ "Geçersiz tutar. Pozitif bir miktar giriniz."
                DEVAM
            BITIR_EGER

            EGER cekilecekTutar MOD 20 ≠ 0 ISE
                YAZ "Tutar 20 TL'nin katı olmalıdır."
                DEVAM
            BITIR_EGER

            EGER cekilecekTutar > bakiye ISE
                YAZ "Yetersiz bakiye. Mevcut bakiye: " + bakiye + " TL"
                DEVAM
            BITIR_EGER

            EGER gunlukCekilen + cekilecekTutar > gunlukLimit ISE
                kalanLimit ← gunlukLimit - gunlukCekilen
                YAZ "Günlük çekim limitinizi aştınız."
                YAZ "Kalan günlük limit: " + kalanLimit + " TL"
                DEVAM
            BITIR_EGER

            # İşlem onayı
            YAZ "İşlemi onaylıyor musunuz? (E/H)"
            OKU onay

            EGER onay = "E" VEYA onay = "e" ISE
                bakiye ← bakiye - cekilecekTutar
                gunlukCekilen ← gunlukCekilen + cekilecekTutar
                YAZ "Lütfen paranızı alınız: " + cekilecekTutar + " TL"
                YAZ "Kalan bakiye: " + bakiye + " TL"
                YAZ "Kalan günlük limit: " + (gunlukLimit - gunlukCekilen) + " TL"
            DEGILSE
                YAZ "İşlem iptal edildi."
            BITIR_EGER

            # Tekrar işlem sorusu
            YAZ "Yeni bir işlem yapmak ister misiniz? (E/H)"
            OKU tekrar

            EGER tekrar = "E" VEYA tekrar = "e" ISE
                DEVAM
            DEGILSE
                YAZ "Kartınızı almayı unutmayınız. Hoşça kalın."
                CIK
            BITIR_EGER

        BITIR_DONGU

    BITIR_DONGU

BITIR
