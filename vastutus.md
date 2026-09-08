# Ülesanne 2 - Vastutuse jaotus

## Klassifikatsioon

| #   | Rakenduse osa                             | Klient | Server | Põhjendus                                                                        |
| --- | ----------------------------------------- | ------ | ------ | -------------------------------------------------------------------------------- |
| 1   | Mängukordade nimekirja kuvamine           | Jah    | Jah    | Klient kuvab kasutajale, aga andmed peab andmebaasist turvaliselt andma server.  |
| 2   | Kuupäevavälja vorming                     | Jah    | Ei     | Kliendi poolt mugavuse ja visuaalse vorminduse tagamiseks.                       |
| 3   | "Vabu kohti vähemalt" filtri rakendamine  | Jah    | Jah    | Kliendis kiireks filtreerimiseks, serveris täpseks andmete pärimiseks.           |
| 4   | Kontroll, kas väli on täitmata            | Jah    | Ei     | Kiire tagasiside kasutajale vormi täitmisel (mugavus).                           |
| 5   | Kontroll, kas mängukord on juba täis      | Ei     | Jah    | Kriitiline reegel, mida klient saab petta – server peab rangelt kaitsma.         |
| 6   | Kontroll, kas kasutaja on sisse logitud   | Ei     | Jah    | Turvalisuse tagamiseks peab server alati kontrollima sessiooni/tokenit.          |
| 7   | Kontroll, kas kasutaja on admin           | Ei     | Jah    | Admin-õiguste kontroll peab toimuma serveris, et vältida volitamata juurdepääsu. |
| 8   | Paarikutse saatmine                       | Ei     | Jah    | Ärifunktsioon, mida server peab valideerima ja andmebaasi salvestama.            |
| 9   | Mängutabeli genereerimine ringmeetodiga   | Ei     | Jah    | Keeruline äriloogika, mis tehakse turvaliselt serveri poolel.                    |
| 10  | Edetabeli arvutamine                      | Ei     | Jah    | Andmete töötlemine ja arvutamine peab toimuma serveris.                          |
| 11  | Tulemuste sortimine juba laaditud tabelis | Jah    | Ei     | Kasutajaliideses kiiruse ja mugavuse huvides.                                    |
| 12  | Andmebaasi parool                         | Ei     | Jah    | Paroolid ja salajased võtmed asuvad rangelt ainult serveri keskkonnas.           |

## Kolm asja, mis ei tohi kunagi kliendile jõuda

1. **Andmebaasi parool ja ühendusandmed** – kui kasutaja seda näeb, saab ta täieliku ligipääsu kogu andmebaasile.
2. **Administraatori õiguste andmise kood/funktsioon** – klient ei otsusta, kas kasutaja on admin, muidu igaüks saaks teha endale admin-õigused.
3. **Äriloogika salajased algoritmilised reeglid** – ärisaladused ja turvakontrollide täpne sisu peavad olema peidus serveris.

## Kolm rünnakut (kui kontroll on tehtud ainult kliendis)

### 1. Andmete manipuleerimine (Client Tampering / Bypassing UI)
* **Kuidas juhtub:** Kasutaja muudab brauseris (näiteks DevTools / F12 abil) või saadab otse päringu kaudu andmeid (näiteks muudab hinna nulliks või sisestab negatiivse vanuse), kuna arendaja pani piirangu ainult brauseri JavaScripti tasemel.
* **Miks see on katastroof:** Server usub pimesi kliendilt tulelnud infot ja salvestab rikutud andmed andmebaasi.
* **Kuidas serveripoolne kontroll aitab:** Server ignoreerib brauseri reegleid, arvutab väärtused ise ja teostab ranged valideerimised enne andmete salvestamist.

### 2. Topeltbroneerimine (Race Condition)
* **Kuidas juhtub:** Kaks kasutajat klikivad samal ajal viimasele vabale kohale. Kui kogu loogika toimib ainult kliendis, näevad mõlemad kasutajad edukat broneeringut, kuigi vaba koht oli vaid üks.
* **Miks see on katastroof:** Tekib konflikt ja andmete ebakooskõla (ühele kohale registreerub mitu inimest).
* **Kuidas serveripoolne kontroll aitab:** Server ja andmebaas töötlevad päringuid transaktsioonide ning lukustuste abil rangelt ükshaaval, tagades, et teine kasutaja saab kohe teate koha täituvusest.

### 3. Peidetud funktsioonide ja õiguste ärakasutamine (Privilege Escalation)
* **Kuidas juhtub:** Arendaja peidab nupu „Kustuta kasutaja“ või „Tee adminiks“ tavalise kasutaja liidesest ära, eeldades, et kui nuppu pole näha, siis keegi seda ei kasuta. Ründaja saadab aga vastava käsu/URL-i serverile otse konsoolist.
* **Miks see on katastroof:** Server ei kontrolli päringu saatja tegelikke õigusi ja annab esimesele ette tulnud kasutajale täieliku kontrolli süsteemi üle.
* **Kuidas serveripoolne kontroll aitab:** Server kontrollib iga kriitilise päringu saabumisel alati turvasessiooni ja andmebaasi kirjet (nt kas `is_admin == true`), lükates volitamata päringud koheselt tagasi.