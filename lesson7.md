# A felhasználói felület kezelése
Bárhol is használjuk azonban a JavaScriptet, a futtatókörnyezetnek biztosítania kell egy olyan kapcsolódási felületet, amelyen keresztül a nyelvi elemekkel a gazdakörnyezet szolgáltatásai elérhetőek. Ez a kapcsolódási felület nem más, mint a futtatókörnyezet szolgáltatásai elé készített programozói felület (API).

A böngészők tehát a JavaScript nyelvi maggal való kapcsolattartást alapvetően két modell segítségével biztosítják:
- a böngésző objektummodell (angolul Browser Object Model, BOM) a böngésző szolgáltatásait reprezentáló API-objektumokat összefogó óriási hierarchikus objektum. Ezen keresztül érjük el a böngésző szolgáltatásait, de ez adja programunk globális névterét is.
- a dokumentum objektummodell (angolul Document Object Model, DOM) a megjelenített HTML oldal elé tett API-objektum. Majd látni fogjuk, hogy nem más, mint a HTML struktúránk JavaScript objektumokra való leképezése. Segítségével az egész oldalunkat képesek vagyunk elérni egyetlen objektumon keresztül. Természetesen ahogy a dokumentum része a böngészőablaknak, úgy a DOM is része a BOM-nak.

## 1 A böngésző objektummodell – BOM
A **window** objektumnak két főbb szerepe van:
- egyrészt ez alá vannak becsatolva a böngésző szolgáltatásait képviselő további objektumok,
- másrészt ez lesz az oldalhoz tartozó JavaScript programnak a globális névtere, ennek részeként jönnek létre a változóink, metódusaink, további objektumaink. A JavaScript nyelvi magnak tehát ez biztosítja a futási környezetét.

kód #1:
```javascript
//Globális változó létehozása
var globalis = 42;
ok( globalis === window.globalis, 'A window objektumon keresztül elérhető a globális változó');
ok( globalis === window['globalis'], 'A globális változók a window objektum adattagjaként jönnek létre');
```

Néhány fontosabb szolgáltatás:
- document
- location
- history
- screen
- localStorage

### 1.1 A böngészőablak (window)
A window objektum néhány fontosabb tulajdonsága és metódusa:

- **innerHeight, innerWidth**: a tartalmi rész (dokumentum) magassága és szélessége.
- **outerHeight, outerWidth**: a böngészőablak magassága és szélessége.
- **screenX, screenY**: az ablaknak a főképernyő bal felső sarkától való távolsága. Több monitoros elrendezés esetén a főképernyő az elrendezés bal felső sarkának megfelelő képernyő. Egyes böngészőkben **screenLeft** és **screenTop** néven is elérhető ez a tulajdonság.
- **scrollX, scrollY**: a tetejétől hány pixelnyire van lejjebb gördítve a dokumentum.
- **resizeTo(szél, mag)**: ablak átméretezése a megadott szélességre és magasságra. A modern böngészők már különböző megkötésekkel élnek az átméretezést illetően. Általában csak a főablakot nem engedik átméretezni, csak azt, amit már eleve JavaScripttel nyitottunk, valamint azoknak az ablakoknak az átméretezése is tiltott, amelyben egynél több oldal van megnyitva füleken. Ennek okai érthetőek: az egyik oldalbeli szkriptnek ne engedjük meg, hogy elrontsa a másik oldal méreteit, illetve hogy elállítsák a felhasználó által beállított böngészőméretet.
- **resizeBy(dszél, dmag)**: ablak méretének megváltoztatása a paraméterben megadott értékekkel. Az átméretezésre ugyanolyan megkötések igazak, mint a resizeTo esetén.
- **moveTo(x, y)**: az ablak mozgatása a paraméterként megadott pozícióra. A fenti megkötések itt is érvényesek.
- **moveBy(dx, dy)**: az ablak pozíciójának megváltoztatása a paraméterként megadott értékekkel (ld. megkötések).
- **scrollTo(x, y)**: a dokumentum görgetése a megadott pozícióra.
- **scrollBy(dx, dy)**: a dokumentum görgetése a paraméterben megadott értékekkel.
- **scrollByLines(l)**: a dokumentum görgetése paraméternyi sorral.
- **scrollByPages(p)**: a dokumentum görgetése paraméternyi oldallal.
- **window.open(url, név, tulajdonságok)**: új böngészőablak vagy -lap nyitását teszi lehetővé. A megnyitni kívánt oldal **elérhetőségét az első**, a **hivatkozási alapnak szolgáló ablaknevet a második**, az **új ablak megjelenését és viselkedését befolyásoló paramétereket a harmadik** paraméterben kell megadni.

Az ablak méretéhez kapcsolódóan két eseményt kell kiemelnünk:
- **resize**: az ablak átméretezésekor hívódik meg.
- **scroll**: a dokumentum görgetésekor aktiválódik.

### 1.2 A képernyő (window.screen)
- **width, height**: a képernyő szélessége és magassága.
- **availWidth, availHeight**: a rendelkezésre álló szélesség és magasság a képernyőn. A tálca vagy egyéb elemek miatt ez az érték kisebb lehet, mint a width és height paraméterek.
- **availTop, availLeft**: egy monitor esetében ez a két érték általában 0. Több monitor esetén a legbaloldalibb monitortól való távolságát adja meg a képernyőnek. Például két monitoros elrendezés esetén a jobb oldali képernyőn elhelyezett böngészőben lekérdezve az értéket a bal oldali képernyő felbontását kapjuk az availLeft paraméterre.
- **colorDepth, pixelDepth**: a képernyő szín- és bitmélységét adja meg.

### 1.3 URL-információk (window.location)
- **hash**: az URL # utáni része.
- **host**: a hostname és protocol együttesen.
- **hostname**: a hosztnév.
- **href**: az egész URL.
- **origin**: a protocol és a host együtt.
- **pathname**: a host után megadott útvonal.
- **port**: portszám.
- **protocol**: az URL protokollja.
- **search**: a ? után megadott szövegrész.

A metódusai:
- **assign(url)**: a paraméterül megadott URL betöltését végzi el.
- **replace(url)**: ugyancsak a megadott URL betöltése a feladata, de úgy, hogy lecseréli az oldalt, így az előzmények között nem lesz új bejegyzés.
- **reload()**: újratölti az oldalt.

### 1.4 Böngészési előzmények (window.history)
- **back()**: előző oldal
- **forward()**: következő oldal
- **go(n)**: ugrás n állapotot

HTML5:
- **state**: állapotleíró objektum
- **pushState(state, title [, url])**: új előzményt rögzít újratöltődés nélkül
- **popstate**: amikor a felhasználó megnyomja a vissza gombot
- **replaceState(state, title [, url])**: az aktuális url-t cseréli le


### 1.5 Böngészőinformációk lekérdezése
- böngésző detektálás elavult
- helyette inkább tulajdonság detektálás

kód #2:
```javascript
//Böngésződetektálás (elavult)
if (navigator.userAgent.indexOf('MSIE') !== -1) {
    document.attachEvent('onclick', console.log);
}
//Tulajdonságdetektálás
if (document.attachEvent) {
    document.attachEvent('onclick', console.log);
}
//Vagy még specifikusabban
if (typeof document.attachEvent !== "undefined") {
    document.attachEvent('onclick', console.log);
}
```

### 1.6 Időzítők
- **setTimeout(fn, ms)**: késleltetés
- **setInterval(fn, ms)**: ütemezés

- **clearTimeout(fn, ms)**: késleltetés megállítása
- **clearInterval(fn, ms)**: ütemezés megállítása

kód #3:
```javascript
// setInterval alternatíva
var d;
(function tick() {
    console.log('de lehet nem ette meg!');
    d = setTimeout(tick, 1000);
}());
clearTimeout(d);
```

## 2 A dokumentum objektummodell – DOM
### 2.1 Elemek kiválasztása
### 2.2 A struktúra bejárása
### 2.3 Attribútumok kezelése
### 2.4 Elemek kezelése
## 3 jQuery
### 3.1 A jQuery története
### 3.2 Filozófiája
### 3.3 Elemek kiválasztása
### 3.4 Elemek bejárása
### 3.5 Iterálás a szelekcióban
### 3.6 HTML struktúra megváltoztatása
### 3.7 Hatékony DOM manipuláció
### 3.8 Stílusok kezelése
## 4 Eseménykezelés
### 4.1 Eseménykezelő regisztrálása
### 4.2 Eseményobjektum megszerzése
### 4.3 Eseményobjektum tulajdonságai
### 4.4 Az alapértelmezett művelet megakadályozása
### 4.5 Buborékolás és megakadályozása
### 4.6 Delegálás
### 4.7 Esemény kiváltása
### 4.8 jQuery