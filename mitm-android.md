### MITM - Android Emulátor

S android emulátorem je několik problémů. Cílem je nahrát do spuštěného AVD root certifikát z cuberact-proxy a to tak aby se zařadil mezi ostatní root CA.

#### Problém 1. Android emulator

Dále v návodu si připravíme certifikát, který potom budeme chtít nahrát mezi ostatní v AVD do složky `/system/etc/security/cacerts`. Android emulátor ale tento adresář vrací do výchozího stavu při každém restartu AVD.
Je tedy třeba zajistit aby nahraný certifikát přežil restart AVD i emulátoru.
Toho docílíme tak, že zajistíme aby se emulátor vždy spouštěl s parametrem `-writable-system`

Nepřišel jsem na lepší řešení než:

Android studio ve svém AVD manageru spouští soubor `~/Android/Sdk/emulator/emulator` a toho využijeme.
Tento soubor si přejmenujeme na `emulator-orig` a vytvoříme fake soubor `emulator`, což bude script, který nám zajistí přidání potřebného parametru.

V novém souboru `emulator` tedy bude:

```batch
 #!/bin/bash
~/Android/Sdk/emulator/emulator-orig -writable-system $@
```

Nastavte souboru `emulator` práva na spuštění
```batch
chmod +x emulator
```

Nyní si emulátor všechny soubory přidané do `/system` bude ukládat do svých souborů a tak přežijí restart AVD. Pro zvídavé jde o file `~/.android/avd/Pixel8.avd/system.img.qcow2`

#### Problém 2. Konverze root cert PEM file 

Z cuberact proxy máme uložený root certifikát jako PEM file ([Save root certificate](https://github.com/cuberact/cuberact-proxy/blob/master/save-root-certificate.md)). Pro Android je potřeba připravit trošku upravený formát.

1. Otevřeme příkazovou řádku v adresáři s uloženým PEM file.
2. Je třeba nejprve zjistit jak se bude soubor certifátu jmenovat

    ```batch 
    openssl x509 -inform PEM -subject_hash_old -in cuberact-proxy-root-cert.pem | head -1
    ```
    tento příkaz vypíše hash/název budoucího souboru
    
3. Převést PEM file na formát pořebný pro android (název souboru z kroku 2 + přípona .0)

    ```batch 
    openssl x509 -inform PEM -text -in cuberact-proxy-root-cert.pem  -out 7636ecd9.0
    ```

    Výsledný soubor `7636ecd9.0` by měl vypadat takhle:
    
    ```text
    -----BEGIN CERTIFICATE-----
    MIIDgDCCAmigAwIBAgIEKwjEqzANBgkqhkiG9w0BAQsFADBPMRcwFQYDVQQDDA5D
    dWJlcmFjdC1Qcm94eTEXMBUGA1UECgwOQ3ViZXJhY3QtUHJveHkxDjAMBgNVBAcM
    BWN6ZWNoMQswCQYDVQQGEwJjejAeFw0xOTA2MDkyMTU2NTBaFw0yMTA1MTkyMTU2
    NTBaME8xFzAVBgNVBAMMDkN1YmVyYWN0LVByb3h5MRcwFQYDVQQKDA5DdWJlcmFj
    dC1Qcm94eTEOMAwGA1UEBwwFY3plY2gxCzAJBgNVBAYTAmN6MIIBIjANBgkqhkiG
    9w0BAQEFAAOCAQ8AMIIBCgKCAQEAu3ChcQeUz8v6Rijou1ZGHKdaGJ2Vo4cV0QLn
    YTC/bTbdo52c9Bre6+mXsE1AeTCoFNk4sic0aZ5RzWHG5i3vrjYtj+LULa7VJx72
    gbNqGYPwrZIDk7MvOlCUbVp42yWHhClsQ+5HFC+pM9yghtIxK/1BWkyJTqD2F7Mo
    Esce9lMxdlH24arSo/G1dh0dWeXajaExKB4299VMI7jC9uU7rWHL+pZ+oiRAaeyd
    DoYn+wN0L9YyARxZVv3R0p/SHsDhq0IDimdEdp+g/F3Ry91QW3GJKTwJXN9vOHnN
    M8wu8CNnjtbXWFp6unSqliwUdWWwUdig4G4jnxzFUKYnE3MK+QIDAQABo2QwYjAd
    BgNVHQ4EFgQUaSe5cI9QHB+yFJbABtpP2qOzLJwwDwYDVR0TAQH/BAUwAwEB/zAL
    BgNVHQ8EBAMCAbYwIwYDVR0lBBwwGgYEVR0lAAYIKwYBBQUHAwEGCCsGAQUFBwMC
    MA0GCSqGSIb3DQEBCwUAA4IBAQAFz4KDm9EfeJ/vVGR4QcYkI+swdyefisEeKayH
    4GS+yJ7FPuGR1x43nsT8CTEhcKUE+E08adY56jzrQ5gKbJ5dnwZ+k3kIIMB2rICc
    TVXNTYCY0VQctj+hG9DhZ4DI4FRFow4D/gNk+FxdA9G4GT+2REqb5BbauWm2hVHR
    9L+A9lj07uhnMBKmyLZRVVR+aGM9slI1d2FCMRau8TADplNJNsPHZnQAZEXG5YKE
    8+sgCe/Fbl0Dg1H8L8kr0xFIIh9XCvMgyXDYB6+QZneu3AFMwObwsCOz7bZVyo6i
    372OmhurAVbAAYLWFsP7DHffV9uZvsaEm+90aZUx00JogawT
    -----END CERTIFICATE-----
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number: 721994923 (0x2b08c4ab)
            Signature Algorithm: sha256WithRSAEncryption
            Issuer: CN = Cuberact-Proxy, O = Cuberact-Proxy, L = czech, C = cz
            Validity
                Not Before: Jun  9 21:56:50 2019 GMT
                Not After : May 19 21:56:50 2021 GMT
            Subject: CN = Cuberact-Proxy, O = Cuberact-Proxy, L = czech, C = cz
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    RSA Public-Key: (2048 bit)
                    Modulus:
                        00:bb:70:a1:71:07:94:cf:cb:fa:46:28:e8:bb:56:
                        46:1c:a7:5a:18:9d:95:a3:87:15:d1:02:e7:61:30:
                        bf:6d:36:dd:a3:9d:9c:f4:1a:de:eb:e9:97:b0:4d:
                        40:79:30:a8:14:d9:38:b2:27:34:69:9e:51:cd:61:
                        c6:e6:2d:ef:ae:36:2d:8f:e2:d4:2d:ae:d5:27:1e:
                        f6:81:b3:6a:19:83:f0:ad:92:03:93:b3:2f:3a:50:
                        94:6d:5a:78:db:25:87:84:29:6c:43:ee:47:14:2f:
                        a9:33:dc:a0:86:d2:31:2b:fd:41:5a:4c:89:4e:a0:
                        f6:17:b3:28:12:c7:1e:f6:53:31:76:51:f6:e1:aa:
                        d2:a3:f1:b5:76:1d:1d:59:e5:da:8d:a1:31:28:1e:
                        36:f7:d5:4c:23:b8:c2:f6:e5:3b:ad:61:cb:fa:96:
                        7e:a2:24:40:69:ec:9d:0e:86:27:fb:03:74:2f:d6:
                        32:01:1c:59:56:fd:d1:d2:9f:d2:1e:c0:e1:ab:42:
                        03:8a:67:44:76:9f:a0:fc:5d:d1:cb:dd:50:5b:71:
                        89:29:3c:09:5c:df:6f:38:79:cd:33:cc:2e:f0:23:
                        67:8e:d6:d7:58:5a:7a:ba:74:aa:96:2c:14:75:65:
                        b0:51:d8:a0:e0:6e:23:9f:1c:c5:50:a6:27:13:73:
                        0a:f9
                    Exponent: 65537 (0x10001)
            X509v3 extensions:
                X509v3 Subject Key Identifier: 
                    69:27:B9:70:8F:50:1C:1F:B2:14:96:C0:06:DA:4F:DA:A3:B3:2C:9C
                X509v3 Basic Constraints: critical
                    CA:TRUE
                X509v3 Key Usage: 
                    Digital Signature, Key Encipherment, Data Encipherment, Certificate Sign, CRL Sign
                X509v3 Extended Key Usage: 
                    Any Extended Key Usage, TLS Web Server Authentication, TLS Web Client Authentication
        Signature Algorithm: sha256WithRSAEncryption
             05:cf:82:83:9b:d1:1f:78:9f:ef:54:64:78:41:c6:24:23:eb:
             30:77:27:9f:8a:c1:1e:29:ac:87:e0:64:be:c8:9e:c5:3e:e1:
             91:d7:1e:37:9e:c4:fc:09:31:21:70:a5:04:f8:4d:3c:69:d6:
             39:ea:3c:eb:43:98:0a:6c:9e:5d:9f:06:7e:93:79:08:20:c0:
             76:ac:80:9c:4d:55:cd:4d:80:98:d1:54:1c:b6:3f:a1:1b:d0:
             e1:67:80:c8:e0:54:45:a3:0e:03:fe:03:64:f8:5c:5d:03:d1:
             b8:19:3f:b6:44:4a:9b:e4:16:da:b9:69:b6:85:51:d1:f4:bf:
             80:f6:58:f4:ee:e8:67:30:12:a6:c8:b6:51:55:54:7e:68:63:
             3d:b2:52:35:77:61:42:31:16:ae:f1:30:03:a6:53:49:36:c3:
             c7:66:74:00:64:45:c6:e5:82:84:f3:eb:20:09:ef:c5:6e:5d:
             03:83:51:fc:2f:c9:2b:d3:11:48:22:1f:57:0a:f3:20:c9:70:
             d8:07:af:90:66:77:ae:dc:01:4c:c0:e6:f0:b0:23:b3:ed:b6:
             55:ca:8e:a2:df:bd:8e:9a:1b:ab:01:56:c0:01:82:d6:16:c3:
             fb:0c:77:df:57:db:99:be:c6:84:9b:ef:74:69:95:31:d3:42:
             68:81:ac:13
    ```
    Poznámka Pokud jsou v souboru sekce v jiném pořadí tak je přesuňte v textovém editoru
    
#### Problém 3. Nahrání souboru do AVD

Spusťte android emulátor s příslušným AVD. Jako AVD volte vždy image/target označený jako `Google APIs`, to proto, že jinak bude AVD jako produkční a nepodaří se vám `adb root` a `adb remount`. Postup jsem odzkoušel na verzích Android 8 a Android 9+. S verzí Android 6 jsem měl problémy, ale možná i zde to bude fungovat.


1. přepnutí `adb` do root režimu

    ```batch 
    adb root
    ```
2. zpřístupnění zápisu do systémových složek

    ```batch
    adb remount 
    ```
3. Zkopírování souboru na příslušné místo

    ```batch
    adb push 7636ecd9.0 /system/etc/security/cacerts/
    ```
4. Restart AVD

    ```batch
    adb reboot
    ```
AVD v emulátoru by se mělo restartovat a poté zkontrolujte přítomnost certifikátu:

`settings -> Security & locations -> Encryption & credentials -> Trusted credentials (tab system) `

![](https://raw.githubusercontent.com/cuberact/cuberact-proxy/master/images/mitm-android/mitm-android-01.png)

A to je vše, nyní bude Android emulátor s přislušným AVD věřit cuberact-proxy.

Nastavení Android proxy [zde](https://github.com/cuberact/cuberact-proxy/blob/master/android-proxy-settings.md)
