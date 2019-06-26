### Android emulátor proxy settings

Android emulátor má dvě možnosti jak nastavit proxy server.

1. Přímo v emulátoru ---> !NEPOUŽÍVAT!

    velmi zabugovaná věc, selhává ve spoustě scénářů. SSL se chová velice divně, sockety jsou ukončované uprostřed komunikace apod.
    
    ![](https://raw.githubusercontent.com/cuberact/cuberact-proxy/master/images/android-proxy/android-proxy-03.png)
    
2. Přímo v AVD (v nastavení uvnitř androidu)
  
    `settings -> Mobile network -> Access point names`

    ![](https://raw.githubusercontent.com/cuberact/cuberact-proxy/master/images/android-proxy/android-proxy-01.png)
    
    ![](https://raw.githubusercontent.com/cuberact/cuberact-proxy/master/images/android-proxy/android-proxy-02.png)
    
    Tato proxy se zdá funguje jak má.

    
Nastavení Android Emulatoru pro MITM [zde](https://github.com/cuberact/cuberact-proxy/blob/master/mitm-android.md)
   