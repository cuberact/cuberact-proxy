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
    openssl x509 -inform PEM -text -in cuberact-proxy-root-cert.pem  -out 453ba6c0.0
    ```

    Výsledný soubor `453ba6c0.0` by měl vypadat takhle:
    
    ```text
    -----BEGIN CERTIFICATE-----
    MIIDgjCCAmqgAwIBAgIEWOA37jANBgkqhkiG9w0BAQsFADBPMRcwFQYDVQQDDA5D
    dWJlcmFjdC1wcm94eTEXMBUGA1UECgwOQ3ViZXJhY3QtcHJveHkxDjAMBgNVBAcM
    BUN6ZWNoMQswCQYDVQQGEwJDWjAgFw0xODEyMzEyMzAwMDBaGA8yMDk5MTIzMTIz
    MDAwMFowTzEXMBUGA1UEAwwOQ3ViZXJhY3QtcHJveHkxFzAVBgNVBAoMDkN1YmVy
    YWN0LXByb3h5MQ4wDAYDVQQHDAVDemVjaDELMAkGA1UEBhMCQ1owggEiMA0GCSqG
    SIb3DQEBAQUAA4IBDwAwggEKAoIBAQCLZfMDgAr4S0NuQ7h6fbZrwZvNKEPyayVs
    ac8+gi+2pVPBpPmNM/oNMoMWaFW1BUXaxanPeOD9WD3ES2dA/LaHYL0LqEJQG6gS
    35yovxEo4ZO8gKplDYfvuhwHlZz3E0XJV8oMLqPPNIN+k3J1F+vzGXshwsN0wBb/
    5MKIG3B+/sBzoF9JJ+srkgjdqM/p+uyOqoySfoldAEHojlE19ukDhg2vYfQSGveD
    4xZcGxYaEnfhgtwSHXMRFuue14z2yuaeYY0FA+AE6g2DdHr30AHxmiwWBKwvN+Cj
    gJBqo9xJLRSwbVKVYt8LGhAa6mvfjC3H5h87tzEa8yldzaJkhkNrAgMBAAGjZDBi
    MB0GA1UdDgQWBBQwY9r415rIpIUy8iLojEx449I5CTAPBgNVHRMBAf8EBTADAQH/
    MAsGA1UdDwQEAwIBtjAjBgNVHSUEHDAaBgRVHSUABggrBgEFBQcDAQYIKwYBBQUH
    AwIwDQYJKoZIhvcNAQELBQADggEBAIN3cDG6N7hmcqQ5Y73reyKDD1RGMe00pu6y
    /2BiYRIedPCex2xZ6s7gyrm0Fvh6lg/aeWhpjw7kRNI9daPc9L7TzHgtHO1sZMD2
    lJNvB0u+RxnXbwIB/q2TStrsRvIHnpUZQ+ILyZTiWG8CmiqBWNUah3W47PIYtvCv
    xi7IGw/cw+/0zBsvolD5QGuGCtUt1V17G/95kn8PXcu4PNhqye8KLuOb4pjv11z4
    sWWa/fRkDgeQvbq18CzxkPi1v7Ed2HjCPcirFsGDZai0GoyZ96eTZyo2upq6Pe7g
    9n16fLOUZo88a6G5Uqb0OuR6zoboNtS9zScsLmdWHgLMCWeduYM=
    -----END CERTIFICATE-----
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number: 1491089390 (0x58e037ee)
            Signature Algorithm: sha256WithRSAEncryption
            Issuer: CN = Cuberact-proxy, O = Cuberact-proxy, L = Czech, C = CZ
            Validity
                Not Before: Dec 31 23:00:00 2018 GMT
                Not After : Dec 31 23:00:00 2099 GMT
            Subject: CN = Cuberact-proxy, O = Cuberact-proxy, L = Czech, C = CZ
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    RSA Public-Key: (2048 bit)
                    Modulus:
                        00:8b:65:f3:03:80:0a:f8:4b:43:6e:43:b8:7a:7d:
                        b6:6b:c1:9b:cd:28:43:f2:6b:25:6c:69:cf:3e:82:
                        2f:b6:a5:53:c1:a4:f9:8d:33:fa:0d:32:83:16:68:
                        55:b5:05:45:da:c5:a9:cf:78:e0:fd:58:3d:c4:4b:
                        67:40:fc:b6:87:60:bd:0b:a8:42:50:1b:a8:12:df:
                        9c:a8:bf:11:28:e1:93:bc:80:aa:65:0d:87:ef:ba:
                        1c:07:95:9c:f7:13:45:c9:57:ca:0c:2e:a3:cf:34:
                        83:7e:93:72:75:17:eb:f3:19:7b:21:c2:c3:74:c0:
                        16:ff:e4:c2:88:1b:70:7e:fe:c0:73:a0:5f:49:27:
                        eb:2b:92:08:dd:a8:cf:e9:fa:ec:8e:aa:8c:92:7e:
                        89:5d:00:41:e8:8e:51:35:f6:e9:03:86:0d:af:61:
                        f4:12:1a:f7:83:e3:16:5c:1b:16:1a:12:77:e1:82:
                        dc:12:1d:73:11:16:eb:9e:d7:8c:f6:ca:e6:9e:61:
                        8d:05:03:e0:04:ea:0d:83:74:7a:f7:d0:01:f1:9a:
                        2c:16:04:ac:2f:37:e0:a3:80:90:6a:a3:dc:49:2d:
                        14:b0:6d:52:95:62:df:0b:1a:10:1a:ea:6b:df:8c:
                        2d:c7:e6:1f:3b:b7:31:1a:f3:29:5d:cd:a2:64:86:
                        43:6b
                    Exponent: 65537 (0x10001)
            X509v3 extensions:
                X509v3 Subject Key Identifier: 
                    30:63:DA:F8:D7:9A:C8:A4:85:32:F2:22:E8:8C:4C:78:E3:D2:39:09
                X509v3 Basic Constraints: critical
                    CA:TRUE
                X509v3 Key Usage: 
                    Digital Signature, Key Encipherment, Data Encipherment, Certificate Sign, CRL Sign
                X509v3 Extended Key Usage: 
                    Any Extended Key Usage, TLS Web Server Authentication, TLS Web Client Authentication
        Signature Algorithm: sha256WithRSAEncryption
             83:77:70:31:ba:37:b8:66:72:a4:39:63:bd:eb:7b:22:83:0f:
             54:46:31:ed:34:a6:ee:b2:ff:60:62:61:12:1e:74:f0:9e:c7:
             6c:59:ea:ce:e0:ca:b9:b4:16:f8:7a:96:0f:da:79:68:69:8f:
             0e:e4:44:d2:3d:75:a3:dc:f4:be:d3:cc:78:2d:1c:ed:6c:64:
             c0:f6:94:93:6f:07:4b:be:47:19:d7:6f:02:01:fe:ad:93:4a:
             da:ec:46:f2:07:9e:95:19:43:e2:0b:c9:94:e2:58:6f:02:9a:
             2a:81:58:d5:1a:87:75:b8:ec:f2:18:b6:f0:af:c6:2e:c8:1b:
             0f:dc:c3:ef:f4:cc:1b:2f:a2:50:f9:40:6b:86:0a:d5:2d:d5:
             5d:7b:1b:ff:79:92:7f:0f:5d:cb:b8:3c:d8:6a:c9:ef:0a:2e:
             e3:9b:e2:98:ef:d7:5c:f8:b1:65:9a:fd:f4:64:0e:07:90:bd:
             ba:b5:f0:2c:f1:90:f8:b5:bf:b1:1d:d8:78:c2:3d:c8:ab:16:
             c1:83:65:a8:b4:1a:8c:99:f7:a7:93:67:2a:36:ba:9a:ba:3d:
             ee:e0:f6:7d:7a:7c:b3:94:66:8f:3c:6b:a1:b9:52:a6:f4:3a:
             e4:7a:ce:86:e8:36:d4:bd:cd:27:2c:2e:67:56:1e:02:cc:09:
             67:9d:b9:83
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
    adb push 453ba6c0.0 /system/etc/security/cacerts/
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
