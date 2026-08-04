# Home Lab 2. rész: Domain Login & Group Policy Management

<img src="https://i.imgur.com/EoXkdxQ.png" height="80%" width="100%" alt="GPO"/>

## Bevezetés
Ez az útmutató az Active Directory lab beállításait folytatja, a tartományi bejelentkezési problémák elhárítására, valamint a Windows 10-es kliens (Client1) központosított kezelését szolgáló Group Policy Object-ek(GPO) alkalmazására összpontosítva.

## 1. lépés - Domain bejelentkezési hiba elhárítása

### Probléma összefoglalása
   - A Client1 (Windows 10, tartományhoz csatlakoztatott) nem tudta hitelesíteni a tartományi felhasználókat.
   - A Client1 hiányzott az Active Directory „Computers” tárolójából.

### Megoldási lépések
1. **Ellenőrzött felhasználói fiók** az Active Directoryban:
   - A hitelesítő adatok és a tartománynév formátuma megfelelőnek bizonyult
2. **Az AD „Computers” tárolójának ellenőrzése**:
   - A Client1 nem szerepelt a listában, ami arra utalt, hogy a tartományhoz való csatlakoztatás nem sikerült megfelelően
3. **A Client1 bejelentkezési képernyőjén**:
   - Kiválasztva az "Other user" lehetőséget → a tartománynév helyesen jelent meg
   - Bejelentkezés helyi rendszergazdai fiókkal

### Megoldás-implementáció
1. **Client1 leválasztása a tartományról**:
   - Beállítások > Rendszer > Névjegy > A számítógép átnevezése (speciális)
   - Módosítva munkacsoportra, majd újraindítva
2. **Client1 újbóli csatlakoztatása a tartományhoz**:
   - Megismételtem a lépéseket, megadtam a tartományt: mydomain.com
   - A kérésnek megfelelően megadtam a tartományi rendszergazdai hitelesítő adatokat
   - Újraindítottam a gépet
3. **Ellenőrzés**:
   - A „Client1” immár megjelent az AD „Computers” tárolójában
   - Sikeres bejelentkezés tartományi felhasználói fiókkal

### Gyökérok-elemzés
   Valószínű okok:
  - Hiányos/sérült kezdeti tartományhoz csatlakozás
  - Megszakadt biztonságos csatorna a Client1 és a DC között

## 2. lépés - Group Policy beállítása

<img src="https://i.imgur.com/0zyRyhZ.png" height="80%" width="100%" alt="GPO"/>

### Group Policy Management telepítése
1. **Server Manager**.
2. **Manage > Add Roles and Features**
3. **Features > Group Policy Management**
4. **Install**

### Organizational Unit létrehozása
1. **Group Policy Management Console**
2. domain → **New Organizational Unit**
3. **TestComputers**
4. Áthelyeztem a Client1-et a „Computers” tárolóból az új OU-ba.


## 3. lépés - Asztali háttérképre vonatkozó irányelvek végrehajtása

<img src="https://i.imgur.com/xUBIESZ.png" height="80%" width="100%" alt="GPO"/>

### Megosztott mappa beállítása
1. Mappa létrehozása: `C:\Wallpapers`
2. Háttérkép hozzáadása: `Magyarország.jpg`
3. Megosztás konfigurálása:
   - Megosztás neve: Wallpapers
   - Jogosultságok:
     - „Everyone” eltávolítva
     - „Domain Computers” hozzáadva (olvasási jog)
   - UNC-útvonal: `\\DC\Wallpapers\Magyarország.jpg`

### GPO létrehozása
1. **TestComputers OU** → **Create a GPO**
2. Név: **Set Desktop Wallpaper**
3. Konfiguráció:
   - User Configuration > Policies > Admin Templates > Desktop > Desktop Wallpaper
   - Enabled policy
   - Set wallpaper path to UNC
   - Style: Fill

### Hibaelhárítás
- Kezdeti hiba jogosultságok miatt → Hozzáadva: <b>Authenticated Users<b>
- Házirend nem érvényesül → Engedélyezve: <b>Loopback Processing (Merge mode)<b>
- Végső ellenőrzés: Háttérkép alkalmazva a `gpupdate /force` parancs és az újraindítás után

<img src="https://i.imgur.com/Mdu8pDz.png" height="80%" width="100%" alt="GPO"/>

## 4. lépés - USB-korlátozási házirend bevezetése

### GPO-konfiguráció
1. Létrehozott új GPO: **Disable USB Storage**
2. Konfiguráció:
   - Computer Configuration > Policies > Admin Templates > System > Removable Storage Access
   - Enabled:
     - Deny execute access
     - Deny read access
     - Deny write access

### Tesztelés
- Megerősítve:
  - USB-s adattároló eszközök letiltva
  - A HID-eszközök (egerek/billentyűzetek) továbbra is működnek.

<img src="https://i.imgur.com/wtWxUhZ.png" height="80%" width="100%" alt="GPO"/>

## 5. lépés - A Chrome telepítése GPO-n keresztül

### Előkészítés
1. Chrome MSI letöltése: [Chrome Enterprise](https://chromeenterprise.google/browser/download/)
2. Megosztott mappa:
   - Útvonal: `C:\Software\Chrome`
   - Megosztva mint`\\DC\Software` olvasási jogosultsággal a következő számára:
     - <b>Domain Computers<b>
     - <b>Authenticated Users<b>

### GPO-implementáció
1. Létrehozott új GPO: **Deploy Chrome**
2. Konfiguráció:
   - Computer Configuration > Policies > Software Settings > Software Installation
   - Csomag hozzáadva: `\\DC\Software\Chrome\GoogleChromeStandaloneEnterprise64.msi`

### Hibaelhárítás
1. Kezdeti jogosultsági problémák → Security beállítások ellenőrzése
2. UNC-útvonal hiba → Csomag újbóli létrehozása a helyes útvonallal
3. Végső ellenőrzés: A Chrome telepítése házirend-frissítés és újraindítás után

<img src="https://i.imgur.com/9aNjKj0.png" height="80%" width="100%" alt="GPO"/>

## 6. lépés - Jelszó Policy Létrehozása

1. Létrehozott új GPO: **Password Policy** 
2. Konfiguráció:
   - Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Password Policy

<img src="https://i.imgur.com/mmzZCrO.png" height="80%" width="100%" alt="GPO"/>

### Gyakori problémák
- **Nem érvényesülnek a GPO-k?**
  - Ellenőrizd a <b>„Loopback Processing<b> beállítást
  - Ellenőrizd a megfelelő OU-ba való besorolást
  - Futtasd a `gpupdate /force` parancsot, és indítsa újra a gépet
- **Jogosultsági hibák?**
  - Győződj meg arról, hogy a <b>„Domain Computers”<b> és az <b>„Authenticated Users”<b> csoportok megfelelő hozzáféréssel rendelkeznek
  - Ellenőrizd mind a megosztási, mind a biztonsági jogosultságokat
- **Nem telepíthető a szoftver?**
  - Ellenőrizd a UNC-útvonal elérhetőségét
  - Részletes hibaüzenetekért tekintsd meg az <b>Event Viewer-t<b>
  
