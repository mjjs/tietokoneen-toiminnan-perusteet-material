---
path: '/luku-2/2-suoritussykli'
title: 'Käskyjen nouto- ja suoritussykli'
---

On kertakaikkiaan nerokasta, kuinka yksinkertaista tietokoneen toiminta on konekäskyjen suorittimen tasolla. Rekisteri PC ilmoittaa seuraavaksi suoritettavan konekäskyn muistiosoitteen. Syklin ensimmäisessä vaiheessa noudamme PC:n osoittaman konekäskyn muistista ja talletamme sen käskyrekisteriin IR. Käytännössä tuo konekäsky löytyy useimmiten välimuistista, mutta se on nyt sivuasia.

Sen jälkeen kasvatamme PC:n arvoa yhdellä, koska oletusarvoisesti seuraavaksi suoritettava konekäsky on aina välittömästi seuraavana muistissa oleva käsky. Tätä seuraavan konekäskyn osoitetta voi vielä (esim.) hyppykäskyn suoritus muuttaa, mutta oletusarvoisesti seuraavaksi suoritetaan nykykäskyn jälkeen muistissa seuraavana oleva käsky.

-- kuva: ch-2-2-nouto-suor-sykli-draft   # kalvo 5.3
<div>
<illustrations motive="ch-2-2-nouto-suor-sykli-draft"></illustrations>
</div>

Sitten suoritamme IR:ssä olevan käskyn. Käsky jaetaan ensin eri kenttiin. Näissä kentissä on mm. operaatiokoodi ja niiden rekistereiden numerot, joita käsky käsittelee. Operaatiokoodi voisi olla yhteenlaskukäskyn koodi (17) ja operandeina numerot 3 ja 5 (viitaten rekistereihin R3 ja R5). Kenttien avulla määritellään myös, mihin muistiosoitteeseen käsky mahdollisesti kohdistuu. Yleensä konekäskyissä on korkeintaan yksi muistiviite.

<div>
 <example
    heading="Yksinkertainen konekäsky"
description="ADD  R3, R5   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ; kentät: 17 3 0 5 0   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;     laske R3:n ja R5:n arvot yhteen ja talleta tulos R3:een">
  </example>
</div>

Seuraavaksi käsky suoritetaan sen operaatiokoodin mukaisesti. Jos se on esimerkiksi kertolasku, niin käskyssä annettujen operandien (esim R3 ja R5) arvot kopioidaan ALU:un, ALU:lle annetaan kertolaskumääräys ja tulos talletetaan käskyssä annettuun rekisteriin (esim. R3). Jos se on hyppykäsky tai ehdollinen haarautuskäsky, niin mahdollisesti käskyn suorituksessa muutetaankin PC:n arvoa. Jos käskyn suorituksen aikana tapahtuu jokin virhetilanne, niin virhe indikoidaan tilarekisteriin ja käskyn suoritus keskeytetään. Tämä voi tapahtua esimerkiksi jakolaskun yhteydessä, jos suoritusaikana havaitaan jakajan arvon olevan nolla.

Sitten palataan syklissä seuraavan konekäskyn noutoon muistista ja sykli toistuu. Tämä on hämmästyttävän suoraviivaista ja opiskelijoiden on usein vaikea sisäistää, että tässäkö se kaikki on. Tietokone on tosiaankin vain tyhmä kone, joka suorittaa hyvin yksinkertaisia konekäskyjä yksi kerrallaan.


-- note: konekäskyjen samanaikainen suoritus
<div>
  <note heading="Konekäskyjen suorituksen optimointi" description="
Todellisissa tehokkaissa suorittimissa peräkkäisten konekäskyjen suoritusnopeutta on optimoitu useallakin eri menetelmällä, mutta lopputulos on silti samanlainen kuin käskyjen suorittaminen yksi kerrallaan. Tällainen menetelmä on mm. liukuhihnoitus, jossa peräkkäiseten konekäskyjen eri vaiheita suoritetaan samanaikaisesti. Esimerkiksi, seuraavaa konekäskyä voidaan olla hakemassa muistista samaan aikaan kuin edellistä vasta suoritetaan. Usein ennakointi kannattaa, mutta joskus tulee huteja. Toinen tapa on tehdä konekäskyistä supertehokkaita, jolloin ne voisivat vaikkapa samalla kertaa useaa laskutoimitusta. Kolmas tapa on toteuttaa suorittimen sisällä monta pienempää suoritinta eli ydintä, joista kukin voi olla suorittamassa omaa ohjelmaansa samanaikaisesti. Neljäs tapa hypersäikeistys mainittiinkin jo aikaisemmin. Siinä suorittimella on esim. kaksi joukkoa rekistereitä, joiden avulla voidaan suorittaa toista ohjelmaa sillä aikaa kun ensimmäinen odottaa muistiviitteen tekemistä. Näitä kaikki menetelmiä voidaan käyttää yhtä aikaa, mutta ne eivät sisälly tämän kurssin oppimistavoitteisiin.
"></note>
</div>

## Etuoikeutettu suoritustila

Ensimmäiset tietokoneet koetettiin vain saada toimimaan. Pian kuitenkin havaittiin, että suorituksessa oleva ohjelman toimintamahdollisuuksia pitäisi jotenkin rajoittaa, jotta ohjelma ei vahingossa tai tahallaan pääsisi sotkemaan muiden ohjelmien tai käyttöjärjestelmän tietoja. Yksi askel tähän ongelmaan oli muistin suojaus kanta- ja rajarekistereiden (BASE, LIMIT) avulla. Näitä käyttämällä suorituksessa oleva ohjelma ei mitenkään voi viitata kuin omaan muistialueeseensa. Toki rajarekisteriden käyttö myös sallii ohjelman muistialueen sijoittamisen joustavasti mihin päin tahansa keskusmuistia, mikä sinällään on erityisen hyödyllistä.

Kanta- ja rajarekisterit ovat kuitenkin vain rekistereitä ja käyttöjärjestelmän pitää pystyä asettamaan niille uudet arvot. Sitä varten käskykannassa pitää olla konekäskyt BASE ja LIMIT-rekistereiden arvojen muuttamiseen. Ongelmana on nyt, että miten voisimme estää tavallista ohjelmaa käyttämästä noita konekäskyjä. Jos ohjelma voi muuttaa omien kanta- ja rajarekistereidensä arvoja, niin sitä kautta se pääsee käsiksi kaikkeen tietoon muistissa. Ei hyvä!

Ratkaisuna tähän ongelmaan on konekäskyjen jakaminen kahteen luokkaan, tavallisiin ja etuoikeutettuihin (privileged). Etuoikeutetut konekäskyt on varattu käyttöjärjestelmän käyttöön ja tavalliset ohjelmat eivät voi niitä käyttää. Ne ovat olemassa nimenomaan luotettavien käyttöjärjestelmien tekemiseen. Suorittimen _etuoikeutettu suoritustila_-bitti tilarekisterissä SR ilmaisee, että suorituksessa oleva prosessi on nyt etuoikeutetussa tilassa (privileged state, supervisor state) ja saa käyttää näitä etuoikeutettuja konekäskyjä. Jos suoritin on tavallisessa suoritustilassa (user state), niin etuoikeutetun käskyn suoritusyritys aiheuttaa virhetilanteen.

Etuoikeutetussa tilassa prosessi voi muuttaa kanta- ja rajarekistereiden arvoja, ja sitä kautta sillä on pääsy koko muistiin. Voidaan myös ajatella, että etuoikeutettuun tilaan siirtyessään prosessin kanta- ja rajarekistereiden arvot asetetaan koko muistialueen kattavaksi. Muita etuoikeutettuja konekäskyjä ovat mm. välimuistin tyhjennyskäsky ja etuoikeutetusta tilasta paluukäsky.

Nyt ymmärrät, miksi käyttöjärjestelmässä olevat virheet ovat niin vaarallisia. Jos tavallinen ohjelma toimii väärin, se voi sotkea vain oman muistialueensa. Käyttöjärjestelmän palikat voivat sotkea ihan mitä vain, minkä vuoksi niiden ohjelmointi pitää (pitäisi?) tehdä erityisen huolella.

-- note: tavallinen käyttäjä vs. ylläpitäjä
<div>
  <note heading="Tavallinen käyttäjä vai ylläpitäjä" description="
Olisi hyvä, että suorittaisit kotitietokonettasi yleensä tavallisena käyttäjänä, etkä ylläpitäjän oikeuksilla. Jos vahingossa päästät haittaohjelman koneellesi, niin ero on merkittävä. Ylläpitäjän oikeuksilla toimiva haittaohjelma on koko ajan etuoikeutetussa tilassa ja saa tehdä ihan mitä haluaa koneellasi. Prosessien oikeudet määräytyvät sen käynnistävän käyttäjän oikeuksien mukaan. Tavallinen käyttäjän käynnistämät ohjelmat ovat turvallisesti tavallisessa suoritustilassa, kun ylläpitäjän käynnistämät ohjelmat ovat jo heti valmiiksi etuoikeutetussa tilassa suoritettavia.
"></note>
</div>

### Siirtymät etuoikeutetun ja tavallisen suoritustilan välillä
Jotkut (ei kaikki!) käyttöjärjestelmän palvelut suoritetaan aina etuoikeutetussa tilassa. Kun prosessi (suorituksessa oleva ohjelma) kutsuu jotain etuoikeutettua palvelua, käyttöjärjestelmä tarkistaa kutsun yhteydessä onko kyseisellä prosessilla oikeus tehdä näin. Jos kaikki on hyvin, niin suoritus jatkuu etuoikeutetussa tilassa. Jos prosessi yrittää kutsua palvelua, johon sillä ei ole oikeuksia, niin prosessin suoritus päättyy virhetilanteeseen.

Kun etuoikeutetussa tilassa suorittava palvelu päättyy, niin palvelusta paluun yhteydessä suoritustila palaa ennalleen. Tätä varten on yleensä ihan oma (etuoikeutettu) konekäskynsä.

## Virhetilanteet ja normaalit keskeytykset

Tietokoneissa tuntuu aina jotain menevän vikaan. Mutta, aivan kuten älykkyydenkin kanssa, nuo käyttäjän havaitsemat virheet tai laitteiston hyytymiset ovat pelkästään ohjelmiston ominaisuuksia. Itse suoritin yksinkertaisine konekäskyineen toimii aina oikein. Sitä käyttävissä ohjelmissa on voi kuitenkin olla _vikoja_, jotka voivat suoritusaikana aiheuttaa _virheitä_, jotka taas joskus aiheuttavat _häiriöitä_ käyttäjille tai järjestelmille. Liukuestetarran puuttuminen kylpyammeesta voisi olla vika, joka joskus aiheuttaa liukastumisvirhetilanteen, joka joskus voi aiheuttaa ranteenmurtumishäiriön.

Kaikki suoritusaikaiset mahdolliset virhetilanteet konekäskyjen suorituksessa on ennalta tunnettu ja niihin on varauduttu. Tällainen virhetilanne on esimerkiksi jo aikaisemmin mainittu kokonaislukujen nollalla jako. Sellainen on myös viittaus jonkun muun prosessin tai käyttöjärjestelmän tietoihin, tai yritys suorittaa niiden koodia. Nämä kaikki virhetilanteet ovat virheitä ohjelmistossa ja niiden tyypit ovat kaikki ennalta tunnettuja. Ainoa yllättävä asia niissä on, että emme tiedä etukäteen niiden tapahtumisajankohtaa. Mitään todella yllättäviä virhetilanteita ei kertakaikkiaan siis voi tapahtua.

Laitteiston virhetilanteet ovat vähän sama kuin että uimahallissa voi joku uimari joutua milloin tahansa hätätilaan. Vaara on tunnistettu, siihen on varauduttu, mutta sen tapahtumisajankohtaa ei tiedetä. Se voi kuitenkin tapahtua ihan millä hetkellä hyvänsä ja tuolloin keskeyttää hallin normaalin toiminnan vähäksi aikaa.

Virhetilanteiden käsittelyn lisäksi samalla tavalla käsitellään kaikki muutkin poikkeustilanteet käskyjen nouto- ja suoritussyklissä. Tällainen poikkeustilanne on vaikkapa Windows-järjestelmissä [CTR-ALT-DEL](https://en.wikipedia.org/wiki/Control-Alt-Delete) näppäimien yhtäaikainen painaminen, mikä avulla halutaan käynnistää nyt just tällä hetkellä jokin tietty käyttöjärjestelmäpalvelu. Sillä hetkellä suorituksessa olevan ohjelman suorituksen täytyy katketa ainakin vähäksi aikaa. Toinen esimerkki mahdollisesta poikkeustilanteesta on I/O-laitekeskeytys, jonka avulla joku ulkoinen laite haluaa ilmoittaa käyttöjärjestelmälle, että sille annettu tehtävä on saatu päätökseen. Kolmas esimerkki on käyttöjärjestelmän palvelupyyntökäsky (esim. SVC eli Superviser Call). Sen toteutus nyt on vain helpointa tehdä poikkeustilanteena, vaikka se muutoin onkin vain tavallinen konekäsky.

Tällaisista virhe- ja poikkeustilanteet käytetään yleisnimeä keskeytys, koska ne kaikki keskeyttävät normaalin käskyjen nouto- ja suoritussyklin, ja siten juuri nyt suorituksessa olevan prosessin suorituksen. Kaikkiin mahdollisiin keskeytyksiin on ennalta varauduttu ja kutakin keskeytystyyppiä varten on olemassa käyttöjärjestelmän oma etuoikeutettu [aliohjelma](https://fi.wikipedia.org/wiki/Aliohjelma), _keskeytyskäsittelijä_. Keskeytystyyppejä ei ole muutamaa kymmentä enempää.

Keskeytyksiä on kolmenlaisia. Meneillään olevan konekäskyn aiheuttamat virhetilanteet käsiteltiin jo. Seuraava ryhmä on järjestelmän omat keskeytykset, joista esimerkinä kellolaitekeskeytys. Se voi "pärähtää" päälle esimerkiksi joka 10 ms ja sen avulla käyttöjärjestelmä saa suoritusvuoron järjestelmän ylläpitoon säännöllisin välein. Kolmas ryhmä on perusjärjestelmän ulkopuolelta tulevat keskeytykset. Tällainen on esimerkiksi jo aiemmin mainittu I/O-laitekeskeytys, jonka avulla vaikkapa wifi-verkkosovitin voi ilmoittaa käyttöjärjestelmälle juuri saapuneesta tietoliikennepaketista.

### Keskeytysten käsittely laitteistossa ja käyttöjärjestelmässä
Kaikkiin keskeytyksiin reagoidaan laitteistossa samalla tavalla. Keskeytystä vastaava bitti laitetaan päälle tilarekisterissä ja käskyn aiheuttaman virhetilanteen sattuessa käskyn suoritus lopetetaan. Käskyjen nouto- ja suoritussykliin on laitettu sen loppuun yksi vaihe lisää ja siinä tarkistetaan, että onko tapahtunut jokin keskeytys. Jos keskeytys havaitaan, niin siihen reagoidaan seuraavasti. Ensin talletetaan riittävä määrä tietoa (vähintään PC ja SR) nyt suorituksessa olevasta prosessista, jotta sen suoritus voi mahdollisesti jatkua normaalisti vähän ajan päästä. Seuraavaksi asetetaan PC:n arvoksi keskeytyskäsittelijän alkuosoite. Samalla laitetaan suorittimen etuoikeutettu suoritustila SR:ssä päälle. Keskeytyskäsittelijät ovat tärkeitä osa käyttöjärjestelmää ja ne saavat tehdä ihan mitä vaan. Kaikki tämä tehdään yhdessä humauksessa laitteistotasolla CPU:n kontrolliyksikön ohjaamana.

-- kuva: ch-2-2-nouto-suor-kesk-sykli-draft   # kalvo 5.3
<div>
<illustrations motive="ch-2-2-nouto-suor-kesk-sykli-draft"></illustrations>
</div>

Käskyn nouto- ja suoritussykli jatkuu sitten normaalisti, mutta nyt ollaankin jo suorittamassa keskeytyskäsittelijää etuoikeutetussa tilassa. Se tekee tarvittavat toimet ja voi tarvittaessa lopettaa keskeytyneen ohjelman suorituksen. Kun kaikki hallintotoimet on tehty, keskeytyskäsittelijä suorittaa etuoikeutetun keskeytyksestä paluu konekäskyn (esim, IRET eli Interrupt Return). Se palauttaa PC:n ja SR:n arvot ennalleen ja nyt keskeytynyt ohjelma voi jatkaa suoritustaan aivan niin kuin keskeytystä ei olisi koskaan tapahtunutkaan. Jos keskeytetty ohjelma esim. virhetilanteen vuoksi oli lopetettu, niin suoritusvuoro annetaan jollekin toiselle prosessille.

On myös mahdollista, että käyttöjärjestelmän saatua kontrollin (suoritusvuoron suorittimella) keskeytyskäsittelijän kautta se päättääkin antaa suoritusvuoron jollekin toiselle prosessille. Näin voi tapahtua esimerkiksi ns. aikaviipalekeskeytyksen käsittelyssä, kun käyttöjärjestelmä yrittää antaa kaikille järjestelmässä oleville prosesseille reilusti suoritinaikaa, pieninä aikaviipaleina kerrallaan. Suorituksessa oleva ohjelma voi vaihtua minkä tahansa konekäskyn jälkeen, koska keskeytyksiä voi tulla ihan milloin vain. Tämä on erityisen harmillista, koska ohjelman suorituksessa voi olla ajanjaksoja, jolloin tietyn koodinpätkän suoritus pitäisi pystyä tekemään alusta loppuun yhteen menoon. Tilanne on vähän sama kuin että sinä et halua kenenkään keskeyttävän itseäsi juuri kun olet tekemässä maksutapahtumaa verkkopankissa. Se pitää tehdä alusta loppuun oikein. Tämä samanaikaisuudenhallintaongelma on tunnettu ja sitä voi ratkoa erilaisilla menetelmillä käyttöjärjestelmässä. Ihan aina ongelmaa ei ole hoksattu ja järjestelmä voi mennä tuolloin sekaisin ja "hyytyä".

-- quiz 2.2.1-7 Väitteet käskyjen nouto- ja suoritussyklistä
<div><quiznator id="5c502951c41ed4148d96abd9"></quiznator></div>
<div><quiznator id="5c502a16fd9fd71425c62335"></quiznator></div>
<div><quiznator id="5c502aa899236814c5bb838d"></quiznator></div>
<div><quiznator id="5c502b31244fe21455cb6741"></quiznator></div>
<div><quiznator id="5c502c4114524713f95a0a30"></quiznator></div>
<div><quiznator id="5c502cbd14524713f95a0a33"></quiznator></div>
<div><quiznator id="5c502d293972a9147410267d"></quiznator></div>